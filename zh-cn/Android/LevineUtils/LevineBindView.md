> 需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

### 1.LevineBindView和LevineOnClick简介

这是两个注解用于绑定view和设置onclick方法.

定义如下：

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LevineOnClick {

}

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LevineBindView {

    /**
     * int name of widget,e.g,R.id.mPhoneBtn,used in both Android Application and Android Library.
     * @return
     */
    int value() default -1;

    /**
     * string name of  widget,e.g,R.id.mPhoneBtn,"mPhoneBtn" is needed .Usually，it is used in a Android Library.
     * @return
     */
    String strValue() default "";
}
```



> 注：在LevineBindView 创建value和strValue两个方法是为了解决在AndroidLibrary中不能使用R.id.xx的问题：
>
> ```java
> @LevineBindView(R.id.mLoginBtn)
> private Button mLogin;
> ```
>
> 上面的代码在library中是不能通过编译的，所以，作者开辟了另一条道路，则使用字符串来初始化，其内部原理是：通过getResources().getIdentity("mLoginBtn","id",getPackageName()),所以就能解决这个问题

>  注：其中LevineAnnotationUtils是用于实现注解功能的关键，需要调用其中的bind方法来初始化

### 2.LevineBindView和LevineOnClick的使用

#### 在主module中初始化

- 在Activity中

  ```java
  public class MainActivity extends AppCompatActivity{
      @LevineOnClick
      @LevineBindView(R.id.textView)
      private TextView textView;
  
      List<BaseExpandBean<HashMap, SelectableBean>> list;
      private List<SelectableBean> selectableBeanList;
  
      @Override
      protected void onCreate(Bundle savedInstanceState) {
          super.onCreate(savedInstanceState);
          setContentView(R.layout.activity_select_meeting_member);
          LevineAnnotationUtils.bind(this);
          init();
          initDatas();
      }
  }
  ```

  

- 在Fragment中

  ```java
  public class Fragment extends Fragment implements View.OnClickListener{
      @LevineBindView(R.id.mHahaTV)
      @LevineOnClick
      private TextView mHahaTV;
  
      @Nullable
      @Override
      public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
          View view=inflater.inflate(R.layout.fragment2,container,false);
          LevineAnnotationUtils.bind(this,view);//初始化
          mHahaTV.setText("嘿嘿");
          init();
          return view;
  ```

  

#### LevineOnClick

使用了LevineOnClick方法后，需要Activity或者Fragment实现View.OnClickListener接口：

```java

    
    public class MainActivity extends AppCompatActivity implements View.OnClickListener{
        
    @LevineOnClick
    @LevineBindView(R.id.textView)
    private TextView textView;

    List<BaseExpandBean<HashMap, SelectableBean>> list;
    private List<SelectableBean> selectableBeanList;
        
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_select_meeting_member);
        LevineAnnotationUtils.bind(this);
        init();
        initDatas();
    }
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.textView:
              //todo:点击了TextView
                textView.setText("哈哈哈");
                break;
        }
    }
}
```



#### 在Library的module中初始化

```
   public class LibraryActivity extends AppCompatActivity implements View.OnClickListener{
        
    @LevineOnClick
    @LevineBindView(strValue="textView") // 这里需要使用strValue
    private TextView textView;
        
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.XXX);
        LevineAnnotationUtils.bind(this);
 
    }
    @Override
    public void onClick(View v) {
      // 这里不能使用switch，因为不能使用R.id.textView(在Library中非常量)
            if(v.getId()==R.id.textView){
            	// ...
            }
    }
}
```



### 拓展：至于为什么不能再Library中使用R.id.xxx?

这是因为在android中，每个module极其依赖的library/aar文件，都会生成一个R文件，R文件生成顺序：**从底层到上层，逐层生成**，最后编译成一个完整的R文件，每个资源都能在R文件中找到对应的编号，如：

```
 public static final class id {
        private id() {}

        public static final int accessibility_action_clickable_span = 0x7f080006;
        public static final int accessibility_custom_action_0 = 0x7f080007;
        public static final int accessibility_custom_action_1 = 0x7f080008;
        public static final int accessibility_custom_action_10 = 0x7f080009;
        public static final int accessibility_custom_action_11 = 0x7f08000a;
        public static final int accessibility_custom_action_12 = 0x7f08000b;
        public static final int accessibility_custom_action_13 = 0x7f08000c;
        .....
```

而且每个资源前面都使用static final来修饰，以此来表示其为常量。比较早的aapt的版本生成的非主模块的资源id确实都是final修饰的，这样会带来一个问题，这些资源id全部内联到代码中，一旦新增或者删除，修改了资源，资源id就会有变化，所有的代码都需要重新编译，造成严重的编译耗时。所以android新的解决方式是：

> 主模块final常量方式内联，非主模块引用方式，这样等按照从下到上编译到App模块的时候，所有的资源id都已经确定了，底层模块的资源只需要通过引用就能拿到自己对应的id，而修改(新增，删除，修改)了资源之后，也只需要重新生成R文件就好了。编译耗时大大减少