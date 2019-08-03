# 只有一种ViewType下的复用情况分析
下面，我们来分析一下两个关键方法的调用时机：
- onCreateViewHolder
- onBindViewHolder

## 初始进入
刚开始进入界面的时候,RecyclerView只实例化了屏幕内可见的ViewHolder，并且onBindViewHolder是在对应的onCreateViewHolder调用完后立即调用的。

## 开始滑动
当我们手指触摸到屏幕，并开始向下滑动，会预加载一个屏幕以外的Item。回调onCreateViewHolder()。当Item开始可见的时候回调onBindViewHolder()方法。紧接着会回调下一个item的onCreateViewHolder()进行预加载（如果有下一个item的话）。

## 继续滑动
当我们继续往下滑动，会预加载一个屏幕以外的Item。回调onCreateViewHolder()。当Item开始可见的时候回调onBindViewHolder()方法。紧接着会回调下一个item的onCreateViewHolder()进行预加载（如果有下一个item的话）。

## 复用
在复用时，只回调了onBindViewHolder方法

# 多种类型的布局
## 基本使用
当我们需要在列表当中展示不同类型的Item时，我们一般需要重写下面的方法，告诉RecyclerView在对应的position上需要展示什么类型的Item。
```java
public int getItemViewType(int position)
```

RecyclerView在回调onCreateViewHolder的时候，同时也会把viewType传递进来，我们根据viewType来创建不同的布局。

# 多种viewType下的复用情况分析
## 初始进入
此时，屏幕中可见的Item的onCreateViewHolder和onBindViewHolder的回调，只会生成屏幕内可见的ViewHolder

## 开始滑动和继续滑动
这两种情况都和单个viewType时相同，会预加载屏幕以外的一个Item回调onCreateViewHolder()。当Item开始可见的时候回调onBindViewHolder()方法。紧接着会回调下一个item的onCreateViewHolder()进行预加载（如果有下一个item的话）。

## 复用
在复用时，只回调了onBindViewHolder方法

# 数据更新
## 更新方式
当数据源发生变化的时候，我们一般会通过Adatper. notifyDataSetChanged()来进行界面的刷新，RecyclerView.Adapter也提供了相同的方法：
```java
public final void notifyDataSetChanged() 
```

除此之外，它还提供了下面几种方法，让我们进行局部的刷新：
```java
//position的数据变化
notifyItemChanged(int postion)
//在position的下方插入了一条数据
notifyItemInserted(int position)
//移除了position的数据
notifyItemRemoved(int postion)
//从position开始，往下n条数据发生了改变
notifyItemRangeChanged(int postion, int n)
//从position开始，插入了n条数据
notifyItemRangeInserted(int position, int n)
//从position开始，移除了n条数据
notifyItemRangeRemoved(int postion, int n)
```

##  比较
数据的更新分为两种：

- **Item changes：**除了Item所对应的数据被更新外，没有其它的变化，对应notifyXXXChanged()方法。

- **Structural changes：**Items在数据集中被插入、删除或者移动，对应notifyXXXInsert/Removed/Moved方法。

notifyDataSetChanged会把当前所有的Item和结构都视为已经失效的，因此它会让LayoutManager*重新绑定*Items，并对他们*重新布局*，这在我们知道已经需要更新某个Item的时候，其实是不必要的，这时候就可以选择进行局部更新来提高效率。

# 监听ViewHolder的状态
RecyclerView.Adapter中还提供了一些回调，让我们能够监听某个ViewHolder的变化：
```java
    //item超出不可见范围（相反方向上的两个ViewHolder）后调用。
    @Override
    public void onViewRecycled(NormalViewHolder holder) {
        Log.d("NormalAdapter", "onViewRecycled=" + holder);
        super.onViewRecycled(holder);
    }

    //item移出可见范围的时候调用
    @Override
    public void onViewDetachedFromWindow(NormalViewHolder holder) {
        Log.d("NormalAdapter", "onViewDetachedFromWindow=" + holder);
        super.onViewDetachedFromWindow(holder);
    }

    //item可见的时候调用
    @Override
    public void onViewAttachedToWindow(NormalViewHolder holder) {
        Log.d("NormalAdapter", "onViewAttachedToWindow=" + holder);
        super.onViewAttachedToWindow(holder);
    }
```

# 监听RecyclerView和RecyclerView.Adapter的关系
RecyclerView和Adapter是通过setAdapter方法来绑定的，因此在Adapter中也通过了绑定的监听：
```java
public void onAttachedToRecyclerView(RecyclerView recyclerView) {}
public void onDetachedFromRecyclerView(RecyclerView recyclerView) {}
```
