>  使用ItemDecoration需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)，共```LevineItemDecoration```,```PaddingItemDecoration```,```TitleItemDecoration```3种

### 1.LevineItemDecoration

#### LevineItemDecoration的简介

```java
 public LevineItemDecoration(Context context, int orientation) {
        final TypedArray a = context.obtainStyledAttributes(ATTRS);
        mDivider = a.getDrawable(0);
        if (mDivider == null) {
            Log.w(TAG, "@android:attr/listDivider was not set in the theme used for this "
                    + "DividerItemDecoration. Please set that attribute all call setDrawable()");
        }
        a.recycle();
        setOrientation(orientation);
    }
```

*LevineItemDecoration*是在*DividerItemDecoration*上修改了添加了可以修改左右两端的Padding值的方法.

#### LevineItemDecoration的使用

```java
LevineItemDecoration levineItemDecoration =
                        new LevineItemDecoration(mContext, LinearLayout.VERTICAL)
                                .setmPaddingStart(30)
                                .setmPaddingEnd(30)
                                .setDrawable(ContextCompat.getDrawable(mContext, R.drawable.item_divider))
                               
recyclerListView.addItemDecoration(levineItemDecoration);
```

设置左右两端的Padding值，并且样式是一个drawable类型的资源，效果如下：

![image-20191117164249671](C:\Users\summer\AppData\Roaming\Typora\typora-user-images\image-20191117164249671.png)



###  2.PaddingItemDecoration

#### PaddingItemDecoration的简介

```PaddingItemDecoration```其实是LevineItemDecoration的简易版，只是少了一个setDrawable方法.



#### PaddingItemDecoration的使用

```java
 PaddingItemDecoration variableItemDecoration=
     new PaddingItemDecoration(LinearLayout.VERTICAL)
     .setmDividerWidth(3)
     .setmPaddingStart(30)
     .setmPaddingEnd(30);               
 recyclerListView.addItemDecoration(variableItemDecoration);
```

设置左右两端的Padding值：

![image-20191117165050546](C:\Users\summer\AppData\Roaming\Typora\typora-user-images\image-20191117165050546.png)



> 另外，```PaddingItemDecoration```和```LevineItemDecoration```都有一个```setDataList(List<?extends BaseBean> dataList)```方法,其内部需要一个数据类型为*BaseBean*的List，用于remove某些位置的*itemDecoration*.调用示例如下：
>
> ```java
> public class CustomerBean extends BaseBean {
> 
>     String name;
> 
>     public CustomerBean(String name) {
>         this.name = name;
>     }
> 
>     public String getName() {
>         return name;
>     }
> 
>     public void setName(String name) {
>         this.name = name;
>     }
> 
>     @Override
>     public String getBeanType() {
>         //重写BaseBean的getBeanType方法
>         return name;
>     }
> }
> 
> 
> List<BaseBean> list=new ArrayList();
> List<BaseBean> list = new ArrayList<>();
>         BaseBean b1 = new CustomerBean("b");
>         BaseBean b2 = new CustomerBean("c");
>         BaseBean b3 = new CustomerBean("c");
>         BaseBean b4 = new CustomerBean("d");
>         BaseBean b5 = new CustomerBean("d");
>        .....
>         list.add(b1);
>         list.add(b2);
>         list.add(b3);
>         list.add(b4);
>         list.add(b5);
>       .....
> 
> LevineItemDecoration levineItemDecoration =
>                         new LevineItemDecoration(mContext, LinearLayout.VERTICAL)
>                                 .setmPaddingStart(30)
>                                 .setmPaddingEnd(30)
>                                 .setDrawable(ContextCompat.getDrawable(mContext, R.drawable.item_divider))
>                                 .setDataList(list);
> ```
>
> 在```LevineItemDecoration```和```PaddingItemDecoration```中根据传入的list中的*BaseBean*的*getBeanType*方法的返回值，判断当前item和下一个item是否是同一个分组(类别)，如果当前item和下一个item不是同一个类别则remove掉ItemDecoration.
>
> 效果如下：
>
> ![image-20191117171503453](C:\Users\summer\AppData\Roaming\Typora\typora-user-images\image-20191117171503453.png)



### 3.TitleItemDecoration

#### TitleItemDecoration简介

TitleItemDecoration适用于设置某个item的title而定制的:

```java

    public TitleItemDecoration(List<? extends BaseBean> datas, int height) {
        this.mDatas = datas;
        this.mTitleHeight = height;
        mPaint = new Paint();
        mPaint.setTextSize(mTitleTextSize);
        mBounds = new Rect();
    }
```

其内部有以下方法：

- ***setmTitleBgColor*** 设置Title的背景颜色
- ***setmTitleFontColor***  设置Title的字体颜色
- ***setmTitleTextSize*** 设置Title的字体大小
- ***setmTitleMarginLeft*** 设置Title的marginleft
- ***setSelectedAnimation*** 设置Title的更换动画，含有```TRANSLATION```，```OVERLAY```两种选择
- ***setHeaderViewLayoutId*** 可以设置一个复杂布局的layout id.

#### TitleItemDecoration的使用

```java
TitleItemDecoration titleItemDecoration = new TitleItemDecoration(list, 80);//高度为80
//titleItemDecoration.setHeaderViewLayoutId(R.layout.complex_header_item_decoration);//设置复杂头部
recyclerListView.addItemDecoration(titleItemDecoration);
```

同样的list和上面的list是一样的，也是根据getBeanType返回值判断是否是同一个类型，若是，则不添加```TitleItemDecoration```；若否，则不添加.

效果如下：

![image-20191117173447722](C:\Users\summer\AppData\Roaming\Typora\typora-user-images\image-20191117173447722.png)

红框所勾选的区域便是TitleItemDecoration.

