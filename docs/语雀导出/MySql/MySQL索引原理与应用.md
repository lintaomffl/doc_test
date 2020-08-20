# MySQL索引原理与应用

主要参考文章：[https://open.atatech.org/articles/142815](https://open.atatech.org/articles/142815)<br />
<br />索引与B+树的内容：[https://juejin.im/post/5c3b22a1f265da61776c2bf2#heading-1](https://juejin.im/post/5c3b22a1f265da61776c2bf2#heading-1)<br />[http://mysql.taobao.org/monthly/2018/09/01/#](http://mysql.taobao.org/monthly/2018/09/01/#)<br />
<br />
<br />M阶B+特点：<br />根节点有[2,M]个子树；<br />非根节点有[m/2,m]个子树；<br />非叶子节点仅作为叶子节点的索引不保存任何数据；<br />数据全部保存在叶子节点，且叶子节点连接成了一个链表。<br />B+树比B树更适合的原因：<br />1.B+树的非叶子节点都不保存数据，所以每次查询都是查询到叶子节点才得到数据，查询效率稳定；<br />2.B+树的非叶子节点不保存数据，那么以一个盘来存一个节点的数据的话，B+树的非叶子节点只有索引没有数据，会包含更多的关键字；<br />3.B+树更容易归库，因为数据都存在叶子结点，不像B树要进行树的遍历获取各个节点上的数据。
