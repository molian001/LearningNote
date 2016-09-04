
#android基础四大组件 
 
##Activity
###生命周期与启动模式
![](http://pic002.cnblogs.com/images/2012/325852/2012120122450787.png)

(1) onStart和onResume的区别是onStart可见，还没有出现在前台，无法和用户进行交互。onResume获取到焦点可以和用户交互。

(2)新Activity是透明主题时，旧Activity不会走onStop；

(3)Activity切换时，旧Activity的onPause会先执行，然后才会启动新的Activity；

(4)Activity在异常情况下被回收时，onSaveInstanceState方法会被回调，回调时机是在onStop之前，当Activity被重新创建的时候，onRestoreInstanceState方法会被回调，时序在onStart之后；

<font color=#0099ff size=4 face="黑体">系统只会在Activity即将被销毁并且有机会重新显示的情况下才会去调用它。</font>

(5)Activity的LaunchMode

a. standard 系统默认。每次启动会重新创建新的实例，谁启动了这个Activity，这个Activity就在谁的栈里。

b. singleTop 栈顶复用模式。该Activity的onNewIntent方法会被回调，onCreate和onStart并不会被调用。

c. singleTask 栈内复用模式。只要该Activity在一个栈中存在，都不会重新创建，onNewIntent会被回调。如果不存在，系统会先寻找是否存在需要的栈，如果不存在该栈，就创建一个任务栈，然后把这个Activity放进去；如果存在，就会创建到已经存在的这个栈中。

d. singleInstance。具有此种模式的Activity只能单独存在于一个任务栈。
	
<font color=#0099ff size=4 face="黑体">可在androidmenifest为activity指定模式,也可在代码里使用flag，若两者同时存在，以flags为准</font>

(6) 标识Activity任务栈名称的属性：TaskAffinity，默认为应用包名。

(7) IntentFilter匹配规则。

a. action匹配规则：要求intent中的action存在且必须和过滤规则中的其中一个相同 区分大小写；

b. category匹配规则：系统会默认加上一个android.intent.category.DEFAULT，所以intent中可以不存在category，但如果存在就必须匹配其中一个；

c. data匹配规则：data由两部分组成，mimeType和URI，要求和action相似。如果没有指定URI，URI的默认值为content和file（schema）

###activity启动过程:
Android系统中，有两种操作会引发Activity的启动，一种用户点击应用程序图标时，Launcher会为我们启动应用程序的主Activity；应用程序的默认Activity启动起来后，它又可以在内部通过调用startActvity接口启动新的Activity，依此类推，每一个Activity都可以在内部启动新的Activity。通过这种连锁反应，按需启动Activity，从而完成应用程序的功能。

无论是通过点击应用程序图标来启动Activity，还是通过Activity内部调用startActivity接口来启动新的Activity，都要借助于应用程序框架层的ActivityManagerService服务进程。在前面一篇文章Android系统在新进程中启动自定义服务过程（startService）的原理分析中，我们已经看到，Service也是由ActivityManagerService进程来启动的。在Android应用程序框架层中，ActivityManagerService是一个非常重要的接口，它不但负责启动Activity和Service，还负责管理Activity和Service。

Android应用程序框架层中的ActivityManagerService启动Activity的过程大致如下图所示：

![](http://hi.csdn.net/attachment/201108/14/0_1313305334OkCc.gif)


##广播

使用的场景如下：      
1.同一app内部的同一组件内的消息通信（单个或多个线程之间）不推荐；   

2.同一app内部的不同组件之间的消息通信（单个进程）不推荐；    

3.同一app具有多个进程的不同组件之间的消息通信；

4.不同app之间的组件之间消息通信；

5.Android系统在特定情况下与App之间的消息通信。

Android中的广播使用了观察者模式，基于消息的发布/订阅事件模型。因此，从实现的角度来看，Android中的广播将广播的发送者和接受者极大程度上解耦，使得系统能够方便集成，更易扩展。具体实现流程要点粗略概括如下：

1.广播接收者BroadcastReceiver通过Binder机制向AMS(Activity Manager Service)进行注册；

2.广播发送者通过binder机制向AMS发送广播；

3.AMS查找符合相应条件（IntentFilter/Permission等）的BroadcastReceiver，将广播发送到BroadcastReceiver（一般情况下是Activity）相应的消息循环队列中；

4.消息循环执行拿到此广播，回调BroadcastReceiver中的onReceive()方法。

广播发送者和广播接收者分别属于观察者模式中的消息发布和订阅两端，AMS属于中间的处理中心。广播发送者和广播接收者的执行是异步的，发出去的广播不会关心有无接收者接收，也不确定接收者到底是何时才能接收到。显然，整体流程与EventBus非常类似。

**-）静态注册**
	
	<receiver android:enabled=["true" | "false"]
	android:exported=["true" | "false"]
	android:icon="drawable resource"
	android:label="string resource"
	android:name="string"
	android:permission="string"
	android:process="string" >
	. . .
	</receiver>

android:enabled

这个属性用于定义系统是否能够实例化这个广播接收器，如果设置为true，则能够实例化，如果设置为false，则不能被实例化。默认值是true。

<application>元素有它自己的enabled属性，这个属性会应用给应用程序的所有组件，包括广播接收器。<application>和<receiver>元素的这个属性都必须是true，这个广播接收器才能够被启用。如果有一个被设置为false，该广播接收器会被禁止实例化。

android:exported  ——此broadcastReceiver能否接收其他App的发出的广播，这个属性默认值有点意思，其默认值是由receiver中有无intent-filter决定的，如果有intent-filter，默认值为true，否则为false。（同样的，activity/service中的此属性默认值一样遵循此规则）同时，需要注意的是，这个值的设定是以application或者application user id为界的，而非进程为界（一个应用中可能含有多个进程）；

android:name  —— 此broadcastReceiver类名；

android:permission  ——如果设置，具有相应权限的广播发送方发送的广播才能被此broadcastReceiver所接收；

android:process  ——broadcastReceiver运行所处的进程。默认为app的进程。可以指定独立的进程（Android四大基本组件都可以通过此属性指定自己的独立进程）

**-）动态注册** 

动态注册时，无须在AndroidManifest中注册<receiver/>组件。直接在代码中通过调用Context的registerReceiver函数，可以在程序中动态注册BroadcastReceiver。registerReceiver的定义形式如下：
	
	1 registerReceiver(BroadcastReceiver receiver, IntentFilter filter)

	2 registerReceiver(BroadcastReceiver receiver, IntentFilter filter, String broadcastPermission, Handler scheduler)

常见写法：
	
	 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mBroadcastReceiver = new MyBroadcastReceiver();
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(BROADCAST_ACTION);
        registerReceiver(mBroadcastReceiver, intentFilter);
    }
	 @Override
    protected void onDestroy() {
        super.onDestroy();
        unregisterReceiver(mBroadcastReceiver);
    }

注：Android中所有与观察者模式有关的设计中，一旦涉及到register，必定在相应的时机需要unregister。因此，上例在onDestroy()回到中需要unregisterReceiver(mBroadcastReceiver)。

当此Activity实例化时，会动态将MyBroadcastReceiver注册到系统中。当此Activity销毁时，动态注册的MyBroadcastReceiver将不再接收到相应的广播。

**-）广播发送及广播类型**

经常说”发送广播“和”接收“，表面上看广播作为Android广播机制中的实体，实际上这一实体本身是并不是以所谓的”广播“对象存在的，而是以”意图“（Intent）去表示。定义广播的定义过程，实际就是相应广播”意图“的定义过程，然后通过广播发送者将此”意图“发送出去。被相应的BroadcastReceiver接收后将会回调onReceive()函数。

下段代码片段显示的是一个普通广播的定义过程，并发送出去。其中setAction(..)对应于BroadcastReceiver中的intentFilter中的action。

	1 Intent intent = new Intent();
	2 intent.setAction(BROADCAST_ACTION);
	3 intent.putExtra("name", "qqyumidi");
	4 sendBroadcast(intent);

根据广播的发送方式，可以将其分为以下几种类型：         
1.Normal Broadcast：普通广播

2.System Broadcast: 系统广播

3.Ordered broadcast：有序广播

4.Sticky Broadcast：粘性广播(**在 android 5.0/api 21中deprecated,不再推荐使用，相应的还有粘性有序广播，同样已经deprecated)**

5.Local Broadcast：App应用内广播

下面分别总结下各种类型的发送方式及其特点。

**1).Normal Broadcast：普通广播**

此处将普通广播界定为：开发者自己定义的intent，以context.sendBroadcast_"AsUser"(intent, ...)形式。具体可以使用的方法有：
sendBroadcast(intent)/sendBroadcast(intent, receiverPermission)/sendBroadcastAsUser(intent, userHandler)/sendBroadcastAsUser(intent, userHandler,receiverPermission)。
普通广播会被注册了的相应的感兴趣（intent-filter匹配）接收，且顺序是无序的。如果发送广播时有相应的权限要求，BroadCastReceiver如果想要接收此广播，也需要有相应的权限。

**2).System Broadcast: 系统广播**

Android系统中内置了多个系统广播，只要涉及到手机的基本操作，基本上都会发出相应的系统广播。如：开启启动，网络状态改变，拍照，屏幕关闭与开启，点亮不足等等。每个系统广播都具有特定的intent-filter，其中主要包括具体的action，系统广播发出后，将被相应的BroadcastReceiver接收。系统广播在系统内部当特定事件发生时，有系统自动发出。

**3)Ordered broadcast：有序广播**

有序广播的有序广播中的“有序”是针对广播接收者而言的，指的是发送出去的广播被BroadcastReceiver按照先后循序接收。有序广播的定义过程与普通广播无异，只是其的主要发送方式变为：sendOrderedBroadcast(intent, receiverPermission, ...)。

对于有序广播，其主要特点总结如下：

1>多个具当前已经注册且有效的BroadcastReceiver接收有序广播时，是按照先后顺序接收的，先后顺序判定标准遵循为：将当前系统中所有有效的动态注册和静态注册的BroadcastReceiver按照priority属性值从大到小排序，对于具有相同的priority的动态广播和静态广播，动态广播会排在前面。

2>先接收的BroadcastReceiver可以对此有序广播进行截断，使后面的BroadcastReceiver不再接收到此广播，也可以对广播进行修改，使后面的BroadcastReceiver接收到广播后解析得到错误的参数值。当然，一般情况下，不建议对有序广播进行此类操作，尤其是针对系统中的有序广播。

**4)Sticky Broadcast：粘性广播(在 android 5.0/api 21中deprecated,不再推荐使用，相应的还有粘性有序广播，同样已经deprecated)。**

既然已经deprecated，此处不再多做总结。

**5)Local Broadcast：App应用内广播（此处的App应用以App应用进程为界）**

由前文阐述可知，Android中的广播可以跨进程甚至跨App直接通信，且注册是exported对于有intent-filter的情况下默认值是true，由此将可能出现安全隐患如下：
<font color=#0099ff size=4 face="黑体">

1.其他App可能会针对性的发出与当前App intent-filter相匹配的广播，由此导致当前App不断接收到广播并处理；

2.其他App可以注册与当前App一致的intent-filter用于接收广播，获取广播具体信息。

无论哪种情形，这些安全隐患都确实是存在的。由此，最常见的增加安全性的方案是：

1.对于同一App内部发送和接收广播，将exported属性人为设置成false，使得非本App内部发出的此广播不被接收；

2.在广播发送和接收时，都增加上相应的permission，用于权限验证；

3.发送广播时，指定特定广播接收器所在的包名，具体是通过intent.setPackage(packageName)指定在，这样此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。

App应用内广播可以理解成一种局部广播的形式，广播的发送者和接收者都同属于一个App。实际的业务需求中，App应用内广播确实可能需要用到。同时，之所以使用应用内广播时，而不是使用全局广播的形式，更多的考虑到的是Android广播机制中的安全性问题。

相比于全局广播，App应用内广播优势体现在：

1.安全性更高；

2.更加高效。

</font>
为此，Android v4兼容包中给出了封装好的LocalBroadcastManager类，用于统一处理App应用内的广播问题，使用方式上与通常的全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将主调context变成了LocalBroadcastManager的单一实例。

	//registerReceiver(mBroadcastReceiver, intentFilter);
	//注册应用内广播接收器
	localBroadcastManager = LocalBroadcastManager.getInstance(this);
	localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
	        
	//unregisterReceiver(mBroadcastReceiver);
	//取消注册应用内广播接收器
	localBroadcastManager.unregisterReceiver(mBroadcastReceiver);
	
	Intent intent = new Intent();
	intent.setAction(BROADCAST_ACTION);
	intent.putExtra("name", "qqyumidi");
	//sendBroadcast(intent);
	//发送应用内广播
	localBroadcastManager.sendBroadcast(intent);

**-)不同注册方式的广播接收器onReceive(context, intent)中的context具体类型**

1).对于静态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是ReceiverRestrictedContext；

2).对于全局广播的动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Activity Context；

3).对于通过LocalBroadcastManager动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Application Context。

注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册的ContextReceiver才有可能接收到（静态注册或其他方式动态注册的ContextReceiver是接收不到的）。

**-)android3.1后  静态注册   app关闭后不再能接收广播**

1).Android5.0/API level 21开始粘滞广播和有序粘滞广播过期，以后不再建议使用；

2).”静态注册的广播接收器即使app已经退出，主要有相应的广播发出，依然可以接收到，但此种描述自Android 3.1开始有可能不再成立“

Android 3.1开始系统在Intent与广播相关的flag增加了参数，分别是FLAG_INCLUDE_STOPPED_PACKAGES和FLAG_EXCLUDE_STOPPED_PACKAGES。

FLAG_INCLUDE_STOPPED_PACKAGES：包含已经停止的包（停止：即包所在的进程已经退出）

FLAG_EXCLUDE_STOPPED_PACKAGES：不包含已经停止的包

主要原因如下：

自Android3.1开始，系统本身则增加了对所有app当前是否处于运行状态的跟踪。在发送广播时，不管是什么广播类型，系统默认直接增加了值为FLAG_EXCLUDE_STOPPED_PACKAGES的flag，导致即使是静态注册的广播接收器，对于其所在进程已经退出的app，同样无法接收到广播。

详情参加Android官方文档：http://developer.android.com/about/versions/android-3.1.html#launchcontrols

由此，对于系统广播，由于是系统内部直接发出，无法更改此intent flag值，因此，3.1开始对于静态注册的接收系统广播的BroadcastReceiver，如果App进程已经退出，将不能接收到广播。

但是对于自定义的广播，可以通过复写此flag为FLAG_INCLUDE_STOPPED_PACKAGES，使得静态注册的BroadcastReceiver，即使所在App进程已经退出，也能能接收到广播，并会启动应用进程，但此时的BroadcastReceiver是重新新建的。

	1 Intent intent = new Intent();
	2 intent.setAction(BROADCAST_ACTION);
	3 intent.addFlags(Intent.FLAG_INCLUDE_STOPPED_PACKAGES);
	4 intent.putExtra("name", "qqyumidi");
	5 sendBroadcast(intent);
注1：对于动态注册类型的BroadcastReceiver，由于此注册和取消注册实在其他组件（如Activity）中进行，因此，不受此改变影响。

注2：在3.1以前，不少app可能通过静态注册方式监听各种系统广播，以此进行一些业务上的处理（如即时app已经退出，仍然能接收到，可以启动service等..）,3.1后，静态注册接受广播方式的改变，将直接导致此类方案不再可行。于是，通过将Service与App本身设置成不同的进程已经成为实现此类需求的可行替代方案。

##service
###service生命周期
**Managing the Lifecycle of a Service**  

　　service的生命周期，从它被创建开始，到它被销毁为止，可以有两条不同的路径：

**A started service**

　　被开启的service通过其他组件调用 startService()被创建。

　　这种service可以无限地运行下去，必须调用stopSelf()方法或者其他组件调用stopService()方法来停止它。

　　当service被停止时，系统会销毁它。

 

**A bound service**    
 
　　被绑定的service是当其他组件（一个客户）调用bindService()来创建的。

　　客户可以通过一个IBinder接口和service进行通信。

　　客户可以通过 unbindService()方法来关闭这种连接。

　　一个service可以同时和多个客户绑定，当多个客户都解除绑定之后，系统会销毁service。

 

　　这两条路径并不是完全分开的。

　　即是说，你可以和一个已经调用了 startService()而被开启的service进行绑定。

　　比如，一个后台音乐service可能因调用 startService()方法而被开启了，稍后，可能用户想要控制播放器或者得到一些当前歌曲的信息，可以通过bindService()将一个activity和service绑定。这种情况下，stopService()或 stopSelf()实际上并不能停止这个service，除非所有的客户都解除绑定。

 

**Implementing the lifecycle callbacks**   

　　和activity一样，service也有一系列的生命周期回调函数，你可以实现它们来监测service状态的变化，并且在适当的时候执行适当的工作。

　　下面的service展示了每一个生命周期的方法：

	public class ExampleService extends Service
	{
    int mStartMode; // indicates how to behave if the service is killed
    IBinder mBinder; // interface for clients that bind
    boolean mAllowRebind; // indicates whether onRebind should be used

    @Override
    public void onCreate()
    {
        // The service is being created
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId)
    {
        // The service is starting, due to a call to startService()
        return mStartMode;
    }

    @Override
    public IBinder onBind(Intent intent)
    {
        // A client is binding to the service with bindService()
        return mBinder;
    }

    @Override
    public boolean onUnbind(Intent intent)
    {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }

    @Override
    public void onRebind(Intent intent)
    {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }

    @Override
    public void onDestroy()
    {
        // The service is no longer used and is being destroyed
    }
	}

 

　　不像是activity的生命周期回调函数，你不需要调用基类的实现。

 ![](http://images.cnitblog.com/blog/325852/201303/24233205-ccefbc4a326048d79b111d05d1f8ff03.png)

　　这个图说明了service典型的回调方法，尽管这个图中将开启的service和绑定的service分开，但是你需要记住，任何service都潜在地允许绑定。

　　所以，一个被开启的service仍然可能被绑定。

　　实现这些方法，你可以看到两层嵌套的service的生命周期：

 

**The entire lifetime**

　　service整体的生命时间是从onCreate()被调用开始，到onDestroy()方法返回为止。

　　和activity一样，service在onCreate()中进行它的初始化工作，在onDestroy()中释放残留的资源。

　　比如，一个音乐播放service可以在onCreate()中创建播放音乐的线程，在onDestory()中停止这个线程。

 　　onCreate() 和 onDestroy()会被所有的service调用，不论service是通过startService()还是bindService()建立。

 

**The active lifetime**
  
　　service积极活动的生命时间（active lifetime）是从onStartCommand() 或onBind()被调用开始，它们各自处理由startService()或 bindService()方法传过来的Intent对象。

　　如果service是被开启的，那么它的活动生命周期和整个生命周期一同结束。

　　如果service是被绑定的，它们它的活动生命周期是在onUnbind()方法返回后结束。

　　注意：尽管一个被开启的service是通过调用 stopSelf() 或 stopService()来停止的，没有一个对应的回调函数与之对应，即没有onStop()回调方法。所以，当调用了停止的方法，除非这个service和客户组件绑定，否则系统将会直接销毁它，onDestory()方法会被调用，并且是这个时候唯一会被调用的回调方法。

 

**Managing the Lifecycle of a Bound Service**


　　当绑定service和所有客户端解除绑定之后，Android系统将会销毁它，（除非它同时被onStartCommand()方法开启）。

　　因此，如果你的service是一个纯粹的绑定service，那么你不需要管理它的生命周期。

　　然而，如果你选择实现onStartCommand()回调方法，那么你必须显式地停止service，因为service此时被看做是开启的。

　　这种情况下，service会一直运行到它自己调用 stopSelf()或另一个组件调用stopService()，不论它是否和客户端绑定。

　　另外，如果你的service被开启并且接受绑定，那么当系统调用你的 onUnbind()方法时，如果你想要在下次客户端绑定的时候接受一个onRebind()的调用（而不是调用 onBind()），你可以选择在 onUnbind()中返回true。

　　onRebind()的返回值为void，但是客户端仍然在它的 onServiceConnected()回调方法中得到 IBinder 对象。

　　下图展示了这种service（被开启，还允许绑定）的生命周期：
![](http://images.cnitblog.com/blog/325852/201303/24233647-0a03689c14da415ebbcb2913f2932d7d.png)


###AIDL

Binder学习：[http://blog.csdn.net/my9074/article/details/44097351](http://blog.csdn.net/my9074/article/details/44097351)

[http://blog.csdn.net/boyupeng/article/details/47011383](http://blog.csdn.net/boyupeng/article/details/47011383)

**首先新建一个aidl文件**

注意：尽管book类已经和IBookManager在一个包中，但仍然需要导入book类，这是aidl的特殊之处
	
	// IBookManager.aidl
	package com.example.zyj010.ipc;
	
	// Declare any non-default types here with import statements
	import com.example.zyj010.ipc.Book;
	import com.example.zyj010.ipc.IOnNewBookArrivedListener;
	interface IBookManager {
	    /**
	     * Demonstrates some basic types that you can use as parameters
	     * and return values in AIDL.
	     */
    List<Book> getBookList();
    void addBook(in Book book);
    void registeListener(IOnNewBookArrivedListener listener);
    void unregisteListener(IOnNewBookArrivedListener listener);
	}


然后编译会**自动生成**IBookManager.java文件  ，其中声明了一个内部类Stub，这个Stub就是一个Binder类

注意：所有可以在Binder中传输的接口都需要继承IInterface这个接口

	package com.example.zyj010.ipc;
	public interface IBookManager extends android.os.IInterface{
	.....
	}

IBooKManager.java中的方法：

**DESCRIPTOR**

binder的唯一标识，一般用当前Binder的类名表示，如："com.zyj.test.aidl.IbookManager"

**asInterface（android.os.Ibinder obj)**

用于将服务端中的Binder对象转换成客户端所需的AIDL接口类型的对象，这种转换过程是区分进程的，如果客户端和服务端位于同一进程，那么此方法返回的就是服务端的Stub对象本身，否则返回的是系统封装后的Stub.proxy对象  

**asBinder**

此方法返回当前Binder对象

**onTransact**

这个方法运行在服务端中的binder线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理，该方法原型为 public Boolean onTransact(int code,android.os.Parcel data,android.os.Parcel reply,int falgs).服务端通过code可以确定客户端请求的目标方法，接着从date取出目标方法所需的参数，然后执行目标方法。当目标方法执行完毕后，就向reply中写入返回值，onTransact方法的执行过程就是这样。需要注意的是，如果方法返回为false，客户端的请求会失败。

**Proxy#getBookList**

aidl中定义的方法，这些方法运行在客户端，当客户端调用此方法，它的内部实现为：创建该方法所需要的输入型Parcel 对象data、输出型Parcel对象_reply和返回值对象List；然后把该方法需要的参数信息写入_data中（如果有参数的话）；接着调用transact方法来发起RPC（远程过程调用）请求，同时当前线程挂起；然后服务端的onTransact会被调用，知道RPC过程返回后，当前线程继续执行，并从reply中取出RPC过程的返回结果，最后返回_reply中的数据。

 

然后在自己新建的service里面实现IBookManager接口并实现方法

	public class IBookService extends Service {
    private static  final String TAG="BMS";
    private CopyOnWriteArrayList<Book> mBookList=new CopyOnWriteArrayList<Book>();

    private Binder mBinder= new IBookManager.Stub() {
        @Override
        public List<Book> getBookList() throws RemoteException {
            return mBookList;
        }

        @Override
        public void addBook(Book book) throws RemoteException {
            mBookList.add(book);
        }

        @Override
        public void registeListener(IOnNewBookArrivedListener listener) throws RemoteException {

        }

        @Override
        public void unregisteListener(IOnNewBookArrivedListener listener) throws RemoteException {

        }
    };

    @Override
    public void onCreate() {
        super.onCreate();
        mBookList.add(new Book(1,"Android"));
        mBookList.add(new Book(2,"ios"));
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
	}
	

在activity在onServiceConnected里调用service里实现的接口方法：  

	public class MainActivity extends AppCompatActivity {

    @BindView(R.id.add) FloatingActionButton add;
    @BindView(R.id.get) FloatingActionButton get;
    @BindView(R.id.slideview) slideview slideview;
    ImageView imageView;
    private static final String TAG="BMA";
    private Button mbutton;
    private ServiceConnection mConnection=new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            IBookManager bookManager=IBookManager.Stub.asInterface(service);
            try {
                List<Book> list=bookManager.getBookList();
               Log.i(TAG,"query book list,list type:"+list.getClass().getCanonicalName());
                Log.i(TAG,"query book list,list type:"+list.toString());
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        ButterKnife.bind(this);
        Intent intent=new Intent(this,IBookService.class);
        bindService(intent,mConnection, Context.BIND_AUTO_CREATE);
        slideview=new slideview(this)；
       
    }

##ContentProvider

**1.ContentProvider概念讲解**
![](http://7xjqvu.com1.z0.glb.clouddn.com/15-8-21/58811327.jpg)


**2.使用系统提供的ContentProvider**

其实很多时候我们用到ContentProvider并不是自己暴露自己的数据，更多的时候通过 
ContentResolver来读取其他应用的信息，最常用的莫过于读取系统APP，信息，联系人， 
多媒体信息等！如果你想来调用这些ContentProvider就需要自行查阅相关的API资料了！ 
另外，不同的版本，可能对应着不同的URL！这里给出如何获取URL与对应的数据库表的字段， 
这里以最常用的联系人为例，其他自行google~
 
①来到系统源码文件下:all-src.rar -> TeleponeProvider -> AndroidManifest.xml查找对应API 

②打开模拟器的file exploer/data/data/com.android.providers.contacts/databases/contact2.db 

导出后使用SQLite图形工具查看，三个核心的表:raw_contact表，data表，mimetypes表！ 
下面演示一些基本的操作示例：



-）简单的读取手机联系人

核心代码：  
 
	private void getContacts(){
        //①查询raw_contacts表获得联系人的id
        ContentResolver resolver = getContentResolver();
        Uri uri = ContactsContract.CommonDataKinds.Phone.CONTENT_URI;
        //查询联系人数据
        cursor = resolver.query(uri, null, null, null, null);
        while(cursor.moveToNext())
        {
            //获取联系人姓名,手机号码
            String cName = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.DISPLAY_NAME));
            String cNum = cursor.getString(cursor.getColumnIndex(ContactsContract.CommonDataKinds.Phone.NUMBER));
            System.out.println("姓名:" + cName);
            System.out.println("号码:" + cNum);
            System.out.println("======================");
        }
        cursor.close();
    }

别忘了加读联系人的权限：

<uses-permission android:name="android.permission.READ_CONTACTS"/>



-）查询指定电话的联系人信息

核心代码：

 		
	private void queryContact(String number){
        Uri uri = Uri.parse("content://com.android.contacts/data/phones/filter/" + number);
        ContentResolver resolver = getContentResolver();
        Cursor cursor = resolver.query(uri, new String[]{"display_name"}, null, null, null);
        if (cursor.moveToFirst()) {
            String name = cursor.getString(0);
            System.out.println(number + "对应的联系人名称：" + name);
        }
    cursor.close();
	}

-）添加一个新的联系人

核心代码：

	private void AddContact() throws RemoteException, OperationApplicationException {
        //使用事务添加联系人
        Uri uri = Uri.parse("content://com.android.contacts/raw_contacts");
        Uri dataUri =  Uri.parse("content://com.android.contacts/data");

        ContentResolver resolver = getContentResolver();
        ArrayList<ContentProviderOperation> operations = new ArrayList<ContentProviderOperation>();
        ContentProviderOperation op1 = ContentProviderOperation.newInsert(uri)
                .withValue("account_name", null)
                .build();
        operations.add(op1);

        //依次是姓名，号码，邮编
        ContentProviderOperation op2 = ContentProviderOperation.newInsert(dataUri)
                .withValueBackReference("raw_contact_id", 0)
                .withValue("mimetype", "vnd.android.cursor.item/name")
                .withValue("data2", "Coder-pig")
                .build();
        operations.add(op2);

        ContentProviderOperation op3 = ContentProviderOperation.newInsert(dataUri)
                .withValueBackReference("raw_contact_id", 0)
                .withValue("mimetype", "vnd.android.cursor.item/phone_v2")
                .withValue("data1", "13798988888")
                .withValue("data2", "2")
                .build();
        operations.add(op3);

        ContentProviderOperation op4 = ContentProviderOperation.newInsert(dataUri)
                .withValueBackReference("raw_contact_id", 0)
                .withValue("mimetype", "vnd.android.cursor.item/email_v2")
                .withValue("data1", "779878443@qq.com")
                .withValue("data2", "2")
                .build();
        operations.add(op4);
        //将上述内容添加到手机联系人中~
        resolver.applyBatch("com.android.contacts", operations);
        Toast.makeText(getApplicationContext(), "添加成功", Toast.LENGTH_SHORT).show();
    }


别忘了权限：

    <uses-permission android:name="android.permission.WRITE_CONTACTS"/>
    <uses-permission android:name="android.permission.WRITE_PROFILE"/>

**3.自定义ContentProvider**

我们很少会自己来定义ContentProvider，因为我们很多时候都不希望自己应用的数据暴露给 
其他应用，虽然这样，学习如何ContentProvider还是有必要的，多一种数据传输的方式，是吧~ 
这是一个流程图：
![](http://7xjqvu.com1.z0.glb.clouddn.com/15-8-22/40787698.jpg)

接下来我们就来一步步实现：

在开始之前我们先要创建一个数据库创建类(数据库内容后面会讲~)：

	DBOpenHelper.Java

	public class DBOpenHelper extends SQLiteOpenHelper {

    final String CREATE_SQL = "CREATE TABLE test(_id INTEGER PRIMARY KEY AUTOINCREMENT,name)";

    public DBOpenHelper(Context context, String name, CursorFactory factory,
            int version) {
        super(context, name, null, 1);
    }


    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_SQL);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        // TODO Auto-generated method stub

    }

	}

Step 1：自定义ContentProvider类，实现onCreate()，getType()，根据需求重写对应的增删改查方法：

	
	NameContentProvider.java

	public class NameContentProvider extends ContentProvider {

    //初始化一些常量
     private static UriMatcher matcher = new UriMatcher(UriMatcher.NO_MATCH);        
     private DBOpenHelper dbOpenHelper;

    //为了方便直接使用UriMatcher,这里addURI,下面再调用Matcher进行匹配

     static{  
         matcher.addURI("com.jay.example.providers.myprovider", "test", 1);
     }  

    @Override
    public boolean onCreate() {
        dbOpenHelper = new DBOpenHelper(this.getContext(), "test.db", null, 1);
        return true;
    }

    @Override
    public Cursor query(Uri uri, String[] projection, String selection,
            String[] selectionArgs, String sortOrder) {
        return null;
    }

    @Override
    public String getType(Uri uri) {
        return null;
    }

    @Override
    public Uri insert(Uri uri, ContentValues values) {

        switch(matcher.match(uri))
        {
        //把数据库打开放到里面是想证明uri匹配完成
        case 1:
            SQLiteDatabase db = dbOpenHelper.getReadableDatabase();
            long rowId = db.insert("test", null, values);
            if(rowId > 0)
            {
                //在前面已有的Uri后面追加ID
                Uri nameUri = ContentUris.withAppendedId(uri, rowId);
                //通知数据已经发生改变
                getContext().getContentResolver().notifyChange(nameUri, null);
                return nameUri;
            }
        }
        return null;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        return 0;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection,
            String[] selectionArgs) {
        return 0;
    }

	}

Step 2：AndroidManifest.xml中为ContentProvider进行注册：

<!--属性依次为：全限定类名,用于匹配的URI,是否共享数据 -->
<provider android:name="com.jay.example.bean.NameContentProvider"
            android:authorities="com.jay.example.providers.myprovider"
            android:exported="true" />

好的，作为ContentProvider的部分就完成了！

接下来，创建一个新的项目，我们来实现ContentResolver的部分，我们直接通过按钮点击插入一条数据：

	MainActivity.java

	public class MainActivity extends Activity {

    private Button btninsert;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        btninsert = (Button) findViewById(R.id.btninsert);

        //读取contentprovider 数据  
        final ContentResolver resolver = this.getContentResolver();


        btninsert.setOnClickListener(new OnClickListener() {

            @Override
            public void onClick(View v) {
                 ContentValues values = new ContentValues();
                 values.put("name", "测试");
                 Uri uri = Uri.parse("content://com.jay.example.providers.myprovider/test");
                resolver.insert(uri, values);
                Toast.makeText(getApplicationContext(), "数据插入成功", Toast.LENGTH_SHORT).show();

            }
        });

    }
	}

如何使用？ 
好吧，代码还是蛮简单的，先运行作为ContentProvider的项目，接着再运行ContentResolver的项目， 
点击按钮插入一条数据，然后打开file exploer将ContentProvider的db数据库取出，用图形查看工具 
查看即可发现插入数据，时间关系，就不演示结果了~

4.通过ContentObserver监听ContentProvider的数据变化
![](http://7xjqvu.com1.z0.glb.clouddn.com/15-8-22/34168859.jpg)
