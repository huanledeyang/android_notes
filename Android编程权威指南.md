###使用adb工具从命令行启动activity
1. 配置activity
  android:exported = "true"
2. adb 命令
  adb shell am start -n 包名/activity名
  
###使用格式化字符串
'<string name="name">%1$s! hello, how are u ? Hi, %2$S, I am fine !</string>

String s = getString(R.string.name, "Tom", "lucy");'

###检查可以响应的activity
    'PackageManager pm = getPackageManager();
    List<ResolveInfo> lists = pm.queryIntentActivities(yourIntent, 0);
    boolean isIntentSafe = lists.size() > 0;'

###检查后台网络的可用性
'ConnectivityManager cm = (ConnectivityManager)
            getSystemService(Context.CONNECTIVITY_SERVICE);
    @SuppressWarnings("deprecation")
    boolean isNetworkAvailable = cm.getBackgroundDataSetting() &&
        cm.getActiveNetworkInfo() != null;'

###Notification用法

'Resources r = getResources();
        PendingIntent pi = PendingIntent
            .getActivity(this, 0, new Intent(this, PhotoGalleryActivity.class), 0);

        Notification notification = new NotificationCompat.Builder(this)
            .setTicker(r.getString(R.string.new_pictures_title))
            .setSmallIcon(android.R.drawable.ic_menu_report_image)
            .setContentTitle(r.getString(R.string.new_pictures_title))
            .setContentText(r.getString(R.string.new_pictures_text))
            .setContentIntent(pi)
            .setAutoCancel(true)
            .build();

        NotificationManager notificationManager = (NotificationManager)
            getSystemService(NOTIFICATION_SERVICE);

        notificationManager.notify(0, notification);'


###FragmentStatePagerAdapter与FragmentPagerAdapter
-使用FragmentStatePagerAdapter会销毁掉不需要的fragment。事务提交后，可将fragment从activity的FragmentManager中彻底移除。
FragmentStatePagerAdapter类名中的“state”表明：在销毁fragment时，它会将其onSaveInstanceState(Bundle)方法中的Bundle信息保存下来。
用户切换回原来的页面后，保存的实例状态可用于恢复生成新的fragment（如图11-3所示）。

-相比之下，FragmentPagerAdapter的做法大不相同。对于不再需要的fragment，FragmentPagerAdapter则选择调用事务的detach(Fragment)方法，
而非remove(Fragment)方法，来处理它。也就是说，FragmentPagerAdapter只是销毁了fragment的视图，但仍将fragment实例保留在FragmentManager中
。因此，FragmentPagerAdapter创建的fragment永远不会被销毁。
