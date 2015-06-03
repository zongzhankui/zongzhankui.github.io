---
layout: post
title: BroadcastReceiver
---

# BroadcastReceiver
Android系统的四大组件还有一种BroadcastReceiver，这种组件本质上就是一种全局的监听器，用于监听系统全局的广播消息。由于BroadcastReceiver是一种全局的监听器，因此它可以非常方便地实现系统中不同组件之间的通信。例如我们希望客户端程序与startService()方法启动的Service之间通信，就可以借助于BroadcastReceiver来实现。

## BroadcastReceiver简介
BroadcastReceiver用于接收程序（包括用户开发的程序和系统内建的程序）所发出的Broadcast Intent，与一块嗯用程序启动Activity、Service相同的是，程序启动BroadcastReceiver也只需要两步。

1. 创建需要启动的BroadcastReceiver的Intent。
2. 调用Context的sendBroadcast()或sendOrderedBroadcast()方法来启动指定的BroadcastReceiver。

当应用程序发出一个BroadcastReceiver Intent之后，所有匹配该Intent的BroadcastReceiver都有可能被启动。

与Activity、Service具有完整的生命周期不同，BroadcastReceiver本质上只是一个系统级的监听器——它专门负责监听个程序所发出的Broadcast。

由于BroadcastReceiver本质上属于一个监听器，因此实现BroadcastReceiver的方法也十分简单，只要重写BroadcastReceiver的onReceive(Context context, Intent intent)方法即可。

一旦实现了BroadcastReceiver，接下来就应该指定该BroadcastReceiver能匹配的Intent，此时有两种方式：

- 使用代码进行指定，调用BroadcastReceiver的Context的registerReceiver(BroadcastReceiver receiver, IntentFilter filter)方法指定。例如如下代码：

	```java
IntentFilter filter = new IntentFilter("android.provider.Telephony.SMS_RECEIVED");
IncomingSMSReceiver receiver = new IncomingSMSReceiver();
registerReceiver(receiver, filter);
    ```

- 在AndroidManifest.xml文件中配置。例如如下代码：

	```xml
<receiver android:name=".IncomingSMSReceiver">
	<intent-filter>
		<action android:name="android.provider.Telephony.SMS_RECEIVED"/>
	</intent-filter>
</receiver>
	```

每次系统Broadcast事件发生后，系统就会创建对应的BroadcastReceiver的实例，并自动触发它的onReceiver()方法，onReceiver()方法执行完后，BroadcastReceiver的实例就会被销毁。

如果BroadcastReceiver的方法不能在10秒内执行完成，Android会认为该程序无响应，所以不要在BroadcastReceiver的onReceiver()方法里执行―些耗时的操作，否则会弹出ANR（Application No Response）的对话框。

如果确实需要根据Broadcast来完成一项比较耗时的操作，则可以考虑通过Intent启动一个Service来完成该操作。不应考虑使用新线程去完成耗时的操作，因为BroadcastReceiver本身的生命周期很短，可能出现的情况是子线程可能还没有结束，BroadcastReceiver就已经退出了。

如果BroadcastReceiver所在的进程结束，虽然该进程内还有用户启动的新线程，但由于该进程内不包含任何活动组件，因此系统可能在内存紧张时优先结束该进程。这样就可能导致BroadcastReceiver启动的子线程不能执行完成。

## 发送广播
在程序中发送广播十分简单，只要调用Context的sendBroadcast(Intent intent)方法即可，这条广播将会启动intent参数所对应的BroadcastReceiver。

下面一个简单的程序示范了如何发送Broadcast、使用BroadcastReceiver接收广播，该程序的Activity界面中包含一个按钮，当用户单击该按钮时程序会向外发送一条广播。该程序的代码如下。

```java
public class MainActivity extends Activity {
    Button send;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        send = (Button) findViewById(R.id.send);
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction("tk.zongzhankui.broadcast.BROADCAST");
                intent.putExtra("msg", "简单的消息");
                sendBroadcast(intent);
            }
        });
    }

}
```

```java
public class MyReceiver extends BroadcastReceiver {
    public MyReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "接收到的Intent的Action为：" + intent.getAction()
            + "\n消息内容是：" + intent.getStringExtra("msg"), Toast.LENGTH_LONG).show();
    }
}
```

正如上面的程序中看到的，当符合该MyReceiver的广播出现时，该MyReceiver的onReceive()方法将会被触发，从而在该方法中显示广播所携带的消息。

上面发送广播的程序中指定发送广播时所用的Intent的Action为tk.zongzhankui.broadcast.BROADCAST，这就需要配置上面的BroadcastReceiver应监听Action为该字符串的Intent，在AndroidManifest.xml文件中增加如下配置即可：

```xml
<receiver android:name=".MyReceiver">
	<intent-filter>
    	<action android:name="tk.zongzhankui.broadcast.BROADCAST" />
    </intent-filter>
</receiver>
```

[示例代码下载](http://pan.baidu.com/share/link?shareid=2601019306&uk=1543963338)

## 有序广播
Broadcast被分为如下两种。

- Normal Broadcast（普通广播）：Normal Broadcast是完全异步的，可以在同一时刻（逻辑上）被所有接收者接收到，消息传递的效率比较高。但缺点是接收者不能将处理结果传递给下一个接收者，并且无法终止Broadcast Intent的传播。
- Ordered Broadcast（有序广播）：Ordered Broadcast的接收者将按预先声明的优先级依次接收Broadcast。如：A的级别高于B、B的级别高于C，那么，Broadcast先传给A，再传给B，最后传给C。优先级别声明在<intent-filter.../>元素的android:priority属性中，数越大优先级别越高，取值范围为-1000~1000,优先级别也可以调用IntentFilter对象的setPriority()进行设置。Ordered Broadcast接受者可以终止Broadcast Intent的传播，Broadcast Intent的传播一旦终止，后面的接收者就无法接收到Broadcast。另外，Ordered Broadcast的接收者可以将数据传递给下一个接收者，如：A得到Broadcast后，可以往它的结果对象中存入数据，当Broadcast传给B时，B可以从A的结果对象中得到A存入的数据。

Context提供的如下两个方法用于发送广播。

- sendBroadcast()：发送Normal Broadcast。
- sendOrderedBroadcast()：发送Ordered Broadcast。

对于Ordered Broadcast而言，系统会根据接收者声明的优先级别按顺序逐个执行接收者，优先接收到Broadcast的接收者可以终止Broadcast，调用BroadcastReceiver的abortBroadcast()方法即可终止Broadcast。如果Broadcast被前面的接收者终止，后面的接收者就再也无法获取到Broadcast。

不仅如此，对于Ordered Broadcast而言，优先接收到Broadcast的接收者可以通过setResultExtras(Bundle)方法将处理结果存入Broadcast中，然后传给下一个接收者，下一个接收者通过代码：Bundle bundle = getResultExtras(true)可以获取上一个接收者存入的数据。

	系统收到短信，发出的Broadcast属于Ordered Broadcast。如果想阻止用户接到短信，可以通过设置优先级，让自定义的BroadcastReceiver先获取到Broadcast，然后终止Broadcast。

接下来介绍一个发送有序广播的示例，该程序的Activity界面上只有一个普通按钮，该按钮用于发送一条有序广播。该程序代码如下。

```java
public class MainActivity extends Activity {
    Button send;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        send = (Button) findViewById(R.id.send);
        send.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent();
                intent.setAction("tk.zongzhankui.sortedbroadcast.BROADCAST");
                intent.putExtra("msg", "简单的消息");
                sendOrderedBroadcast(intent, null);
            }
        });
    }

}
```

上面的程序中粗体字代码指定了Intent的Action属性，再调用sendOrderedBroadcast()方法来发送有序广播。对于有序广播而言，它会按优先级依次出发每个BroadcastReceiver的onReceive()方法。

下面的程序先定义第一个BroadcastReceiver。

```java
public class MyReceiver extends BroadcastReceiver {
    public MyReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context, "接收到的Intent的Action为："
                + intent.getAction()
                + "\n消息内容是：" + intent.getStringExtra("msg")
                , Toast.LENGTH_LONG).show();
        Bundle bundle = new Bundle();
        bundle.putString("first", "第一个BroadcastReceiver存入的消息");
        setResultExtras(bundle);
//        abortBroadcast();

    }
}
```

上面的BroadcastReceiver不仅处理了它所接收到的消息，而且向处理结果中存入了key为frist的消息，这个消息将可以被第二个BroadcastReceiver解析出来。

在AndroidManifest.xml文件中部署该BroadcastReceiver，并指定其优先级为20,配置片段如下：

```xml
<receiver android:name=".MyReceiver" >
	<intent-filter android:priority="20" >
		<action android:name="tk.zongzhankui.sortedbroadcast.BROADCAST" />
	</intent-filter>
</receiver>
```

接下来再为程序提供第二个BroadcastReceiver，这个BroadcastReceiver将会解析前一个BroadcastReceiver所存入的key为first的消息。该BroadcastReceiver的代码如下。

```java
public class MyReceiver2 extends BroadcastReceiver {
    public MyReceiver2() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        Bundle bundle = getResultExtras(true);
        String first = bundle.getString("first");
        Toast.makeText(context, "第一个Broadcast存入的消息为：" + first, Toast.LENGTH_LONG).show();
    }
}
```

上面的程序中粗体字代码用于解析前一个BroadcastReceiver存入结果中key为first的消息，在AndroidManifest.xml文件中配置该BroadcastReceiver，并指定其优先级为0.配置片段如下：

```xml
<receiver android:name=".MyReceiver2" >
	<intent-filter android:priority="0">
		<action android:name="tk.zongzhankui.sortedbroadcast.BROADCAST" />
	</intent-filter>
</receiver>
```

[示例代码下载](http://pan.baidu.com/share/link?shareid=2603760333&uk=1543963338)