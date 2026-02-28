

# DFS 核心
**一条路走到黑，走不通再回头，换条路继续走。**
本质就是：**递归 + 回溯**。

---

# 1. DFS 通用模板思路（背会这个）
```
dfs(当前状态) {
    if (已经到终点/满足条件) {
        记录答案 / 输出
        return
    }

    for (遍历所有可能的下一步) {
        if (下一步合法、没走过) {
            标记走过
            dfs(下一步)
            取消标记（回溯）
        }
    }
}
```

拆成 4 步：
1. **出口**：什么时候停止递归
2. **枚举**：所有能走的方向/选择
3. **判断**：能不能走（不越界、没访问过）
4. **递归 + 回溯**：先走，再撤销

---

# 2. 最常见场景：二维网格 DFS（比如迷宫、岛屿）
给你一个网格，上下左右走，不走回头路。

### 思路
1. 越界直接 return
2. 走过/不能走 return
3. 标记当前为已走
4. 往 **上下左右** 四个方向 DFS
5. 不需要回溯（因为一般只需要遍历，不回头）

### 伪代码
```
dfs(grid, x, y) {
    if (x,y 越界 || 不能走 || 已走过)
        return

    标记已走过

    dfs(x-1, y)  // 上
    dfs(x+1, y)  // 下
    dfs(x, y-1)  // 左
    dfs(x, y+1)  // 右
}
```

---

# 3. 另一个常见场景：排列/组合（需要回溯）
比如：全排列、子集、选数组合。

### 思路
1. 选一个数
2. 递归进去
3. **退掉这个数（回溯）**
4. 继续选下一个

---

# 4. DFS 三大关键点（一定要记住）
1. **递归方向**：下一步能去哪
2. **访问标记**：别重复走
3. **递归出口**：防止死循环

---

# 5. 最简单对比
- BFS：**一圈一圈扩散**（像水波纹）
- DFS：**一条路走到头**（像走迷宫钻到底）
你想要的C++版本DFS模板我已经整理好了，包含**最通用的递归模板** + **二维网格（迷宫/岛屿）** + **排列组合（回溯）** 三个核心场景，代码可以直接复制运行，注释也写得很详细，新手能轻松看懂。

### 一、C++ DFS 通用递归模板（核心）
这个模板是所有DFS场景的基础，记住它就能应对绝大多数问题：
```cpp
#include <iostream>
#include <vector>
#include <unordered_set>
using namespace std;

// 通用DFS模板（以“从0开始选数，选k个数的组合”为例）
void dfs(
    int start,               // 当前选择的起点
    int k,                   // 需要选的总数
    vector<int>& path,       // 已选的路径（当前状态）
    vector<vector<int>>& res // 最终结果
) {
    // 1. 递归出口：路径长度等于k，找到一个有效组合
    if (path.size() == k) {
        res.push_back(path);
        return;
    }

    // 2. 枚举所有可能的下一步选择
    for (int i = start; i <= 9; ++i) { // 示例：选1-9的数，避免重复选
        // 3. 做出选择（标记已选）
        path.push_back(i);
        // 4. 递归深入：选下一个数（i+1避免重复）
        dfs(i + 1, k, path, res);
        // 5. 回溯：撤销选择（恢复状态）
        path.pop_back();
    }
}

// 测试：选2个数的所有组合（比如[1,2],[1,3]...[8,9]）
int main() {
    vector<vector<int>> res;
    vector<int> path;
    dfs(1, 2, path, res); // 从1开始选，选2个数

    // 输出结果
    for (auto& comb : res) {
        cout << "[" << comb[0] << "," << comb[1] << "] ";
    }
    return 0;
}
```

### 二、C++ 二维网格DFS（迷宫/岛屿问题，最常用）
比如“统计岛屿数量”“找迷宫出口”，核心是上下左右遍历：
```cpp
#include <iostream>
#include <vector>
using namespace std;

// 方向数组：上、下、左、右（核心！简化四个方向的遍历）
int dirs[4][2] = {{-1,0}, {1,0}, {0,-1}, {0,1}};

// grid：网格（1=陆地，0=海水），x/y：当前坐标，visited：标记是否访问过
void dfs_grid(vector<vector<int>>& grid, int x, int y, vector<vector<bool>>& visited) {
    int m = grid.size();    // 网格行数
    int n = grid[0].size(); // 网格列数

    // 1. 递归出口：越界 / 不是陆地 / 已访问过
    if (x < 0 || x >= m || y < 0 || y >= n || grid[x][y] == 0 || visited[x][y]) {
        return;
    }

    // 2. 标记当前位置已访问（避免重复遍历）
    visited[x][y] = true;

    // 3. 遍历四个方向
    for (auto& dir : dirs) {
        int nx = x + dir[0]; // 新x坐标
        int ny = y + dir[1]; // 新y坐标
        dfs_grid(grid, nx, ny, visited); // 递归访问相邻位置
    }
    // 注：岛屿问题不需要回溯，因为只需要“遍历一次”，不用恢复状态
}

// 测试：统计岛屿数量
int main() {
    // 示例网格：3行4列，2个岛屿
    vector<vector<int>> grid = {
        {1,1,0,0},
        {1,0,0,0},
        {0,0,1,1}
    };
    int m = grid.size(), n = grid[0].size();
    vector<vector<bool>> visited(m, vector<bool>(n, false));
    int count = 0;

    // 遍历每个网格点
    for (int i = 0; i < m; ++i) {
        for (int j = 0; j < n; ++j) {
            // 遇到未访问的陆地，启动DFS，岛屿数+1
            if (grid[i][j] == 1 && !visited[i][j]) {
                count++;
                dfs_grid(grid, i, j, visited);
            }
        }
    }

    cout << "岛屿数量：" << count << endl; // 输出：2
    return 0;
}
```

### 三、关键代码解释
1. **递归出口**：所有DFS的第一步必须先写出口，避免死循环（比如越界、满足结果条件）。
2. **方向数组**：二维网格必备，用`dirs`把“上下左右”四个方向简化成循环，避免重复写4行代码。
3. **回溯操作**：`path.pop_back()`是核心——递归返回后撤销选择，让路径回到之前的状态，才能尝试下一个选择。
4. **访问标记**：`visited`数组（或`unordered_set`）用来记录已走过的位置，避免重复遍历（比如迷宫走回头路）。

### 总结
1. DFS核心逻辑：**出口判断 → 枚举选择 → 做出选择 → 递归深入 → 回溯撤销**。
2. 二维网格DFS重点：方向数组+越界判断，无需回溯（只遍历一次）。
3. 排列/组合DFS重点：回溯操作（恢复状态），避免重复组合。

如果需要针对某道具体的DFS题目（比如全排列、迷宫找路）讲解，我可以把代码改成对应题目版本，带你一步步分析执行过程。
---
