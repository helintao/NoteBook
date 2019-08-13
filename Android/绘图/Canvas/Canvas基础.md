# Canvas三个概念
Canvas的主要有以下三个概念：画板、画布和图层，画板包含了多个画布，每个画布中间又可能会包含多个图层，同一层级的元素会叠加在一块，在上一层中间进行展示。

## 画板
画板用来承载画布。

## 画布
每一个画布就是一个bitmap，所有的图像都是画在bitmap上的。

画布有两种：

- View的原始画布，也就是onDraw(Canvas canvas)当中传入的canvas实例。

- 人造画布，即通过saveLayer或者new Canvas(Bitmap)这些方法来人为新建一个画布，saveLayer新建一个画布之后，以后所有的draw函数所画的图像都是在这个画布上的，只有在restore、restoreToCount函数以后，才会返回到原始画布上绘制。

## 图层
图层是绘制的最小单元，每次调用canvas.drawXXX函数，都会生成一个专门的透明图层来画这个图形，画完之后，就盖在了画布上。连续调用draw方法，那么就会生成多个透明图层，画完之后依次在当前的画布上显示。

# Canvas变换
Canvas的变换可以分为两类：可逆变换和不可逆变换。

可逆变换包括：平移、旋转、缩放和倾斜四种。

不可逆变换包括：裁剪。

## 可逆变换
- 平移

    void translate(float dx, dy)，向右和向下为正方向，此时移动的是canvas原点的位置。

- 旋转

    rotate(float degrees, float px, float py)，正数表示顺时针旋转，此外还可以指定旋转的中心，默认是(0, 0)

- 缩放

    scale(float sx, float sy, float px, py)，对x和y进行缩放，并且可以指定缩放的中心点。

- 倾斜

    skew(float sx, float sy)，倾斜画布，分别表示倾斜角度的tan值。

## 不可逆变换

    调用clipXXX函数进行裁剪，通过与Rect、Path、Region取交、并、差等集合运算来获得最新的画布状态，这个操作是不可逆的，一旦canvas被裁剪，那么就不能被恢复。

# Canvas绘制

## 绘制基本图形

- 线

    ```java
    drawLine(float startX, float startY, float stopX, float stopY, Paint paint)

    drawLines(float[] pts, Paint paint)

    drawLines(float[] pts, int offset, int count, Paint paint)
    ```

- 点

    ```java
    drawPoint(float x, float y, Paint paint)//x、y为绘制点的坐标

    drawPoints(float[] pts, Paint paint)

    drawPoints(float[] pts, int offset, int count, Paint paint)
    ```

- 矩形

    ```java
    drawRect(float left, float top, float right, float bottom, Paint)

    drawRect(RectF rect, Paint paint)

    drawRect(Rect rect, Paint paint)
    ```

- 圆角矩形

    ```java
    drawRoundRect(RectF rect, float rx, float ry, Paint paint)
    ```

- 圆

    ```java
    drawCircle(float cx, float cy, float radius, Paint paint)
    ```

- 椭圆

    ```java
    drawOval(RectF oval, Paint paint)
    ```

- 扇形

    ```java
    drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)
    ```

## 调用Path绘制

在canvas中，绘制路径时使用drawPath(Path path, Paint paint)方法

- 直线路径

    调用moveTo(float x, float y)可以移动到某个绘制点，lineTo(float x, float y)连到直线的结束点，如果连续画了几条直线，但没有形成闭环，调用close()会将路径首尾连接起来。

- 矩形路径

    addRect(RectF rect, Path.Direction dir)和drawRect(float l, float t, float r, float b, Path.Direction dir)，用来绘制一条封闭的矩形路径，第2个参数决定了该路径的方向。

- 圆角矩形路径

    addRoundRect(RectF rect, float[] radii, Path.Direction dir)和addRoundRect(RectF, float rx, float ry, Path.Direction dir)

- 圆形路径

    addCircle(float x, float y, float radius, Path.Direction dir)

- 椭圆路径

    addOval(RectF oval, Path.Direction dir)

- 弧形路径

    addArc(RectF oval, float startAngle, float sweepAngle)

## 绘制文字
canvas的文字绘制提供以下3种：

- 普通水平绘制

    drawText(String text, float x, float y, Paint paint)

- 指定各个文字位置

    drawPosText(String text, float[] pos, Paint)

- 沿路径绘制

    drawTextOnPath(String text, Path path, float hOffset, float vOffset, Paint paint)，其中hOffset表示与路径起始点的水平偏移量，vOffset表示与路径中心的水平偏移量。