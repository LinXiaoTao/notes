[Matrix_Basic](http://www.gcssloop.com/customview/Matrix_Basic/)

[矩阵乘法](https://baike.baidu.com/item/%E7%9F%A9%E9%98%B5%E4%B9%98%E6%B3%95)



## 基础知识

### 矩阵乘法

矩阵相乘最重要的方法是一般矩阵乘积。它只有在第一个矩阵的列数和第二个矩阵的行数相同时才有意义。一般单指矩阵乘积时，指的是一般矩阵乘积。

设 A 为 m * p 的矩阵，B 为 p * n 的矩阵，那么 m * n 的矩阵 C 为矩阵 A 和矩阵 B 的乘积，记住 C= AB。其中矩阵 C 中的第 i 行第 j 列元素可以表示为：

![img](https://wikimedia.org/api/rest_v1/media/math/render/svg/a7c766d397f2efd80375b0d2c64c729d38d8036f)

> 1. 当矩阵 A 的列数等于矩阵 B 的行数时，A 与 B 才可以相乘
>
> 2. 矩阵 C 的行数等于矩阵 A 的行数，C 的列数等于 B 的列数
>
> 3. 矩阵 C 的第 m 行第 n 列的元素等于矩阵 A 的第 m 行与矩阵 B 的第 n 列对应元素乘积之和。
>
> 4. 矩阵乘法满足结合律：(AB)C = A(BC)
>
> 5. 矩阵乘法满足分配律：(A+B)C = AC+BC || C(A+B) = CA+CB
>
> 6. 对数乘的结合性：k(AB) = k(A)N = A(kB)
>
> 7. 装置
>
> 8. $$
>    (AB)^T=B^TA^T
>    $$
>

**矩阵乘法一般不满足交换律**



## Android 上的使用

Android 上的矩阵操作，是基于 3*3 的矩阵。



### 基本变换

使用 Matrix 实现的基本变换有四种：平移（translate）、缩放（scale）、旋转（rotate）、错切（skew）。

四种变换都是由矩阵中的哪些元素影响的：

![img](http://ww2.sinaimg.cn/large/005Xtdi2jw1f60gwrhlnyj30c008zdgy.jpg)

![img](http://ww2.sinaimg.cn/large/005Xtdi2jw1f633hvklfnj30c008zdge.jpg)



### set、pre 与 post

对于四种基本变换，平移（translate）、缩放（scale）、旋转（rotate）、错切（skew），每一种都对应三种操作方法，分别为 set、pre、post。而它们的基础是 `concat` ，通过先构造出特殊矩阵，然后再用原始矩阵 `concat` 特殊矩阵，来达到变换的结果。

| 方法   | 作用                       |
| ---- | ------------------------ |
| set  | 设置，会覆盖之前的值               |
| pre  | 前乘，`M' = M * S`（S 为特殊矩阵） |
| post | 后乘，`M' = S * M`（S 为特殊矩阵） |



> 1. 一开始从 Canvas 中获取的 Matrix 并不是初始矩阵，而是经过偏移后的矩阵，且偏移距离就是距离屏幕左上角的位置。
> 2. 因为矩阵相乘一般不满足交换律，所以前乘（pre）和后乘（post）结果可能有偏差。
> 3. 受矩阵乘法影响，后面执行的操作可能会影响到之前的操作。使用时要注意构造顺序。



### 基于特定点的矩阵操作

矩阵的操作都是基于坐标原点，即左上角，并且之前操作的坐标系状态会保留，并且影响后续状态。

为了实现基于特定点的矩阵操作，一般需要以下步骤：

1. 先将坐标系原点移动到指定位置，使用矩阵平移（使用 T 表示）
2. 对坐标系进行特定的矩阵操作（使用 S 表示）
3. 将坐标系平移回原来的位置，使用矩阵平移（使用 -T 表示）

具体公式如下：

> M 为原始矩阵，是一个单位矩阵，M' 为结果矩阵

```
M' = M * T * S * -T
```

代码如下：

``` java
Matrix matrix = new Matrix();
matrix.preTranslate(pivotX,pivotY);
// 各种操作，旋转，缩放，错切等，可以执行多次。
matrix.preTranslate(-pivotX, -pivotY);
```

但是使用这种方式的话，`preTranslate(pivotX,pivotY)` 和 `preTranslate(-pivotX, -pivotY)`可能会拉的太开，所有通常使用这种写法：

``` java
Matrix matrix = new Matrix();
// 各种操作，旋转，缩放，错切等，可以执行多次。
matrix.postTranslate(pivotX,pivotY);
matrix.preTranslate(-pivotX, -pivotY);
```

> 注意：上面这两种写法的结果是一致的。
>
> 1. 对于第一种写法可表示为：`M * T * ... * -T`
> 2. 对应第二种写法可表示为：`T * M * ... * -T`
>
> 因为 M 是单位矩阵，所以不会影响计算结果，所有上面两种写法都可以表示为：`T * … * -T`

