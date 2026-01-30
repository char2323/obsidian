# 滑动窗口

滑动窗口本质上可以归类为快慢指针，一快一慢的指针中间夹的就是所谓的窗口。滑动窗口技巧主要用于处理子数组问题，比如**寻找某个符合条件的最长、最短子数组**。

暴力解也简单，使用两个 `for` 循环枚举即可：

```cpp
for (int i = 0; i < nums.length; i++) {
    for (int j = i; j < nums.length; j++) {
        // nums[i, j] 是一个子数组
    }
}
```

滑动窗口的代码框架也不难，使用两个 `while` 来实现窗口滑动：

```cpp
// 索引区间 [left, right) 是窗口
int left = 0, right = 0;

while (right < nums.size()) {
    // 增大窗口
    window.addLast(nums[right]);
    right++;
    
    while (window needs shrink) {
        // 缩小窗口
        window.removeFirst(nums[left]);
        left++;
    }
}
```

多数相关问题都是字符串，以字符串为例写一个模板：

```cpp
// 滑动窗口算法伪码框架
void slidingWindow(string s) {
    // 用合适的数据结构记录窗口中的数据，根据具体场景变通
    // 比如说，我想记录窗口中元素出现的次数，就用 map
    // 如果我想记录窗口中的元素和，就可以只用一个 int
    auto window = ...

    int left = 0, right = 0;
    while (right < s.size()) {
        // c 是将移入窗口的字符
        char c = s[right];
        window.add(c);
        // 增大窗口
        right++;

        // 进行窗口内数据的一系列更新
        ...

        // *** debug 输出的位置 ***
        printf("window: [%d, %d)\n", left, right);
        // 注意在最终的解法代码中不要 print
        // 因为 IO 操作很耗时，可能导致超时

        // 判断左侧窗口是否要收缩
        while (window needs shrink) {
            // d 是将移出窗口的字符
            char d = s[left];
            window.remove(d);
            // 缩小窗口
            left++;

            // 进行窗口内数据的一系列更新
            ...
        }
    }
}
```

框架简单，但需要考虑三个问题：
1. 什么时候需要移动 `right` 来扩大窗口？窗口扩大以后数据要如何更新？
2. 什么时候需要移动 `left` 来缩小窗口？窗口缩小以后数据要如何更新？
3. 什么时候应该更新结果？

## [最小覆盖子串](https://leetcode.cn/problems/minimum-window-substring/description/)

> [!question]
> 给你一个字符串 `s` 、一个字符串 `t` 。返回 `s` 中涵盖 `t` 所有字符的最小子串。如果 `s` 中不存在涵盖 `t` 所有字符的子串，则返回空字符串 `""` 。
> **注意：**
> 
> - 对于 `t` 中重复字符，我们寻找的子字符串中该字符数量必须不少于 `t` 中该字符数量。
> - 如果 `s` 中存在这样的子串，我们保证它是唯一的答案。
> 
> **示例 1：**
> 
> **输入：** s = "ADOBECODEBANC", t = "ABC"
> **输出：**"BANC"
> **解释：** 最小覆盖子串 "BANC" 包含来自字符串 t 的 'A'、'B' 和 'C'。
> 
> **示例 2：**
> 
> **输入：** s = "a", t = "a"
> **输出：**"a"
> **解释：** 整个字符串 s 是最小覆盖子串。
> 
> **示例 3:**
> 
> **输入:** s = "a", t = "aa"
> **输出:** ""
> **解释:** t 中两个字符 'a' 均应包含在 s 的子串中，
> 因此没有符合条件的子字符串，返回空字符串。
> 
> **提示：**
> 
> - `m == s.length`
> - `n == t.length`
> - `1 <= m, n <= 105`
> - `s` 和 `t` 由英文字母组成
> 
> **进阶：** 你能设计一个在 `o(m+n)` 时间内解决此问题的算法吗？


```cpp
class Solution {
public:
    string minWindow(string s, string t) {
        unordered_map<char, int> need, window;
        for (char c : t) {
            need[c]++;
        }

        int left = 0, right = 0;
        // 记录window中的字符满足need条件的字符个数
        int valid = 0;
        // 记录最小覆盖子串的起始索引及长度
        int start = 0, len = INT_MAX;
        while (right < s.size()) {
            // c 是将移入窗口的字符
            char c = s[right];
            // 扩大窗口
            right++;
            // 进行窗口内数据的一系列更新
            if (need.count(c)) {
                window[c]++;
                if (window[c] == need[c])
                    valid++;
            }

            // 判断左侧窗口是否要收缩
            while (valid == need.size()) { /**<extend down -200>![](/images/algo/slidingwindow/2.png) */
                // 在这里更新最小覆盖子串
                if (right - left < len) {
                    start = left;
                    len = right - left;
                }
                // d 是将移出窗口的字符
                char d = s[left];
                // 缩小窗口
                left++;
                // 进行窗口内数据的一系列更新
                if (need.count(d)) {
                    if (window[d] == need[d])
                        valid--;
                    window[d]--;
                }
            } /**<extend up -50>![](/images/algo/slidingwindow/4.png) */
        }
        // 返回最小覆盖子串
        return len == INT_MAX ? "" : s.substr(start, len);
    }
};
```

## [字符串排列](https://leetcode.cn/problems/permutation-in-string/)

> [!question]
> 给你两个字符串 `s1` 和 `s2` ，写一个函数来判断 `s2` 是否包含 `s1` 的排列。如果是，返回 `true` ；否则，返回 `false` 。
> 换句话说，`s1` 的排列之一是 `s2` 的 **子串** 。
> 
> **示例 1：**
> 
> **输入：** s1 = "ab" s2 = "eidbaooo"
> **输出：** true
> **解释：** s2 包含 s1 的排列之一 ("ba").
> 
> **示例 2：**
> 
> **输入：** s1= "ab" s2 = "eidboaoo"
> **输出：** false
> 
> **提示：**
> 
> - `1 <= s1.length, s2.length <= 104`
> - `s1` 和 `s2` 仅包含小写字母

```cpp
class Solution {
public:
    // 判断 s 中是否存在 t 的排列
    bool checkInclusion(string t, string s) {
        unordered_map<char, int> need, window;
        for (char c : t) need[c]++;

        int left = 0, right = 0;
        int valid = 0;
        while (right < s.size()) {
            char c = s[right];
            right++;
            // 进行窗口内数据的一系列更新
            if (need.count(c)) {
                window[c]++;
                if (window[c] == need[c])
                    valid++;
            }

            // 判断左侧窗口是否要收缩
            while (right - left >= t.size()) {
                // 在这里判断是否找到了合法的子串
                if (valid == need.size())
                    return true;
                char d = s[left];
                left++;
                // 进行窗口内数据的一系列更新
                if (need.count(d)) {
                    if (window[d] == need[d])
                        valid--;
                    window[d]--;
                }
            }
        }
        // 未找到符合条件的子串
        return false;
    }
};
```

## [找所有字母异位词](https://leetcode.cn/problems/find-all-anagrams-in-a-string/)

> [!question]
> 给定两个字符串 `s` 和 `p`，找到 `s` 中所有 `p` 的 **异位词** 的子串，返回这些子串的起始索引。不考虑答案输出的顺序。
> **异位词** 指由相同字母重排列形成的字符串（包括相同的字符串）。
> 
> **示例 1:**
> 
> **输入:** s = "cbaebabacd", p = "abc"
> **输出:** [0,6]
> **解释:**
> 起始索引等于 0 的子串是 "cba", 它是 "abc" 的异位词。
> 起始索引等于 6 的子串是 "bac", 它是 "abc" 的异位词。
> 
> **示例 2:**
> 
> **输入:** s = "abab", p = "ab"
> **输出:** [0,1,2]
> **解释:**
> 起始索引等于 0 的子串是 "ab", 它是 "ab" 的异位词。
> 起始索引等于 1 的子串是 "ba", 它是 "ab" 的异位词。
> 起始索引等于 2 的子串是 "ab", 它是 "ab" 的异位词。
> 
> **提示:**
> 
> - `1 <= s.length, p.length <= 3 * 104`
> - `s` 和 `p` 仅包含小写字母

```cpp
#include <vector>
#include <string>
#include <unordered_map>

using namespace std;

class Solution {
public:
    vector<int> findAnagrams(string s, string t) {
        unordered_map<char, int> need;
        unordered_map<char, int> window;
        for (char c : t) {
            need[c]++;
        }

        int left = 0, right = 0;
        int valid = 0;
        // 记录结果
        vector<int> res;
        while (right < s.size()) {
            char c = s[right];
            right++;
            // 进行窗口内数据的一系列更新
            if (need.count(c)) {
                window[c]++;
                if (window[c] == need[c]) {
                    valid++;
                }
            }
            // 判断左侧窗口是否要收缩
            while (right - left >= t.size()) {
                // 当窗口符合条件时，把起始索引加入 res
                if (valid == need.size()) {
                    res.push_back(left);
                }
                char d = s[left];
                left++;
                // 进行窗口内数据的一系列更新
                if (need.count(d)) {
                    if (window[d] == need[d]) {
                        valid--;
                    }
                    window[d]--;
                }
            }
        }
        return res;
    }
};
```

## [最长无重复子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

> [!question]
> 给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长子串** 的长度。
> 
**示例 1:**
> 
> **输入:** s = "abcabcbb"
> **输出:** 3 
> **解释:** 因为无重复字符的最长子串是 `"abc"`，所以其长度为 3。
> 
> **示例 2:**
> 
> **输入:** s = "bbbbb"
> **输出:** 1
> **解释:** 因为无重复字符的最长子串是 `"b"`，所以其长度为 1。
> 
> **示例 3:**
> 
> **输入:** s = "pwwkew"
> **输出:** 3
> **解释:** 因为无重复字符的最长子串是 `"wke"`，所以其长度为 3。
>      请注意，你的答案必须是 **子串** 的长度，`"pwke"` 是一个_子序列，_不是子串。
> 
> **提示：**
> 
> - `0 <= s.length <= 5 * 104`
> - `s` 由英文字母、数字、符号和空格组成
> 
> 

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_map<char, int> window;

        int left = 0, right = 0;
        // 记录结果
        int res = 0;
        while (right < s.size()) {
            char c = s[right];
            right++;
            // 进行窗口内数据的一系列更新
            window[c]++;
            // 判断左侧窗口是否要收缩
            while (window[c] > 1) {
                char d = s[left];
                left++;
                // 进行窗口内数据的一系列更新
                window[d]--;
            }
            // 在这里更新答案
            res = max(res, right - left);
        }
        return res;
    }
};
```

## [长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)

> [!question]
> 给定一个字符串 `s` ，请你找出其中不含有重复字符的 **最长**的长度。
> 
> **示例 1:**
> 
> **输入:** s = "abcabcbb"
> **输出:** 3 
> **解释:** 因为无重复字符的最长子串是 `"abc"`，所以其长度为 3。注意 "bca" 和 "cab" 也是正确答案。
> 
> **示例 2:**
> 
> **输入:** s = "bbbbb"
> **输出:** 1
> **解释:** 因为无重复字符的最长子串是 `"b"`，所以其长度为 1。
> 
> **示例 3:**
> 
> **输入:** s = "pwwkew"
> **输出:** 3
> **解释:** 因为无重复字符的最长子串是 `"wke"`，所以其长度为 3。
>      请注意，你的答案必须是 **子串** 的长度，`"pwke"` 是一个_子序列，_不是子串。
> 
> **提示：**
> 
> - `0 <= s.length <= 5 * 104`
> - `s` 由英文字母、数字、符号和空格组成

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left = 0, right = 0;
        // 维护窗口内元素之和
        int windowSum = 0;
        int res = INT_MAX;

        while (right < nums.size()) {
            // 扩大窗口
            windowSum += nums[right];
            right++;
            while (windowSum >= target && left < right) {
                // 已经达到 target，缩小窗口，同时更新答案
                res = min(res, right - left);
                windowSum -= nums[left];
                left++;
            }
        }
        return res == INT_MAX ? 0 : res;
    }
};
```

问题大同小异，核心还是那三个问题：
1. 什么时候应该扩大窗口？
2. 什么时候应该缩小窗口？
3. 什么时候应该更新答案？
弄清楚这三个问题，在加上上面的框架，就没有什么问题了。

# 矩阵的螺旋遍历

## 矩阵旋转

先考虑一下简单的矩阵旋转：顺时针旋转九十度和逆时针旋转九十度。常规的思路就是去寻找原始坐标和旋转后坐标的映射规律，但我们可以让思维跳跃跳跃，尝试把矩阵进行反转、镜像对称等操作，可能会出现新的突破口。

我们可以先将 `n x n` 矩阵 `matrix` 按照左上到右下的对角线进行镜像对称，然后再对矩阵的每一行进行反转，所得结果就是顺时针旋转九十度所得的矩阵。逆时针就用另一条对角线，即右上到左下的对角线来镜像然后逐行反转即可，但代码要更加烧脑一些：

```cpp
class Solution {
public:
    // 将二维矩阵原地顺时针旋转 90 度
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // 先沿对角线镜像对称二维矩阵
        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                // swap(matrix[i][j], matrix[j][i]);
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }
        // 然后反转二维矩阵的每一行
        for(auto &row : matrix) {
            reverse(row.begin(), row.end());
        }
    }
	// 将二维矩阵原地逆时针旋转 90 度
    void rotate2(vector<vector<int>>& matrix) {
        int n = matrix.size();
        // 沿左下到右上的对角线镜像对称二维矩阵
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n - i; j++) {
                // swap(matrix[i][j], matrix[n-j-1][n-i-1])
                int temp = matrix[i][j];
                matrix[i][j] = matrix[n - j - 1][n - i - 1];
                matrix[n - j - 1][n - i - 1] = temp;
            }
        }
        // 然后反转二维矩阵的每一行
        for (auto& row : matrix) {
            reverse(row.begin(), row.end());
        }
    }
};
```

## [螺旋矩阵](https://leetcode.cn/problems/spiral-matrix/)

代码和思路都比较简单，但初次接触不好想，可以积累下来。

> [!question]
> 给你一个 `m` 行 `n` 列的矩阵 `matrix` ，请按照 **顺时针螺旋顺序** ，返回矩阵中的所有元素。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2020/11/13/spiral1.jpg)
> 
> **输入：**matrix = [[1,2,3],[4,5,6],[7,8,9]]
> **输出：**[1,2,3,6,9,8,7,4,5]
> 
> **示例 2：**
> 
> ![](https://assets.leetcode.com/uploads/2020/11/13/spiral.jpg)
> 
> **输入：**matrix = [[1,2,3,4],[5,6,7,8],[9,10,11,12]]
> **输出：**[1,2,3,4,8,12,11,10,9,5,6,7]
> 
> **提示：**
> 
> - `m == matrix.length`
> - `n == matrix[i].length`
> - `1 <= m, n <= 10`
> - `-100 <= matrix[i][j] <= 100`

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        if (matrix.empty()) return {};

        int m = matrix.size();    // 行数
        int n = matrix[0].size(); // 列数
        vector<int> res;

        // 定义四个边界
        int upper = 0, lower = m - 1;
        int left = 0, right = n - 1;

        while (true) {
            // 1. 从左向右遍历顶部
            for (int j = left; j <= right; ++j) res.push_back(matrix[upper][j]);
            // 走完后上边界下移，若超过下边界则结束
            if (++upper > lower) break;

            // 2. 从上向下遍历右侧
            for (int i = upper; i <= lower; ++i) res.push_back(matrix[i][right]);
            // 走完后右边界左移，若小于左边界则结束
            if (--right < left) break;

            // 3. 从右向左遍历底部
            for (int j = right; j >= left; --j) res.push_back(matrix[lower][j]);
            // 走完后下边界上移，若小于上边界则结束
            if (--lower < upper) break;

            // 4. 从下向上遍历左侧
            for (int i = lower; i >= upper; --i) res.push_back(matrix[i][left]);
            // 走完后左边界右移，若超过右边界则结束
            if (++left > right) break;
        }

        return res;
    }
};
```

## [螺旋矩阵 II](https://leetcode.cn/problems/spiral-matrix-ii/)

> [!question]
> 给你一个正整数 `n` ，生成一个包含 `1` 到 `n2` 所有元素，且元素按顺时针顺序螺旋排列的 `n x n` 正方形矩阵 `matrix` 。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2020/11/13/spiraln.jpg)
> 
> **输入：**n = 3
> **输出：**[[1,2,3],[8,9,4],[7,6,5]]
> 
> **示例 2：**
> 
> **输入：**n = 1
> **输出：**[[1]]
> 
> **提示：**
> 
> - `1 <= n <= 20`

```cpp
#include <vector>
using namespace std;

class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        // 1. 初始化一个 n x n 的二维 vector，初始值填 0
        vector<vector<int>> matrix(n, vector<int>(n, 0));
        
        // 2. 定义边界和计数器
        int upper = 0, lower = n - 1;
        int left = 0, right = n - 1;
        int num = 1; // 从 1 开始填
        int target = n * n; // 填到 n^2 为止

        while (num <= target) {
            // 顶部：从左向右
            for (int j = left; j <= right; j++) {
                matrix[upper][j] = num++;
            }
            if (++upper > lower) break;

            // 右侧：从上向下
            for (int i = upper; i <= lower; i++) {
                matrix[i][right] = num++;
            }
            if (--right < left) break;

            // 底部：从右向左
            for (int j = right; j >= left; j--) {
                matrix[lower][j] = num++;
            }
            if (--lower < upper) break;

            // 左侧：从下向上
            for (int i = lower; i >= upper; i--) {
                matrix[i][left] = num++;
            }
            if (++left > right) break;
        }

        return matrix;
    }
};
```

# 前缀和

前缀和的思想是重复利用计算过的子数组之和，从而降低区间查询需要累加计算的次数——其实就是计算过的东西先存起来，有点类似于动态规划、、

## [区间和](https://kamacoder.com/problempage.php?pid=1070)

> [!question]
> **题目描述**
> 给定一个整数数组 Array，请计算该数组在每个指定区间内元素的总和。
> 
> **输入描述**
> 第一行输入为整数数组 Array 的长度 n，接下来 n 行，每行一个整数，表示数组的元素。随后的输入为需要计算总和的区间下标：a，b （b > = a），直至文件结束。
> 
> **输出描述**
> 输出每个指定区间内元素的总和。
> 
> **输入示例**
> ```
> 5
> 1
> 2
> 3
> 4
> 5
> 0 1
> 1 3
> ```
> 
> **输出示例**
> ```
> 3
> 9
> ```
> 
> **提示信息**
> 数据范围：  
> 0 < n <= 100000
> 

还是比较简单的问题

```cpp
#include <iostream>
#include <vector>
int main() {
	int n, a, b;
	std::cin >> n;
	std::vector<int> vec(n);
	std::vector<int> sum(n);
	int presum = 0;
	for (int i = 0; i < n; i++) {
		std::cin >> vec[i];
		presum += vec[i];
		sum[i] = presum;
	}
	while (std::cin >> a >> b) {
		int sum1;
		if (a == 0)sum1 = sum[b];
		else
			sum1 = sum[b] - sum[a - 1];
		std::cout << sum1 << std::endl;
	}
}
```

## [开发商购买土地](https://kamacoder.com/problempage.php?pid=1044)

> [!question]
> **题目描述**
> 在一个城市区域内，被划分成了n * m个连续的区块，每个区块都拥有不同的权值，代表着其土地价值。目前，有两家开发公司，A 公司和 B 公司，希望购买这个城市区域的土地。 
> 现在，需要将这个城市区域的所有区块分配给 A 公司和 B 公司。
> 然而，由于城市规划的限制，只允许将区域按横向或纵向划分成两个子区域，而且每个子区域都必须包含一个或多个区块。 为了确保公平竞争，你需要找到一种分配方式，使得 A 公司和 B 公司各自的子区域内的土地总价值之差最小。 
> 注意：区块不可再分。
> 
> **输入描述**
> 第一行输入两个正整数，代表 n 和 m。 
> 接下来的 n 行，每行输出 m 个正整数。
> 
> **输出描述**
> 请输出一个整数，代表两个子区域内土地总价值之间的最小差距。
> 
> **输入示例**
> ```
> 3 3
> 1 2 3
> 2 1 3
> 1 2 3
> ```
> 
> **输出示例**
> ```
> 0
> ```
> 
> **提示信息**
> 如果将区域按照如下方式划分：
> 1 2 | 3  
> 2 1 | 3  
> 1 2 | 3 
> 两个子区域内土地总价值之间的最小差距可以达到 0。
> 
> **数据范围**：
> 1 <= n, m <= 100；  
> n 和 m 不同时为 1。

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <cmath>
#include <climits>

using namespace std;

int main() {
    int n, m;
    if (!(cin >> n >> m)) return 0;

    vector<vector<int>> matrix(n, vector<int>(m));
    long long totalSum = 0;

    // 1. 输入数据并计算总价值
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < m; j++) {
            cin >> matrix[i][j];
            totalSum += matrix[i][j];
        }
    }

    long long minDiff = LLONG_MAX;

    // 2. 尝试横向切（按行划分）
    long long horizontalPart = 0;
    for (int i = 0; i < n - 1; i++) { // n-1 保证切开后两边都不为空
        for (int j = 0; j < m; j++) {
            horizontalPart += matrix[i][j];
        }
        long long diff = abs(totalSum - 2 * horizontalPart);
        minDiff = min(minDiff, diff);
    }

    // 3. 尝试纵向切（按列划分）
    long long verticalPart = 0;
    for (int j = 0; j < m - 1; j++) { // m-1 保证切开后两边都不为空
        for (int i = 0; i < n; i++) {
            verticalPart += matrix[i][j];
        }
        long long diff = abs(totalSum - 2 * verticalPart);
        minDiff = min(minDiff, diff);
    }

    cout << minDiff << endl;

    return 0;
}
```

有一个核心公式：
$$
diff=∣(TotalSum−PartSum)−PartSum∣=∣TotalSum−2×PartSum∣
$$
不难看出：我们设有 $S,S_{A},S_{B}$，不难看出 $S_{B}=S-S_{A}$，于是 :
$$
diff=S_{A}-S_{B}=S_{A}-(S-S_{A})=2S_{A}-S
$$
加上绝对值则得到上式。


