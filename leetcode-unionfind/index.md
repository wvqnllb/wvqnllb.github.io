# Leetcode UnionFind


<!--more-->

## UnionFind
### 数组实现
实现还是非常简单的~
实现细节：
1. 用负数标注当前树的大小
2. 将小树附加到大树上，避免出现低效率情况
3. 进行根节点查找时进行路径压缩
```java
    public class UnionFind {
        public int[] parents;
        public UnionFind (int size) {
            parents = new int[size];
            //初始全部为大小为1的单独子树
            Arrays.fill(parents, -1);
        }

        //path compreesion
        public int findRoot(int v) {
            if (parents[v] < 0) {
                return v;
            }
            parents[v] = findRoot(parents[v]);
            return parents[v];
        }

        public void union (int v1, int v2) {
            int r1 = findRoot(v1);
            int r2 = findRoot(v2);
            if (r1 == r2) {
                return;
            }
            //小树的根作为大树根的儿子，更新大树根
            if (-parents[r1] > -parents[r2]) {
                parents[r1] += parents[r2];
                parents[r2] = r1;
            } else {
                parents[r2] += parents[r1];
                parents[r1] = r2;
            }
        }
        //判断是否在同一树中
        public boolean find(int v1, int v2) {
            int r1 = findRoot(v1);
            int r2 = findRoot(v2);
            if (r1 == r2) {
                return true;
            }
            return false;
        }
    }
```
### leetcode200 岛屿数量
[题目链接](https://leetcode-cn.com/problems/number-of-islands/)

dfs: 采用dfs并判断dfs次数
```java
class Solution {
    void dfs(char[][] grid, int r, int c) {
        int nr = grid.length;
        int nc = grid[0].length;

        if (r < 0 || c < 0 || r >= nr || c >= nc || grid[r][c] == '0') {
            return;
        }

        grid[r][c] = '0';
        dfs(grid, r - 1, c);
        dfs(grid, r + 1, c);
        dfs(grid, r, c - 1);
        dfs(grid, r, c + 1);
    }

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) {
            return 0;
        }

        int nr = grid.length;
        int nc = grid[0].length;
        int num_islands = 0;
        for (int r = 0; r < nr; ++r) {
            for (int c = 0; c < nc; ++c) {
                if (grid[r][c] == '1') {
                    ++num_islands;
                    dfs(grid, r, c);
                }
            }
        }

        return num_islands;
    }
}

//来源：LeetCode
```
bfs: 队列迭代实现
```java
class Solution {
    void dfs(char[][] grid, int r, int c) {
        int nr = grid.length;
        int nc = grid[0].length;

        if (r < 0 || c < 0 || r >= nr || c >= nc || grid[r][c] == '0') {
            return;
        }

        grid[r][c] = '0';
        dfs(grid, r - 1, c);
        dfs(grid, r + 1, c);
        dfs(grid, r, c - 1);
        dfs(grid, r, c + 1);
    }

    public int numIslands(char[][] grid) {
        if (grid == null || grid.length == 0) {
            return 0;
        }

        int nr = grid.length;
        int nc = grid[0].length;
        int num_islands = 0;
        for (int r = 0; r < nr; ++r) {
            for (int c = 0; c < nc; ++c) {
                if (grid[r][c] == '1') {
                    ++num_islands;
                    dfs(grid, r, c);
                }
            }
        }

        return num_islands;
    }
}

//来源：LeetCode
```

UnionFind实现
```java
//时间复杂度O(mn) * α(x)
//空间复杂度O(mn)
class Solution {
    int m;
    int n;

    public class UnionFind {
        public int[] parents;
        public UnionFind (char[][] grid) {
            this.parents = new int[m * n];
            Arrays.fill(parents, -1);
        }

        //path compreesion
        public int findRoot(int v) {
            if (parents[v] < 0) {
                return v;
            }
            parents[v] = findRoot(parents[v]);
            return parents[v];
        }

        public void union (int v1, int v2) {
            int r1 = findRoot(v1);
            int r2 = findRoot(v2);
            if (r1 == r2) {
                return;
            }
            if (-parents[r1] > -parents[r2]) {
                parents[r1] += parents[r2];
                parents[r2] = r1;
            } else {
                parents[r2] += parents[r1];
                parents[r1] = r2;
            }
        }

        public boolean find(int v1, int v2) {
            int r1 = findRoot(v1);
            int r2 = findRoot(v2);
            if (r1 == r2) {
                return true;
            }
            return false;
        }
    }

    public int numIslands(char[][] grid) {
        int count0 = 0;
        int res = 0;
        if (grid == null || grid.length == 0) {
            return res;
        }
        m = grid.length;
        n = grid[0].length;

        UnionFind uf = new UnionFind(grid);
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == '1') {
                    int index1 = getIndex(i, j);
                    if (i > 0 && grid[i - 1][j] == '1') {
                        int index2 = getIndex(i - 1, j);
                        uf.union(index1, index2);
                    }
                    if (j > 0 && grid[i][j - 1] == '1') {
                        int index2 = getIndex(i, j - 1);
                        uf.union(index1, index2);
                    }
                } else {
                    count0++;
                }
            }
        }

        for (int i : uf.parents) {
            if (i < 0) {
                res++;
            }
        }

        return res - count0;

    }

    public int getIndex(int r, int c) {
        return r * n + c;
    }
}
```
