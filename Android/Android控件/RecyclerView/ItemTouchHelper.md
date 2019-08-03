# ItemTouchHelper
ItemTouchHelper在RecyclerView的整个体系中，负责监听Item的手势操作，我们通过给它设置一个继承于ItemTouchHelper.Callback的子类，在其中处理Item的UI变化，就可以完成侧滑删除、拖动排序等操作。

# API分析
对于Item的手势操作分为两种：侧滑和拖动，如果需要支持这两种，那么需要给ItemTouchHelper传入一个ItemTouchHelper.Callback的子类，并把ItemTouchHelper和RecyclerView关联起来，下面，我们先来介绍一下ItemTouchHelper.Callback个回调方法的含义：

## 控制相关
- **public boolean isLongPressDragEnabled()**

    该方法返回true时，表示支持长按拖动，即长按ItemView后才可以拖动，我们遇到的场景一般也是这样的。默认是返回true。如果返回false，那么可以通过startDrag(ViewHolder)方法来触发某个特定Item的拖动的机制。

- **public boolean isItemViewSwipeEnabled()**

    该方法返回true时，表示如果用户触摸并左右滑动了View，那么可以执行滑动删除操作，即可以调用到onSwiped（）方法。默认是返回true。

- **public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder)**

    该方法用于返回可以滑动的方向，比如说允许从右到左侧滑，允许上下拖动等。我们一般使用makeMovementFlags（int，int）或makeFlag（int，int）来构造我们的返回值。
    例如：要使RecyclerView的项目可以上下拖动，同时允许从右到左侧滑，但不许允许从左到右的侧滑，我们可以这样写：

    ```java
    @Override
    public int getMovementFlags(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder) {
        int dragFlags = ItemTouchHelper.UP | ItemTouchHelper.DOWN; //允许上下的拖动
        int swipeFlags = ItemTouchHelper.LEFT;   //只允许从右向左侧滑
        return makeMovementFlags(dragFlags,swipeFlags);
    }
    ```

## 结果相关
- **public abstract boolean onMove(RecyclerView recyclerView, ViewHolder viewHolder, ViewHolder target)**

    当用户拖动一个项目进行上下移动从旧的位置到新的位置的时候会调用该方法，在该方法内，我们可以调用适配器的notifyItemMoved方法来交换两个ViewHolder的位置，最后返回真，表示被拖动的ViewHolder已经移动到了目的位置。所以，如果要实现拖动交换位置，可以重写该方法（前提是支持上下拖动）：

    ```java
    @Override
    public boolean onMove(RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, RecyclerView.ViewHolder target) {
        //onItemMove是自定义的接口方法
        mAdapter.onItemMove(viewHolder.getAdapterPosition(),target.getAdapterPosition());  
        return true;
    }
    ```

- **public abstract void onSwiped(ViewHolder viewHolder, int direction)**

    当用户左右滑动项目达到删除条件时，会调用该方法，一般手指触摸滑动的距离达到RecyclerView宽度的一半时，再松开手指，此时该项目会继续向原先滑动方向滑过去并且调用onSwiped方法进行删除，否则会反向滑回原来的位置在该方法内部我们可以这样写：
    ```java
    @Override
    public void onSwiped(RecyclerView.ViewHolder viewHolder, int direction) {
        //onItemDissmiss是自定义的接口方法
        mAdapter.onItemDissmiss(viewHolder.getAdapterPosition());
    }
    ```
    如果在onSwiped方法内我们没有进行任何操作，即不删除已经滑过去的项目，那么就会留下空白的地方，因为实际上该ItemView控件还占据着该位置，只是移出了我们的可视范围内罢了。

## 状态相关
- **public void onSelectedChanged(RecyclerView.ViewHolder viewHolder, int actionState)**

    从静止状态变为拖拽或者滑动的时候会回调该方法，参数actionState表示当前的状态。actionState的取值有ACTION_STATE_IDLE/ACTION_STATE_SWIPE/ACTION_STATE_DRAG。

- **public void clearView(RecyclerView recyclerView, ViewHolder viewHolder)**

    当用户操作完毕某个项并且其动画也结束后会调用该方法，一般我们在该方法内恢复ItemView的初始状态，防止由于复用而产生的显示错乱问题。

- **public void onChildDraw(Canvas c, RecyclerView recyclerView, RecyclerView.ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive)**
    我们可以在这个方法内实现我们自定义的交互规则或者自定义的动画效果。

    - Canvas：绘制RecyclerView的Canvas
    - dx, dy：偏移。
    - actionState：拖拽还是侧滑，对应ACTION_STATE_DRAG和ACTION_STATE_SWIPE。
    - isCurrentlyActive为true表示这个Item正在被用户所控制，false则表示它仅仅是在回到原本状态的动画过程当中。

- **public void onChildDrawOver(Canvas c, RecyclerView recyclerView, ViewHolder viewHolder, float dX, float dY, int actionState, boolean isCurrentlyActive)**

    和上面类似，只不过它是绘制在Item之上。
