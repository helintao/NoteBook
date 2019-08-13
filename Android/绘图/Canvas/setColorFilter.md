# ColorFilter的分类
当我们使用ColorFilter的时候，不一样直接使用它，而是使用它的子类：

- ColorMatrixColorFilter

- LightingColorFilter

- PoterDuffColorFilter

## ColorMatrixColorFilter
ColorMatrixColorFilter的通过ColorMatrix构造，而ColorMatrix则由一个长度为20的float数组构造，传入该数组后把该数组先从左到右，再从上到下排列，形成一个4*5的矩阵。
```
[ a, b, c, d, e,
  f, g, h, i, j,
  k, l, m, n, o,
  p, q, r, s, t ]
```
之后，再用矩阵和目标的RGBA进行计算，最后得到新的RGBA，它的计算方法为：
```
R’ = a*R + b*G + c*B + d*A + e;

G’ = f*R + g*G + h*B + i*A + j;

B’ = k*R + l*G + m*B + n*A + o;

A’ = p*R + q*G + r*B + s*A + t;
```

注意，新的RGBA会被限制在0-255的范围内。在实际使用的时候，我们先通过一个ColorMatrix辅助类来确定需要相乘的这个颜色矩阵，之后再把它作为ColorMatrixColorFilter构造函数的参数来构造它。

我们一般有两种用法：改变画笔的颜色，或者改变整个Bitmap的像素点的颜色。

## LightingColorFilter

用来模拟光照的效果，它定义了两个参数来和原color相乘，第一个colorMultiply原来相乘，而第二个参数colorAdd用来相加，并且会忽略其中的Aplha参数，这个32位的表示0xAARRGGBB，计算的公式为：
```
R' = R * colorMultiply.R + colorAdd.R
G' = G * colorMultiply.G + colorAdd.G
B' = B * colorMultiply.B + colorAdd.B
```

## PorterDuffColorFilter

混合模式，其构造函数为PorterDuffColorFilter(int dstColor, PorterDuff.Mode mode)，其中color为16进制的终点颜色，mode为混合策略，它会根据起点颜色Sc，起点透明度Sa，终点颜色Dc和终点透明度Da最终计算得出要显示的RGBA。


