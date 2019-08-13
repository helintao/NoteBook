# save()、save(int saveFlags)
**保存当前Canvas的状态信息。**

作用：当调用完save方法之后，例如平移、缩放、旋转、倾斜、拼接或者裁剪这些操作，都是和原来的一样，而当调用完restore方法之后，在save()到restore()之间的所有操作都会被遗忘，并且会恢复调用save()之前的所有设置。

- 这两个方法最终都调用native_save方法，而无参方法save()默认是保存Matrix和Clip这两个信息。

- 如果允许，那么尽量使用无参的save()方法，而不是使用有参的save(int saveFlags)方法传入别的Flag。

- 该方法的返回值，对应的是在堆栈中的index，之后可以在restoreToCount(int saveCount)中传入它来讲在它之上的所有保存图层都出栈。

# restore()、restoreToCount(int count)、getSaveCount()
这三个方法用来**恢复图层信息**，也就是将之前保存到栈中的元素出栈。

- restore()方法用来恢复，最近一次调用save()之前的Matrix/Clip信息，如果调用restore()的次数大于save()的一次，也就是栈中已经没有元素，那么会抛出异常。

- getSaveCount()：返回的是当前栈中元素的数量。

- restoreToCount(int count)会将saveCount()之上对应的所有元素都出栈，如果count < 1，那么会抛出异常。

- 它们最终都是调用了native的方法。

# saveLayer、saveLayerAlpha
除了save()方法之外，canvas还提供了saveLayer方法

saveLayer和saveLayerAlpha，它们最终都是调用了两个native方法：

- 对于saveLayer，如果不传入saveFlag，那么默认是采用ALL_SAVE_FLAG。

- 对于saveLayerAlpha，如果不传入saveFlag，那么默认是采用ALL_SAVE_FLAG，如果不传入alpha，那么最终调用的alpha = 0。

## 和save()方法的区别

关于save()和saveLayer()的区别，源码当中是这么解释的，也就是说它会新建一个不在屏幕之内的bitmap，之后的所有绘制都是在这个bitmap上操作的。并且这个方法是相当消耗资源的，因为它会导致内容的二次渲染，特别是当canvas的边界很大或者使用了CLIP_TO_LAYER这个标志时，更推荐使用LAYER_TYPE_HARDWARE，也就是硬件渲染来进行Xfermode或者ColorFilter的操作，它会更加高效。

总结：调用saveLayer之后，创建了一个透明的图层，之后在调用restore()方法之前，我们都是在这个图层上面进行操作，而save方法则是直接在原先的图层上面操作，那么对于某些操作，我们不希望原来图层的状态影响到它，那么我们应该使用saveLayer。