# [反转字符串中的单词](https://leetcode.cn/problems/reverse-words-in-a-string/)

分三步走：
1. 移除多余空格（前后空格，以及单词间多余的空格）
2. 反转整个字符串
3. 反转每个单词

## 1. 第一步：移除多余空格（快慢指针）

这是之前学过的“移除元素”逻辑的变体。我们使用**快慢指针**：`fast` 寻找有效字符，`slow` 指向填充位置。

**逻辑：**
- 跳过前面的空格。
- 单词之间只保留一个空格。
- 跳过后面的空格。

```cpp
void removeExtraSpaces(string& s) {
    int slow = 0;
    for (int fast = 0; fast < s.size(); ++fast) {
        if (s[fast] != ' ') { // 遇到非空格
            // 手动添加单词间的空格：如果 slow 不是起点，说明前面刚填完一个单词
            if (slow != 0) s[slow++] = ' '; 
            
            // 补全整个单词
            while (fast < s.size() && s[fast] != ' ') {
                s[slow++] = s[fast++];
            }
        }
    }
    s.resize(slow); // 重新调整字符串大小，砍掉末尾多余部分
}
```

## 2. 第二步：整体反转 + 局部反转

这是解这道题最精妙的数学逻辑：**负负得正。**

- **例子：** `"the sky is blue"`
- **整体反转：** `"eulb si yks eht"` （此时单词顺序对了，但单词内部字母是反的）
- **局部反转每个单词：** `"blue is sky the"` （大功告成！）

## 3. 代码实现

```cpp
class Solution {
public:
    // 辅助函数：双指针反转任意区间 [left, right]
    void reverse(string& s, int left, int right) {
        while (left < right) {
            swap(s[left++], s[right--]);
        }
    }

    string reverseWords(string s) {
        // 1. 移除多余空格 (快慢指针)
        int slow = 0;
        for (int fast = 0; fast < s.size(); ++fast) {
            if (s[fast] != ' ') {
                if (slow != 0) s[slow++] = ' ';
                while (fast < s.size() && s[fast] != ' ') {
                    s[slow++] = s[fast++];
                }
            }
        }
        s.resize(slow);

        // 2. 整体反转
        reverse(s, 0, s.size() - 1);

        // 3. 局部反转单词 (双指针寻找单词边界)
        int start = 0;
        for (int i = 0; i <= s.size(); ++i) {
            // 当到达空格或字符串末尾时，说明找到了一个单词的终点
            if (i == s.size() || s[i] == ' ') {
                reverse(s, start, i - 1);
                start = i + 1; // 更新下一个单词的起点
            }
        }

        return s;
    }
};
```

## 问题：

>  `if (slow != 0) s[slow++] = ' '` 这里为什么就可以说这里就是新单词的开头了呢

能进入这个 `if (slow != 0)` 逻辑，必须先通过外层的 `if (s[fast] != ' ')`。这意味着：**`fast` 此时正踩在一个真实的字符（单词的开头）上。**

### 1. 为什么这里必然是新单词的开头？

我们可以把代码的执行流程看作一个**“进入检查站”**的过程：

```C++
for (int fast = 0; fast < s.size(); ++fast) {
    if (s[fast] != ' ') { // 【检查站 A】：fast 必须抓到了一个非空格字符
        
        // --- 此时，fast 已经确定站在了一个新单词的第一个字母上 ---

        if (slow != 0) { // 【检查站 B】：如果 slow 已经写过东西了
            s[slow++] = ' '; // 就在写新单词之前，先补一个空格
        }

        // --- 开始搬运这个新单词 ---
        while (fast < s.size() && s[fast] != ' ') {
            s[slow++] = s[fast++];
        }
    }
}
```

### 2. 逻辑推演：为什么它不会在单词中间加空格？

你可能会担心：如果单词是 `apple`，它会不会在 `a` 后面加个空格？

**不会。** 因为 `while` 循环会负责把整个 `apple` 搬运完：

1. `fast` 看到 `a`，外层 `if` 成立。
2. 此时 `while` 循环启动，直接把 `a`, `p`, `p`, `l`, `e` 全部搬到 `slow` 的位置。
3. **关键点**：当 `while` 结束时，`fast` 已经跳过了 `e`，指向了 `apple` 后面的那个空格。
4. 下一次 `for` 循环开始时，`fast` 看到的是空格，**直接被外层的 `if (s[fast] != ' ')` 挡在了门外**。

**结论**：只有当 `fast` 越过了一堆空格，再次撞上**下一个单词的第一个字母**时，外层的 `if` 才会再次开启，此时 `slow` 已经不是 0 了，才会触发补空格的动作。

---

### 3. 用“开关灯”来理解

你可以把外层的 `if (s[fast] != ' ')` 想象成一个“单词探测灯”：

- **灯灭（`fast` 看到空格）**：`slow` 坐着不动，什么都不干。
- **灯亮（`fast` 撞到字符）**：
    - **第一件事**：看看自己是不是第一个亮起的（`slow == 0`？）。如果不是第一个，说明前面已经有单词了，赶紧给 `slow` 补个空格。
    - **第二件事**：把这个单词的所有字母一口气搬走（`while` 循环），直到灯再次熄灭。

---

### 4. 为什么不把 `s[slow++] = ' '` 放在 `while` 后面？

如果你放在 `while` 后面（即“写完一个单词补一个空格”），会产生一个很麻烦的后果：**字符串末尾会多出一个空格。**

比如处理 `"the sky"`：

1. 写完 `the`，补个空格。
2. 写完 `sky`，补个空格。
3. 结果变成了 `"the sky "`。

而现在的写法是“写下一个单词前，看看要不要给前一个单词补空格”。这种“后置补偿”逻辑完美保证了：

- **开头没有空格**（第一个单词前不补）。
- **结尾没有空格**（最后一个单词写完后，循环就结束了，没机会再补）。

---

### 总结

这个 `if (slow != 0)` 就像是一个“衔接器”。它只在 `fast` 探测到新资源（新单词）且已经有旧产物（旧单词）的情况下才工作。

# [右旋字符串](https://kamacoder.com/problempage.php?pid=1065)

> [!question]
> 字符串的右旋转操作是把字符串尾部的若干个字符转移到字符串的前面。给定一个字符串 s 和一个正整数 k，请编写一个函数，将字符串中的后面 k 个字符移到字符串的前面，实现字符串的右旋转操作。
> 
> 例如，对于输入字符串 "abcdefg" 和整数 2，函数应该将其转换为 "fgabcde"。
> 
> 输入：输入共包含两行，第一行为一个正整数 k，代表右旋转的位数。第二行为字符串 s，代表需要旋转的字符串。
> 
> 输出：输出共一行，为进行了右旋转操作后的字符串。

依然是利用“整体反转 + 局部反转”的数学逻辑，也就是常说的 “三反转法”。

> 核心思想：负负得正——要把尾部 k 个字符移到前面，我们可以分三步走：

1. **翻转整个字符串**：把后面的字符带到前面（但顺序是反的）。
2. **翻转前 k 个字符**：让移到前面的这部分恢复正常顺序。
3. **翻转剩余的字符**：让剩下的部分也恢复正常顺序。

```cpp
#include <iostream>
#include <string>
#include <algorithm>

using namespace std;

// 双指针反转函数
void reverseString(string& s, int start, int end) {
    while (start < end) {
        swap(s[start++], s[end--]);
    }
}

int main() {
    int k;
    string s;

    // 输入 k 和字符串 s
    if (!(cin >> k >> s)) return 0;

    int n = s.size();
    // 如果 k 大于长度，取模处理（虽然本题范围 k < n，但这是好习惯）
    k = k % n;

    // 1. 整体反转
    reverseString(s, 0, n - 1);
    
    // 2. 反转前 k 个
    reverseString(s, 0, k - 1);
    
    // 3. 反转剩余部分
    reverseString(s, k, n - 1);

    cout << s << endl;

    return 0;
}
```

# `size()&size()-1`

### 1. `s.size() - 1`：找“末尾位置”

当你需要一个具体的**下标**去访问字符串里的字符时，通常要用到 `-1`。

- **场景**：双指针反转、读取最后一个字符、右边界。
- **直觉**：因为下标从 0 开始，所以最后一个坑位的编号永远比总数少 1。
- **例子**：`"Arch"` 的 `size()` 是 4，但字母 `'h'` 的下标是 3。
    

```C++
int hi = s.size() - 1; // 指向最后一个字符
while (lo < hi) {
    swap(s[lo++], s[hi--]); // 只有指向合法的下标，swap 才有意义
}
```

---

### 2. `s.size()`：代表“整体边界”或“长度”

当你需要描述一个**范围**、进行**循环判断**或**调整大小**时，通常直接用 `size()`。

- **场景**：`for` 循环遍历、`resize()` 空间、作为“结束”的标志。
- **直觉**：`size()` 指向的是字符串末尾字母**后面**的那个虚空位置（通常是 `\0` 的位置）。
- **例子**：

```C++
// 这里的 i < s.size() 实际上包含了 0, 1, 2, ..., size()-1
for (int i = 0; i < s.size(); i++) {
    // 逻辑
}

s.resize(s.size() + 5); // 这里的 size() 是作为数量使用的
```

---

### 3. 核心对比表：一目了然

|**场景**|**使用 s.size()**|**使用 s.size() - 1**|
|---|---|---|
|**循环条件**|`i < s.size()` (常用)|`i <= s.size() - 1` (较少用)|
|**双指针**|很少用（除非是寻找插入位置）|**必须用**（指向最后一个有效元素）|
|**截取区间**|代表区间长度|代表区间最后一个元素的下标|
|**越界风险**|访问 `s[s.size()]` 会报错|访问 `s[s.size()-1]` 是安全的|

---

### 4. 一个容易踩坑的特例：`unsigned` 溢出

在 Arch Linux 下用 `g++` 编译时，如果你写 `i < s.size() - 1` 而字符串刚好是**空**的，程序可能会直接崩溃。

**为什么？**

因为 `s.size()` 返回的是 `size_t`（无符号整数）。

- 如果 `s` 是空的，`s.size()` 是 0。
- `0 - 1` 在无符号运算下会变成一个**巨大的正数**（比如 4294967295）。
- 循环条件 `i < 巨大正数` 会变成死循环并导致越界。

**优雅的解决方案：**

如果你需要处理可能为空的情况，尽量写 `i + 1 < s.size()` 或者先强转：`(int)s.size() - 1`。

---

### 5. 记忆口诀

> **“指哪打哪”用 `-1`，**（需要精确坐标时）
> 
> **“圈地围墙”用 `size`。**（需要界定范围时）

---

### 练手小测试

刚才我们在写 **151. 反转字符串中的单词** 时，局部反转单词的代码是这样的：

```C++
for (int i = 0; i <= s.size(); ++i) {
    if (i == s.size() || s[i] == ' ') {
        reverse(s, start, i - 1); // 思考：为什么这里是 i - 1？
    }
}
```

**解析：** 这里的 `i` 跑到了 `s.size()`（越过了最后一个字母），所以当它发现自己出界或者是空格时，它知道单词的结尾就在它**前一个位置**，所以用 `i - 1`。

# [找出字符串中第一个匹配项的下标](https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/)

先构建 `next` 数组:

```cpp
void getNext(int* next, const string& s) {
    int j = 0; // j 代表前缀末尾，也代表最长相等前后缀的长度
    next[0] = 0;
    for(int i = 1; i < s.size(); i++) { // i 代表后缀末尾
        // 如果不匹配，就回退到上一个可能相等的位置
        while (j > 0 && s[i] != s[j]) {
            j = next[j - 1];
        }
        // 如果匹配，长度加 1
        if (s[i] == s[j]) {
            j++;
        }
        next[i] = j;
    }
}
```

利用 `next` 数组进行匹配：

```cpp
int strStr(string haystack, string needle) {
    if (needle.size() == 0) return 0;
    int next[needle.size()];
    getNext(next, needle);

    int j = 0; // needle 的指针
    for (int i = 0; i < haystack.size(); i++) { // haystack 的指针
        while (j > 0 && haystack[i] != needle[j]) {
            // 关键：匹配失败，j 跳到 next 数组记录的位置，i 不回退！
            j = next[j - 1];
        }
        if (haystack[i] == needle[j]) {
            j++;
        }
        if (j == needle.size()) { // 全部匹配成功
            return (i - needle.size() + 1);
        }
    }
    return -1;
}
```

# 标准 KMP 算法代码

```cpp
#include <iostream>
#include <vector>
#include <string>

using namespace std;

class KMP {
public:
    /**
     * 构建 next 数组（前缀表）
     * j 代表：1. 前缀末尾位置 2. [0, i] 区间内最长相等前后缀的长度
     * i 代表：后缀末尾位置
     */
    void getNext(vector<int>& next, const string& s) {
        int j = 0;
        next[0] = 0; // 第一个字符没有前后缀，长度必为 0

        for (int i = 1; i < s.size(); i++) {
            // 情况 1：前后缀不匹配
            // j 要回退，回退到前一个字符对应的最长相等前后缀长度
            while (j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            // 情况 2：前后缀匹配
            if (s[i] == s[j]) {
                j++;
            }
            // 将当前长度赋给 next[i]
            next[i] = j;
        }
    }

    /**
     * 在 haystack 中搜索 needle
     */
    int strStr(string haystack, string needle) {
        if (needle.empty()) return 0;
        
        int n = haystack.size();
        int m = needle.size();
        
        // 1. 初始化并构建 next 数组
        vector<int> next(m);
        getNext(next, needle);

        int j = 0; // needle 的指针
        for (int i = 0; i < n; i++) { // i 是 haystack 的指针，永不回退
            // 如果当前字符不匹配，j 根据 next 数组回退
            while (j > 0 && haystack[i] != needle[j]) {
                j = next[j - 1];
            }
            // 如果匹配，j 移动到下一位
            if (haystack[i] == needle[j]) {
                j++;
            }
            // 如果 j 走到了 needle 的末尾，说明匹配成功
            if (j == m) {
                return i - m + 1; // 返回起始索引
            }
        }
        return -1;
    }
};

int main() {
    KMP solver;
    string haystack = "hello";
    string needle = "ll";
    cout << "Found at index: " << solver.strStr(haystack, needle) << endl;
    return 0;
}
```

## 核心逻辑拆解

### 1. 为什么 `j = next[j - 1]`？

这是整个 KMP 的“灵魂”。当匹配失败时，我们不需要从头开始，因为 `next[j-1]` 已经记录了 **“当前已匹配部分中，最长的那个相同前缀的后面”**。

### 2. 两个 `while` 循环的意义

- **`getNext` 里的 `while`**：是模式串在“自己匹配自己”。如果后缀的第 `i` 个字符和前缀的第 `j` 个不相等，就看能不能在更短的前缀里找到匹配。
- **`strStr` 里的 `while`**：是主串在匹配模式串。如果断了，就利用已经算好的 `next` 数组快速重排模式串。
    

### 3. 时间复杂度分析

虽然代码里有嵌套的 `while`，但你会发现：

- 主串指针 `i` 一直在增加，从未减小。
- 模式串指针 `j` 虽然会回退，但它增加的总次数等于 `i` 增加的总次数。
- 最终时间复杂度是 **O(n+m)**，其中 n 是文本长，m 是模式串长。

# [重复的子字符串](https://leetcode.cn/problems/repeated-substring-pattern/)

可以观察到一个结论：

如果一个字符串 s 是由重复子串组成的，那么：

1. 它一定存在相等的前后缀。
2. **字符串长度 - 最长相等前后缀长度 = 最小重复单元的长度**。
3. 如果这个“重复单元长度”能被“总长度”整除，那么这个字符串就是由重复子串构成的。

我们可以用“错位”的思想来理解： 假设字符串 S 由子串 T 重复 n 次组成。 那么 S 的前 (n−1) 个 T（前缀）必然等于后 (n−1) 个 T（后缀）。

当这两部分重叠时，剩下的那那一小块（总长 - 前后缀长）就是那个**循环节**。如果总长度能把这个循环节“装满”（整除），说明整个字符串都是由它铺出来的。

```cpp
class Solution {
public:
    void getNext(vector<int>& next, const string& s) {
        int j = 0;
        next[0] = 0;
        for (int i = 1; i < s.size(); i++) {
            while (j > 0 && s[i] != s[j]) {
                j = next[j - 1];
            }
            if (s[i] == s[j]) {
                j++;
            }
            next[i] = j;
        }
    }

    bool repeatedSubstringPattern(string s) {
        int n = s.size();
        if (n <= 1) return false;

        vector<int> next(n);
        getNext(next, s);

        // 获取整个字符串的最长相等前后缀长度
        int maxLPS = next[n - 1];

        // 两个判断条件：
        // 1. maxLPS > 0：必须存在相等的前后缀
        // 2. n % (n - maxLPS) == 0：总长能被“循环节长度”整除
        if (maxLPS > 0 && n % (n - maxLPS) == 0) {
            return true;
        }

        return false;
    }
};
```

另一种想法：

```cpp
bool repeatedSubstringPattern(string s) {
    return (s + s).find(s, 1) != s.size();
}
```

**双倍字符串法**。

1. 将两个 s 拼接在一起：`s + s`。
2. 去掉新字符串的**第一个**和**最后一个**字符。
3. 如果在剩下的字符串里还能找到 s，说明 s 是循环的。

**原理：** 如果 s 有循环节，那么 `s+s` 里面其实藏着好几个 s。掐头去尾是为了破坏最外层的两个 s，如果中间还能拼出一个完整的 s，说明它内部自带旋转对称性。