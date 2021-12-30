## [Java-WebSocket Github地址](https://github.com/TooTallNate/Java-WebSocket)

## [原文地址：Android WebSocket实现即时通讯功能](https://www.cnblogs.com/xch-yang/p/11726497.html)
[Android中Java-WebSocket简介](https://blog.csdn.net/helloworld19870427/article/details/87005478)

1、与websocket建立长连接
2、与websocket进行即时通讯
3、Service和Activity之间通讯和UI更新
4、弹出消息通知（包括锁屏通知）
5、心跳检测和重连（保证websocket连接稳定性）
6、服务（Service）保活


### 配置
#### 根目录Gradle
```groovy
To use Gradle add the maven central repository to your repositories list:
buildscript {
    repositories {
      mavenCentral()
  }
}
```
#### Module的gradle配置
```groovy
implementation 'org.java-websocket:Java-WebSocket:1.5.2'

```
#### 加入网络权限
```
```
### 新建客户端类
onOpen()方法在websocket连接开启时调用
onMessage()方法在接收到消息时调用
onClose()方法在连接断开时调用
onError()方法在连接出错时调用。
构造方法中的new Draft_17()代表使用的协议版本
```java
public class JWebSocketClient extends WebSocketClient {
    public JWebSocketClient(URI serverUri) {
        super(serverUri, new Draft_17());
    }

    @Override
    public void onOpen(ServerHandshake handshakedata) {
        Log.i("JWebSocketClient", "onOpen()");
    }

    @Override
    public void onMessage(String message) {
        Log.i("JWebSocketClient", "onMessage()");
    }

    @Override
    public void onClose(int code, String reason, boolean remote) {
        Log.i("JWebSocketClient", "onClose()");
    }

    @Override
    public void onError(Exception ex) {
        Log.i("JWebSocketClient", "onError()");
    }
}
```
### 4、建立websocket连接
建立连接只需要初始化此客户端再调用连接方法，需要注意的是WebSocketClient对象是不能重复使用的，所以不能重复初始化，其他地方只能调用当前这个Client。
为了方便对接收到的消息进行处理，可以在这重写onMessage()方法。初始化客户端时需要传入websocket地址（测试地址：ws://echo.websocket.org），websocket协议地址大致是这样的:ws:// ip地址 : 端口号

```java
URI uri = URI.create("ws://*******");
JWebSocketClient client = new JWebSocketClient(uri) {
    @Override
    public void onMessage(String message) {
        //message就是接收到的消息
        Log.e("JWebSClientService", message);
    }
};
```

连接时可以使用connect()方法或connectBlocking()方法，建议使用connectBlocking()方法，connectBlocking多出一个等待操作，会先连接再发送。
```java
try {
    client.connectBlocking();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```
运行之后可以看到客户端的onOpen()方法得到了执行，表示已经和websocket建立了连接
### 5、发送消息
发送消息只需要调用send()方法，如下
```java
if (client != null && client.isOpen()) {
    client.send("你好");
}
````
### 6、关闭socket连接
关闭连接调用close()方法，最后为了避免重复实例化WebSocketClient对象，关闭时一定要将对象置空。
```java
/**
 * 断开连接
 */
private void closeConnect() {
    try {
        if (null != client) {
            client.close();
        }
    } catch (Exception e) {
        e.printStackTrace();
    } finally {
        client = null;
    }
}
```
## 二、后台运行
一般来说即时通讯功能都希望像QQ微信这些App一样能在后台保持运行，当然App保活这个问题本身就是个伪命题，我们只能尽可能保活，所以首先就是建一个Service，将websocket的逻辑放入服务中运行并尽可能保活，让websocket保持连接。

##### 1、新建Service
新建一个Service，在启动Service时实例化WebSocketClient对象并建立连接，将上面的代码搬到服务里即可。

##### 2、Service和Activity之间通讯
由于消息是在Service中接收，从Activity中发送，需要获取到Service中的WebSocketClient对象，所以需要进行服务和活动之间的通讯，这就需要用到Service中的onBind()方法了。

首先新建一个Binder类,让它继承自Binder，并在内部提供相应方法，然后在onBind()方法中返回这个类的实例。

这里首先创建了一个ServiceConnection匿名类，在里面重写onServiceConnected()和onServiceDisconnected()方法，这两个方法会在活动与服务成功绑定以及连接断开时调用。在onServiceConnected()首先得到JWebSocketClientBinder的实例，有了这个实例便可调用服务的任何public方法，这里调用getService()方法得到Service实例，得到了Service实例也就得到了WebSocketClient对象，也就可以在活动中发送消息了。

```java
public class JWebSocketClientService extends Service {
    private URI uri;
    public JWebSocketClient client;
    private JWebSocketClientBinder mBinder = new JWebSocketClientBinder();

    //用于Activity和service通讯
    class JWebSocketClientBinder extends Binder {
        public JWebSocketClientService getService() {
            return JWebSocketClientService.this;
        }
    }
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
}
接下来就需要对应的Activity绑定Service，并获取Service的东西，代码如下

public class MainActivity extends AppCompatActivity {
    private JWebSocketClient client;
    private JWebSocketClientService.JWebSocketClientBinder binder;
    private JWebSocketClientService jWebSClientService;

    private ServiceConnection serviceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
            //服务与活动成功绑定
            Log.e("MainActivity", "服务与活动成功绑定");
            binder = (JWebSocketClientService.JWebSocketClientBinder) iBinder;
            jWebSClientService = binder.getService();
            client = jWebSClientService.client;
        }

        @Override
        public void onServiceDisconnected(ComponentName componentName) {
            //服务与活动断开
            Log.e("MainActivity", "服务与活动成功断开");
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        bindService();
    }

    /**
     * 绑定服务
     */
    private void bindService() {
        Intent bindIntent = new Intent(MainActivity.this, JWebSocketClientService.class);
        bindService(bindIntent, serviceConnection, BIND_AUTO_CREATE);
    }
}
```
## 三、从Service中更新Activity的UI
当Service中接收到消息时需要更新Activity中的界面，方法有很多，这里我们利用广播来实现，在对应Activity中定义广播
```java
public class MainActivity extends AppCompatActivity {
    ...
    private class ChatMessageReceiver extends BroadcastReceiver{

        @Override
        public void onReceive(Context context, Intent intent) {
             String message=intent.getStringExtra("message");
        }
    }

   
    /**
     * 动态注册广播
     */
    private void doRegisterReceiver() {
        chatMessageReceiver = new ChatMessageReceiver();
        IntentFilter filter = new IntentFilter("com.xch.servicecallback.content");
        registerReceiver(chatMessageReceiver, filter);
    }
    ...
}
```
上面的代码很简单，首先创建一个内部类并继承自BroadcastReceiver，也就是代码中的广播接收器ChatMessageReceiver，然后动态注册这个广播接收器。当Service中接收到消息时发出广播，就能在ChatMessageReceiver里接收广播了。
发送广播：
```java
client = new JWebSocketClient(uri) {
      @Override
      public void onMessage(String message) {
          Intent intent = new Intent();
          intent.setAction("com.xch.servicecallback.content");
          intent.putExtra("message", message);
          sendBroadcast(intent);
      }
};
```
获取广播传过来的消息后即可更新UI，具体布局就不细说，比较简单，看下我的源码就知道了，demo地址我会放到文章末尾。
## 四、消息通知
消息通知直接使用Notification，只是当锁屏时需要先点亮屏幕，代码如下
```java
/**
   * 检查锁屏状态，如果锁屏先点亮屏幕
   *
   * @param content
   */
  private void checkLockAndShowNotification(String content) {
      //管理锁屏的一个服务
      KeyguardManager km = (KeyguardManager) getSystemService(Context.KEYGUARD_SERVICE);
      if (km.inKeyguardRestrictedInputMode()) {//锁屏
          //获取电源管理器对象
          PowerManager pm = (PowerManager) this.getSystemService(Context.POWER_SERVICE);
          if (!pm.isScreenOn()) {
              @SuppressLint("InvalidWakeLockTag") PowerManager.WakeLock wl = pm.newWakeLock(PowerManager.ACQUIRE_CAUSES_WAKEUP |
                        PowerManager.SCREEN_BRIGHT_WAKE_LOCK, "bright");
              wl.acquire();  //点亮屏幕
              wl.release();  //任务结束后释放
          }
          sendNotification(content);
      } else {
          sendNotification(content);
      }
  }

  /**
   * 发送通知
   *
   * @param content
   */
  private void sendNotification(String content) {
      Intent intent = new Intent();
      intent.setClass(this, MainActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
      NotificationManager notifyManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
      Notification notification = new NotificationCompat.Builder(this)
              .setAutoCancel(true)
              // 设置该通知优先级
              .setPriority(Notification.PRIORITY_MAX)
              .setSmallIcon(R.mipmap.ic_launcher)
              .setContentTitle("昵称")
              .setContentText(content)
              .setVisibility(VISIBILITY_PUBLIC)
              .setWhen(System.currentTimeMillis())
              // 向通知添加声音、闪灯和振动效果
              .setDefaults(Notification.DEFAULT_VIBRATE | Notification.DEFAULT_ALL | Notification.DEFAULT_SOUND)
              .setContentIntent(pendingIntent)
              .build();
      notifyManager.notify(1, notification);//id要保证唯一
  }
```
如果未收到通知可能是设置里通知没开，进入设置打开即可，如果锁屏时无法弹出通知，可能是未开启锁屏通知权限，也需进入设置开启。为了保险起见我们可以判断通知是否开启，未开启引导用户开启，代码如下：

```java
/**
   * 检测是否开启通知
   *
   * @param context
   */
  private void checkNotification(final Context context) {
      if (!isNotificationEnabled(context)) {
          new AlertDialog.Builder(context).setTitle("温馨提示")
                  .setMessage("你还未开启系统通知，将影响消息的接收，要去开启吗？")
                  .setPositiveButton("确定", new DialogInterface.OnClickListener() {
                      @Override
                      public void onClick(DialogInterface dialog, int which) {
                          setNotification(context);
                      }
                  }).setNegativeButton("取消", new DialogInterface.OnClickListener() {
              @Override
              public void onClick(DialogInterface dialog, int which) {

              }
          }).show();
      }
  }
  /**
   * 如果没有开启通知，跳转至设置界面
   *
   * @param context
   */
  private void setNotification(Context context) {
      Intent localIntent = new Intent();
      //直接跳转到应用通知设置的代码：
      if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
          localIntent.setAction("android.settings.APP_NOTIFICATION_SETTINGS");
          localIntent.putExtra("app_package", context.getPackageName());
          localIntent.putExtra("app_uid", context.getApplicationInfo().uid);
      } else if (android.os.Build.VERSION.SDK_INT == Build.VERSION_CODES.KITKAT) {
            localIntent.setAction(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
            localIntent.addCategory(Intent.CATEGORY_DEFAULT);
            localIntent.setData(Uri.parse("package:" + context.getPackageName()));
      } else {
          //4.4以下没有从app跳转到应用通知设置页面的Action，可考虑跳转到应用详情页面
          localIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
          if (Build.VERSION.SDK_INT >= 9) {
                localIntent.setAction("android.settings.APPLICATION_DETAILS_SETTINGS");
                localIntent.setData(Uri.fromParts("package", context.getPackageName(), null));
          } else if (Build.VERSION.SDK_INT <= 8) {
                localIntent.setAction(Intent.ACTION_VIEW);
                localIntent.setClassName("com.android.settings", "com.android.setting.InstalledAppDetails");
                localIntent.putExtra("com.android.settings.ApplicationPkgName", context.getPackageName());
          }
      }
      context.startActivity(localIntent);
  }

  /**
   * 获取通知权限,检测是否开启了系统通知
   *
   * @param context
   */
  @TargetApi(Build.VERSION_CODES.KITKAT)
  private boolean isNotificationEnabled(Context context) {
      String CHECK_OP_NO_THROW = "checkOpNoThrow";
      String OP_POST_NOTIFICATION = "OP_POST_NOTIFICATION";

      AppOpsManager mAppOps = (AppOpsManager) context.getSystemService(Context.APP_OPS_SERVICE);
      ApplicationInfo appInfo = context.getApplicationInfo();
      String pkg = context.getApplicationContext().getPackageName();
      int uid = appInfo.uid;

      Class appOpsClass = null;
      try {
          appOpsClass = Class.forName(AppOpsManager.class.getName());
          Method checkOpNoThrowMethod = appOpsClass.getMethod(CHECK_OP_NO_THROW, Integer.TYPE, Integer.TYPE,
                    String.class);
          Field opPostNotificationValue = appOpsClass.getDeclaredField(OP_POST_NOTIFICATION);

          int value = (Integer) opPostNotificationValue.get(Integer.class);
          return ((Integer) checkOpNoThrowMethod.invoke(mAppOps, value, uid, pkg) == AppOpsManager.MODE_ALLOWED);

      } catch (Exception e) {
          e.printStackTrace();
      }
      return false;
  }
```
## 五、心跳检测和重连
由于很多不确定因素会导致websocket连接断开，例如网络断开，所以需要保证websocket的连接稳定性，这就需要加入心跳检测和重连。
心跳检测其实就是个定时器，每个一段时间检测一次，如果连接断开则重连，Java-WebSocket框架在目前最新版本中有两个重连的方法，分别是reconnect()和reconnectBlocking()，这里同样使用后者。
```java
private static final long HEART_BEAT_RATE = 10 * 1000;//每隔10秒进行一次对长连接的心跳检测
  private Handler mHandler = new Handler();
  private Runnable heartBeatRunnable = new Runnable() {
      @Override
      public void run() {
          if (client != null) {
              if (client.isClosed()) {
                  reconnectWs();
              }
          } else {
              //如果client已为空，重新初始化websocket
              initSocketClient();
          }
          //定时对长连接进行心跳检测
          mHandler.postDelayed(this, HEART_BEAT_RATE);
      }
  };

  /**
   * 开启重连
   */
  private void reconnectWs() {
      mHandler.removeCallbacks(heartBeatRunnable);
      new Thread() {
          @Override
          public void run() {
              try {
                  //重连
                  client.reconnectBlocking();
              } catch (InterruptedException e) {
                  e.printStackTrace();
              }
          }
      }.start();
  }
```

然后在服务启动时开启心跳检测

```java
mHandler.postDelayed(heartBeatRunnable, HEART_BEAT_RATE);//开启心跳检测

```

## 六、服务(Service)保活
如果某些业务场景需要App保活，例如利用这个websocket来做推送，那就需要我们的App后台服务不被kill掉，当然如果和手机厂商没有合作，要保证服务一直不被杀死，这可能是所有Android开发者比较头疼的一个事，这里我们只能尽可能的来保证Service的存活。

1、提高服务优先级（前台服务）
前台服务的优先级比较高，它会在状态栏显示类似于通知的效果，可以尽量避免在内存不足时被系统回收，前台服务比较简单就不细说了。有时候我们希望可以使用前台服务但是又不希望在状态栏有显示，那就可以利用灰色保活的办法，如下

```java
private final static int GRAY_SERVICE_ID = 1001;
  //灰色保活手段
  public static class GrayInnerService extends Service {
      @Override
      public int onStartCommand(Intent intent, int flags, int startId) {
          startForeground(GRAY_SERVICE_ID, new Notification());
          stopForeground(true);
          stopSelf();
          return super.onStartCommand(intent, flags, startId);
      }
      @Override
      public IBinder onBind(Intent intent) {
          return null;
      }
  }

   //设置service为前台服务，提高优先级
   if (Build.VERSION.SDK_INT < 18) {
       //Android4.3以下 ，隐藏Notification上的图标
       startForeground(GRAY_SERVICE_ID, new Notification());
   } else if(Build.VERSION.SDK_INT>18 && Build.VERSION.SDK_INT<25){
       //Android4.3 - Android7.0，隐藏Notification上的图标
       Intent innerIntent = new Intent(this, GrayInnerService.class);
       startService(innerIntent);
       startForeground(GRAY_SERVICE_ID, new Notification());
   }else{
       //暂无解决方法
       startForeground(GRAY_SERVICE_ID, new Notification());
   }
```
AndroidManifest.xml中注册这个服务

 ```xml
  <service android:name=".im.JWebSocketClientService$GrayInnerService"
       android:enabled="true"
       android:exported="false"
       android:process=":gray"/>
```
这里其实就是开启前台服务并隐藏了notification，也就是再启动一个service并共用一个通知栏，然后stop这个service使得通知栏消失。但是7.0以上版本会在状态栏显示“正在运行”的通知，目前暂时没有什么好的解决办法。

2、修改Service的onStartCommand 方法返回值
```java
 @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
      ...
      return START_STICKY;
  }
```
onStartCommand()返回一个整型值，用来描述系统在杀掉服务后是否要继续启动服务，START_STICKY表示如果Service进程被kill掉，系统会尝试重新创建Service。

3、锁屏唤醒
```java
PowerManager.WakeLock wakeLock;//锁屏唤醒
  private void acquireWakeLock()
  {
      if (null == wakeLock)
      {
          PowerManager pm = (PowerManager)this.getSystemService(Context.POWER_SERVICE);
          wakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK|PowerManager.ON_AFTER_RELEASE, "PostLocationService");
          if (null != wakeLock)
          {
              wakeLock.acquire();
          }
      }
  }
```
获取电源锁，保持该服务在屏幕熄灭时仍然获取CPU时，让其保持运行。

**4、其他保活方式**
服务保活还有许多其他方式，比如进程互拉、一像素保活、申请自启权限、引导用户设置白名单等，其实Android 7.0版本以后，目前没有什么真正意义上的保活，但是做些处理，总比不做处理强。这篇文章重点是即时通讯，对于服务保活有需要的可以自行查阅更多资料，这里就不细说了。

最后附上这篇文章源码地址，GitHub：[https://github.com/yangxch/WebSocketClient](https://github.com/yangxch/WebSocketClient)，如果有帮助帮忙点个star吧。

# [Android WebSocket实现即时通讯功能](https://www.cnblogs.com/xch-yang/p/11726497.html)



```java
public class SocketService extends Service {
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
 
    WebSocketClient client;
 
    @Override
    public void onCreate() {
        super.onCreate();
 
        try {
            client = new WebSocketClient(new URI("填入与后台连接的url")) {
 
                @Override
                public void onWebsocketPong(WebSocket conn, Framedata f) {
                    super.onWebsocketPong(conn, f);
//                    LogUtils.showLog(f.getPayloadData().toString());
//                    LogUtils.showLog("socket ping"+f.getOpcode());
                }
 
                @Override
                public void onOpen(ServerHandshake serverHandshake) {
                    // 连接打开以后的回调
 
                }
 
 
                @Override
                public void onMessage(String message) {
                    // 收到消息的回调
                    LogUtils.showLog("socket接收服务器发来的消息：" + message);
                }
 
                @Override
                public void onError(Exception ex) {
                    // 连接出错的回调
                }
 
                @Override
                public void onClose(int code, String reason, boolean remote) {
                    // 连接关闭的回调， remote如果为true表示被服务端cut掉了
 
                }
 
                @Override
                public void onMessage(ByteBuffer bytes) {
                    // 返回的字节流消息
                    LogUtils.showLog("socket bytebuffer bytes");
                }
            };
 
            client.connect();
        } catch (Exception e) {
 
        }
    }
 
 
    /**
     * 重连websocket
     * 此方法在handler中调用，由于重新连接websocket的线程如果与原先连接websocket的线程相同，会报异常，
     * 所以切换到HandlerThread的线程中重连
     */
    private void reConnectWebSocket() {
        if (null != client && !client.isOpen() && !isReConnecting) {
            //LogUtils.showLog("socket onStartConnect");
            client.reconnect();
        }
    }
 
 
    @Override
    public void onDestroy() {
        super.onDestroy();
 
        try {
            LogUtils.showLog("socket service onDestroy()");
            client.close();
        } catch (Exception e) {
 
        }
    }
}
```
1、重连机制需要自己来写：

（1）监听网络连接， 当从断开到连上就可以调用重连

（2）在client回调的onError和onClose中调用重连

2、心跳机制需要自己写

自己写一个循环的handler， 隔20秒就发送一个心跳：client.sendPing()， 这时候onWebsocketPong()会回调一次，这就是一次心跳

3、后端传回来的消息在onMessage中返回来， 我们可以跟后台定格式，哪些业务需要处理什么格式的数据，然后根据type来分发给业务。






## WebSocketManager  WebSocketClient的使用
```java
 /**
     * 继承WebSocketClient，获取消息.注意各个回调函数是子线程
     */
    private class MySocketClient extends WebSocketClient{


        private Disposable disposable;

        private MySocketClient(URI serverUri, Draft draft) {
            super(serverUri, draft);
        }

        @Override
        public void onOpen(ServerHandshake handshakedata) {

            try {
                Log.i("MySocketClient","onOpen:"+handshakedata.getHttpStatusMessage());

                //建立连接之后，先发送一个注册请求
                register("PUSH1001");

                //启动轮询
                disposable = active();
                timeDisposable = disposable;

                webSocketIsActive = true;
            }catch (Exception e){
                e.printStackTrace();
            }

        }

        @Override
        public void onMessage(String message) {
     
            handlerMessage(message);
        }

        @Override
        public void onMessage(final ByteBuffer message) {
            String content = getString(message);
     
            handlerMessage(content);
        }

        @Override
        public void onClose(int code, String reason, boolean remote) {
    

            webSocketIsActive = false;

            try {
                if(disposable!=null && !disposable.isDisposed()){
                    disposable.dispose();
                }
            }catch (Exception e){
                e.printStackTrace();
            }

        }

        @Override
        public void onError(Exception ex) {
 
            if(disposable!=null && !disposable.isDisposed()){
                disposable.dispose();
            }
        }

    }

```