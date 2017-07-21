---
title: 初探大屏设备图片资源适配
date: 2017-07-20 17:36:01
tags: [Android , 适配]
---
### 背景
最近有个需求，将`drawable`下的图片文件通过`getDrawable()`的方式加载为一个`Bitmap`，再获取这个`Bitmap`的宽和高，用`canvas`画到界面上去。
使用平板作为适配对象时，结果发现平板上绘制出来的图片要比手机小一些。后经实验，平板读取的是`xhdpi`资源下的图片，而对比的手机则是读取的`xxhdpi`下的。现在的需求是，要让`xdpi`的平板使用手机`xxdpi`的资源。

<!--more-->

### 尝试
将平板需要的图片放入`drawable-sw600dp`文件夹下，编译后，发现图片要比手机上相同图片大得多，边缘有明显的模糊感。将图片的文件夹名改为`drawable-sw600dp-xhdpi`后，图片正常显示。

### 分析
#### 源码
直接从源码下手，笔者使用的是`Android 6.0`版本源码。首先从`getResource().getDrawable()`方法跳入
```java
public Drawable getDrawable(@DrawableRes int id, @Nullable Theme theme)
        throws NotFoundException {
    final TypedValue value = obtainTempTypedValue();
    try {
        final ResourcesImpl impl = mResourcesImpl;
        impl.getValue(id, value, true);
        return impl.loadDrawable(this, value, id, theme, true);
    } finally {
        releaseTempTypedValue(value);
    }
}
```
没什么好说的，继续进入`loadDrawable()`方法
```java  
@Nullable
    Drawable loadDrawable(TypedValue value, int id, Theme theme) throws NotFoundException {
        .
        .
        .
        if (value.type >= TypedValue.TYPE_FIRST_COLOR_INT
                && value.type <= TypedValue.TYPE_LAST_COLOR_INT) {
            isColorDrawable = true;
            caches = mColorDrawableCache;
            key = value.data;
        } else {
            isColorDrawable = false;
            caches = mDrawableCache;
            key = (((long) value.assetCookie) << 32) | value.data;
        }

        // First, check whether we have a cached version of this drawable
        // that was inflated against the specified theme.
        if (!mPreloading) {
            final Drawable cachedDrawable = caches.getInstance(key, theme);
            if (cachedDrawable != null) {
                return cachedDrawable;
            }
        }
        // Next, check preloaded drawables. These may contain unresolved theme
        // attributes.
        final ConstantState cs;
        if (isColorDrawable) {
            cs = sPreloadedColorDrawables.get(key);
        } else {
            cs = sPreloadedDrawables[mConfiguration.getLayoutDirection()].get(key);
        }

        Drawable dr;
        if (cs != null) {
            dr = cs.newDrawable(this);
        } else if (isColorDrawable) {
            dr = new ColorDrawable(value.data);
        } else {
            dr = loadDrawableForCookie(value, id, null);
        }
        .
        .
        .
        return dr;
    }
```
这段代码，首先从内存里面获取缓存，通过`type`来判断是不是`colorDrawable`，如果是的话获取缓存，如果不是的话通过 `mDrawableCache` 来获取。如果内存里面有的话，就直接返回。如果内存没有的话，会从preload里面取。如果还是没有缓存，那么就会查看到底是不是`color`，如果是`color`的话，就从`TypeValue`里面获取`data`，生成一个新的，如果不是`color`的话就调用 `loadDrawableForCookie` 继续加载。

进入`loadDrawableForCookie()`方法
```java
private Drawable loadDrawableForCookie(TypedValue value, int id, Theme theme) {
    .
    .
    .
      if (file.endsWith(".xml")) {
          final XmlResourceParser rp = loadXmlResourceParser(
                  file, id, value.assetCookie, "drawable");
            dr = Drawable.createFromXml(this, rp, theme);
            rp.close();
        } else {
            final InputStream is = mAssets.openNonAsset(
                    value.assetCookie, file, AssetManager.ACCESS_STREAMING);
            dr = Drawable.createFromResourceStream(this, value, is, file, null);
            is.close();
        }
    }
    .
    .
    .
    return dr;
}
```
接着进入`Drawable.createFromResourceStream()`方法
```java
public static Drawable createFromResourceStream(Resources res, TypedValue value,
          InputStream is, String srcName, BitmapFactory.Options opts) {
      .
      .
      .
      if (opts == null) opts = new BitmapFactory.Options();
      opts.inScreenDensity = res != null
              ? res.getDisplayMetrics().noncompatDensityDpi : DisplayMetrics.DENSITY_DEVICE;
      Bitmap  bm = BitmapFactory.decodeResourceStream(res, value, is, pad, opts);
      if (bm != null) {
          byte[] np = bm.getNinePatchChunk();
          if (np == null || !NinePatch.isNinePatchChunk(np)) {
              np = null;
              pad = null;
          }

          final Rect opticalInsets = new Rect();
          bm.getOpticalInsets(opticalInsets);
          return drawableFromBitmap(res, bm, np, pad, opticalInsets, srcName);
      }
      return null;
  }
```
可以看到，图片是靠`Bitmap  bm = BitmapFactory.decodeResourceStream(res, value, is, pad, opts)`方法读取出来的。

进入`BitmapFactory.decodeResourceStream()`方法
```java
public static Bitmap decodeResourceStream(Resources res, TypedValue value,
        InputStream is, Rect pad, Options opts) {

    if (opts == null) {
        opts = new Options();
    }

    if (opts.inDensity == 0 && value != null) {
        final int density = value.density;
        if (density == TypedValue.DENSITY_DEFAULT) {
            opts.inDensity = DisplayMetrics.DENSITY_DEFAULT;
        } else if (density != TypedValue.DENSITY_NONE) {
            opts.inDensity = density;
        }
    }

    if (opts.inTargetDensity == 0 && res != null) {
        opts.inTargetDensity = res.getDisplayMetrics().densityDpi;
    }

    return decodeStream(is, pad, opts);
}
可以看到，这个方法里面设置了图片对应的dpi，而我们的问题也是出在dpi上的，所以这里应该就是我们的突破口。
```
如果想详细了解每一步干了些什么，请参考[Resource通过resId获取Drawable的流程](http://www.qingpingshan.com/rjbc/az/239746.html)

#### 调试

* 将资源放入`drawable-sw600dp-xhdpi`文件夹时，程序执行到这一步
![sad](/images/big-screen-adaptation/drawable-sw600dp-xhdpi.png)

* 将相同图片资源放入`drawable-sw600dp`文件夹时，程序执行到这一步
![sad](/images/big-screen-adaptation/drawable-sw600dp.png)

可以看出，对于图片解析的分叉点就在这里。第一种情况，文件夹名字中包含密度信息时，图片会根据文件夹所指定的dpi进行缩放，而如果文件夹名字中不包含密度信息，只有屏幕大小限制符，图片会被设置一个默认的dpi进行缩放，那么这个`DisplayMetrics.DENSITY_DEFAULT`指的是什么呢？找到源码
```java    
   /**
     * The reference density used throughout the system.
     */
    public static final int DENSITY_DEFAULT = DENSITY_MEDIUM;
```
`Android`系统中默认的密度是采用了中等，也就是`mdpi`。也就是说，如果不指定图片的缩放等级，它将被当做`mdpi`进行缩放处理，这也就解释了为什么当资源被放入`drawable-sw600dp`文件夹时，图片边缘有明显的模糊感。

回到最开始，为什么笔者会将文件夹设置成`drawable-sw600dp`。一开始笔者认为大屏幕设备的图片资源应该可以通用，就没指定对应的密度信息，其实这个`sw600dp`指的是符合条件的最小屏幕宽度，与密度信息并不冲突，换句话说，`drawable-sw600dp-xhdpi`可以被解读为在宽度大于等于600dp，且屏幕像素密度为`xhdpi`的设备使用该资源。相应的还可以衍生出`drawable-sw600dp-land-xhdpi`等等。

### 总结
首先要了解`Android`系统中默认的密度是`mdpi`,其次在设置资源文件夹的时候要了解各个限定符究竟限定了资源使用的哪些条件，互相有没有干扰等等。遇到了奇奇怪怪的问题时，多分析源码，有条件的可以把源码下载到本地以供调试。
