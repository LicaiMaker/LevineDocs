> BaseExpandableRecyclerViewAdapter是继承自Recyclerview.Adapter,Recyclerview及其子类都可以使用该适配器，需要集成[LevineUtils](/zh-cn/Android/LevineUtils/README)，其实现了可扩展列表的效果

### 1.BaseExpandableRecyclerViewAdapter简介

```java
public abstract class BaseExpandableRecyclerViewAdapter<T, V> extends RecyclerView.Adapter<BaseViewHolder> {
     public BaseExpandableRecyclerViewAdapter(Context mContext, List<BaseExpandBean<T, V>> parentList) {
        this.mDataList = parentList;
        this.mContext=mContext;
        mLayoutInflater = LayoutInflater.from(this.mContext);
        initmParentItemsStatus();
        setOnItemClickListener();
    }
}
```

其中向外暴露出四个方法，用于调用者使用：

```java
 			@Override
            public int getParentLayId() {
                //获取父item的layout id
                return R.layout.parent_item;
            }

            @Override
            public int getChildLayId() {
                //获取子item的layout id
                return R.layout.child_item;
            }

            @Override
            public void convertParentView(BaseViewHolder holder, String itemData) {
         		//为父布局渲染数据
            }

            @Override
            public void convertChildView(BaseViewHolder holder, String itemData) {
               //为子布局渲染数据
            }

```

还有个打开模式：

```java
setOnlyOneItemExpanded(boolean onlyOneItemExpanded);
//为true是一次只打开一个父item，为false则可以同时打开多个父item
```



另外，还有个监听器：

```java
setmOnExpandListener(OnExpandItemListener mOnExpandListener);
```

```java
public interface OnExpandItemListener {
        void onExpanded(View view, int pos);//父布局展开

        void onCollapsed(View view, int pos);//父布局收缩

        void onChildClick(View view, int parentItemIndex, int childItemIndex);//子item的展开
    }
```

### 2.BaseExpandableRecyclerViewAdapter的使用

```java
BaseExpandableRecyclerViewAdapter adapter=new BaseExpandableRecyclerViewAdapter<String,String>(this.getContext(),list){
            @Override
            public int getParentLayId() {
                return R.layout.parent_item;
            }

            @Override
            public int getChildLayId() {
                return R.layout.child_item;
            }

            @Override
            public void convertParentView(BaseViewHolder holder, String itemData) {
                holder.setText(R.id.parentTV,itemData);
//                holder.setImageFromRes(R.id.parentIV,R.drawable.icon_list_unexpand);
            }

            @Override
            public void convertChildView(BaseViewHolder holder, String itemData) {
                holder.setText(R.id.childTV,itemData);
            }
        };

        adapter.setmOnExpandListener(new BaseExpandableRecyclerViewAdapter.OnExpandItemListener() {
            @Override
            public void onExpanded(View view, int pos) {
                LogUtils.e("pos:"+pos+" 展开了");
                ImageView view1=view.findViewById(R.id.parentIV);
                view1.setImageResource(R.drawable.icon_list_unexpand);
            }

            @Override
            public void onCollapsed(View view, int pos) {
                LogUtils.e("pos:"+pos+" 收缩了");
                ImageView view1=view.findViewById(R.id.parentIV);
                view1.setImageResource(R.drawable.icon_list_expand);
            }

            @Override
            public void onChildClick(View view, int parentItemIndex, int childItemIndex) {
                LogUtils.e("pos:（p，c）:"+"("+parentItemIndex+","+childItemIndex+")被点击了");
            }
        });
//        adapter.setOnlyOneItemExpanded(true);
        mFragment2RLV.setAdapter(adapter);
```

效果如下：

![image-20191117182312450](../../../_media/imgs/image-20191117182312450.png)

