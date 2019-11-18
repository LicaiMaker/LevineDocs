> BaseRecyclerViewAdapter是继承自Recyclerview.Adapter,Recyclerview及其子类都可以使用该适配器，需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

### 1.BaseRecyclerViewAdapter简介

BaseRecyclerViewAdapter的定义

```java
public abstract class BaseRecyclerViewAdapter<T> extends RecyclerView.Adapter<BaseViewHolder> {
    
    protected List<T> tList;
    protected Context mContext;
    protected LayoutInflater mLayoutInflater;
    protected int layId;
    
     public BaseRecyclerViewAdapter(List<T> tList, Context mContext, int layId) {
        this.tList = tList;
        this.mContext = mContext;
        this.layId = layId;
        mLayoutInflater = LayoutInflater.from(mContext);
    }
}
```

为什么称为万能的呢，因为它将item的数据的绑定交给了调用者，在BaseRecyclerViewAdapter中添加了一个抽象方法```public abstract void convert(BaseViewHolder holder, T itemData);```并且在onBindViewHolder中调用该方法，

在调用者中实现数据绑定([调用示例](#adapter)).

*<T>*表示每个item的数据类型，*layId*表示item的布局文件id，里面的BaseViewHolder继承自Recyclerview.ViewHolder.这个BaseViewHolder内置了常用的数据绑定方法，如：

```java
 
public class BaseViewHolder extends RecyclerView.ViewHolder {
/**
     * 通过id得到view
     *
     * @param viewId
     * @param <V>
     * @return
     */
    public <V extends View> V getViewAtId(int viewId) {}  


/**
     * 通过id得到view
     *
     * @param viewId
     * @param <V>
     * @return
     */
    public <V extends View> V getViewAtId(int viewId) {}

 /**
     * 设置文字
     *
     * @param viewId
     * @param text
     * @return
     */
    public BaseViewHolder setText(int viewId, CharSequence text) {}

 /**
     * 通过资源来设置图片资源
     *
     * @param viewId
     * @param resId
     * @return
     */
    public BaseViewHolder setImageFromRes(int viewId, int resId) {}

/**
     * 设置控件透明度
     *
     * @param viewId
     * @param visibility
     * @return
     */
    public BaseViewHolder setViewVisibility(int viewId, int visibility) {}

/**
     * 设置textview的字体颜色
     * @param viewId TextView的id
     * @param color 颜色值
     * @return
     */
    public BaseViewHolder setTextColor(int viewId,int color){}
}
```

### 2.BaseRecyclerViewAdapter的调用

> 注：需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)

####  <span id="adapter">创建BaseRecyclerViewAdapter子类</span>

```java
public class Fragment1ListviewAdapter extends BaseRecyclerViewAdapter<CustomerBean> {
    public Fragment1ListviewAdapter(List list, Context mContext, int layId) {
        super(list, mContext, layId);
    }

    @Override
    public void convert(BaseViewHolder holder, CustomerBean bean) {
        //绑定数据
        String text= bean.getName();
        holder.setText(R.id.mListViewTV,text);
    }
}
```

BaseRecyclerViewAdapter将数据绑定交给了调用者(convert方法)，调用者也只需要考虑数据绑定即可.

#### Recyclerview设置适配器

```java
 Fragment1RecyclerViewAdapter adapter = 
     new Fragment1RecyclerViewAdapter(list, this.getActivity(),R.layout.horizontal_viewpager_page_view);
 recyclerView.setAdapter(adapter);
```

### 3.多布局支持

另外，经常会使用不同的布局的item，如：

![多布局支持](../../../_media/imgs/webp)

在BaseRecyclerViewAdapter中也提供了多布局接口setMultiTypeSupport(BaseRecyclerViewAdapter.MultiTypeSupport<T> interface).

#### 设置多布局接口

假设数据类型如下：

```java
	  List pageDatas = new ArrayList<HashMap<String, Object>>();
        pageDatas.add(new HashMap<String, Object>() {{
            put("mFragment1TV", "info1");
            put("mFragment1IV", R.mipmap.ic_launcher);
            put("type", "normal");
        }});
        pageDatas.add(new HashMap<String, Object>() {{
            put("mFragment1TV", "info2");
            put("mFragment1IV", R.mipmap.ic_launcher);
            put("type", "ad");
        }});
```

其中农数据中的type就是用来区分不同的item.

```java
		//多布局支持
        adapter.setMultiTypeSupport(new BaseRecyclerViewAdapter.MultiTypeSupport<HashMap<String, Object>>() {
            @Override
            public int getLayoutId(HashMap<String, Object> item) {
                String type = (String) item.get("type");
                if (type.equals("ad")) {
                    return R.layout.horizontal_viewpager_page_view_ad;
                } else if(type.equals("normal")){
                    return R.layout.horizontal_viewpager_page_view;
                }
            }
        });
```

通过setMultiTypeSupport接口重写getLayoutId方法，在方法里根据需要返回不同的布局id即可.

#### 为多布局item绑定数据

当设置了多布局后，还要根据不同的布局绑定数据。

```java
public class Fragment1RecyclerViewAdapter extends BaseRecyclerViewAdapter<HashMap<String, Object>> {
    public Fragment1RecyclerViewAdapter(List list, Context mContext, int layId) {
        super(list, mContext, layId);
    }

    @Override
    public void convert(BaseViewHolder holder, HashMap<String, Object> itemData) {
        //绑定数据，因为在adapter中支持了多布局，所以要判断一下其数据中的类型，来判断使用的是哪一个布局文件
        String type = (String) itemData.get("type");
        switch (type) {
            case "ad":
                holder.setText(R.id.mFragment1TV, (CharSequence) itemData.get("mFragment1TV"));
                int imgId = (int) itemData.get("mFragment1IV");
                holder.setImageFromRes(R.id.mFragment1IV, imgId);
                break;
            case "normal":
               	holder.setText(R.id.mFragment1TV, (CharSequence) itemData.get("mFragment1TV"));
                break;
        }
    }
```



### 4.设置监听器

设置item的监听器：

```java
 adapter.setOnItemClickListener(new BaseRecyclerViewAdapter.OnBaseItemClickListener() {
            @Override
            public void onItemClick(int position) {
               
            }
        });
```

