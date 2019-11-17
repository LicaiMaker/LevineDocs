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
    int value();
}

```

>  注：其中LevineAnnotationUtils是用于实现注解功能的关键，需要调用其中的bind方法来初始化

### 2.LevineBindView和LevineOnClick的使用

#### 初始化

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

