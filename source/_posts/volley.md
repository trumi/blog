---
title: Volley之对抗内存泄漏
date: 2018-02-26 15:50:03
tags: [Android , Volley]
thumbnail: /images/datasaver/thumbnail.jpg
---

说起网络通信框架，不得不说到[Volley](https://github.com/google/volley)，虽然还有很多其它优秀的第三方框架，比如OkHttp，Retrofit，但Volley凭借着它的轻量化，Google官方出品，还是有一席之地的。但是，Volley也存在着一些坑，这篇博客主要聚焦Volley造成的内存泄漏问题。

<!--more-->

### 分析

#### `Request`
通过Volley构建网络请求的时候，不管是`StringRequest`,还是`JsonRequest`，或者是`ImageRequest`，都要传入`Response.Listener`和`Response.ErrorListener`的参数，以`StringRequest`为例:
```java
StringRequest stringRequest = new StringRequest("http://example.com",  
                        new Response.Listener<String>() {  
                            @Override  
                            public void onResponse(String response) { 

                            }  
                        }, new Response.ErrorListener() {  
                            @Override  
                            public void onErrorResponse(VolleyError error) {    

                            }  
                        }); 
```
在这里面处理网络请求的结果，也就是说，这两个Listener可能会包含一些回调，通知View层去更新数据，这个回调可能是Activity的引用，或者是Presenter的引用，或者是其他实例的引用。

那么我们再来看看，当一个Request被cancel之后，Volley内部执行了什么操作
```java
/**
* Mark this request as canceled.  No callback will be delivered.
*/
public void cancel() {
    mCanceled = true;
}
```
这里只是更新了一下标识符

由于`StringRequest`是继承于`Request`的，所以猜想可能是在子类对Listener进行了置空处理。`StringRequest`源码并不多，索性全部贴出来以供研究。
```java
public class StringRequest extends Request<String> {
    private final Listener<String> mListener;

    public StringRequest(int method, String url, Listener<String> listener,
            ErrorListener errorListener) {
        super(method, url, errorListener);
        mListener = listener;
    }

    public StringRequest(String url, Listener<String> listener, ErrorListener errorListener) {
        this(Method.GET, url, listener, errorListener);
    }

    @Override
    protected void deliverResponse(String response) {
        mListener.onResponse(response);
    }

    @Override
    protected Response<String> parseNetworkResponse(NetworkResponse response) {
        String parsed;
        try {
            parsed = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
        } catch (UnsupportedEncodingException e) {
            parsed = new String(response.data);
        }
        return Response.success(parsed, HttpHeaderParser.parseCacheHeaders(response));
    }
}
```

可以看到，在创建Request时的`Listener`到了构造函数时赋给了全局变量`mListener`，而`ErrorListener`则继续向父类，也就是`Request<T>`传递。其他种类的Request基本保持这一做法。

我们可以发现，不管是父类还是子类，都没有在`cancel()`方法里对`mListener`进行置空操作,这就会带来一个问题：当一个`Request`被取消时，由于`mListener`没有被置空，创建出来的`Request`实例依旧持有`Listener`的实例，当`Listener`实例是一个`Activity`的话，很有可能导致`Activity`退出后，由于`Activity`的引用被`Request`持有而导致内存不能被及时回收，造成泄漏。这种情况同样试用于`ErrorListener`。

#### `ImageLoader`
由于`ImageLoader`内部也是通过`ImageRequest`来请求图片资源的，既然`ImageRequest`会出现内存泄露，`ImageLoader`也肯定会。这里附上`ImageLoader`请求图片资源的关键方法
```java
 public ImageContainer get(String requestUrl, ImageListener imageListener,
            int maxWidth, int maxHeight, ScaleType scaleType) {
        ...
        //ImageLoader的图片缓存机制
        final String cacheKey = getCacheKey(requestUrl, maxWidth, maxHeight, scaleType);
        Bitmap cachedBitmap = mCache.getBitmap(cacheKey);
        if (cachedBitmap != null) {
            ImageContainer container = new ImageContainer(cachedBitmap, requestUrl, null, null);
            imageListener.onResponse(container, true);
            return container;
        }
        //无法找到图片缓存，发起网络请求获取图片资源
        ImageContainer imageContainer =
                new ImageContainer(null, requestUrl, cacheKey, imageListener);
        imageListener.onResponse(imageContainer, true);

        BatchedImageRequest request = mInFlightRequests.get(cacheKey);
        if (request != null) {
            request.addContainer(imageContainer);
            return imageContainer;
        }
        //通过ImageRequest获取图片
        Request<Bitmap> newRequest = makeImageRequest(requestUrl, maxWidth, maxHeight, scaleType,
                cacheKey);

        mRequestQueue.add(newRequest);
        mInFlightRequests.put(cacheKey,
                new BatchedImageRequest(newRequest, imageContainer));
        return imageContainer;
    }
```

### 解决
* `Request`
思路很简单，对于每一个Request，比如`StringRequest`、`JsonRequest`、`ImageRequest`等等，重新写一个类，直接继承于各自功能的`XXRequest<T>`,重写`cancel()`方法,在里面对Listener进行置空操作，同时，需要将`Listener`和`ErrorListener`的所有回调操作全部托付给子类，保证`XXRequest<T>`中不含有`Listener`和`ErrorListener`。这里还是以`StringRequest`举例：
```java
public abstract class LeakFreeStringRequest<T> extends StringRequest<T> {

    protected Map<String, String> mParamMap;
    private Response.Listener<T> mListener;
    private Response.ErrorListener mErrorListener;

     public LeakFreeStringRequest(int method, String url, Map<String, String> params, Listener<T> listener, ErrorListener errorListener) {
        super(method, url, null);
        mListener = listener;
        mErrorListener = errorListener;
        mParamMap = params;
    }

    @Override
    public void cancel() {
        mListener = null;
        mErrorListener = null;
        super.cancel();
    }

    @Override
    protected void deliverResponse(T response) {
        mListener.onResponse(response);
    }

    @Override
    public void deliverError(VolleyError error) {
        mErrorListener.deliverError(error);
    }

    @Override
    public ErrorListener getErrorListener() {
        return mErrorListener;
    }
    ...
```
所有其他特定功能的Request类都可以参照这样来实现。当然，如果你是源码编译引用的Volley，直接改源码就好了

* `ImageLoader`
虽然留了这个坑，好在`ImageLoader`的`makeImageRequest()`方法可以让我们自定义构建请求的操作。
```java
 protected Request<Bitmap> makeImageRequest(String requestUrl, int maxWidth, int maxHeight,
            ScaleType scaleType, final String cacheKey) {
        return new ImageRequest(requestUrl, new Listener<Bitmap>() {
            @Override
            public void onResponse(Bitmap response) {
                onGetImageSuccess(cacheKey, response);
            }
        }, maxWidth, maxHeight, scaleType, Config.RGB_565, new ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                onGetImageError(cacheKey, error);
            }
        });
    }
```
源码中的`makeImageRequest()`中创建的`ImageRequest`是直接继承于`Request<T>`的，根据上文分析的结论必须替换成改造版本的，这里姑且叫做`LeakFreeImageRequest`，代码就不贴了，和上文`LeakFreeStringRequest`类写法类似。接着，需要新建一个类，暂且叫`LeakFreeImageLoader`，重写`makeImageRequest()`方法，将`ImageRequest`改为已经改造好的`LeakFreeImageRequest`：
```java
public class LeakFreeImageLoader extends ImageLoader {

    public LeakFreeImageLoader(RequestQueue queue, ImageCache imageCache) {
        super(queue, imageCache);
    }

    @Override
    protected Request<Bitmap> makeImageRequest(String requestUrl, int maxWidth, int maxHeight,
                                               ImageView.ScaleType scaleType, final String cacheKey) {
        return new LeakFreeImageRequest(requestUrl, new Response.Listener<Bitmap>() {
            @Override
            public void onResponse(Bitmap response) {
                onGetImageSuccess(cacheKey, response);
            }
        }, maxWidth, maxHeight, scaleType, Bitmap.Config.RGB_565, new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                onGetImageError(cacheKey, error);
            }
        });
    }
}
```
之后，替换原有的`ImageLoader`变为`LeakFreeImageLoader`，问题解决