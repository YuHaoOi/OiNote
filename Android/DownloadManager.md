###  DownloadManager实现文件下载
从Android 2.3（API level 9）开始Android用系统服务(Service)的方式提供了Download Manager来优化处理长时间的下载操作。Download Manager处理HTTP连接并监控连接中的状态变化以及系统重启来确保每一个下载任务顺利完成。在大多数涉及到下载的情况中使用Download Manager都是不错的选择，特别是当用户切换不同的应用以后下载需要在后台继续进行，以及当下载任务顺利完成非常重要的情况。

### 使用
#### 下载
```java
downloadManager = (DownloadManager)getSystemService(Context.DOWNLOAD_SERVICE);
Uri uri = Uri.parse("http://media.mp4");//下载地址
DownloadManager.Request request = new DownloadManager.Request(uri);
//文件只能在WiFi下进行下载，如果没有连接Wifi默认不会下载，也不提示。
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);
request.setTitle("正在下载...");//改变默认的Notification标题
request.setDescription("提示下载信息");//Notification提示信息
/**
 * 这里返回的myDownloadId变量是系统为当前的下载请求分配的一个唯一的ID，
 * 我们可以通过这个ID重新获得这个下载任务，
 * 进行一些自己想要进行的操作或者查询下载的状态以及取消下载等等。
 */
//开始下载
myDownloadId = downloadManager.enqueue(request);
```
##### 进度监听
```java
final DownloadManager.Query query = new DownloadManager.Query();
timer = new Timer();
TimerTask task = new TimerTask() {
    @Override
    public void run() {
        Cursor cursor = downloadManager.query(query.setFilterById(myDownloadId));
        if (cursor != null && cursor.moveToFirst()) {
            if (cursor.getInt(cursor.getColumnIndex(DownloadManager.COLUMN_STATUS)) == DownloadManager.STATUS_SUCCESSFUL) {
                //下载完成
                timer.cancel();
                timer = null;
            }
            int bytes_downloaded = cursor.getInt(cursor.getColumnIndex(DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR));
            int bytes_total = cursor.getInt(cursor.getColumnIndex(DownloadManager.COLUMN_TOTAL_SIZE_BYTES));
            //下载进度
            int pro =  (bytes_downloaded * 100) / bytes_total;
        }
        cursor.close();
    }
};
timer.schedule(task, 0,1000);
```

#### 接收下载完成的广播
如果需要得到下载完成的事件，需要注册一个广播接受者来接收 DownloadManager.ACTION_DOWNLOAD_COMPLETE广播，这个广播会包含一个DownloadManager.EXTRA_DOWNLOAD_ID信息在intent中包含了已经完成的这个下载的ID。

```java
//注册广播：
MyDownloadReceiver myDownloadReceiver = new MyDownloadReceiver();
IntentFilter intentFilter = new IntentFilter();
intentFilter.addAction(DownloadManager.ACTION_DOWNLOAD_COMPLETE);
intentFilter.addAction(DownloadManager.ACTION_NOTIFICATION_CLICKED);
registerReceiver(myDownloadReceiver, intentFilter);
```
```java
class MyDownloadReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        //下载完成广播
        if (DownloadManager.ACTION_DOWNLOAD_COMPLETE.equals(intent.getAction())){

            long downloadId = intent.getLongExtra(DownloadManager.EXTRA_DOWNLOAD_ID, -1);
            if (myDownloadId == downloadId) {
                DownloadManager.Query myDownloadQuery = new DownloadManager.Query();
                myDownloadQuery.setFilterById(downloadId);
                Cursor c = downloadManager.query(myDownloadQuery);
                if (c.moveToFirst()) {
                    String fileName = c.getString(c.getColumnIndex(DownloadManager.COLUMN_LOCAL_FILENAME));
                    String fileUri = c.getString(c.getColumnIndexOrThrow(DownloadManager.COLUMN_LOCAL_URI));
                    Log.e("oi", fileName);
                }
                c.close();//关闭游标
            }
            //通知被点击
        } else if( DownloadManager.ACTION_NOTIFICATION_CLICKED.equals(intent.getAction())){
            long[] ids = intent.getLongArrayExtra(DownloadManager.EXTRA_NOTIFICATION_CLICK_DOWNLOAD_IDS);
            //点击通知栏取消下载,或者其他操作
            downloadManager.remove(ids);
            Log.e("oi","已经取消下载");
        }
    }
}
```

---

### 配置功能
#### 取消下载
```java
//如果一个下载被取消了，所有相关联的文件，部分下载的文件和完全下载的文件都会被删除
downloadManager.remove();
```
#### 指定下载位置，及文件名称
```java
//Android/data/包名/files/Download/dxtj.apk
request.setDestinationInExternalFilesDir(this, Environment.DIRECTORY_DOWNLOADS ,  "dxtj.apk");
//SD卡
request.setDestinationInExternalPublicDir("/epmyg/", "dxtj.apk");
```

#### 指定下载的网络类型
```java
//指定在WIFI状态下，执行下载操作。
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_WIFI);
//指定在MOBILE状态下，执行下载操作
request.setAllowedNetworkTypes(DownloadManager.Request.NETWORK_MOBILE);
//是否允许漫游状态下，执行下载操作
request.setAllowedOverRoaming(boolean);
//是否允许“计量式的网络连接”执行下载操作
request.setAllowedOverMetered(boolean); //默认是允许的。
```

#### 定制Notification样式
```java
//设置Notification的标题和描述
request.setTitle("标题");  
request.setDescription("描述");
//设置Notification的显示，和隐藏。
request.setNotificationVisibility(visibility);
```
