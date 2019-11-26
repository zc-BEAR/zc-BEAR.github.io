---
layout: post
title: 更进一步——初探高级二叉树
date: 2019-11-23
tag: algorithm
---

基础的树结构在上两节已经大概梳理了一遍，但是树的内容还是比较多的，所以在这里又开了一章讲一讲期中考试以来这两周我们学到的一些树相关的知识。同时，DSAA的第二次quiz安排在~~11月26日~~(延期到12月2日了),本章同时做一个对树阶段的考点复习。

##### tips：虽然本人用JAVA更熟练，但是考虑到算法题中大多使用C++来实现，并且我也在龟速学习C++，这个标签下的代码都是C++代码。

### 小目录：记录这一章要讲些什么
 
* BST 二叉查找树的构建和操作
* BBST 平衡二叉查找树(解决二叉树失衡退化成链表的问题)[AVL-tree][treap]
* 一些证明：BBST的高度为O(logn),
* 其他重要的不重要的内容

### 二叉查找树：二叉树的基础应用，其他高级二叉树的基本功

二叉查找树(以下简称BST)，顾名思义，就是有查找功能的二叉树，即这个树的结构是有一定规律的——每个节点的左子树中所有节点的值均小于该节点的值，反之，右子树的所有节点的值均大于该节点的值。对于一个二叉查找树，我们需要掌握的基本操作有插入、删除、查找等，这些是后期学到高级树之后我们依旧需要用到的基本功。开始之前，我们先回顾一下二叉树的基本结构：

```cpp
struct TreeNode{
    int val;
    TreeNode *fa;
    TreeNode *lc;
    TreeNode *rc;
};
```

其中，根节点没有父节点，且所有节点的左孩子值一定小于节点值，右孩子一定大于节点值。

#### 搜索

搜索其实范围挺大的，在二叉树中搜索某个值只是其中比较重要的一个部分，还有其他的搜索条件比如：找到最大/最小值、找到比key小的最大/比key大的最小值等

一个简单的递归查找指定值的方法：
```cpp
AVLTreeNode<T>* search(AVLTreeNode<T>* root, T key)
{
    if(root==NULL || x->key==key)
        return root;
    if(key < root->key)
        return search(root->left , key);
    else
        return search(root->right , key);
}
```

查找最大值的简单递归(最小值同理，不再赘述)：
```cpp
AVLTreeNode<T>* maximum(AVLTreeNode<T>* root)
{
    if (tree == NULL) return NULL;
    while(root->right != NULL) tree = tree->right;
    return tree;
}
```

前后缀查找：
```cpp
//代码待补全
```

#### 添加和删除

添加的思路很清晰，即构建一个值为val的节点，然后通过**搜索**找到一个符合要求的节点——节点值大于目标值，且节点的左孩子的值小于目标值 or 节点值小于目标值，且节点的左孩子的值大于目标值。然后做一个类似于链表操作的链接即可。

删除的话只需要找到某个节点然后删除之后用左子树最大或者右子树最小替代一下就好，也并不复杂。因为下面有增加了平衡功能的增删代码，这里就不对这两个操作做过多的描述了。

#### 遍历

遍历也是常规的二叉树操作了，前中后序和层序遍历就是一般二叉树的写法，这里写上这一点只是保证结构的完整性，具体的实现就不多提了。


### 平衡二叉查找树(BBST)

回顾了以上几个二叉查找树的功能以后，我们可以进入平衡二叉查找树的学习了。那么这个平衡的意思是什么呢？为什么需要把二叉树进行一个平衡操作？

通过二叉查找树的代码实现，我们可以看出，二叉查找树有一个十分致命的弊端——插入的条件限制的太少了。对于一棵树而言，满足节点符合插入的情况有很多，这就意味着在最坏的情况下可以一直往左子树插入节点，那二叉树就将退化成一条链表了。为了避免这种情况的出现，我们就需要对二叉查找树做一个“平衡”的操作。平衡是指：在插入节点时如果出现了某个节点(指被插入的根节点)的左子树高度比右子树高度大1以上的情况时，把该节点进行旋转操作，使得旋转后节点处子树高度差不大于1。

**注：**BBST的AVL部分因为没有自己实现全过程的能力，一些代码参考了[遨游网络huster的CSDN博客](https://blog.csdn.net/u013149325/article/details/41381607)
#### 每一个树节点的形状

因为其中很多~~我懒得写~~比较复杂的代码直接转载了原博主的内容，这里有必要贴一下原博主写的树节点，让读者可以比较流畅地看懂每个方法使用的变量名称。其中，如果你想的话，可以增加一个fa节点指向自己的父节点，这样的调用更加灵活，但是这不是必要的。而其中的height变量是平衡二叉树所区别于二叉树的一个重要特点，利用这个变量可以判断出一个节点是否失衡。

```cpp
template <class T>
class AVLTreeNode{
    public:
        T key;                // 关键字(键值)
        int height;         // 高度
        TreeNode *left;    // 左孩子
        AVLTreeNode *right;    // 右孩子
        //构造函数
        AVLTreeNode(T value, AVLTreeNode *l, AVLTreeNode *r):
            key(value), height(0),left(l),right(r) {}
};
```
**注意：这种AVL树只允许不重复的节点插入，如果需要有重复的节点插入应该在每个节点处加入一个用于计数的变量，以及在每次插入和删除时多进行一些判断(假设这些重复节点对树的高度不做贡献)**
#### 失衡的四种姿势：LL，LR，RR，RL

二叉平衡树中最需要理解的部分就是旋转的部分，我们需要掌握四种需要被旋转的失衡方式以及相应的旋转方式，最好自己手写一下这些片段的代码加深对这几种旋转方式的理解。

* **LL**：增删节点后根节点的左子树高度比右子树高度大2，左孩子的左子树比右子树高度大1
* **LR**：增删节点后根节点的左子树高度比右子树高度大2，左孩子的右子树比左子树高度大1
* **RL**：增删节点后根节点的右子树高度比左子树高度大2，右孩子的左子树比右子树高度大1
* **RR**：增删节点后根节点的右子树高度比左子树高度大2，右孩子的右子树比左子树高度大1

以上就是四种旋转的方式，只看文字可能比较抽象，下面贴两张图片举例子来表示失衡的四种情况。

![avatar](\2019-11-24-AdvanceTree\lostBalance1.jpg)

![avatar](\2019-11-24-AdvanceTree\lostBalance2.jpg)

图片来自[CSDN博客](https://blog.csdn.net/u013149325/article/details/41381607)

#### 旋转：如何通过代码旋转节点达到重新平衡的目的

**LL型的旋转**

LL型旋转目的是让左孩子b旋转到根节点a的位置，根节点a变为b的右孩子，最后让b的右孩子成为a的左孩子。可以想象为**一个不平衡的天平向右边倾斜，然后中间偏左的重物往右边掉了，挂在右边的节点上(来自想象力丰富的沈老师)**。即需要做出更改的边有三条：ab之间的边，a连向父亲节点的边以及b的一条子树连到a上面。每条边需要改变双向的节点，所以从思路上来说这需要**六行代码**，这是一定要明确的。写成代码的形式就是下面的六行：

```cpp
AVLTreeNode* LLrotation(AVLTreeNode* a)
{
    AVLTreeNode* b;
    b = a->left;
    a->left = b->right;
    b->right = a;
    //如果写的是有父亲的版本
    b->fa = a->fa;
    a->fa = b;
    a->left->fa = a;
    //如果写的节点还需要记录树的高度
    a->height = max(height(a->left),height(a->right))+1;
    b->height = max(height(b->left),a->height)+1;
    return b;
}
```

补充一点后来发现的问题：这一段代码并不是每一行都是必须的，用不同的方式实现AVL需要用到不同的部分。比如写父亲节点的那一段，在这个实现中完全可以删掉也没有任何影响，因为这个实现对AVL树旋转方法的调用是有返回值的，也就是说在调用旋转方法的时候把以地址来调用，直接把这个节点的地址传入原来的位置，不需要考虑和父节点的链接问题。

**RR型的旋转**

同理，RR型的旋转无非是LL型对称情况，实现起来无非就是换个放下而已，这里就直接贴上代码了：

```cpp
AVLTreeNode* RRrotation(AVLTreeNode* a)
{
    AVLTreeNode<T>* b;
 
    b = a->right;
    a->right = b->left;
    b->left = a;
    //如果写的是有父亲的版本
    b->fa = a->fa;
    a->fa = b;
    a->right->fa = a;
    //如果写的节点还需要记录树的高度
    a->height = max( height(a->left), height(a->right)) + 1;
    b->height = max( height(b->right), a->height) + 1;
 
    return b;

```

**LR型的旋转**

这种情况就和上面的情况有区别了，实际上这种失衡相当于旋转两次，因为是相当于是对三个节点进行操作，要先对左子节点进行RR旋转的操作让该节点从右偏到形成左偏的结构，就可以使得这三个节点形成一个LL型的情况，在进行一次LL旋转，最后达到平衡的目的。因此，这种结构需要更改的边数为**5条**(含父节点的情况)，每条边是双向的，也就是需要**十行代码**，
但是这十行代码怎么写是考试中需要写出来的东西，在实际的代码中我们很容易知道其实这种情况的旋转只是调用了一次RR型旋转和一次LL型旋转而已：
```cpp
AVLTreeNode* LRrotation(AVLTreeNode* a)
{
    a->left = RRrotation(a->left);

    return LLrotation(a);
}
```

**RL的旋转**

RL的旋转与LR的旋转也是对称的情况，所以可以很容易写出这个情况的旋转方法：
```cpp
AVLTreeNode* RLrotation(AVLTreeNode* a)
{
    a->right = LLrotation(a->right);

    return RRrotation(a);
}
```

以上就是四种失去平衡的情况我们需要做的旋转。需要明确的一点是失衡只会在你增加/删除节点(对树有扰动的情况下)时发生，所以所有的旋转操作只需要在插入和删除的时候进行判定就可以了。下面继续复习插入和删除的过程。

#### 插入 

给定一个key值，将这个值插入到二叉平衡树中合适的位置，传入参数为插入值和平衡树的根节点。同样的有递归版本和非递归版，显然一门基础课中不会考的太深入，非递归版的思路不太直观，这里总结递归版的代码，如果有兴趣可以移步原博主的博客参考他写的非递归版代码。

```cpp
template <class T>
AVLTreeNode<T>* insert(AVLTreeNode* &tree , T key)
{
    if(tree==NULL)
    {
        tree = new AVLTreeNode<T>(key,NULL,NULL);
        //避免新建失败应提出错误信息
        if(tree == NULL)
        {
            cout << "create failed!"<<endl;
            return NULL;
        }
    }
    else if(key < tree->key)
    {
        tree->left = insert(tree->left , key);
        //插入节点后。若AVL树失去平衡，则进行相应的调节
        if(height(tree->left) - height(tree->right)==2)
        {
            //其实插入左边的失衡情况只有两种
            if(key < tree->left->key)
                tree = LLrotation;//如果加在左子树的左子树就是LL
            else
                tree = LRrotation;//如果加在左子树的右子树就是LR
        }
        
    }
    else if(key > tree->key)
    {
        tree->right = insert(tree->right , key);
        if(height(tree->left) - height(tree->right)==2)
        {
            //插入右边的失衡情况只有两种
            if(key < tree->right->key)
                tree = RLrotation;//如果加在左子树的左子树就是RL
            else
                tree = RRrotation;//如果加在左子树的右子树就是RR
        }
        
    }
    else//已存在这个key
    {
        cout<<"add failed : you shouldn't add same key"<<endl;
    }
    tree->height = max(height(tree->left),height(tree->right))+1;
    return tree;
}
```

可见，在你写好了几种平衡的方法之后，递归版的插入并没有什么太多需要你去实现的地方，只要做好几个判断调用合适的平衡方法就可以完成一个节点的插入了。接下来让我们来看看删除的操作。

### 删除

删除一个节点要考虑的东西也和插入差不多，大概就是树是否为空/平衡/是否存在这个数。第一个和第三个不必多说，第二个平衡判断则是删除代码的主角内容，以下是实现节点删除的代码，传入的参数为树的根节点和需要删除的值(作者原本传入的是节点，但是个人认为直接传值可以更灵活地修改代码)，返回的值为根节点。
```cpp
template<class T>
AVLTreeNode<T>* remove(AVLTreeNode<T>* &tree, T key)
{
    //先判断一个最简单的根为空
    if(tree==NULL) return NULL;
    if(key < tree->key){
        tree->left = remove(tree->left , key);
        // 以下是失衡调节
        if (height(tree->right) - height(tree-left) == 2)
        {
            AVLTreeNode<T>* r = tree->right;
            if(height(r->left) > height(r->right))
                tree = RLrotation(tree);
            else
                tree = RRrotation(tree);
        }
    }
    else if(key > tree->key)
    {
        tree->right = remove(tree->right , key);
        if(height(tree->left) - height(tree->right) == 2)
        {
            AVLTreeNode<T>* l = tree->left;
            if(height(l->right) > height(l->left))
                tree = LRrotation(tree);
            else
                tree = LLrotation(tree);
        }
    }
    else
    {
        if((tree->left != NULL) && (tree->right != NULL))
        {
            if(height(tree->left) > height(tree->right))
            {
                //如果被删除的节点的左子树比右子树高，那么把左子树中最大的节点换到被删除的位置，这样做的好处是在删除某个节点是仍旧可以保持原本的平衡性
                AVLTreeNode<T>* max = maximum(tree->left);
                tree->key = max->key;
                tree-left = remove(tree->left,max);
            }
            else
            {
                //同样的，再右边比较高时采用的维持平衡的方式
                AVLTreeNode<T>* min = minimum(tree->right);
                tree->key = min->key;
                tree->right = remove(tree->right, min);
            }
        }
        else
        {
            //直接有一边的子树时空的就比较好处理了
            AVLTreeNode<T>* tmp = tree;
            tree = (tree->left != NULL) ? tree->left : tree->right;
            delete tmp;
        }
    }

    return tree;

}
```

以上就是删除代码的所有内容了可以说是以及完成了一个平衡二叉树的绝大部分核心代码，以至于其他一些诸如**摧毁树**(一个后序遍历的删除操作)之类的比较容易一次性写对的操作就不再这里啰嗦了，AVL树到此告一段落。

### Treap

Treap是一种也实现了平衡二叉树功能的二叉树，拥有较容易实现的特点，但是他的稳定性就不如AVL树。因为这个星期的lab是用Treap的板子过的，所以才会接触到这样一种相对比较冷门的树类型，还没有想好究竟要不要展开来讲，这里就先列一个标签在这里吧。

### 各种证明

开这一节的目的是将树这一部分一些重要的证明都记录汇总一下，但是目前来看这一节的内容已经非常的长了，这一段的内容写完后可能会迁移到下一节。

#### 第一个证明：BBST的高度为O(logn)

