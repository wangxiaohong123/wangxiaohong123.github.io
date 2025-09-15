---
title: 1.union find
tags:
  - 算法
categories: 基础
copyright: true
---

union find的定义是有一系列的整数对，每队整数表示互相相连，这些整数对有以下特性：

*   如果整数对p和q是相连的，那么q和p也是相连的；
*   p和p是相连的；
*   如果p和q相连，q和r相连，那么p和r也是相连的；

union find如果整数对表示网卡之类的东西可以实现在网络中快速判断两台计算机是否可以通信，我们把节点之间的连接叫做分量，首先需要设计一份API封装所需要的操作，比如初始化，两个节点新建连接、判断两个节点是否可以连接等：

```java
// 初始化N个节点，每个节点的值为N
UF(int N);
// 在p和q之间建立连接
void union(int p, int q);
// p所在分量的标识符
int find(int p);
// 判断p和q是否在同一个分量中
boolean connected(int p, int q);
// 拿到分量数
int count();
```

基本思路就是把同一个分量内的节点的分量值都设置成一样的，这样在判断两个节点是否互通就可以通过比较两个节点的分量值来实现，所以每个节点都需要有一个存储位保存分量值，基础的数据就可以满足这种需求，数组的索引就是节点值，数组值表示分量值，比如id[1]=4，表示节点1的分量值是4，id[3]=9，表示节点3的分量值是9，这样1和3就不是互通的，因为节点值不同。

第一种最简单的实现思路就出来了，使用id变量的索引表示要查询的变量的值，当两个分量连接时，把两个分量的所有节点都改成相同的值，需要遍历数据，同时分量数减1，属于union-find的quick-find版本，因为他的查找很快，但是union方法很慢，代码如下：

```java
public class Uf1 {

    /**
     * 分量数
     */
    private int count;
	/**
     * 所有节点
     */
    private final int[] id;

    public Uf1(int n) {
        this.id = new int[n];
        this.count = n;
        for (int i = 0; i < n; i ++) {
            id[i] = i;
        }
    }

    /**
     * 连接两个节点
     *
     * 很慢，最坏的情况就是每次调用这个方法都需要遍历整个数据
     * 这个 无法处理大量的数据
     */
    public void union(int p, int q) {
        int valueP = find(p);
        int valueQ = find(q);
        if (valueP == valueQ) {
            return;
        }
        for (int i = 0; i < this.id.length; i ++) {
            if (this.id[i] == valueP) {
                this.id[i] = valueQ;
            }
        }
        this.count--;
    }

    /**
     * 获取节点所在分量
     *
     * 这种写法的find是很快的，他只是根据索引拿到元素
     */
    public int find(int p) {
        return this.id[p];
    }

    /**
     * 判断两个节点是否在一个分量里
     */
    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    /**
     * 获取分量数
     */
    public int count() {
        return this.count;
    }

    public static void main(String[] args) {
        int[] test = {2,32,1,4,5,6,7,8,23,21,7,88,14,75,32,1,7,8};
        Uf1 uf1 = new Uf1(100);
        for (int i = 0; i < test.length; i += 2) {
            uf1.union(test[i] , test[i +1]);
        }

        System.out.println("count:" + uf1.count());
        System.out.println("是否连通:" + uf1.connected(2, 20));
    }
}
```

这种实现的缺点很明显，每次连接都需要遍历整个数组，数组非常大的时候union()方法很快会把资源耗尽，原因就是要把所有的分量值都修改一下，优化的思路就是修改分量值的时候不全部修改，只把判断的节点的分量值指向要连接的节点，这样一个分量下的节点就变成了一棵树，判断两个节点是否连通只要判断根节点是否相同就可以了，根节点就是值和索引相同的节点，这个是union-find的quick-union版本，这种写法有个缺点就是最坏的情况下需要遍历整个数组，所以他只是执行union()方法的时候很快，而connected()方法可能会很慢，代码如下：

```java
public class Uf2 {
    /**
     * 分量数
     */
    private int count;
	/**
     * 所有节点
     */
    private final int[] id;

    public Uf2(int n) {
        this.id = new int[n];
        this.count = n;
        for (int i = 0; i < n; i ++) {
            id[i] = i;
        }
    }

    /**
     * 连接两个节点
     */
    public void union(int p, int q) {
        int valueP = find(p);
        int valueQ = find(q);
        if (valueP == valueQ) {
            return;
        }
        this.id[p] = this.id[q];
        this.count--;
    }

    /**
     * 获取节点所在分量
     */
    public int find(int p) {
        while (p != this.id[p]) {
            p = this.id[p];
        }
        return p;
    }

    /**
     * 判断两个节点是否在一个分量里
     */
    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    /**
     * 获取分量数
     */
    public int count() {
        return this.count;
    }

    public static void main(String[] args) {
        int[] test = {2,32,1,4,5,6,7,8,23,21,7,88,14,75,32,1,7,8};
        Uf2 uf2 = new Uf2(100);
        for (int i = 0; i < test.length; i += 2) {
            uf2.union(test[i] , test[i +1]);
        }

        System.out.println("count:" + uf2.count());
        System.out.println("是否连通:" + uf2.connected(2, 32));
    }
}
```

考虑connected()方法最坏的情况的原因是因为在链接的时候导致树只有左节点或者右节点，变成了单向链表，查询的时候变成了遍历链表，换个说法就是树的深度过深，解决这个问题只需要在修改向量值的时候改成根节点，然后在优化一下就是向量里节点少的指向节点多的，这样还需要加一个变量记录每个向量里的节点值，他数据quick-union的加权版，代码如下：

```java
public class Uf3 {
	/**
     * 分量数
     */
    private int count;
	/**
     * 所有节点
     */
    private final int[] id;
	/**
     * 向量里的节点数量
     */
    private final int[] tSize;

    public Uf3(int n) {
        this.id = new int[n];
        this.count = n;
        this.tSize = new int[n];
        for (int i = 0; i < n; i ++) {
            this.id[i] = i;
            this.tSize[i] = 1;
        }
    }

    /**
     * 连接两个节点
     */
    public void union(int p, int q) {
        int valueP = find(p);
        int valueQ = find(q);
        if (valueP == valueQ) {
            return;
        }
        if (this.tSize[p] < this.tSize[q]) {
            this.id[p] = valueQ;
            this.tSize[q] += this.tSize[p];
        } else {
            this.id[q] = valueP;
            this.tSize[p] += this.tSize[q];
        }
        this.count--;
    }

    /**
     * 获取节点所在分量
     */
    public int find(int p) {
        while (p != this.id[p]) {
            p = this.id[p];
        }
        return p;
    }

    /**
     * 判断两个节点是否在一个分量里
     */
    public boolean connected(int p, int q) {
        return find(p) == find(q);
    }

    /**
     * 获取分量数
     */
    public int count() {
        return this.count;
    }

    public static void main(String[] args) {
        int[] test = {2,32,1,4,5,6,7,8,23,21,7,88,14,75,32,1,7,8};
        Uf3 uf2 = new Uf3(100);
        for (int i = 0; i < test.length; i += 2) {
            uf2.union(test[i] , test[i +1]);
        }

        System.out.println("count:" + uf2.count());
        System.out.println("是否连通:" + uf2.connected(2, 32));
    }
}
```

