# [反转字符串](https://leetcode.cn/problems/reverse-string/)

> [!question]
> 编写一个函数，其作用是将输入的字符串反转过来。输入字符串以字符数组 `s` 的形式给出。
> 不要给另外的数组分配额外的空间，你必须**[原地](https://baike.baidu.com/item/原地算法)修改输入数组**、使用 O(1) 的额外空间解决这一问题。
> 
> **示例 1：**
> 
> **输入：** s = ["h","e","l","l","o"]
> **输出：** ["o","l","l","e","h"]
> 
> **示例 2：**
> 
> **输入：** s = ["H","a","n","n","a","h"]
> **输出：** ["h","a","n","n","a","H"]
> 
> **提示：**
> 
> - `1 <= s.length <= 105`
> - `s[i]` 都是 [ASCII](https://baike.baidu.com/item/ASCII) 码表中的可打印字符

```cpp
class Solution {
public:
  void reverseString(vector<char> &s) {
    int left = 0, right = s.size() - 1;
    while (left < right) {
      char temp = s[left];
      s[left] = s[right];
      s[right] = temp;
      right--;
      left++;
    }
  }
};
```

双指针秒杀、

# [反转字符串 II](https://leetcode.cn/problems/reverse-string-ii/)

> [!question]
> 给定一个字符串 `s` 和一个整数 `k`，从字符串开头算起，每计数至 `2k` 个字符，就反转这 `2k` 字符中的前 `k` 个字符。
> 
> - 如果剩余字符少于 `k` 个，则将剩余字符全部反转。
> - 如果剩余字符小于 `2k` 但大于或等于 `k` 个，则反转前 `k` 个字符，其余字符保持原样。
> 
> **示例 1：**
> 
> **输入：** s = "abcdefg", k = 2
> **输出：** "bacdfeg"
> 
> **示例 2：**
> 
> **输入：** s = "abcd", k = 2
> **输出：** "bacd"
> 
> **提示：**
> 
> - `1 <= s.length <= 104`
> - `s` 仅由小写英文组成
> - `1 <= k <= 104`

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {
        // i 每次跳 2k 步，这就是我们的“外层步进指针”
        for (int i = 0; i < s.length(); i += (2 * k)) {
            // 规则：翻转前 k 个。如果剩余字符少于 k 个，则全部翻转。
            // 所以右边界是 min(当前起点 + k - 1, 字符串末尾)
            int left = i;
            int right = min((int)s.length() - 1, i + k - 1);
            
            // 这里就是标准的“头尾双指针”翻转逻辑
            while (left < right) {
                swap(s[left++], s[right--]);
            }
        }
        return s;
    }
};
```

# 💡 深度思考：什么时候该用双指针？

- **头尾双指针**：用于翻转、判断回文、2Sum（有序）。
- **快慢双指针**：用于找环、找中点、滑动窗口。
- **分块/步进指针（本题）**：用于格式化处理字符串。

# [替换数字](https://kamacoder.com/problempage.php?pid=1064)

> [!question]
> 给定一个字符串 s，它包含小写字母和数字字符，请编写一个函数，将字符串中的字母字符保持不变，而将每个数字字符替换为number。
> 
> 例如，对于输入字符串 "a 1 b 2 c 3"，函数应该将其转换为 "anumberbnumbercnumber"。
> 
> 对于输入字符串 "a 5 b"，函数应该将其转换为 "anumberb"
> 
> 输入：一个字符串 s, s 仅包含小写字母和数字字符。
> 
> 输出：打印一个新的字符串，其中每个数字字符都被替换为了 number
> 
> 样例输入：a 1 b 2 c 3
> 
> 样例输出：anumberbnumbercnumber
> 
> 数据范围：1 <= s.length < 10000。
> 

## 1. 核心思想：先扩容，后填充（从后往前）

在处理字符串替换时，如果从前往后替换，每碰到一个数字，后面的字符都要往后挪，时间复杂度会飙升到 O(n2)。

**更优雅的做法是：**

1. **统计数字个数**：计算出替换后字符串需要的总长度。
2. **扩容**：使用 `s.resize()` 将原字符串空间拉大。
3. **双指针填充**：一个指针 `i` 指向旧长度的末尾，一个指针 `j` 指向新长度的末尾。**从后往前**进行填充。

```cpp
#include <iostream>
#include <string>

using namespace std;

string replaceNumber(string s) {
    int oldSize = s.size();
    int count = 0;

    // 1. 统计数字出现的次数
    for (char c : s) {
        if (c >= '0' && c <= '9') count++;
    }

    // 2. 扩容：每个数字 '1' 变成 "number"，长度增加了 5 (6 - 1)
    // 比如 '1' 是 1位，"number" 是 6位，所以每个数字多出 5 个坑位
    s.resize(oldSize + count * 5);
    int newSize = s.size();

    // 3. 双指针从后往前填充
    // i 指向旧末尾，j 指向新末尾
    for (int i = oldSize - 1, j = newSize - 1; i >= 0; i--) {
        if (s[i] >= '0' && s[i] <= '9') {
            // 发现数字，倒着填入 "number"
            s[j--] = 'r';
            s[j--] = 'e';
            s[j--] = 'b';
            s[j--] = 'm';
            s[j--] = 'u';
            s[j--] = 'n';
        } else {
            // 发现字母，直接搬运
            s[j--] = s[i];
        }
    }
    return s;
}

int main() {
    string s;
    while (cin >> s) {
        cout << replaceNumber(s) << endl;
    }
    return 0;
}
```

