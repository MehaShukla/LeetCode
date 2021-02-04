### 444.Sequence-Reconstruction

本题巧妙地应用了拓扑排序的原理。我们根据seqs可以得到每个节点的入度和（有向的）邻接节点。我们每个回合里，统计当前入度为零的节点，这样的节点必须只有一个。如果有若干个，那么他们彼此之间的先后顺序必然是不确定的。此外，这唯一的入度为零的节点（整个图的起点），也必须是org里当前的首元素。如果不满足这两个条件，我们就返回false。

在满足这两个条件之后，我们就弹出org的首元素，考察它所邻接的节点及其更新后的入度（它们的入度要减一），继续寻找其中入度为零的节点。不断重复直至遍历完所有的拓扑关系。

当遍历完所有的拓扑关系后，我们要求也恰好遍历完org里面所有的节点。否则返回false。

此外，seqs里面所有的节点必须是在[1,n]的范围里。