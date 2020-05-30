# CodeCraft2020-Final
华为2020软件精英挑战赛决赛，2147483647队决赛代码，练习赛第6，决赛第14。

## 写在前面
感谢两个月来队友们的不懈努力，也恭喜获奖的。去年软挑喜提八强之后，今年本是带着更进一步的期望来的，无奈能力确实有限，最后决赛的Bonus图中两个大SCC的情况队友在赛前提出来过，但限于时间紧迫以及还有其他提升鲁棒性的方案需要实现，没有针对这种情况进行处理，决赛的大部分时间把优化目标放在了12核的NUMA上，但也没有取得更好的成绩。虽然最后结果小有遗憾，但总的来说不论是经历上还是知识上都是收获颇丰的。

## 初赛复赛
初赛复赛阶段赛题组有许多失误，不多赘述，我们队侥幸没有中枪，代码也许会在整理后开源。

## 决赛
### 赛题
简单来说是计算带权有向图的关键位置中心性（介数中心性，Betweeness Centrality，BC），数据规模为250w条边，节点的id进行了离散化，权重为32位无符号正整数。
### 数据结构
主要采用了前向星的存图方式优化访存，每个线程所用到的数据存储在一个结构体内部，相互之间隔离开。能用静态数组的地方绝不用动态数组或vector，空间换时间。
针对不同的数据权重范围切换使用uint32或uint64。根据练习赛数据特点，采用了HashMin和八叉堆两种方式实现优先队列用于dijkstra。之前的版本中尝试了使用重写的vector存储前驱图，效果还不错，最后阶段改为使用一个栈按照拓扑序存储最短路上的边以减少一层寻址。
### 算法思路
主要使用Brandes算法，对每个源点使用dijkstra求最短路，同时可以按拓扑序存储最短路径构成的DAG图上的点和边。然后可以按正/逆拓扑序遍历所有的边以递推的方式求出从源点到达每个其他点的最短路径数量，从而求出介数中心性。
针对一般情况，用八叉堆实现优先队列，不带modify操作。为了减少堆的push和pop开销，我们维护了一个等待队列WaitList和一个代表优先队列中最大距离的MaxDist，搜索时只将距离小于MaxDist的点(及其当前距离)放入优先队列中，其余点暂时放入WaitList。当优先队列为空时，再将WaitList中的点push到优先队列里。Push时可以过滤掉存储时距离大于该点当前距离的记录。这一方法减小了优先队列的平均大小，减轻了push pop的开销，在从WaitList向优先队列添加元素的过程中也有一定的剪枝。这一方法优于我们实现的其他基于线段树、配对堆等数据结构的dijkstra。另外仅针对第一张图这种极端稀疏图，SPFA也是可以考虑的，在我们的早期版本中，SPFA有明显优势。
针对练习数据这种图半径在10000以内的情况，我们采用了一种对距离进行Hash的方式，插入删除的时间复杂度接近O(1)，只在pop到某一距离的元素全部清空或第一次出现时才会触发O(ln(n))的优先队列操作。这在图半径较小时有60%以上的速度提升。
### 其他小trick
未完待续
