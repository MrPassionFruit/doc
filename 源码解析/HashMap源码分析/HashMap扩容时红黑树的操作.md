# HashMap扩容时红黑树的操作

## 1、红黑树操作：split函数

```java
/**
* map 当前map
* tab 新数组
* index 旧数组下标
* bit 旧数组长度
**/
final void split(HashMap<K,V> map, Node<K,V>[] tab, int index, int bit) {
    TreeNode<K,V> b = this;
    // Relink into lo and hi lists, preserving order
    //原节点Node，表示扩容后位置不变的节点
    TreeNode<K,V> loHead = null, loTail = null;
    //新节点Node，表示扩容后位置变动的节点，且变为：原索引+oldCap
    TreeNode<K,V> hiHead = null, hiTail = null;
    //原索引、新索引个数
    int lc = 0, hc = 0;
    for (TreeNode<K,V> e = b, next; e != null; e = next) {
        next = (TreeNode<K,V>)e.next;
        //以便GC回收
        e.next = null;
        //与运算，如果为0，则说明扩容后，位置不变，若不为0，则为新的链表
        if ((e.hash & bit) == 0) {
            //如果loTail为空，说明是第一个节点
            if ((e.prev = loTail) == null)
                loHead = e;
            else
                //将节点loTail.next指向e节点
                loTail.next = e;
            //将e赋值给loTail，作为新的loTail
            loTail = e;
            //原索引个数+1
            ++lc;
        }
        //与运算不为0的，放入新的链表结构，扩容后的索引位置为:老表的索引位置＋oldCap
        else {
            //hiTail，说明是第一个节点
            if ((e.prev = hiTail) == null)
                hiHead = e;
            else
                //hiTail.next指向e节点
                hiTail.next = e;
             //将e赋值给hiTail，作为新的hiTail
            hiTail = e;
             //新索引个数+1
            ++hc;
        }
    }
	
    //原节点不为空
    if (loHead != null) {
        //当原节点个数<=6，即链表长度<=6，此时红黑树转链表
        if (lc <= UNTREEIFY_THRESHOLD)
            //红黑树转链表
            tab[index] = loHead.untreeify(map);
        else {
            //将原index位置的节点设置对应的头节点
            tab[index] = loHead;
            //如果新的hiHead节点不为空，表示原来的红黑树被拆分成2个链表
            //则以loHead为根节点, 重新构建红黑树
            if (hiHead != null) // (else is already treeified)
                loHead.treeify(tab);
        }
    }
    //新索引节点不为空，新索引位置=旧索引位置＋oldCap
    if (hiHead != null) {
        //当新索引节点个数<=6，即链表长度<=6，此时红黑树转链表
        if (hc <= UNTREEIFY_THRESHOLD)
            tab[index + bit] = hiHead.untreeify(map);
        else {
            //将新index位置的节点设置对应的头节点
            tab[index + bit] = hiHead;
            //如果loHead节点不为空，表示原来的红黑树被拆分成2个链表
            //则以hiHead为根节点, 重新构建红黑树
            if (loHead != null)
                hiHead.treeify(tab);
        }
    }
}
```

## 2、红黑树转链表：untreeify函数

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    //遍历红黑树
    for (Node<K,V> q = this; q != null; q = q.next) {
        //将节点的next置空，返回node
        Node<K,V> p = map.replacementNode(q, null);
        //如果tl为空，说明是头节点
        if (tl == null)
            hd = p;
        else
            //将tl.next指向下一个节点
            tl.next = p;
        //将p节点作为二次遍历的起始节点
        tl = p;
    }
    //返回头节点
    return hd;
}
```

## 3、链表转红黑树：treeify函数

```java
final void treeify(Node<K,V>[] tab) {
    TreeNode<K,V> root = null;
    //this表示当前链表的头节点
    for (TreeNode<K,V> x = this, next; x != null; x = next) {
        next = (TreeNode<K,V>)x.next;
        //左右节点置空
        x.left = x.right = null;
        //根节点，黑色
        if (root == null) {
            x.parent = null;
            x.red = false;
            root = x;
        }
        else {
            K k = x.key;
            int h = x.hash;
            Class<?> kc = null;
            //根节点开始，查询子节点
            for (TreeNode<K,V> p = root;;) {
                int dir, ph;
                K pk = p.key;
                //如果x节点的hash值小于p节点的hash值，则将dir赋值为-1, 代表向p的左边查找
                if ((ph = p.hash) > h)
                    dir = -1;
                //如果x节点的hash值大于p节点的hash值，则将dir赋值为1, 代表向p的右边查找
                else if (ph < h)
                    dir = 1;
                //hash值相等，则比较key
                else if (
                //comparableClassFor:如果一个对象实现了Comparable<C>，就返回该类，否则返回null
                    (kc == null && (kc = comparableClassFor(k)) == null) 
                    //比较大小，k等于pk
                         ||(dir = compareComparables(kc, k, pk)) == 0)
                    //使用自定义的方法，判断k、pk大小
                    dir = tieBreakOrder(k, pk);

                TreeNode<K,V> xp = p;
                //-1取左节点，1取右节点
                if ((p = (dir <= 0) ? p.left : p.right) == null) {
                    //x的父节点为xp
                    x.parent = xp;
                    if (dir <= 0)
                        xp.left = x;
                    else
                        xp.right = x;
                    //红黑树平衡
                    root = balanceInsertion(root, x);
                    break;
                }
            }
        }
    }
    //如果root节点不在table索引位置的头节点, 则将其调整为头节点
    moveRootToFront(tab, root);
}
```

```java
static int tieBreakOrder(Object a, Object b) {
    int d;
    if (a == null || b == null ||
        (d = a.getClass().getName().compareTo(b.getClass().getName())) == 0){
        d = (System.identityHashCode(a) <= System.identityHashCode(b) ? -1 : 1);
    }
    return d;
}
```

## 4、红黑树平衡：balanceInsertion函数

```java
/**
* root 当前根节点
* x 新插入的节点
**/
static <K,V> TreeNode<K,V> balanceInsertion(TreeNode<K,V> root,
                                            TreeNode<K,V> x) {
    x.red = true;
    //xp父节点，xpp父节点的父节点, xppl父父节点的左节点, xppr父父节点的右节点
    for (TreeNode<K,V> xp, xpp, xppl, xppr;;) {
        //如果x.parent为空，说明当前节点是root节点，为黑色，
        if ((xp = x.parent) == null) {
            x.red = false;
            return x;
        }
        //父节点为黑色或者父父节点不存在
        else if (!xp.red || (xpp = xp.parent) == null)
            return root;
        //父节点为父父节点的左孩子
        if (xp == (xppl = xpp.left)) {
            //xp为红色，xppr也为红色时
            if ((xppr = xpp.right) != null && xppr.red) {
            //由于x节点是红色，则此时需要将xp、xppr转变为黑色
            //为了保持红黑树的特征：每个分支的黑节点数量一致，即xpp转变为红色
            //若xpp为红色之后，可能导致2个红色节点相连，此时需要进行旋转，则将x=xpp，以便进行下次操作旋转
                xppr.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {
                //如果x为父节点的右节点
                if (x == xp.right) {
                    //父节点左旋
                    root = rotateLeft(root, x = xp);
                    //xpp为旋转完成之后的xp的父节点
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        //父节点的父节点右旋
                        root = rotateRight(root, xpp);
                    }
                }
            }
        }
        //xp为右孩子
        else {
            if (xppl != null && xppl.red) {
                xppl.red = false;
                xp.red = false;
                xpp.red = true;
                x = xpp;
            }
            else {
                //左节点
                if (x == xp.left) {
                    //右旋
                    root = rotateRight(root, x = xp);
                    xpp = (xp = x.parent) == null ? null : xp.parent;
                }
                if (xp != null) {
                    xp.red = false;
                    if (xpp != null) {
                        xpp.red = true;
                        //左旋
                        root = rotateLeft(root, xpp);
                    }
                }
            }
        }
    }
}
```

```java
/**
* 左旋：  黑pp            黑pp 
		/               /
	   红p             红r
	    \             /
	      红r        红p
	     /            \
	    黑rl           黑rl
**/
static <K,V> TreeNode<K,V> rotateLeft(TreeNode<K,V> root,
                                      TreeNode<K,V> p) {
    //r为p的右子节点，pp为p的父节点，rl为r的左子节点
    TreeNode<K,V> r, pp, rl;
    //
    if (p != null && (r = p.right) != null) {
        //rl移动到p的右子节点，即rl的父节点为p
        if ((rl = p.right = r.left) != null)
            rl.parent = p;
        //r的父节点为p的父节点，如果为空，则表示是根节点，当前节点颜色为黑色
        if ((pp = r.parent = p.parent) == null)
            (root = r).red = false;
        else if (pp.left == p)
            pp.left = r;
        else
            pp.right = r;
        //旋转完成之后，r的左节点为p，p的父节点为r
        r.left = p;
        p.parent = r;
    }
    return root;
}
```

```java
/**
* 右旋
**/
static <K,V> TreeNode<K,V> rotateRight(TreeNode<K,V> root,
                                       TreeNode<K,V> p) {
    //l为p的左孩子，pp为p的父节点，lr为l的右孩子
    TreeNode<K,V> l, pp, lr;
    //p的左孩子不为空
    if (p != null && (l = p.left) != null) {
        //l的右孩子赋值给父节点p的左孩子
        if ((lr = p.left = l.right) != null)
            lr.parent = p;
        //p的父节点赋值给l的父节点，若为空，则为根节点
        if ((pp = l.parent = p.parent) == null)
            (root = l).red = false;
        else if (pp.right == p)
            pp.right = l;
        else
            pp.left = l;
        //p作为l的右孩子
        l.right = p;
        p.parent = l;
    }
    return root;
}
```

## 5、Root节点调整：moveRootToFront函数

```java
static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
    int n;
    if (root != null && tab != null && (n = tab.length) > 0) {
        //计算root节点的index
        int index = (n - 1) & root.hash;
        //root节点
        TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
        if (root != first) {
            Node<K,V> rn;
            //root赋值
            tab[index] = root;
            //root节点的头节点
            TreeNode<K,V> rp = root.prev;
            //如果next节点不为空，则将next节点的头节点，指向root的头节点
            if ((rn = root.next) != null)
                ((TreeNode<K,V>)rn).prev = rp;
            if (rp != null)
                //next赋值
                rp.next = rn;
            if (first != null)
                //原头节点指向root
                first.prev = root;
            //root的next为first
            root.next = first;
            root.prev = null;
        }
        //检查树是否正常，是否满足红黑树的标准
        assert checkInvariants(root);
    }
}
```

```java
static <K,V> boolean checkInvariants(TreeNode<K,V> t) {
    TreeNode<K,V> tp = t.parent, tl = t.left, tr = t.right,
        tb = t.prev, tn = (TreeNode<K,V>)t.next;
    if (tb != null && tb.next != t)
        return false;
    if (tn != null && tn.prev != t)
        return false;
    if (tp != null && t != tp.left && t != tp.right)
        return false;
    if (tl != null && (tl.parent != t || tl.hash > t.hash))
        return false;
    if (tr != null && (tr.parent != t || tr.hash < t.hash))
        return false;
    if (t.red && tl != null && tl.red && tr != null && tr.red)
        return false;
    if (tl != null && !checkInvariants(tl))
        return false;
    if (tr != null && !checkInvariants(tr))
        return false;
    return true;
}
```

