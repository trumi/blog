---
title: Android平台MarkDown的解析及显示
date: 2016-08-02 22:01:01
tags: [Android, View, MarkDown]
<<<<<<< HEAD
thumbnail: /images/markdown/thumbnail.jpg
---
很久没有更博了，期间也没怎么闲着。由于现在在实习，弄自己项目的时间就没有以前多了。最近想搞个da新闻，接触到了一点MarkDown的解析以及显示，遂更。
<!--more-->
=======
---
很久没有更博了，期间也没怎么闲着。由于现在在实习，弄自己项目的时间就没有以前多了。最近想搞个da新闻，接触到了一点MarkDown的解析以及显示，遂更。
>>>>>>> 7f71da62acc8d31c1677818e3ea3d9faaba8a63c

### 了解

#### 解析
Android平台上的MarkDown有解析库，比较出名的有：[pegdown](https://github.com/sirthias/pegdown/)、[txtmark](https://github.com/rjeschke/txtmark)、[markdown4j](https://code.google.com/archive/p/markdown4j/)(fork自txtmark)。
不过实测发现，pegdown的效果虽好，但依赖其他的包，且导入容易遇到问题。txtmark和markdown4j会出现解析错误的情况。
#### 显示
Android上解析并显示MarkDown文本最简单的方案就是：将MarkDown文本解析为HTML，再通过WebView显示。GitHub上已经出现了这样的自定义控件，如[zzhoujay/Markdown](https://github.com/falnatsheh/MarkdownView)

说明：GitHub上已经出现了这种自定义控件，本博的目的也仅限于为想自己做MarkDownView的猿们抛砖引玉，顺便分享一下本人踩过的坑。
<<<<<<< HEAD

=======
<!--more-->
>>>>>>> 7f71da62acc8d31c1677818e3ea3d9faaba8a63c
### 开始
这里我选用了[chjj/marked](https://github.com/chjj/marked)作为解析器，这是一个使用javascript开发的解析库，你懂的。
* 将marked.js下载后，放在工程Assets文件夹下，修改第1219行
将
```javascript
return Parser.parse(Lexer.lex(src, opt), opt);
```
    改为
```javascript
document.getElementById('content').innerHTML =Parser.parse(Lexer.lex(src, opt), opt);
```

* 创建一个WebView，并初始化

```java
WebView webview = new WebView(context);
setContentView(webview);
webview.getSettings().setDefaultTextEncodingName("UTF-8");
webview.getSettings().setJavaScriptEnabled(true);
```
* 创建一个HTML文件，同样放在工程Assets文件夹下，用于承载js处理后的结果

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8"/>
  <script src="marked.js"></script> //引入js库
</head>
<body style="line-height: 161%; padding: 25px;">
  <div id="content"></div>
</body>
</html>
```

* webview加载HTML文件

```java
webview.loadUrl("file:///android_asset/index.html");
```

* 显示MarkDown文本时，将MarkDown文本从文件中读入，变为字符串data，执行

```java
webview.loadUrl("javascript:marked(\'"+data+"\')");
```

### 避坑

##### 全文无法解析为MarkDown，显示的是MarkDown源文件字符串
如果你直接把从MarkDown文件中读取的字符串放入js里执行，显示效果简直没法看，解析器压根就不认你的MarkDown语法。
解决方案：在读取的源字符串的每一行末尾加上一个‘\n’，例如：

```java
StringBuilder sb=new StringBuilder();
BufferedReader br=new BufferedReader(new FileReader(filename));
String line;
while ((line=br.readLine())!=null){
    sb.append(line).append("\\n");//注意这一行，通常应该是sb.append(line);
}
```

##### 部分解析异常，排版部分错乱
这个问题是由于源文件中存在特殊字符导致javascript库解析异常
解决方案：直接上代码

```java
//result为从MarkDown源文件中读取出的字符串
result=result.replace("\"","\\"+"\"");
result=result.replace("\'","\\"+"\'");
result=result.replace("//","\\/\\/");
```

##### WebView不显示任何内容
发生这个问题，有很大概率是进行了在WebView执行加载HTML文件后，马上执行解析步骤这种操作，即

```java
webview.loadUrl("file:///android_asset/index.html");
webview.loadUrl("javascript:marked(\'"+data+"\')");
```
如果是这样写的，将会造成WebView还未完全加载完javascript库，便先执行了javascript中的marked方法。通常情况下，在加载完HTML之后，再去执行解析操作
解决方案：在WebView的setWebViewClient中监听加载进度，等加载完成后，再进行解析

```java
webview.setWebViewClient(new WebViewClient()
    {
      @Override
     public void onPageFinished(WebView view, String url)
    {
        //HTML加载完成
        super.onPageFinished(view, url);
        String call = "javascript:marked(\'"+data+"\')";
        loadUrl(call);
     }
      @Override
      public void onPageStarted(WebView view, String url, Bitmap favicon)
      {
          super.onPageStarted(view, url, favicon);
      }
 });
```

### 效果
不说废话，直接上图
![devices](/images/markdown/markdown.png)
<<<<<<< HEAD
具体的样式，可以自行修改，毕竟它最终会被渲染成一个web。我改了部分样式，包括引用块，超链接等
=======
具体的样式，可以自行修改，毕竟它最终会被渲染成一个web。我改了部分样式，包括引用块，超链接等
>>>>>>> 7f71da62acc8d31c1677818e3ea3d9faaba8a63c
