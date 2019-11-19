> 这篇文章专门用于讲述LevineUtils中的`FragmentFactory`，需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

###  1.FragmentFactory简介

* `FragmentFactory`是利用apt技术，即通过注解的方式来管理整个应用中的自定义的Fragment,通过`FragmentFactory`对象的showFragment(String tag)方法来控制fragment的显示和隐藏，从而实现了fragment的切换. 

- `FragmentFactory`同时也对fragment的重影问题给出了解决方案,通过使用<font color=green>saveCurrentFragmentInfo(Bundle bundle)</font>和<font color=green>restoreCurrentFragmentInfo(Bundle bundle)</font>方法保存状态和恢复状态.

###  2.使用FragmentFactory

####  **初始化FragmentFactory**

在activity中的onCreate方法中初始化`FragmentFactory`对象，但是需要注意的是需要继承自FragmentActicity 或者AppCompatActicity，因为在`FragmentFactory`中使用的是getSupportFragmentManager,所以你的activity必须继承自FragmentActivity或

AppCompatActivity:

```java
public class MainActivity extends AppCompatActivity {
    private FragmentFactory mFactory;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_main);
         //使用单例模式创建`FragmentFactory`对象,R.id.mContentnF1是要显示的Fragment的布局容器的id
         mFactory = FragmentFactory.getInstance()
                .init(this, R.id.mContentFl);
        ....
    }
}
```

#### **自定义Fragment**

例如我的主界面是这样的布局，里面含有两个tab，用于两个Fragment的切换：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical">
    <FrameLayout
        android:id="@+id/mContentFl"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="0.9" />
    <com.google.android.material.tabs.TabLayout
        android:layout_gravity="bottom"
        android:id="@+id/mTabLayout"
        android:layout_weight="0.1"
        android:layout_width="match_parent"
        android:layout_height="0dp">
        <com.google.android.material.tabs.TabItem
            android:id="@+id/tab1"
            android:text="page1"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent"/>
        <com.google.android.material.tabs.TabItem
            android:id="@+id/tab2"
            android:text="page2"
            android:layout_weight="1"
            android:layout_width="0dp"
            android:layout_height="match_parent"/>
    </com.google.android.material.tabs.TabLayout>
</LinearLayout>

```

自定义两个Fragment用于tab点击时，切换使用（此时要在Fragment的外部使用`@TargetFragmentTag(String tag)`注解，里面需要接收一个string类型的tag，用于和当前的fragment 一 一对应):

```java
@TargetFragmentTag("fragment1")
public class Fragment1 extends Fragment {
    .....
}

@TargetFragmentTag("fragment2")
public class Fragment2 extends Fragment {
    .....
}
```

> 注： 相当于`Fragment1`的tag就是“ <font color=green face="consolas">fragment1</font>”，`Fragment2`的tag就是“ <font color=green face="consolas">fragment2</font>”，但是鉴于方便管理整个应用中所有Fragment的标签,所以建议创建一个类来存贮所有fragment的标签，比如，建立一个接口类FragmentTag：
>
> ```java
> public interface FragmentTag {
>     String FRAGMENT1="fragment1";
>     String FRAGMENT2="fragment2";
>     
> }
> ```
>
> 那么我们的代码就可以这样写：
>
> ```java
> @TargetFragmentTag(FragmentTag.FRAGMENT1)
> public class Fragment1 extends Fragment {
>     .....
> }
> 
> @TargetFragmentTag(FragmentTag.FRAGMENT2)
> public class Fragment2 extends Fragment {
>     .....
> }
> ```

然后，这样就能统一调度fragment了。

#### **使用FragmentFactory来显示指定的Fragment**

在activity中调用showFragment(String tag)来显示fragment：

```java
public class MainActivity extends AppCompatActivity {
    private FragmentFactory mFactory;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_main);
         //使用单例模式创建`FragmentFactory`对象,R.id.mContentnF1是要显示的Fragment的布局容器的id
         mFactory = FragmentFactory.getInstance()
                .init(this, R.id.mContentFl);
        //默认显示fragment1
         mFactory.showFragment(FragmentTag.FRAGMENT1)
        ....
    }
}
```

另外在tab切换时，同时切换Fragment的显示：

```java
 @Override
    public void onTabSelected(TabLayout.Tab tab) {
        switch (tab.getPosition()) {
            case 0:
                mFactory.showFragment(FragmentTag.FRAGMENT1);
                break;
            case 1:
                mFactory.showFragment(FragmentTag.FRAGMENT2);
                break;
        }
    }
```

> Fragment在切换时，会自动将当前的Fragment隐藏，但并非销毁掉，下次重新切换回来时，会再将这个Fragment显示出来.

#### **跨Activity使用FragmentFactory**

在APP中可能有多个Activity，每个Activity中有多个Fragment，所以经常会在全局中使用`FragmentFactory`(跨Activity)，那也是可以的，因为`FragmentFactory`在初始化时，传入了一个Activity的对象，所以在跨Activity使用时，只需要重新初始化一下即可，各个Activity中使用`FragmentFactory`是相互独立的.

例如：在另一个Activity中有一个有Fragment3和Fragment4,同样地：

```java
@TargetFragmentTag(FragmentTag.FRAGMENT3)
public class Fragment3 extends Fragment {
    .....
}

@TargetFragmentTag(FragmentTag.FRAGMENT4)
public class Fragment4 extends Fragment {
    .....
}

public class AnotherActivity extends AppCompatActivity {
    private FragmentFactory mFactory;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
         setContentView(R.layout.activity_another);
         //使用单例模式创建`FragmentFactory`对象,R.id.mContentnF2是要显示的Fragment的布局容器的id
         mFactory = FragmentFactory.getInstance()
                .init(this, R.id.mContentF2);
        //默认显示fragment3
         mFactory.showFragment(FragmentTag.FRAGMENT3)
        ....
    }
    
     @Override
    public void onTabSelected(TabLayout.Tab tab) {
        switch (tab.getPosition()) {
            case 0:
                mFactory.showFragment(FragmentTag.FRAGMENT3);
                break;
            case 1:
                mFactory.showFragment(FragmentTag.FRAGMENT4);
                break;
        }
    }
}
```



### 3.延伸

> Fragment确实能减少Activity的数量，使用灵活方便，但是也会引发一些问题，例如Fragment重影问题.重影问题主要是由于Activity会保存当前视图层级引起的，在某些事件发生后，Activity会调用onSaveInstance方法进行保存状态，在重新打开activity后，Activity会调用onRestoreInstance方法恢复之前保存的状态，再切换到另外的Fragment，就会出现重影.

关于onSaveInstance和onRestoreInstance方法的时机大致是：当activity可能会被回收时，调用onSaveInstance,当activity被回收后，又重新创建一个实例后，会执行onRestoreInstance方法 

重影问题主要有几种方式引发：

- **横竖屏切换**

- **当任务在后台被回收，重新打开页面时**
- **在开发者模式下设置 不保存活动**

第一个问题解决方案：

在AndroidManifest.xml中的Activity中添加```android:configChanges="keyboardHidden|orientation|screenSize"```,则可以让Activity在横竖屏切换时，不调用这些生命周期的方法即可.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.levine.ucall">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true">
       
        <activity
            android:name=".ui.activity.MainActivity"
            android:configChanges="keyboardHidden|orientation|screenSize">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

另外，其他情况下的解决方案，在`FragmentFactory`中也给出了解决方案：

- 1.首先重写Activity的OnSaveInstance和OnRestoreInstance方法

  ```java
   @SuppressLint("MissingSuperCall")
   @Override
   protected void onSaveInstanceState(@NonNull Bundle outState) {
  		//去掉super.onSaveInstanceState(outState),这是重影问题的根源
          mFactory.saveCurrentFragmentInfo(outState);
      }
   @Override
   protected void onRestoreInstanceState(Bundle savedInstanceState) {
          if (savedInstanceState != null) {
              mFactory.restoreCurrentFragmentInfo(savedInstanceState);
          }
      }
  ```

  

- 2.在onCreate中也调用restoreCurrentFragmentInfo方法

  ```java
    private FragmentFactory mFactory;
      @Override
      protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_another);
           //使用单例模式创建`FragmentFactory`对象,R.id.mContentnF2是要显示的Fragment的布局容器的id
           mFactory = FragmentFactory.getInstance()
                  .init(this, R.id.mContentF2);
          
          if (savedInstanceState != null) {
              mFactory.restoreCurrentFragmentInfo(savedInstanceState);
          } else {
             //默认显示fragment1
           	mFactory.showFragment(FragmentTag.FRAGMENT1)
          }
          ....
      }
  
  ```
  

这样就解决了重影问题。



> 补充：虽然重影问题解决了，但是如果想保存Fragment中某些控件状态该怎么办呢？

在Fragment重写onSaveInstanceState和onViewStateRestored方法即可，比如我的Fragment中有一个两个Tab，需要保存tab的位置：

```java
@Override
    public void onSaveInstanceState(@NonNull Bundle outState) {
        // 保存fragment的状态
        super.onSaveInstanceState(outState);
        outState.putInt("tabPosition",tabLayout.getSelectedTabPosition());
    }

    @Override
    public void onViewStateRestored(@Nullable Bundle savedInstanceState) {
        super.onViewStateRestored(savedInstanceState);
        if(savedInstanceState!=null){
            //恢复fragment的状态
            int position=savedInstanceState.getInt("tabPosition");
            tabLayout.getTabAt(position).select();
        }
    }
```