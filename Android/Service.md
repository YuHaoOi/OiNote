### Start Service

#### 使用
```xml
<!--
  代表服务可以被远程启动：可以从别的应用启动服务
  android:enabled="true"
  android:exported="true"-->
<service
    android:name=".MyService"
    android:enabled="true"
    android:exported="true"/>
```
```java
public class MyService extends Service{

    @Nullable
    @Override
    public IBinder onBind(Intent intent){
        return null;
    }
     //可以调用多次
    /**
     * onStartCommand的返回值就代表了服务的粘性
     * START_STICKY:粘性服务，被意外杀死后，服务会在资源足够的情况下重建,不会重传intent
     * START_REDELIVER_INTENT:粘性服务， 被意外杀死后，服务会在资源足够的情况下重建，同时会重传intent
     * START_NOT_STICKY:非粘性服务，被意外中止后，服务不能自动重建
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId){
        //在主线程运行,需要开线程运行耗时操作
        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy(){
        super.onDestroy();
    }
}
```
```java
Intent intent = new Intent(this, MyService.class);
startService(intent);

//开启的Server记得关闭
stopSelf(); //自己关闭
stopService(intent); //外界
```

### IntentService

#### 使用场景
它拥有较高的优先级，不易被系统杀死（继承自Service的缘故），因此比较适合执行一些高优先级的异步任务。

#### 弊端
因为 IntentService 内置的是 HandlerThread 作为异步线程，所以每一个交给 IntentService 的任务都将以队列的方式逐个被执行到，一旦队列中有某个任务执行时间过长，那么就会导致后续的任务都会被延迟处理。

#### 使用
```java
/**
 * 不需要自己开启线程，onHandleIntent方法就是在子线程上运行的
 * 下载完成后，不需要手动关闭服务，会自行关闭
 *
 * 必须重写两个方法：
 * 1.构造方法:必须是无参构造，没有无参构造，会报错。原因：service的对象是系统创建的，系统创建时需要调用无参构造。
 * 2.onHandleIntent方法
*/
public class MyIntentService extends IntentService{

    public MyIntentService(){
        super("MyIntentService"); //调用有参构造，参数是线程名称
    }

    @Override
    protected void onHandleIntent(Intent intent){
      //这个方法运行在子线程
    }

    public static void start(Context context) {
        Intent intent = new Intent(context, MyIntentService.class);
        context.startService(intent);
    }
}
```


### bindService

#### 使用

```java
/**
 *
 *
 * 绑定服务：
 * 特点：1.服务和绑定他的组件之间可以通信
 *      2.绑定他的组件退出，服务也就会解除绑定并停止,就算没有手动解绑，服务也会解绑并退出， 如果Conn对象没有解绑的话会出现leak异常,所以最好在绑定服务的组件退出的时候，手动解除绑定。
 *
 *
 * 绑定服务实现步骤：
 * 客户端：
 * 1.创建类实现ServiceConnection接口，重写方法，实例化onServiceConnected被回调时会接收服务端传来的IBinder对象
 * 2.创建intent对象
 * 3.通过调用bindService方法，传入intent和ServiceConnection对象
 *
 * 服务端：
 * 1.定义类继承于Service,重写方法必须重写onBind方法，这个方法需要返回一个服务和外界交互的IBinder对象
 *   1.1 定义类实现IBinder接口，将服务可以提供给外界的方法封装进去可以直接继承IBinder的实现类Binder
 * 2.注册到清单
 *
 */

public class MyServer extends Service{
    MyIBinder binder = new MyIBinder(); //实例化Ibinder对象

    //绑定服务的时候回调
    //返回一个IBinder对象，服务端用于和外界进行联络的对象（经纪人）
    @Nullable
    @Override
    public IBinder onBind(Intent intent){
        return binder;
    }

    public class MyIBinder extends Binder {
        public MyServer getServiceInstance(){
            return MyServer.this;
        }
    }

    public void MyFun(){
        Log.d("MyServer", "aaaa");
    }
}
```


```java
Intent intent = new Intent(MainActivity.this, MyServer.class);
bindService(intent, conn, BIND_AUTO_CREATE); //一个标记，一般都填BIND_AUTO_CREATE
private ServiceConnection conn = new ServiceConnection(){
    //当服务连接成功回调
    //其中参数二：就是服务端的onBind方法返回来的IBinder
    @Override
    public void onServiceConnected(ComponentName name, IBinder service){
        MyServer.MyIBinder binder = (MyServer.MyIBinder) service;
        myServer = binder.getServiceInstance();
    }

    //当服务连接断开时回调
    @Override
    public void onServiceDisconnected(ComponentName name){
    }
};
```
```java
myServer.MyFun(); //调用服务,暴露Server--不采用,用于理解
```
```java
//最后记得解绑服务
if(conn != null){ //多次解绑会出异常
    unbindService(conn);
    conn = null;
}
```

#### 调用服务方法的推荐做法
```java
public class MyBinder extends Binder implements IMyServer{
    @Override
    public void movie(String name){
        Log.d("MyBinder", name);
    }
}
```
```java
public interface IMyServer{
    void movie(String name);
}

@Override
public IBinder onBind(Intent intent){
    return new MyBinder();
}
```
```java
private ServiceConnection conn = new ServiceConnection(){
    @Override
    public void onServiceConnected(ComponentName name, IBinder service){
        server = (IMyServer) service; //服务实现接口,给主线程使用
    }

    @Override
    public void onServiceDisconnected(ComponentName name){
    }
};
```
