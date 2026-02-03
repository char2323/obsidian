注意虚拟头节点的使用。

# [移除链表元素](https://leetcode.cn/problems/remove-linked-list-elements/)

> [!question]
> 给你一个链表的头节点 `head` 和一个整数 `val` ，请你删除链表中所有满足 `Node.val == val` 的节点，并返回 **新的头节点** 。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2021/03/06/removelinked-list.jpg)
> 
> **输入：** head = [1,2,6,3,4,5,6], val = 6
> **输出：**[1,2,3,4,5]
> 
> **示例 2：**
> 
> **输入：** head = [], val = 1
> **输出：**[]
> 
> **示例 3：**
> 
> **输入：** head = [7,7,7,7], val = 7
> **输出：**[]
> 
> **提示：**
> 
> - 列表中的节点数目在范围 `[0, 104]` 内
> - `1 <= Node.val <= 50`
> - `0 <= val <= 50`

```cpp
class Solution {
public:
  ListNode *removeElements(ListNode *head, int val) {
    ListNode dummy(-1);
    ListNode *p = &dummy;
    ListNode *curr = head;
    while (curr != nullptr) {
      if (curr->val != val) {
        p->next = curr;
        p = p->next;
      }
      curr = curr->next;
    }
    p->next = nullptr;
    return dummy.next;
  }
};
```

题目比较简单没有什么好说的，但是关于**节点的创建**有两种方式：
1. `ListNode* dummy = new ListNode; ListNode* p = dummy;`
2. `ListNode dummy(-1); ListNode* p = &dummy;`
哪一种更好呢？

结论先行：**在绝大多数情况下，使用 `ListNode dummy(-1)`（栈分配）更好。**

---

## 1. 两种方式的本质区别

|特性|`ListNode dummy(-1)`|`ListNode *dummy = new ListNode(-1)`|
|---|---|---|
|**分配位置**|**栈 (Stack)**|**堆 (Heap)**|
|**内存管理**|**自动释放**（函数结束时自动销毁）|**手动释放**（必须 `delete`，否则内存泄漏）|
|**访问速度**|**更快**（CPU 栈指针移动，缓存友好）|**较慢**（涉及内存分配器寻找空闲块）|
|**语法**|使用 `&dummy` 获取指针|本身就是指针|
## 2. 为什么推荐“栈分配”（方式二）？

### A. 零内存泄漏风险 (RAII)

在 C++ 中，`new` 出来的对象必须有对应的 `delete`。
- 如果写了 `new` 但没写 `delete`，在长期运行的程序中会导致内存持续上涨。
- 栈对象会在函数执行完毕后由编译器自动调用析构函数。这符合 C++ 的 **RAII**（资源获取即初始化）核心哲学。

### B. 性能优势

栈分配仅仅是移动一下 CPU 的栈顶指针，极其高效。而 `new` 操作符需要调用操作系统的内存分配接口（如 `malloc`），在多线程环境下还可能涉及锁竞争，开销远比栈分配大。

### C. 代码更简洁

不需要在函数末尾写一堆 `delete`，逻辑更清爽。

## 3. 两种方式的代码写法对比

### 推荐写法（栈分配）

注意这里如何通过 `&` 取地址来给 `p` 指针赋值：

```cpp
ListNode dummy(-1); 
ListNode* p = &dummy; // 获取栈对象的地址

// ... 逻辑处理 ...

return dummy.next; // 直接返回，无需清理
```

### 堆分配写法 （需手动清理）

```cpp
ListNode* dummy = new ListNode(-1);
ListNode* p = dummy;

// ... 逻辑处理 ...

ListNode* res = dummy->next;
delete dummy; // 必须手动删除！
return res;
```

## 4. 什么时候必须用 `new`（堆分配）？

虽然 `dummy` 节点建议用栈，但有些场景**必须**用堆：
1. **生存期超过函数：** 如果这个节点在函数结束后还需要存在（比如你要创建一个新链表并返回它），那么除了 `dummy` 节点外，其他实际节点必须在堆上。
2. **对象非常大：** 栈的空间通常很小（Arch Linux 默认通常是 8MB）。如果你创建了一个巨大的数组或对象，可能会导致 **Stack Overflow**。

## 总结与建议

在算法题或函数内部的临时 `dummy` 节点：
- **请坚持使用 `ListNode dummy(-1);`**。
- 它能让你养成良好的 C++ 内存管理习惯。

# [反转链表](https://leetcode.cn/problems/reverse-linked-list/)

> [!question]
> 给你单链表的头节点 `head` ，请你反转链表，并返回反转后的链表。
> **示例 1：**
> 
> ![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex1.jpg)
> 
> **输入：** head = [1,2,3,4,5]
> **输出：** [5,4,3,2,1]
> 
> **示例 2：**
> 
> ![](https://assets.leetcode.com/uploads/2021/02/19/rev1ex2.jpg)
> 
> **输入：** head = [1,2]
> **输出：** [2,1]
> 
> **示例 3：**
> 
> **输入：** head = []
> **输出：** []
> 
> **提示：**
> 
> - 链表中节点的数目范围是 `[0, 5000]`
> - `-5000 <= Node.val <= 5000`

这里可以考虑使用递归来处理，也可以单个节点反转，使用循环来处理。

## 递归

```cpp
class Solution {
public:
    ListNode* reverseList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        ListNode* last = reverseList(head->next);
        head->next->next = head;
        head->next = nullptr;
        return last;
    }
};
```

## 非递归

```cpp
class Solution {
public:
    // 反转以 head 为起点的单链表
    ListNode* reverseList(ListNode* head) {
        if (head == nullptr || head->next == nullptr) {
            return head;
        }
        // 由于单链表的结构，至少要用三个指针才能完成迭代反转
        // cur 是当前遍历的节点，pre 是 cur 的前驱结点，nxt 是 cur 的后继结点
        ListNode *pre, *cur, *nxt;
        pre = nullptr; cur = head; nxt = head->next;
        while (cur != nullptr) {
            // 逐个结点反转
            cur->next = pre;
            // 更新指针位置
            pre = cur;
            cur = nxt;
            if (nxt != nullptr) {
                nxt = nxt->next;
            }
        }
        // 返回反转后的头结点
        return pre;
    }
};
```

# [设计链表](https://leetcode.cn/problems/design-linked-list/)

> [!question]
> 你可以选择使用单链表或者双链表，设计并实现自己的链表。
> 单链表中的节点应该具备两个属性：`val` 和 `next` 。`val` 是当前节点的值，`next` 是指向下一个节点的指针/引用。
> 
> 如果是双向链表，则还需要属性 `prev` 以指示链表中的上一个节点。假设链表中的所有节点下标从 **0** 开始。
> 
> 实现 `MyLinkedList` 类：
> 
> - `MyLinkedList()` 初始化 `MyLinkedList` 对象。
> - `int get(int index)` 获取链表中下标为 `index` 的节点的值。如果下标无效，则返回 `-1` 。
> - `void addAtHead(int val)` 将一个值为 `val` 的节点插入到链表中第一个元素之前。在插入完成后，新节点会成为链表的第一个节点。
> - `void addAtTail(int val)` 将一个值为 `val` 的节点追加到链表中作为链表的最后一个元素。
> - `void addAtIndex(int index, int val)` 将一个值为 `val` 的节点插入到链表中下标为 `index` 的节点之前。如果 `index` 等于链表的长度，那么该节点会被追加到链表的末尾。如果 `index` 比长度更大，该节点将 **不会插入** 到链表中。
> - `void deleteAtIndex(int index)` 如果下标有效，则删除链表中下标为 `index` 的节点。
> 
> **示例：**
> 
> **输入**
> ["MyLinkedList", "addAtHead", "addAtTail", "addAtIndex", "get", "deleteAtIndex", "get"]
> \[[], [1], [3], [1, 2], [1], [1], [1]\]
> **输出**
> [null, null, null, null, 2, null, 3]
> 
> **解释**
> MyLinkedList myLinkedList = new MyLinkedList();
> myLinkedList.addAtHead(1);
> myLinkedList.addAtTail(3);
> myLinkedList.addAtIndex(1, 2);    // 链表变为 1->2->3
> myLinkedList.get(1);              // 返回 2
> myLinkedList.deleteAtIndex(1);    // 现在，链表变为 1->3
> myLinkedList.get(1);              // 返回 3
> 
> **提示：**
> 
> - `0 <= index, val <= 1000`
> - 请不要使用内置的 LinkedList 库。
> - 调用 `get`、`addAtHead`、`addAtTail`、`addAtIndex` 和 `deleteAtIndex` 的次数不超过 `2000` 。

使用单链表即可。

```cpp
#include <iostream>

class MyLinkedList {
public:
    // 定义节点结构
    struct LinkedNode {
        int val;
        LinkedNode* next;
        LinkedNode(int val) : val(val), next(nullptr) {}
    };

    MyLinkedList() {
        // 初始化虚拟头节点，它不存储实际数据
        _dummyHead = new LinkedNode(0);
        _size = 0;
    }

    // 获取下标为 index 的节点值
    int get(int index) {
        if (index < 0 || index >= _size) return -1;
        
        LinkedNode* cur = _dummyHead->next;
        while (index--) {
            cur = cur->next;
        }
        return cur->val;
    }

    // 在链表最前面插入
    void addAtHead(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        newNode->next = _dummyHead->next;
        _dummyHead->next = newNode;
        _size++;
    }

    // 在链表最后面追加
    void addAtTail(int val) {
        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        // 找到当前的最后一个节点
        while (cur->next != nullptr) {
            cur = cur->next;
        }
        cur->next = newNode;
        _size++;
    }

    // 在下标 index 之前插入
    void addAtIndex(int index, int val) {
        if (index > _size) return;
        if (index < 0) index = 0;

        LinkedNode* newNode = new LinkedNode(val);
        LinkedNode* cur = _dummyHead;
        // 找到 index 的前驱节点
        while (index--) {
            cur = cur->next;
        }
        newNode->next = cur->next;
        cur->next = newNode;
        _size++;
    }

    // 删除下标为 index 的节点
    void deleteAtIndex(int index) {
        if (index < 0 || index >= _size) return;

        LinkedNode* cur = _dummyHead;
        // 找到 index 的前驱节点
        while (index--) {
            cur = cur->next;
        }
        LinkedNode* tmp = cur->next;
        cur->next = cur->next->next;
        delete tmp; // 释放内存，好习惯
        _size--;
    }

private:
    int _size;
    LinkedNode* _dummyHead;
};
```

`_size` 非必需：
	虽然可以通过遍历来计算长度，但维护一个 `_size` 变量可以让 `get` 和 `addAtIndex` 的合法性检查达到 O(1) 的效率。

`_dummy` 也是非必需：
	观察 `addAtHead` 和 `addAtIndex(0, val)`。
	- **如果没有虚拟头节点**：需要写 `if (index == 0) { head = newNode; }`。
	- **有了虚拟头节点**：无论插入哪里，逻辑统一为 `prev->next = newNode; newNode->next = next;`。这就是为什么 `cur` 初始化为 `_dummyHead` 的原因。

在 `deleteIndex` 部分，为了删除第 `index` 个节点， `curr` 指针必须停在第 `index - 1` 个节点（即前驱节点）上。

跟踪一下 `index = 2`（删除第三个节点）的情况：
- **初始**：`curr` 指向 `dummyHead`（虚拟头节点，可以看作下标 -1）。
- **第 1 次循环**：`index` 变成 1，`curr` 移动到下标 0 的节点。
- **第 2 次循环**：`index` 变成 0，`curr` 移动到下标 1 的节点。
- **结束**：循环停止。此时 `curr` 恰好停在下标 1 的节点上，也就是下标 2 的**前一个**。

这就是使用 **虚拟头节点 (Dummy Head)** 的精妙之处：它让你即使想删除下标为 0 的节点，也能找到它的“前一个”（即 `dummyHead`），从而统一了所有删除逻辑。

## 总结

- 如果想 **修改/删除** 下标为 `n` 的节点，指针必须停在 `n-1`。
- 如果只想 **读取 (get)** 下标为 `n` 的节点，指针最后停在 `n` 即可。