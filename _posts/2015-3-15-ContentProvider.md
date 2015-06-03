---
layout: post
title: ContentProvider
---

# 使用ContentProvider实现数据共享
当在系统中部署一个又一个Android应用之后，系统里将会包含多个Android应用，有时候就需要在不同的应用之间共享数据，比如现在有一个短信接收应用，用户想把接收到的陌生短信的发信人添加到联系人管理应用中，就需要在不同应用之间共享数据。对于这种需要在不同应用之间共享数据的需求，当然可以让一个应用程序直接去操作另一个应用程序所记录的数据，比如操作它所记录的SharedPreferences、文件或数据库等，这种方式显得太杂乱了；不同的应用程序记录数据的方式差别很大，这种方式不利于应用程序之间进行数据交换。

为了在应用程序之间交换数据，Android提供了ContentProvider，ContentProvider是不同应用程序之间进行数据交换的标准API，当一个应用程序需要把自己的数据暴露给其他程序使用时，该应用程序就可通过提供ContentProvider来实现；其他应用程序就可通过ContentResolver来操作ContentProvider暴露的数据。

ContentProvider也是Android应用的四大组件之一，与Activity、Service、BroadcastReceiver相似，它们都需要在AndroidManifest.xml文件中进行配置。

一旦某个应用程序通过ContentProvider暴露了自己的数据操作接口，那么不管该应用程序是否启动，其他应用程序都可通过该接口来操作该应用程序的内部数据，包括增加数据、删除数据、修改数据、查询数据等。
## 数据共享标准：ContentProvider简介
ContentProvider是不同应用程序之间进行数据交换的标准API，ContentProvider以某种Uri的形式对外提供数据，允许其他应用访问或修改数据；其他应用程序使用ContentResolver根据Uri去访问操作指定数据。

	可以把ContentProvider当成Android系统内部的“网站”,这个网站以固定的Uri对外提供服务；而ContentResolver则可当成Android系统内部的HttpClient，它可以向指定Uri发送“请求”（实际上是调用ContentResolver的方法），这种请求最后委托给ContentProvider处理，从而实现对“网站”（即ContentProvider）内部数据进行操作。

### ContentProvider简介
如果把ContentProvider当成一个“网站”来看，那么如何对外提供数据呢？是否需要像Java Web开发一样编写JSP、Servlet之类呢？不需要。如果那样就太复杂了，毕竟ContentProvider只是提供数据的访问接口，并不是像一个网站一样对外提供完整的页面。

如果把ContentProvider当成一个网站来看，那么如何完整地开发一个ContentProvider呢？步骤其实很简单：
1. 定义自己的ContentProvider类，该类需要继承Android提供的ContentProvider基类。
2. 向Android系统注册这个“网站”，也就是在AndroidManifest.xml文件中注册这个ContentProvider，就像注册Activity一样。注册ContentProvider时需要为它绑定一个URL。

向Android系统中注册ContentProvider主要在<application.../>元素下添加如下子元素即可：

```xml
<!-- 下面配置中name属性指定ContentProvider类
	authorities就相当于为该ContentProvider指定URL -->
<provider android:name=".DictProvider" android:authorities="tk.zongzhankui.providers.dictprovider">
```

当我们向Android系统注册了ContentProvider网站之后，其他应用程序就可通过该Uri来访问DictProvider所暴露的数据了。

### Uri简介
在介绍Android系统的Uri之前，先来看一下最常用的互联网URL，例如想访问疯狂Java联盟的某个页面，应该在浏览器中输入如下Uri：

	http://www.crazyit.org/ethos.php

对于上面这个URL，可分为如下三个部分：

- http://：URL的协议部分，只要通过HTTP协议来访问网站，这个部分是固定的。
- www.crazyit.org：网站域名部分。只要访问指定的网站，这个部分总是固定的。
- ethos.php：网站资源部分。当访问者需要访问不同资源时，这个部分是动态改变的。

Android的Uri与此类似，例如如下Uri：

	content://org.crazyit.providers.dictprovider/words

它也可分为三个部分：

- content://：这个部分是Android所规定的，是固定的。
- org.crazyit.providers.dictprovider：这个部分就是ContentProvider的authority。系统就是由这个部分来找到操作哪个ContentProvider。只要访问指定的ContentProvider，这个部分总是固定的。
- words：资源部分（或者说数据部分），当访问者需要访问不同资源时，这个部分是动态改变的。

需要指出的是，Android的Uri所能表达的功能更丰富，它还可以支持如下Uri：

	content://org.crazyit.providers.dictprovider/word/2

此时它要访问的资源为word/2，这意味着访问word数据中ID为2的记录。

还有如下形式：

	content://org.crazyit.providers.dictprovider/word/2/word

此时它要访问的资源为word/2，这意味着访问word数据中ID为2的记录的word字段。

如果想访问全部数据，即可使用上面所示的形式：
