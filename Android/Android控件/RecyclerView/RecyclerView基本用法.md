# 导入远程依赖包
```xml
implementation 'androidx.recyclerview:recyclerview:1.0.0'
```

# 继承于RecyclerView.Adapter来实现自定义的Adapter
当我们实现自己的Adapter时，至少要做四个工作：

- **第一：继承于RecyclerView.ViewHolder，编写自己的ViewHolder**  
这个子类用来描述RecyclerView中每个Item的布局以及和它关联的数据，它同时也是RecyclerView.Adapter<VH>中需要指定的VH类型。
在构造方法中，除了需要调用super(View view)方法来传入Item的跟布局来给基类中itemView变量赋值，还应当提前执行findViewById来获得其中的子View以便我们之后对它们进行更新。

- **第二：实现onCreateViewHolder(ViewGroup parent, int viewType)**  
当RecyclerView需要我们提供类型为viewType的新ViewHolder时，会回调这个方法。
在这里，我们实例化出了Item的根布局，并返回一个和它绑定的ViewHolder。

- **第三：实现onBindViewHolder(VH viewHolder, int position)** 
当RecyclerView需要展示对应position位置的数据时会回调这个方法。
通过viewHolder中持有的对应position上的View，我们可以更新视图。

- **第四：实现getItemCount()**  
返回Item的总数。

# 在Activity中，我们给Adapter传递数据，使用方法和ListView基本相同，只是多了一句在设置LayoutManager的操作