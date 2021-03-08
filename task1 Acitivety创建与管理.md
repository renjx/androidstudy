## task1 Activity创建与管理

在Android中，Activity是四大组件之一，主要用于显示界面和用户进行交互。首先，我们从简单的界面开始，比如闪屏界面。闪屏可以让应用打开比较自然，不是突然进入应用内部，而是出现一个图片，让你在等待的过程中不那么无聊。系统也可以在闪屏的同时加载资源或者播放广告。下面我们在我们的第一个程序上面创建一个闪频，效果是程序打开3s后跳转到原来的主界面。

###  创建闪频Activity

右键左边栏中任何位置New->Activity->Empty Activity，出现创建Activity对话框，如图所示。

![img](images/1.1createsplashactivity.png)

设置Activity Name为SplashActivity。勾选Generate a Layout File，这样系统会在res->layout下生成一个布局文件。Layout Name默认为activity_splash。然后勾选Launcher Activity，表示app启动时首先运行SplashActivity。后面的都默认，点击Finish。

系统自动在AndroidManifest.xml中加入下面一节activity配置，表示从SplashActivity启动。如图所示。

![img](images/1.1splashactivitymanifest.png)

系统在java->pub.renge.myfirstapp下自动生成了SplashActivity.java文件，我们可以在这个文件中编写程序设置闪屏几秒中，下一个屏幕是什么。

系统在res->layout下自动生成activity_splash.xml配置文件，在这里可以设置闪屏的界面。

### 修改闪频布局

activity_splash.xml文件以xml文件保存布局样式，编辑方式有两种，代码模式和设计模式。如果所示，左边是代码，纯文本，右边是设计器，拖拽的模式，所见即所得，这是一种文件的两种不同编辑方式而已，最终还是文本文件的方式保存。

![img](images/1.1activitysplashxml.png)

具体文件内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".SplashActivity">

    <ImageView
        android:id="@+id/splash_image"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:scaleType="fitXY"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:srcCompat="@drawable/task1_1splashimage" />

    <TextView
        android:id="@+id/splash_app_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="150dp"
        android:text="@string/app_name"
        android:textColor="@color/white"
        android:textSize="50sp"
        android:textStyle="bold"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/splash_slogan"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="50dp"
        android:text="@string/splash_slogan"
        android:textColor="@color/alpha_90_white"
        android:textSize="14sp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/splash_app_name" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="500dp"
        android:layout_marginBottom="100dp"
        android:background="@color/alpha_10_white"
        android:orientation="vertical"
        android:visibility="visible"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/splash_slogan"
        >

        <TextView
            android:id="@+id/splash_version_name"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:textColor="@color/white"
            android:textSize="15sp" />

        <TextView
            android:id="@+id/splash_copyright"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="10dp"
            android:gravity="center"
            android:textColor="@color/white"
            android:textSize="12sp" />

    </LinearLayout>


</androidx.constraintlayout.widget.ConstraintLayout>
```

设计效果如图所示，版本号和版权声明并没有填，这个主要是给大家演示下，如何用程序填写。

![img](images/task1.1activitysplashdesign.png)

图片素材来自网络，大家可以自行搜索相关图片，然后把图片导入到res的drawable下，具体方法是点击图片中ImageView的左边栏选择图片标记，然后点击Import Drawables上传图片即可，记得xml文件中图片来源要和实际图片文件名称一致。

![img](images/task1.1pickimage.png)

对于TextView控件的text和color属性，这里并没有写死，而是用一个常量来表示，比如@string/app_name,表示在strings.xml里面有个变量app_name可以设置，@color表示在colors.xml中设置颜色。这种方式可以很方便修改这些值。如图所示。

![img](images/task1.1stringandcolor.png)



###　设置动画

在Android中动画也可以使用xml文件来表示，我们采用放大缩小并改变透明度的方式作为动画，每种效果2秒种，文件至于res的anim下，名称为splash.xml具体xml如下。

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <scale android:fromXScale="2.0" android:toXScale="1.0"
        android:fromYScale="2.0" android:toYScale="1.0"
        android:pivotX="50%p" android:pivotY="50%p"
        android:duration="2000" />
    <scale android:fromXScale="1.0" android:toXScale=".5"
        android:fromYScale="1.0" android:toYScale=".5"
        android:pivotX="50%p" android:pivotY="50%p"
        android:duration="2000" />
    <alpha android:fromAlpha="1.0" android:toAlpha="0"
        android:duration="2000"/>
</set>
```

### 设置AndroidManifest.xml

这个文件中我们要修改splashactivity的配置为：

```
<activity android:name=".SplashActivity"
    android:theme="@style/Theme.AppCompat.NoActionBar"
    android:configChanges="keyboardHidden|orientation|screenSize"
    android:screenOrientation="portrait"
    >
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />

        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

NoActionBar表示没有工具栏，不然闪频显示个工具栏会很难看。

keyboardHidden表示隐藏键盘。

portrait表示方向是竖直的。

intent-filter 设为MAIN和LAUNCHER表示首先启动SplashActivity。

### 编写代码

前面资源和配置都差不多了，最后就是写代码。具体代码如下：

```
public class SplashActivity extends AppCompatActivity {
    private ImageView mSplashImage;
    private TextView mVersionName;
    private TextView mCopyRight;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_splash);

        mSplashImage = (ImageView)findViewById(R.id.splash_image);
        mVersionName = (TextView)findViewById(R.id.splash_version_name);
        mCopyRight = (TextView)findViewById(R.id.splash_copyright);

        mCopyRight.setText(getResources().getString(R.string.copyright));
        mVersionName.setText(getVersionName(getApplication()));

        Animation animation = AnimationUtils.loadAnimation(getApplicationContext(),R.anim.splash);

        animation.setAnimationListener(new Animation.AnimationListener() {
            @Override
            public void onAnimationStart(Animation animation) {

            }

            @Override
            public void onAnimationEnd(Animation animation) {
                Intent intent = new Intent(SplashActivity.this, MainActivity.class);
                startActivity(intent);
                finish();
            }

            @Override
            public void onAnimationRepeat(Animation animation) {

            }
        });

        mSplashImage.startAnimation(animation);
    }

    private String getVersionName(Context context) {
        String versionName = null;
        try {
            versionName = context.getPackageManager().getPackageInfo(context.getPackageName(),0).versionName;
        }catch (PackageManager.NameNotFoundException e){
            e.printStackTrace();
        }
        return String.format(context.getResources().getString(R.string.app_version),versionName);
    }
}
```