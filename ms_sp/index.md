# ShortestPathTree & MinimumSpanningTree


Dijkstra/A*/Prim/Kruskal Algorithms

<!--more-->


## ShortestPathTree
[SlideLink](https://docs.google.com/presentation/d/1X_HRo2Wr9FwFrzRcH8Ppq6L5r1bEGrlhLS2L9hY4-Ec/edit#slide=id.g54762a0157_3_13)
### problem: Single Source Single Target Shortest Path
在图中规定一个起始节点，那么从该起始节点到其余每个结点的最短路径形成的就是最短路径树。在边的权值不为负数的情况下不可能出现闭环的情况。
### Dijkstra算法
[DemoLink](https://docs.google.com/presentation/d/1_bw2z1ggUkquPdhl7gwdVBoTaoJmaZdpkV6MoAgxlJc/pub?start=false&loop=false&delayms=3000)

#### 概述
采用该算法解决最短路径问题的流程：
1. 根据描述建立图的邻接表
2. 初始化一个优先级队列，储存的是图中的所有结点，每个节点对应的优先度又它当前距离起点的距离决定。优先度在结点未被发现的情况下只有root为0，其余皆为正无穷。
#### 简单证明
#### Pseudocode

## Minimum Spanning Tree



