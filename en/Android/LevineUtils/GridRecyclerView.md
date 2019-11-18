>  GridRecyclerView是继承自Recyclerview,需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

### 1.GridRecyclerView简介

```java
public class GridRecyclerView extends RecyclerView {
    public GridRecyclerView(@NonNull Context context) {
        this(context,null);
    }

    public GridRecyclerView(@NonNull Context context, @Nullable AttributeSet attrs) {
        this(context,attrs,0);
    }

    public GridRecyclerView(@NonNull Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init(context,attrs);
    }
    
    其中init方法：
    private  GridLayoutManager gridLayoutManager ;
    private void init(Context context,AttributeSet attrs) {
        gridLayoutManager = new GridLayoutManager(this.getContext(),spanCount);
        TypedArray a = context.obtainStyledAttributes(attrs,
                R.styleable.GridRecyclerViewPager);

        spanCount = a.getInteger(
                R.styleable.GridRecyclerViewPager_spanCount, spanCount);// 默认为每行3个
        a.recycle();


        gridLayoutManager.setSpanCount(spanCount);
        this.setLayoutManager(gridLayoutManager);
    }
```

其中自定义了一个属性*spaceCount*，用于指定GridLayout中每一行中的item数量。

### 2.GridRecyclerView的使用

GridRecyclerView其实也是*RecyclerView*的一种，同样可以使用我们的[BaseRecyclerViewAdaper](/zh-cn/Android/LevineUtils/BaseRecyclerViewAdapter万能适配器).

#### 在xml文件中使用GridRecyclerView

```xml
 <com.levine.utils.app.view.GridRecyclerView
        android:layout_alignParentStart="true"
        android:id="@+id/mGridView"
        app:spanCount="3"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
```



`spaceCount`可以指定每行的item数量.  

#### 设置适配器



```java
  BaseRecyclerViewAdapter adapter=new BaseRecyclerViewAdapter<GridBean>(list,getContext(),R.layout
        .fragment3_grid_item) {
            @Override
            public void convert(BaseViewHolder holder, GridBean itemData) {
                holder.setImageFromRes(R.id.mFragmentGridItemViewAvatarIV,itemData.getAvatar());
                holder.setText(R.id.mFragmentGridItemViewNameTV,itemData.getName());
            }
        };
  mGridView.setAdapter(adapter);
```

#### 设置ItemDecoration

```java
ItemDecoration itemDecoration=new DividerItemDecoration(RecyclerView.VERTICAL);
recyclerListView.addItemDecoration(itemDecoration);
```

效果如下：

![gridview](../../../_media/imgs/gridview.png)

当然可以使用自定义其他样式的*[ItemDecoration](/zh-cn/Android/LevineUtils/ItemDecoration)*