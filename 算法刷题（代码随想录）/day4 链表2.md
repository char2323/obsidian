# [两两交换链表中的节点](https://leetcode.cn/problems/swap-nodes-in-pairs/)

> [!question]
> 给你一个链表，两两交换其中相邻的节点，并返回交换后链表的头节点。你必须在不修改节点内部的值的情况下完成本题（即，只能进行节点交换）。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2020/10/03/swap_ex1.jpg)
> 
> **输入：** head = [1,2,3,4]
> **输出：**[2,1,4,3]
> 
> **示例 2：**
> 
> **输入：** head = []
> **输出：** []
> 
> **示例 3：**
> 
> **输入：** head = [1]
> **输出：** [1]
> 
> **提示：**
> 
> - 链表中节点的数目在范围 `[0, 100]` 内
> - `0 <= Node.val <= 100`
> 

可以使用递归或者一般方法。

## 递归

递归注意不要陷入每一层中，只要注意**递归的本质是“自底向上”处理。**

写任何递归函数，只需要想清楚这三件事：

1. **终止条件（Base Case）**：什么时候停？
    - 如果没有节点，或者只剩一个节点，没法交换，直接返回。
2. **递归单元（Recursive Step）**：我要做什么？
    - 把当前的两个节点（`head` 和 `next`）换个位置。
3. **返回值**：我要给上一层交什么差？
    - 返回交换后的这这一对的“新头节点”。

假设链表是 `1 -> 2 -> 3 -> 4`。

1. **第一层（处理 1, 2）**：
    - `head` 是 `1`，`next` 是 `2`。
    - **关键动作**：我们要让 `2` 指向 `1`。
    - **悬念**：`1` 应该指向谁？它应该指向**后面那一串（3, 4）交换完后的结果**。
2. **第二层（处理 3, 4）**：
    - 重复上面的动作。
    - `4` 指向 `3`，`3` 指向后面交换完的结果（此时后面是 `nullptr`）。
    - 这一层返回 `4` 给第一层。

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        // 1. 终止条件：如果不满足两个节点，就不用换了，直接原样返回
        if (head == nullptr || head->next == nullptr) {
            return head;
        }

        // 2. 确定当前要交换的两个节点
        ListNode* firstNode = head;
        ListNode* secondNode = head->next;

        // 3. 递归：先把后面的“烂摊子”处理好
        // 让 firstNode（原来的 1）指向后面已经交换好的结果（原来的 4）
        firstNode->next = swapPairs(secondNode->next);

        // 4. 交换当前这对：让 secondNode（原来的 2）指向 firstNode（原来的 1）
        secondNode->next = firstNode;

        // 5. 返回给上一层：现在 secondNode 成了这一对的新头
        return secondNode;
    }
};
```

## 非递归

假设我们要交换节点 `A` 和 `B`。为了保证链表不断裂，必须要能控制以下三个角色：

1. **`prev`**：指向要交换的一对节点的前驱（这样交换完后，前驱才能指向新的头）。
2. **`node1`**：要交换的第一对中的第一个（原来的 A）。
3. **`node2`**：要交换的第一对中的第二个（原来的 B）。

有了 `dummy` 节点（下标 -1），我们的逻辑会非常统一。初始时 `curr` 指向 `dummy`。

交换过程（以交换 `1` 和 `2` 为例）：

1. **记录位置**： `node1 = curr->next;` `node2 = curr->next->next;`
2. **进行交换**：
    - `curr->next = node2;` （`dummy` 指向 `2`）
    - `node1->next = node2->next;` （`1` 指向 `3`）
    - `node2->next = node1;` （`2` 指向 `1`）
3. **移动 `curr`**： `curr = node1;` （现在 `1` 是这对的末尾了，变成下一对的前驱）

```cpp
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        // 1. 依然是老规矩，祭出 dummy 节点（栈分配）
        ListNode dummy(-1);
        dummy.next = head;
        ListNode* curr = &dummy;

        // 2. 只有当后面至少有两个节点时，才需要交换
        while (curr->next != nullptr && curr->next->next != nullptr) {
            // 临时保存节点
            ListNode* node1 = curr->next;
            ListNode* node2 = curr->next->next;

            // 开始交换（三步走）
            node1->next = node2->next; // 1 -> 3
            node2->next = node1;       // 2 -> 1
            curr->next = node2;        // dummy -> 2

            // 3. 移动指针，准备处理下一对
            // 交换后 node1 变成了这一对的后面那个，也就是下一对的前驱
            curr = node1;
        }

        return dummy.next;
    }
};
```

- **顺序很重要**：如果先写 `curr->next = node2` 再写 `node2->next = node1`，看似没问题，但如果不小心把 `node1->next` 给弄丢了（没先连到 `3` 上），链表就断了。

- **循环条件**：一定要先检查 `curr->next` 是否为空，再检查 `curr->next->next`。在 C++ 中，`&&` 是**短路运算符**，如果第一个条件不满足，就不会去访问第二个，从而避免了空指针崩溃。

# [删除链表的倒数第 N 个结点](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/)

> [!question]
> 给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2020/10/03/remove_ex1.jpg)
> 
> **输入：** head = [1,2,3,4,5], n = 2
> **输出：** [1,2,3,5]
> 
> **示例 2：**
> 
> **输入：** head = [1], n = 1
> **输出：** []
> 
> **示例 3：**
> 
> **输入：** head = [1,2], n = 1
> **输出：** [1]
> 
> **提示：**
> 
> - 链表中结点的数目为 `sz`
> - `1 <= sz <= 30`
> - `0 <= Node.val <= 100`
> - `1 <= n <= sz`

## 三步口诀：分清“第几个”和“步数”

为了以后不再混淆，请记住这三条法则：

### 法则一：下标法则 (Index Rule)

> **从 `dummy` 开始，走 K 步，到达下标 K−1 的节点。**

- 想操作下标为 `i` 的节点？从 `dummy` 开始走 `i` 步（到达 `i-1`）进行删除，或走 `i+1` 步到达 `i`。

### 法则二：倒数法则 (Backwards Rule)

> **想找倒数第 N 个，快指针就领先慢指针 N 步。**

- 如果快指针先走 N 步，当快指针指向 `nullptr` 时，慢指针刚好指向倒数第 N 个。
- **注意**：如果你要删除倒数第 n 个，你需要停在倒数第 n+1 个。所以快指针要领先 **n+1** 步。

### 法则三：距离法则 (Distance Rule)

> **总步数 = 跨过的节点数。**

如果从 `dummy` 出发，链表长度为 L（不含 dummy），那么：

- 到达最后一个节点（下标 L−1）需要走 L 步。
- 到达 `nullptr`（越过末尾）需要走 L+1 步。

> 实际上只需要记住这一句话就够：**终点下标 = 起点下标 + 步数**

```cpp
ListNode *removeNthFromEnd(ListNode *head, int n) {
    ListNode *dummy = new ListNode(0);
    dummy->next = head;
    ListNode *p1 = dummy;
    ListNode *p2 = dummy;
    for (int i = 0; i < n + 1; i++) {
      p1 = p1->next;
    }
    while (p1 != nullptr) {
      p1 = p1->next;
      p2 = p2->next;
    }
    ListNode *x = p2;
    x->next = x->next->next;
    return dummy->next;
  }
```

# [链表相交](https://leetcode.cn/problems/intersection-of-two-linked-lists-lcci/)

> [!question]
> 给你两个单链表的头节点 `headA` 和 `headB` ，请你找出并返回两个单链表相交的起始节点。如果两个链表没有交点，返回 `null` 。
> 图示两个链表在节点 `c1` 开始相交**：**
> 
> [![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/14/160_statement.png)
> 
> 题目数据 **保证** 整个链式结构中不存在环。
> 
> **注意**，函数返回结果后，链表必须 **保持其原始结构** 。
> 
> **示例 1：**
> 
> [![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/14/160_example_1.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_1.png)
> 
> **输入：** intersectVal = 8, listA = [4,1,8,4,5], listB = [5,0,1,8,4,5], skipA = 2, skipB = 3
> **输出：** Intersected at '8'
> **解释：** 相交节点的值为 8 （注意，如果两个链表相交则不能为 0）。
> 从各自的表头开始算起，链表 A 为 [4,1,8,4,5]，链表 B 为 [5,0,1,8,4,5]。
> 在 A 中，相交节点前有 2 个节点；在 B 中，相交节点前有 3 个节点。
> 
> **示例 2：**
> 
> [![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/14/160_example_2.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_2.png)
> 
> **输入：** intersectVal = 2, listA = [0,9,1,2,4], listB = [3,2,4], skipA = 3, skipB = 1
> **输出：** Intersected at '2'
> **解释：** 相交节点的值为 2 （注意，如果两个链表相交则不能为 0）。
> 从各自的表头开始算起，链表 A 为 [0,9,1,2,4]，链表 B 为 [3,2,4]。
> 在 A 中，相交节点前有 3 个节点；在 B 中，相交节点前有 1 个节点。
> 
> **示例 3：**
> 
> [![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/14/160_example_3.png)](https://assets.leetcode.com/uploads/2018/12/13/160_example_3.png)
> 
> **输入：** intersectVal = 0, listA = [2,6,4], listB = [1,5], skipA = 3, skipB = 2
> **输出：** null
> **解释：** 从各自的表头开始算起，链表 A 为 [2,6,4]，链表 B 为 [1,5]。
> 由于这两个链表不相交，所以 intersectVal 必须为 0，而 skipA 和 skipB 可以是任意值。
> 这两个链表不相交，因此返回 null 。
> 
> **提示：**
> 
> - `listA` 中节点数目为 `m`
> - `listB` 中节点数目为 `n`
> - `0 <= m, n <= 3 * 104`
> - `1 <= Node.val <= 105`
> - `0 <= skipA <= m`
> - `0 <= skipB <= n`
> - 如果 `listA` 和 `listB` 没有交点，`intersectVal` 为 `0`
> - 如果 `listA` 和 `listB` 有交点，`intersectVal == listA[skipA + 1] == listB[skipB + 1]`

直接看代码：

```cpp
class Solution {
public:
    ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
        // p1 指向 A 链表头结点，p2 指向 B 链表头结点
        ListNode* p1 = headA;
        ListNode* p2 = headB;
        while (p1 != p2) {
            // p1 走一步，如果走到 A 链表末尾，转到 B 链表
            p1 = (p1 == nullptr) ? headB : p1->next;
            // p2 走一步，如果走到 B 链表末尾，转到 A 链表
            p2 = (p2 == nullptr) ? headA : p2->next;
        }
        return p1;
    }
};
```

# [环形链表](https://leetcode.cn/problems/linked-list-cycle/)

> [!question]
> 给你一个链表的头节点 `head` ，判断链表中是否有环。
> 如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（索引从 0 开始）。**注意：`pos` 不作为参数进行传递** 。仅仅是为了标识链表的实际情况。
> 
> _如果链表中存在环_ ，则返回 `true` 。 否则，返回 `false` 。
> 
> **示例 1：**
> 
> ![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist.png)
> 
> **输入：** head = [3,2,0,-4], pos = 1
> **输出：** true
> **解释：** 链表中有一个环，其尾部连接到第二个节点。
> 
> **示例 2：**
> 
> ![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)
> 
> **输入：** head = [1,2], pos = 0
> **输出：** true
> **解释：** 链表中有一个环，其尾部连接到第一个节点。
> 
> **示例 3：**
> 
> ![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)
> 
> **输入：** head = [1], pos = -1
> **输出：** false
> **解释：** 链表中没有环。
> 
> **提示：**
> 
> - 链表中节点的数目范围是 `[0, 104]`
> - `-105 <= Node.val <= 105`
> - `pos` 为 `-1` 或者链表中的一个 **有效索引** 。

用快慢指针就可以秒解：

```cpp
class Solution {
public:
    bool hasCycle(ListNode *head) {
        // 快慢指针初始化指向 head
        ListNode *slow = head, *fast = head;
        // 快指针走到末尾时停止
        while (fast != nullptr && fast->next != nullptr) {
            // 慢指针走一步，快指针走两步
            slow = slow->next;
            fast = fast->next->next;
            // 快慢指针相遇，说明含有环
            if (slow == fast) {
                return true;
            }
        }
        // 不包含环
        return false;
    }
};
```

# [环形链表 II](https://leetcode.cn/problems/linked-list-cycle-ii/)

> [!question]
> 给定一个链表的头节点  `head` ，返回链表开始入环的第一个节点。 _如果链表无环，则返回 `null`。_
> 如果链表中有某个节点，可以通过连续跟踪 `next` 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 `pos` 来表示链表尾连接到链表中的位置（**索引从 0 开始**）。如果 `pos` 是 `-1`，则在该链表中没有环。**注意：`pos` 不作为参数进行传递**，仅仅是为了标识链表的实际情况。
> 
> **不允许修改** 链表。
> 
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2018/12/07/circularlinkedlist.png)
> 
> **输入：** head = [3,2,0,-4], pos = 1
> **输出：** 返回索引为 1 的链表节点
> **解释：** 链表中有一个环，其尾部连接到第二个节点。
> 
> **示例 2：**
> 
> ![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test2.png)
> 
> **输入：** head = [1,2], pos = 0
> **输出：** 返回索引为 0 的链表节点
> **解释：** 链表中有一个环，其尾部连接到第一个节点。
> 
> **示例 3：**
> 
> ![](https://assets.leetcode.cn/aliyun-lc-upload/uploads/2018/12/07/circularlinkedlist_test3.png)
> 
> **输入：** head = [1], pos = -1
> **输出：** 返回 null
> **解释：** 链表中没有环。
> 
> **提示：**
> 
> - 链表中节点的数目范围在范围 `[0, 104]` 内
> - `-105 <= Node.val <= 105`
> - `pos` 的值为 `-1` 或者链表中的一个有效索引

基于上一个问题，我们还需要多思考一步：

## 核心推导：路程公式

假设从链表头到环起点的距离为 a，环起点到快慢指针相遇点的距离为 b，相遇点再走回到环起点的距离为 c。

当快慢指针相遇时：

- **慢指针（Slow）** 走过的路程：Sslow​=a+b
- **快指针（Fast）** 走过的路程：Sfast​=a+n(b+c)+b （n 是快指针在环里转的圈数）

因为快指针的速度是慢指针的 **2 倍**，所以：
$$2×(a+b)=a+n(b+c)+b$$
化简这个等式：
$$a+b=n(b+c)a=(n−1)(b+c)+c$$

即得：**从链表头出发到环起点的距离 (a)，等于从相遇点走回到环起点的距离 (c)**（即便快指针多转了几圈，最终落点是一样的）。

因此，定位起点的策略如下：

1. **第一步**：快慢指针同时出发，找到相遇点。
2. **第二步**：此时，让一个指针回到 **链表头（head）**，另一个指针停在 **相遇点**。
3. **第三步**：两个指针以 **相同速度（每次 1 步）** 同时前进。
4. **最后**：当它们再次相遇时，那个位置就是 **环的起点**。

```cpp
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *slow = head, *fast = head;
        // 1. 寻找相遇点
        while (fast != nullptr && fast->next != nullptr) {
            slow = slow->next;
            fast = fast->next->next;
            if (slow == fast) { // 相遇了
                // 2. 定位起点
                ListNode *index1 = head;
                ListNode *index2 = fast;
                while (index1 != index2) {
                    index1 = index1->next;
                    index2 = index2->next;
                }
                return index1; // 相遇处即为环起点
            }
        }
        return nullptr; // 无环
    }
};
```

