# 回溯法套路

> 抽象的看，解决一个回溯问题，实际上就是遍历一棵决策树的过程，树的每个叶子节点存放着一个合法答案。将整棵树遍历一遍，把叶子节点上的答案都收集起来，就可以得到所有的合法答案。

站在回溯树的一个节点上，只需要考虑三个问题：

1. 路径：就是已经做出的选择；
2. 选择列表：就是当前可以做的选择；
3. 结束条件：就是达到决策树底层，无法再做选择的条件。

回溯算法的框架：

```cpp
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

> [!note]
> 核心就是 for 循环里面的递归，在递归调用之前“做选择”，在递归调用以后“撤销选择”。

上面描述的各种抽象概念，接下来用全排列问题来解析：

## [全排列](https://leetcode.cn/problems/permutations/)

> [!question]
> 给定一个不含重复数字的数组 `nums` ，返回其 _所有可能的全排列_ 。你可以 **按任意顺序** 返回答案。

很容易画出这样的回溯树：

![[Pasted image 20260311091547.png]]

只要从根遍历这棵树，记录路径上的数字，其实就是所有的全排列。**我们不妨把这棵树称为回溯算法的「决策树」**。

**为啥说这是决策树呢，因为在每个节点上其实都在做决策**。比如说站在下图的红色节点上：

![[Pasted image 20260311091654.png]]

此时就在做决策：可以选择 1 那条树枝，也可以选择 3 那条树枝。因为 2 在身后，已经做过选择了，而全排列是不允许有重复数字的。所以：

> [2]就是“路径”，记录已经做过的选择；[1,2]就是“选择列表”，表示当前可以做出的选择；“结束条件”就是遍历到树的底层叶子节点，这里也就是选择列表为空的时候。

**我们定义的 `backtrack` 函数其实就像一个指针，在这棵树上游走，同时要正确维护每个节点的属性，每当走到树的底层叶子节点，其「路径」就是一个全排列**。

在 [[day12 基础二叉树]]中写过，各种搜索问题其实都是树的遍历问题。而多叉树的遍历框架是这样：

```cpp
void traverse(TreeNode* root) {
    for (TreeNode* child : root->childern) {
        // 前序位置需要的操作
        traverse(child);
        // 后序位置需要的操作
    }
}
```

**前序遍历的代码在进入某一个节点之前的那个时间点执行，后序遍历代码在离开某个节点之后的那个时间点执行**。

回想我们刚才说的，「路径」和「选择」是每个节点的属性，函数在树上游走要正确处理节点的属性，那么就要在这两个特殊时间点搞点动作：

![[Pasted image 20260311092526.png]]

所以再回看这个框架：

```cpp
for 选择 in 选择列表:
    # 做选择
    将该选择从选择列表移除
    路径.add(选择)
    backtrack(路径, 选择列表)
    # 撤销选择
    路径.remove(选择)
    将该选择再加入选择列表
```

**我们只要在递归之前做出选择，在递归之后撤销刚才的选择**，就能正确得到每个节点的选择列表和路径。

下面，直接看全排列代码：

```cpp
#include <vector>
#include <list>

class Solution {

    vector<vector<int>> res;

public:
    // 主函数，输入一组不重复的数字，返回它们的全排列
    vector<vector<int>> permute(vector<int>& nums) {
        // 记录「路径」
        list<int> track;
        // 「路径」中的元素会被标记为 true，避免重复使用
        vector<bool> used(nums.size(), false);

        backtrack(nums, track, used);
        return res;
    }

private:
    // 路径：记录在 track 中
    // 选择列表：nums 中不存在于 track 的那些元素（used[i] 为 false）
    // 结束条件：nums 中的元素全都在 track 中出现
    void backtrack(const vector<int>& nums, list<int>& track, vector<bool>& used) {
        // 触发结束条件
        if (track.size() == nums.size()) {
            res.push_back(vector<int>(track.begin(), track.end()));
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            // 排除不合法的选择
            if (used[i]) {
                // nums[i] 已经在 track 中，跳过
                continue;
            }
            // 做选择
            track.push_back(nums[i]);
            used[i] = true;
            // 进入下一层决策树
            backtrack(nums, track, used);
            // 取消选择
            track.pop_back();
            used[i] = false;
        }
    }
};
```

我们这里稍微做了些变通，没有显式记录「选择列表」，而是通过 `used` 数组排除已经存在 `track` 中的元素，从而推导出当前的选择列表。至此，我们就通过全排列问题详解了回溯算法的底层原理。

但是必须说明的是，不管怎么优化，都符合回溯框架，而且时间复杂度都不可能低于 O(N!)，因为穷举整棵决策树是无法避免的，你最后肯定要穷举出 N! 种全排列结果。

**这也是回溯算法的一个特点，不像动态规划存在重叠子问题可以优化，回溯算法就是纯暴力穷举，复杂度一般都很高**。

## 总结

回溯算法就是个多叉树的遍历问题，关键就是在前序遍历和后序遍历的位置做一些操作，算法框架如下：

```cpp
def backtrack(...):
    for 选择 in 选择列表:
        做选择
        backtrack(...)
        撤销选择
```

**写 `backtrack` 函数时，需要维护走过的「路径」和当前可以做的「选择列表」，当触发「结束条件」时，将「路径」记入结果集**。

其实想想看，回溯算法和动态规划是不是有点像呢？我们在动态规划系列文章中多次强调，动态规划的三个需要明确的点就是「状态」「选择」和「base case」，是不是就对应着走过的「路径」，当前的「选择列表」和「结束条件」？

尽管动态规划和回溯算法底层都把问题抽象成了树的结构，但这两种算法在思路上是完全不同的。

# 数独与 n 皇后问题

选择这两个问题，主要是它们的解法思路非常相似，而且它们都是回溯算法在实际生活中的有趣应用。

## [解数独](https://leetcode.cn/problems/sudoku-solver/)

> [!question]
> 编写一个程序，通过填充空格来解决数独问题。
> 数独的解法需 **遵循如下规则**：
> 1. 数字 `1-9` 在每一行只能出现一次。
> 2. 数字 `1-9` 在每一列只能出现一次。
> 3. 数字 `1-9` 在每一个以粗实线分隔的 `3x3` 宫内只能出现一次。（请参考示例图）
> 
数独部分空格内已填入了数字，空白格用 `'.'` 表示。

函数签名：

```cpp
void solveSudoku(vector<vector<char>>& board);
```

三个问题：

1. 数独游戏是二维数组，如何映射到一维数组呢？
2. $9^{81}$ 是一个非常大的数字，穷举那么多节点，即便写出回溯代码，也会超时吧？
3. 数独游戏需要一个可行解即可，所以当我们遇到一个可行解，就要及时结束代码，减少时间复杂度

对于一个 `MxN` 的二维数组，可以把二维坐标 `(i, j)` 映射到一维坐标 `index = i * N + j`，反之可以通过 `i = index / N` 和 `j = index % N` 得到原来的二维坐标。

有了这个编解码算法，数独游戏的 $9\times 9$ 二维数组就可以在逻辑上看做一个长度为 81 的一维数组，然后按照一维数组的递归树来穷举即可。

**对于第二个问题**，$9^{81}$ 只是一个非常粗略的估算，实际写代码时，要考虑数独的规则。

首先，初始棋盘会给你预先填充一些数字，这些数字是不能改变的，所以这些格子不用穷举，应该从 81 个格子去除。

另外，数独规则要求每行、每列和每个 $3\times 3$ 的九宫格内数字都不能重复，所以实际上每个空格子可以填入的数字并不是真的有 9 种选择，而是要根据当前棋盘的状态，去除不合法的数字（剪枝）。

而且，$9^{81}$ 是穷举所有可行解的复杂度估算，实际上题目只要求我们寻找一个可行解，所以只要找到一个可行解就可以结束算法了，没必要穷举所有解。

讲到这里，我们应该可以发现一个有趣的现象：

按照人类的一般理解，规则越多，问题越难；但是对于计算机来说，规则越多，问题反而越简单。

因为它要穷举嘛，规则越多，越容易剪枝，搜索空间就越小，就越容易找到答案。反之，如果没有任何规则，那就是纯暴力穷举，效率会非常低。

我们对回溯算法的剪枝优化，本质上就是寻找规则，提前排除不合法的选择，从而提高穷举的效率。

第三个问题，可以使用一个全局变量来标记是否找到可行解，辅助算法提前终止。

代码如下：

```cpp
class Solution {
    // 标记是否已经找到可行解
    bool found = false;

public:
    void solveSudoku(vector<vector<char>>& board) {
        backtrack(board, 0);
    }

private:
    // 路径：board 中小于 index 的位置所填的数字
    // 选择列表：数字 1~9
    // 结束条件：整个 board 都填满数字
    void backtrack(vector<vector<char>>& board, int index) {
        int m = 9, n = 9;
        int i = index / n, j = index % n;

        if (found) {
            // 已经找到一个可行解，立即结束
            return;
        }

        if (index == m * n) {
            // 找到一个可行解，触发 base case
            found = true;
            return;
        }

        if (board[i][j] != '.') {
            // 如果有预设数字，不用我们穷举
            backtrack(board, index + 1);
            return;
        }

        for (char ch = '1'; ch <= '9'; ch++) {
            // 剪枝：如果遇到不合法的数字，就跳过
            if (!isValid(board, i, j, ch))
                continue;

            // 做选择
            board[i][j] = ch;

            backtrack(board, index + 1);
            if (found) {
                // 如果找到一个可行解，立即结束
                // 不要撤销选择，否则 board[i][j] 会被重置为 '.'
                return;
            }

            // 撤销选择
            board[i][j] = '.';
        }
    }

    // 判断是否可以在 (r, c) 位置放置数字 num
    bool isValid(vector<vector<char>>& board, int r, int c, char num) {
        for (int i = 0; i < 9; i++) {
            // 判断行是否存在重复
            if (board[r][i] == num) return false;
            // 判断列是否存在重复
            if (board[i][c] == num) return false;
            // 判断 3 x 3 方框是否存在重复
            if (board[(r/3)*3 + i/3][(c/3)*3 + i%3] == num)
                return false;
        }
        return true;
    }
};
```

还是再回头看看回溯框架，会有更深的感触：

```cpp
result = []
def backtrack(路径, 选择列表):
    if 满足结束条件:
        result.add(路径)
        return
    
    for 选择 in 选择列表:
        做选择
        backtrack(路径, 选择列表)
        撤销选择
```

## [N 皇后](https://leetcode.cn/problems/n-queens/)

---
按照国际象棋的规则，皇后可以攻击与之处在同一行或同一列或同一斜线上的棋子。

**n 皇后问题** 研究的是如何将 `n` 个皇后放置在 `n×n` 的棋盘上，并且使皇后彼此之间不能相互攻击。

给你一个整数 `n` ，返回所有不同的 **n 皇后问题** 的解决方案。

每一种解法包含一个不同的 **n 皇后问题** 的棋子放置方案，该方案中 `'Q'` 和 `'.'` 分别代表了皇后和空位。

**示例 1：**

![](https://assets.leetcode.com/uploads/2020/11/13/queens.jpg)

**输入：** n = 4
**输出：** [\[".Q..","...Q","Q...","..Q."],["..Q.","Q...","...Q",".Q.."]]
**解释：** 如上图所示，4 皇后问题存在两个不同的解法。

**示例 2：**

**输入：** n = 1
**输出：** [\["Q"]]

---

函数签名：

```cpp
vector<vector<string>> solveNQueens(int n);
```

其实 N 皇后问题和数独游戏的解法思路是一样的，具体有以下差别：

1、数独的规则和 N 皇后问题的规则不同，我们需要修改 `isValid` 函数，判断一个位置是否可以放置皇后。

2、题目要求找到 N 皇后问题所有解，而不是仅仅找到一个解。我们需要去除算法提前终止的逻辑，让算法穷举并记录所有的合法解。

这两个问题都比较容易解决，按理说可以直接参照数独游戏的代码实现 N 皇后问题。

但 N 皇后问题相对数独游戏还有一个优化：**我们可以以行为单位进行穷举，而不是像数独游戏那样以格子为单位穷举**。

举个直观的例子，在数独游戏中，如果我们设置 `board[i][j] = 1`，接下来呢，要去穷举 `board[i][j+1]` 了对吧？而对于 N 皇后问题，我们知道每行必然有且只有一个皇后，所以如果我们决定在 `board[i]` 这一行的某一列放置皇后，那么接下来就不用管 `board[i]` 这一行了，应该考虑 `board[i+1]` 这一行的皇后要放在哪里。

所以 N 皇后问题的穷举对象是棋盘中的行，每一行都持有一个皇后，可以选择把皇后放在该行的任意一列。

接下来直接看代码和注释吧：

```cpp
#include <vector>
#include <string>

class Solution {
public:
    // 输入棋盘边长 n，返回所有合法的放置
    vector<vector<string>> solveNQueens(int n) {
        // '.' 表示空，'Q' 表示皇后，初始化空棋盘。
        vector<string> board(n, string(n, '.'));
        backtrack(board, 0);
        return res;
    }

private:
    vector<vector<string>> res;

    // 路径：board 中小于 row 的那些行都已经成功放置了皇后
    // 选择列表：第 row 行的所有列都是放置皇后的选择
    // 结束条件：row 超过 board 的最后一行
    void backtrack(vector<string>& board, int row) {
        // 触发结束条件
        if (row == board.size()) {
            res.push_back(board);
            return;
        }

        int n = board[row].size();
        for (int col = 0; col < n; col++) {
            // 排除不合法选择
            if (!isValid(board, row, col)) {
                continue;
            }
            // 做选择
            board[row][col] = 'Q';
            // 进入下一行决策
            backtrack(board, row + 1);
            // 撤销选择
            board[row][col] = '.';
        }
    }

    // 是否可以在 board[row][col] 放置皇后？
    bool isValid(const vector<string>& board, int row, int col) {
        int n = board.size();
        // 检查列是否有皇后互相冲突
        for (int i = 0; i < row; i++) {
            if (board[i][col] == 'Q')
                return false;
        }
        // 检查右上方是否有皇后互相冲突
        for (int i = row - 1, j = col + 1; i >= 0 && j < n; i--, j++) {
            if (board[i][j] == 'Q')
                return false;
        }
        // 检查左上方是否有皇后互相冲突
        for (int i = row - 1, j = col - 1; i >= 0 && j >= 0; i--, j--) {
            if (board[i][j] == 'Q')
                return false;
        }
        return true;
    }
};
```

