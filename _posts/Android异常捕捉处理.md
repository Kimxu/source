title: Android异常捕捉处理
date: 2015-12-02 14:34:51
tags: Android Thread.UncaughtExceptionHandler
---

*通常情况下，我们做Android开发都回遇到难以避免的bug，导致应用意外退出，给用户弹出一个系统的意外退出对话框，但是这样的用户体验很不好。我们可以使用UncaughtExceptionHandler捕获全局异常*
---- 


先来看代码
/**
 * Application基类
 * Created by xuzhiguo on 15/11/19.
 */
public class KBaseApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        /** 异常自己处理 */Thread.setDefaultUncaughtExceptionHandler(mUncaughtExceptionHandler);
    }

    /**
     * 以下为uncaught exception的处理代码 防止程序崩溃
     */
    private Thread.UncaughtExceptionHandler mDefaultHandler = Thread.getDefaultUncaughtExceptionHandler();
    private Thread.UncaughtExceptionHandler mUncaughtExceptionHandler = new Thread.UncaughtExceptionHandler() {
        @Override
        public void uncaughtException(Thread thread, Throwable ex) {
            if (!handleException(ex) && mDefaultHandler != null) {
                mDefaultHandler.uncaughtException(thread, ex);
            } else {
                restartApplication();
            }
            System.exit(0);
            android.os.Process.killProcess(android.os.Process.myPid());
        }
    };

    /**
     * 处理异常
     *
     * @param ex
     * @return boolean
     */
    private boolean handleException(Throwable ex) {
        if (ex == null) {
            return false;
        }
        new Thread(
                new Runnable() {
                    @Override
                    public void run() {
                        Looper.prepare();
                        ////传入参数必须为Activity，否则Toast不显示
                        Toast.makeText(KBaseApplication.this,"哎呀出现点小问题",Toast.LENGTH_LONG).show();
                        Looper.loop();
                    }
                }
        ).start();
        return true;
    }

    /**
     * 重启程序
     */
    private void restartApplication() {
        Intent intent = getPackageManager().getLaunchIntentForPackage(getPackageName());
        intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
        startActivity(intent);
    }
}


我们只需要在**Oncreate**方法里面调用* Thread.setDefaultUncaughtExceptionHandler(mUncaughtExceptionHandler);*就可以把自定义的全局异常捕获绑定到自己写的方法中啦。
