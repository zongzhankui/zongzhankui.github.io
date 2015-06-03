---
layout: post
title: SharedPreferences
---

# 使用SharedPreferences
有些时候，应用程序有少量的数据需要保存，而且这些数据的格式很简单：都是普通的字符串、标量类型的值等，比如应用程序的各种配置信息（如是否打开音效、是否使用振动效果等）、小游戏的玩家积分（如扫雷英雄榜之类的）等，对于这种数据，Android提供了SharedPreferences进行保存。
## SharedPreferences与Editor简介
SharedPreferences保存的数据主要是类似于配置信息格式的数据，因此它保存的数据主要是简单类型的key-value对。

SharedPreferences接口主要负责读取应用程序的Preferences数据，它提供了如下常用方法来访问SharedPreferences中的key-value对。

+ boolean contains(String key)：判断SharedPreferences是否包含特定key的数据。
+ abstract Map(String, ?) getAll()：获取SharedPreferences数据里全部的key-value对。
+ boolean getXxx(String key, xxx defValue)：获取SharedPreferences数据里指定key对应的value。如果该key不存在，返回默认值defValue。其中xxx可以是boolean、float、int、long、String等各种基本类型的值。

SharedPreferences接口本身并没有提供写入数据的能力，而是通过SharedPreferences的内部接口，SharedPreferences调用edit()方法即可获取它所对应的Editor对象。Editor提供了如下方法来向SharedPreferences写入数据。

+ SharedPreferences.Editor clear()：清空SharedPreferences里所有数据。
+ SharedPreferences.Editor putXxx(String key, xxx value)：向SharedPreferences存入指定key对应的数据。其中xxx可以是boolean、float、int、long、String等各种基本类型的值。
+ SharedPreferences.Editor remove(String key)：删除SharedPreferences里指定key对应的数据项。
+ boolean commit()：当Editor编辑完成后，调用该方法提交修改。

SharedPreferences本身是一个接口，程序无法直接创建SharedPreferences实例，只能通过Context提供的getSharedPreferences(String name, int mode)方法来获取SharedPreferences实例，该方法的第二个参数支持如下几个值。

+ Context.MODE_PRIVATE：指定该SharedPreferences数据只能被本应用程序读、写。
+ Context.MODE_WORLD_READABLE：指定该SharedPreferences数据能被其他应用程序读，但不能写。
+ Context.MODE_WORLD_WRITEABLE：指定该SharedPreferences数据能被其他应用程序读、写。

## SharedPreferences的存储位置和格式
下面程序示范了如何向SharedPreferences中写入、读取数据，该程序的界面很简单，它只是提供了两个按钮，其中一个用于写入数据，另外一个用于读取数据，故此处不再给出界面布局文件。程序代码如下。
```python

```
SharedPreferences数据总是保存在/data/data/<package name>/shared_prefs目录下，SharedPreferences数据总是以XML格式保存。通过File Explorer面板的导出文件按钮将该XML文件导出到XML文档，打开该XML文档可看到如下文件内容：
```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<map>
    <int name="random" value="56" />
    <string name="time">2015年02月24日 10:40:21</string>
</map>

```
从上面的文件不难看出，SharedPreferences数据文件是一个根元素为<map.../>的根元素，该元素里每个子元素代表一个key-value对，当value是整数类型的值时使用<int.../>子元素，当value是字符串类型时，使用<string.../>子元素……以此类推。
## 实例：记录应用程序的使用次数
这个简单的示例可以记住应用程序的使用次数：当用户第一次启动该应用程序时，系统创建SharedPreferences来记录使用次数。用户以后启动应用程序时，系统先读取SharedPreferences中记录的使用次数，然后将使用次数加1。

本示例程序的代码如下。
```java
```
## 读、写其他应用SharedPreferences
要读、写其他应用的SharedPreferences，前提是创建该SharedPreferences的应用程序指定相应的访问权限，例如指定了MODE_WORLD_READABLE，这表明该SharedPreferences可被其他应用程序读取；指定了MODE_WORLD_WRITABLE，这表明该SharedPreferences可被其他程序写入。

例如上一个示例创建SharedPreferences时就指定了MODE_WORLD_READABLE模式，这表明该SharedPreferences数据可以被其他程序读取。

为了读取其他程序的SharedPreferences，可按如下步骤进行。
1. 需要创建其他程序对应的Context，例如如下代码。
```java
useCount = createPackageContext("tk.zongzhankui.io", Context.CONTEXT_IGNORE_SECURITY);
```
上面的程序中tk.zongzhankui.io就是其他程序的包名——实际上Android系统就是用应用程序的包名来作为该程序的标识的。
2. 调用其他程序的Context的getSharedPreferences(String name, int mode)即可获取相应的SharedPreferences对象。
3. 如果需要向其他应用的SharedPreferences数据写入数据，调用SharedPreferences的edit()方法获取相应的Editor即可。

下面的程序示范如何读取上一个程序所保存的SharedPreferences数据。
```java
```
事实上，如果开发者不通过先获取其他应用程序的Context，再获取SharedPreferences的方式也可读取SharedPreferences的数据——开发者完全使用以IO流的方式先读取SharedPreferences对应的XML文件，再通过XML解析来获取数据也是可行的，只是这种方式过去烦琐，而使用SharedPreferences来读写数据则简单得多。
