[教你初步了解红黑树](http://blog.csdn.net/v_july_v/article/details/6105630)

[wiki](https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91)



### 红黑树的介绍

红黑树，一种二叉查找树，但在每个节点上增加一个存储位表示节点的颜色，可以是 Red 或 Black。

通过对任何一条从根到叶子的路径上各个节点着色方式的限制，红黑树确保没有一条路径会比其他路径长出两倍，因而是接近平衡的。

> 二叉树的定义：也称为有序二叉树，是指一颗空树或者具有下列性质的二叉树：
>
> * 若任意节点的左子树不空，则左子树上所有的节点的值均小于它的根节点的值；
> * 若任意节点的右子树不空，则右子树上所有的节点的值均大于它的根节点的值；
> * 任意节点的左、右子树也分别为二叉查找树；
> * 没有键值相等的节点（no duplicate nodes）；

红黑树的 5 个性质：

1. 每个节点要么是红色要么是黑色；
2. 根节点是黑色的；
3. 每个叶节点（即树尾端 NIL 指针或 NULL 节点）都是黑色的；
4. 如果一个节点是红色的，那么它的两个儿子都是黑色的；
5. 对于任意节点而言，其到叶节点尾端 NIL 指针的每条简单路径都包含相同数目的黑色节点；

这 5 个性质使得一个 n 个节点的红黑树始终保持 logn 的高度，从而保证了红黑树的查找、插入、删除的时间复杂度最坏为 O(log n)。

![红黑树](http://img.my.csdn.net/uploads/201212/12/1355319681_6107.png)

### 插入操作

以二叉查找树的方法增加节点并标记为红色。

> 如果设为黑色，就会导致根到叶的路径有一条路上，多了额外的黑色节点，很难调整。

对于插入操作来说：

* 性质 1 和 性质 3 总是保持着；
* 性质 4 只在增加红色节点、重绘黑色节点为红色、或做旋转时受到威胁；
* 性质 5 只在增加黑色节点、重绘红色节点为黑色、或做旋转时受到威胁；

> 将要插入的节点为 N，N 的父节点为 P，N 的祖父节点为 G，N 的叔父节点为 U。

#### 情形1：新节点位于树的根上，没有父节点

只需要将新节点绘制为黑色即可。

``` c
void insert_case1(node *n) {
    if(n->parent == NULL)
        n->color = BLACK;
    else
        insert_case2(n);
}
```

#### 情景2：新节点的父节点 P 是黑色

> 注意这里假定新节点为红色。

不需要修改，仍然满足红黑树的性质。因为二叉查找树的插入节点都是叶子节点，并且新节点是红色节点，所以通过它的每个子节点的路径就都有，同通过它所取代的黑色的叶子的路径同样的黑色节点。

``` c
void insert_case2(node *n){
    if(n->parent->color == BLACK)
        return;
    else 
        insert_case3(n);
}
```

**注意：在以下的情形中，我们假定新节点的父节点为红色，所有它都有祖父节点，因为如果父节点是根节点，那么父节点应该是黑色的，所以新节点总有一个叔父节点。**

#### 情形3：如果父节点 P 和叔父节点 U 二者都是红色

将它们两个重绘为黑色并重绘祖父节点 G 为红色。

``` c
void insert_case3(node *n){
    if(uncle(n) != NULL && uncle(n)->color == RED){
        // 下面三步保持了性质 5
        n->parent->color = BLACK;
        uncle(n)->color = BLACK;
        grandparent(n)->color = RED;
        // 可能违反了性质 2，如果祖父节点是根节点
        // 可能违法了性质 4，如果祖父节点的父节点是红色
        // 所以重新进行检查
        insert_case1(grandparent(n));
    }
    else 
        insert_case4(n);
}
```

![insert_case3](https://upload.wikimedia.org/wikipedia/commons/c/c8/Red-black_tree_insert_case_3.png)

**注意：在余下的情形中，我们假定父节点 P 是其父亲 G 的左子节点，如果它是右子节点，情形 4 和 5 的左和右应该对调。**

#### 情形4：父节点 P 是红色而叔父节点 U 是黑色或缺少，并且新节点 N 是其父节点 P 的右子节点而父节点 P 又是其父节点的左子节点

针对父节点 P 进行一次**左旋**转调换新节点和其父节点的角色

``` c
void insert_case4(node *n){
    // 上面已经判断已经检查过父节点是红色和叔父节点不是红色
    if( n == n->parent->right && n->parent == grandparent(n)->left) [
        rotate_left(n->parent);
        n = n->left;
    ]else if(n == n->parent->left && n->parenty == grandparent(n)->right) {
        rotate_right(n->parent);
        n = n->right;
    }
    insert_case5(n);
}
```

![情形4](https://upload.wikimedia.org/wikipedia/commons/5/56/Red-black_tree_insert_case_4.png)

#### 情形5：父节点 P 是红色而叔父节点 U 是黑色或缺少，新节点 N 是其父节点的左子节点，而父节点 P 又是其父节点 G 的左子节点

切换父节点 P 和祖父节点 G 的颜色，针对祖父节点 G 进行一次**右旋**

``` c
// 根据前面的判断，存在祖父节点是黑色的
void insert_case5(node *n){
    // 父节点从 红色->黑色
    n->parent-color = BLACK;
    // 祖父节点从 黑色->红色
    grandparent(n)->color = RED;
    if(n == n->parent->left && n->parent == grandparent(n)->left){
        rotate_right(grandparent(n));
    }else {
        rotate_left(grandparent(n));
    }
    
}
```

