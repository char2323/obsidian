# [四数相加 II](https://leetcode.cn/problems/4sum-ii/)

> [!question]
> 给你四个整数数组 `nums1`、`nums2`、`nums3` 和 `nums4` ，数组长度都是 `n` ，请你计算有多少个元组 `(i, j, k, l)` 能满足：
> 
> - `0 <= i, j, k, l < n`
> - `nums1[i] + nums2[j] + nums3[k] + nums4[l] == 0`
> 
> **示例 1：**
> 
> **输入：** nums1 = [1,2], nums2 = [-2,-1], nums3 = [-1,2], nums4 = [0,2]
> **输出：** 2
> **解释：**
> 两个元组如下：
> 1. (0, 0, 0, 1) -> nums1[0] + nums2[0] + nums3[0] + nums4[1] = 1 + (-2) + (-1) + 2 = 0
> 2. (1, 1, 0, 0) -> nums1[1] + nums2[1] + nums3[0] + nums4[0] = 2 + (-1) + (-1) + 0 = 0
> 
> **示例 2：**
> 
> **输入：** nums1 = [0], nums2 = [0], nums3 = [0], nums4 = [0]
> **输出：** 1
> 
> **提示：**
> 
> - `n == nums1.length`
> - `n == nums2.length`
> - `n == nums3.length`
> - `n == nums4.length`
> - `1 <= n <= 200`
> - `-228 <= nums1[i], nums2[i], nums3[i], nums4[i] <= 228`

## 核心思路：二二分组

我们可以把四个数组分成两组：

- **第一组**：数组 `A` 和 `B`。
- **第二组**：数组 `C` 和 `D`。

**第一步：** 算出 `A[i] + B[j]` 的所有可能结果，并记录每个结果出现的**次数**。存入一个哈希表中。

**第二步：** 遍历 `C` 和 `D`，算出 `C[k] + D[l]`。如果这个和的相反数（即 `-(C[k] + D[l])`）在哈希表里，就说明这四个数加起来等于 0！

- **暴力法**：$n \times n \times n \times n = n^4$
- **分组哈希法**：$n^2$（处理 A, B） + $n^2$（处理 C, D） = **$2n^2$**

从 $n^4$ 降到 $n^2$.

```C++
#include <unordered_map>
#include <vector>

class Solution {
public:
    int fourSumCount(std::vector<int>& nums1, std::vector<int>& nums2, 
                     std::vector<int>& nums3, std::vector<int>& nums4) {
        std::unordered_map<int, int> countAB;
        
        // 1. 统计 nums1 和 nums2 的所有两两相加之和
        for (int a : nums1) {
            for (int b : nums2) {
                countAB[a + b]++; // 记录这个和出现了几次
            }
        }
        
        int result = 0;
        // 2. 遍历 nums3 和 nums4
        for (int c : nums3) {
            for (int d : nums4) {
                int target = -(c + d);
                // 3. 看看之前 A+B 的结果里有没有 target
                if (countAB.find(target) != countAB.end()) {
                    result += countAB[target]; // 注意：是加上出现的次数
                }
            }
        }
        
        return result;
    }
};
```

## 为什么是 `result += countAB[target]`？

在“两数之和”中，我们只要找到一对就返回下标。但在这一题中，我们要统计的是**元组的个数**。

**举个例子：**

- 假设在 `A` 和 `B` 中，能凑出和为 `5` 的组合有 **3 种**。
- 现在在 `C` 和 `D` 中，你凑出了一个和为 `-5`。
- 那么这一个 `-5` 就可以和之前那 **3 种** `5` 分别配对，产生 **3 个**满足条件的四元组。
- 所以我们要累加的是 `countAB[target]` 的频率。

## 什么时候该用哈希表分组？

当你发现题目要求计算 $A+B+C+D = 0$ 这种平衡关系，且数组之间互相独立时，**“对半拆分”** 是最有效的策略。

在算法题里，除非你需要 key 是有序的，否则**请永远优先使用 `std::unordered_map`**。因为它的底层是哈希表，查询效率是 $O(1)$，而 `std::map` 是红黑树，查询效率是 $O(\log n)$。

# [赎金信](https://leetcode.cn/problems/ransom-note/)

> [!question]
> 给你两个字符串：`ransomNote` 和 `magazine` ，判断 `ransomNote` 能不能由 `magazine` 里面的字符构成。
> 
> 如果可以，返回 `true` ；否则返回 `false` 。
> 
> `magazine` 中的每个字符只能在 `ransomNote` 中使用一次。
> 
> **示例 1：**
> 
> **输入：** ransomNote = "a", magazine = "b"
> **输出：** false
> 
> **示例 2：**
> 
> **输入：** ransomNote = "aa", magazine = "ab"
> **输出：** false
> 
> **示例 3：**
> 
> **输入：** ransomNote = "aa", magazine = "aab"
> **输出：** true
> 
> **提示：**
> 
> - `1 <= ransomNote.length, magazine.length <= 105`
> - `ransomNote` 和 `magazine` 由小写英文字母组成
> 

既然“[[day6 哈希表1#^4fe338 |字母异位词]]”是要求两个计数器 **完全相等**，那么“赎金信”就是要求 **A 计数器里的每一项都不能超过 B 计数器**。

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        // 如果信比杂志还长，肯定写不出来
        if (ransomNote.length() > magazine.length()) return false;

        int record[26] = {0};

        // 1. 先把杂志里所有的字符“存入仓库”
        for (char c : magazine) {
            record[c - 'a']++;
        }

        // 2. 遍历信件，消耗仓库里的字符
        for (char c : ransomNote) {
            record[c - 'a']--;
            // 3. 如果某个字符减成了负数，说明杂志里的不够用了
            if (record[c - 'a'] < 0) {
                return false;
            }
        }

        return true;
    }
};
```

- **242 (异位词)**：要求 A=B。最后数组必须 **全为 0**。
- **383 (赎金信)**：要求 A⊆B。只要减完之后 **没有负数** 即可（仓库里剩下字符没关系）。
- **383 赎金信**：字符可以 **散落在任何地方**。只要杂志里有 3 个 'a'，信里用 3 个 'a'，管它们在哪都行。
- **567 [[day2 滑动窗口、矩阵的螺旋遍历、前缀和#^87710d|字符串排列]]**：要求字符必须是 **连续的**。也就是说，`s2` 中必须存在一个 **子串** 是 `s1` 的排列。

**解法升级**： 567 不能只靠一个简单的计数器了，它需要用到 **“滑动窗口 (Sliding Window)”** 技术。

- 需要维护一个和 `s1` 等长的窗口。
- 窗口在 `s2` 上向右滑动。
- 每次滑动时，进来一个字符，出去一个字符，动态更新计数器并判断。

# [三数之和](https://leetcode.cn/problems/3sum/)

> [!question]
> 给你一个整数数组 `nums` ，判断是否存在三元组 `[nums[i], nums[j], nums[k]]` 满足 `i != j`、`i != k` 且 `j != k` ，同时还满足 `nums[i] + nums[j] + nums[k] == 0` 。请你返回所有和为 `0` 且不重复的三元组。
> 
> **注意：** 答案中不可以包含重复的三元组。
> 
> **示例 1：**
> 
> **输入：** nums = [-1,0,1,2,-1,-4]
> **输出：** \[[-1,-1,2],[-1,0,1]\]
> **解释：**
> nums[0] + nums[1] + nums[2] = (-1) + 0 + 1 = 0 。
> nums[1] + nums[2] + nums[4] = 0 + 1 + (-1) = 0 。
> nums[0] + nums[3] + nums[4] = (-1) + 2 + (-1) = 0 。
> 不同的三元组是 [-1,0,1] 和 [-1,-1,2] 。
> 注意，输出的顺序和三元组的顺序并不重要。
> 
> **示例 2：**
> 
> **输入：** nums = [0,1,1]
> **输出：** []
> **解释：** 唯一可能的三元组和不为 0 。
> 
> **示例 3：**
> 
> **输入：** nums = [0,0,0]
> **输出：** \[[0,0,0]\]
> **解释：**唯一可能的三元组和为 0 。
> 
> **提示：**
> 
> - `3 <= nums.length <= 3000`
> - `-105 <= nums[i] <= 105`

**核心思想**：固定一个数 `nums[i]`，然后在 `i+1` 之后的区间内寻找两个数，使得它们的和等于 `-nums[i]`。

```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> res;
        sort(nums.begin(), nums.end()); // 排序是双指针的前提
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            // 优化 1: 如果当前数字 > 0，后面都是正数，和不可能为 0
            if (nums[i] > 0) break; 
            // 优化 2: 跳过重复的第一个数（去重关键！）
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int target = -nums[i];
            int lo = i + 1, hi = n - 1;

            while (lo < hi) {
                int sum = nums[lo] + nums[hi];
                if (sum < target) {
                    lo++;
                } else if (sum > target) {
                    hi--;
                } else {
                    res.push_back({nums[i], nums[lo], nums[hi]});
                    // 找到一组解后，跳过左右所有的重复项
                    int left = nums[lo], right = nums[hi];
                    while (lo < hi && nums[lo] == left) lo++;
                    while (lo < hi && nums[hi] == right) hi--;
                }
            }
        }
        return res;
    }
};
```

# [四数之和](https://leetcode.cn/problems/4sum/)

> [!question]
> 给你一个由 `n` 个整数组成的数组 `nums` ，和一个目标值 `target` 。请你找出并返回满足下述全部条件且**不重复**的四元组 `[nums[a], nums[b], nums[c], nums[d]]` （若两个四元组元素一一对应，则认为两个四元组重复）：
> - `0 <= a, b, c, d < n`
> - `a`、`b`、`c` 和 `d` **互不相同**
> - `nums[a] + nums[b] + nums[c] + nums[d] == target`
> 
> 你可以按 **任意顺序** 返回答案 。
> 
> **示例 1：**
> 
> **输入：** nums = [1,0,-1,0,-2,2], target = 0
> **输出：** \[[-2,-1,1,2],[-2,0,0,2],[-1,0,0,1]\]
> 
> **示例 2：**
> 
> **输入：** nums = [2,2,2,2,2], target = 8
> **输出：** \[[2,2,2,2]\]
> 
> **提示：**
> 
> - `1 <= nums.length <= 200`
> - `-109 <= nums[i] <= 109`
> - `-109 <= target <= 109`

**核心思想**：在 3Sum 的基础上再套一层循环。 **注意点**：4Sum 的 `target` 是动态给定的，且 `nums[i]` 的累加可能会溢出 `int`，在 C++ 中建议用 `long` 处理求和。

```cpp
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector<vector<int>> res;
        sort(nums.begin(), nums.end());
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            // 第一层去重
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            for (int j = i + 1; j < n; j++) {
                // 第二层去重
                if (j > i + 1 && nums[j] == nums[j - 1]) continue;

                int lo = j + 1, hi = n - 1;
                // 使用 long 避免溢出
                long remaining = (long)target - nums[i] - nums[j];

                while (lo < hi) {
                    int sum = nums[lo] + nums[hi];
                    if (sum < remaining) {
                        lo++;
                    } else if (sum > remaining) {
                        hi--;
                    } else {
                        res.push_back({nums[i], nums[j], nums[lo], nums[hi]});
                        int left = nums[lo], right = nums[hi];
                        while (lo < hi && nums[lo] == left) lo++;
                        while (lo < hi && nums[hi] == right) hi--;
                    }
                }
            }
        }
        return res;
    }
};
```

# 以上两道题的共同“去重”逻辑总结

这两道题最容易错的地方不是双指针，而是 **去重位置**。请牢记这两种去重形态：

> 形态 A：外层循环的 `continue`：

```cpp
if (i > 0 && nums[i] == nums[i-1]) continue;
```

**目的**：保证同一个“位置”不会选两次相同的数字作为起点。

> 形态 B：内层双指针的 `while`

```cpp
while (lo < hi && nums[lo] == left) lo++;
```

**目的**：在找到一组解后，迅速滑过所有相同的数字，防止在结果集里出现重复的三元组或四元组。

# 通用的 nSum 递归函数

```cpp
/*
 * nums: 已排序数组
 * n: 几数之和
 * start: 本层搜索的起点 (即你问的 i+1)
 * target: 目标值
 */
vector<vector<int>> nSum(vector<int>& nums, int n, int start, int target) {
    int sz = nums.size();
    vector<vector<int>> res;
    
    // 递归底：2Sum 情况
    if (n == 2) {
        int lo = start, hi = sz - 1;
        while (lo < hi) {
            int sum = nums[lo] + nums[hi];
            int left = nums[lo], right = nums[hi];
            if (sum < target) {
                while (lo < hi && nums[lo] == left) lo++;
            } else if (sum > target) {
                while (lo < hi && nums[hi] == right) hi--;
            } else {
                res.push_back({left, right});
                while (lo < hi && nums[lo] == left) lo++;
                while (lo < hi && nums[hi] == right) hi--;
            }
        }
    } else {
        // n > 2 时，递归计算 (n-1)Sum
        for (int i = start; i < sz; i++) {
            // 去重
            if (i > start && nums[i] == nums[i - 1]) continue;
            
            // 递归调用：下层的起点是 i + 1！
            auto sub = nSum(nums, n - 1, i + 1, target - nums[i]);
            
            for (auto& v : sub) {
                v.push_back(nums[i]);
                res.push_back(v);
            }
        }
    }
    return res;
}
```