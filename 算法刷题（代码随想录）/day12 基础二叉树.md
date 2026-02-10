# 二叉树的遍历

## DFS

### 递归遍历

> 三个特殊位置，代码执行的时机不同，带来不同的遍历顺序

递归遍历的代码模板

```cpp
class TreeNode{
public:
	int val;
	TreeNode* left,right;
	TreeNode(int x):val(x),left(nullptr),right(nullptr){}
}

void traverse(TreeNode* root){
	if(root==nullptr){
		return;
	}
	traverse(root->left);
	traverse(root->right);
}

//当然也可以先走右子树，再走左子树
void traverseFlip(TreeNode* root){
	if(root==nullptr){
		return;
	}
	traverseFlip(root->right);
	traverseFlip(root->left);
}
```

递归遍历的顺序都是如此，也就是 `traverse` 函数访问节点的顺序是固定的，但是在 `traverse` 函数的不同位置写代码，效果却又不同。

```cpp
// 二叉树的遍历框架
void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    traverse(root->left);
    // 中序位置
    traverse(root->right);
    // 后序位置
}
```

![[Pasted image 20260210220002.png]]

### 迭代法遍历

利用**栈**实现。
#### 前序遍历

**逻辑：** 每次从栈顶弹出一个节点，放入结果集，然后将其**右孩子**和**左孩子**依次压入栈中（因为栈是先入后出，所以左孩子要后压入，才能先处理）。

```cpp
vector<int> preorderTraversal(TreeNode* root) {
    vector<int> res;
    if (root == nullptr) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        st.pop();
        res.push_back(node->val);           // 中
        if (node->right) st.push(node->right); // 右（先入栈）
        if (node->left) st.push(node->left);   // 左（后入栈，先出栈）
    }
    return res;
}
```

#### 后序遍历

**逻辑：** 这是一个非常精妙的“反转”技巧。

1. 我们按照 **“中-右-左”** 的顺序进行类似于前序遍历的操作。
2. 得到结果集后，将其整个**翻转**（reverse），顺序就变成了 **“左-右-中”**。

```cpp
vector<int> postorderTraversal(TreeNode* root) {
    vector<int> res;
    if (root == nullptr) return res;
    stack<TreeNode*> st;
    st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        st.pop();
        res.push_back(node->val);           // 中
        if (node->left) st.push(node->left);   // 左（注意：这里先左后右）
        if (node->right) st.push(node->right); // 右
    }
    // 此时结果是 中-右-左，反转后变成 左-右-中
    reverse(res.begin(), res.end());
    return res;
}

```

#### 中序遍历

**逻辑：** 需要一个指针 `cur` 来辅助。

1. 只要 `cur` 不为空，就一直往左钻，并把路径上的节点全部压栈。
2. 到头了，就从栈里弹出一个（这就是最左边的节点），处理它。
3. 然后转向它的右子树，重复上述过程。

```cpp
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> res;
    stack<TreeNode*> st;
    TreeNode* cur = root;
    while (cur != nullptr || !st.empty()) {
        if (cur != nullptr) { // 步骤 1: 一路向左
            st.push(cur);
            cur = cur->left;
        } else {              // 步骤 2: 弹出并转向右
            cur = st.top();
            st.pop();
            res.push_back(cur->val); // 处理“中”
            cur = cur->right;        // 转向“右”
        }
    }
    return res;
}
```

## 统一迭代法

为了解决传统迭代法中“访问”和“处理”顺序不一致的问题，我们引入 `nullptr` 作为标记。

- **原理**：将要处理的节点放入栈中，并在其后紧跟一个 `nullptr`。
- **适用性**：只需调整压栈顺序，即可完美统一前中后序。

```cpp
vector<int> inorderTraversal(TreeNode* root) {
    vector<int> result;
    stack<TreeNode*> st;
    if (root) st.push(root);
    while (!st.empty()) {
        TreeNode* node = st.top();
        if (node != nullptr) {
            st.pop(); 
            // 中序顺序：左-中-right -> 压栈顺序：右-中-左
            if (node->right) st.push(node->right); // 右
            st.push(node);                         // 中
            st.push(nullptr);                      // 标记
            if (node->left) st.push(node->left);   // 左
        } else {
            st.pop(); // 弹出空标记
            node = st.top(); st.pop();
            result.push_back(node->val);
        }
    }
    return result;
}
```

## 层序便利（BFS）

> 三种不同写法，适用于不同场景

![[Pasted image 20260210220507.png]]

层序遍历就是一层层遍历嘛、我们可以利用**队列**来实现：

### 写法一

> 这是最简单的写法，但是不能知道节点的层数，然而知道节点的层数是常见的需求，比如收集每一层的节点，或者计算二叉树的最小深度等等、、所以虽然这样写简单，但是实际用的不多

```cpp
void levelOrderTraverse(TreeNode* root){
	if(root==nullptr)
		return;
	std::queue<TreeNode*>q;
	q.push(root);
	while(!q.empty()){
		TreeNode* cur = q.front();
		q.pop();
		std::cout<<cur->val<<std::endl;
		if(cur->left!=nullptr)
			q.push(cur->left);
		if(cur->right!=nullptr)
			q.push(cur->right);
	}
}
```

简单来看，无非就是每次把（子）树的根节点拿出来，然后把这个根节点的左右节点加入队列，循环此步骤即可。

### 写法二

```cpp
void levelOrderTraverse(TreeNode* root){
	if(root==nullptr) return;
	queue<TreeNode*> q;
	q.push(root);
	
	int depth = 1;
	
	while(!q.empty()){
		int sz = q.size();
		for(int i = 0; i <sz; i++){ //这里的for循环也可以写作while(sz-- > 0)，只需要知道这里的i遍历是每一 层 的节点
			TreeNode* cur=q.front();
			q.pop();
			std::cout<<"depth = "<<depth<<",val = "<<cur->val<<std::endl;
			if(cur->left!=nullptr) q.push(cur->left);
			if(cur->right!=nullptr) q.push(cur->right);
		}
		depth++;
	}
}
```

这种写法就可以记录下来每个节点所在的层数，可以解决如二叉树最小深度的问题，最为常用。

### 写法三

注意到写法二中，每遍历一层我们就 `depth++`，也就是我们每一层的权重都是 1, 也可以理解为每条树枝的权重是 1, 二叉树中的深度其实就是从根节点到这个节点的路径权重之和，且每一层的所有节点的路径权重和都是相同的。

但是倘若每条树枝的权重可以是任意的，现在层序遍历整棵树，打印每个节点的路径权重和，就会有不同的结果（类似于带权图），这就引出了我们的三个写法：在写法一的基础上添加一个 `state` 类，让每个节点自己负责维护自己的路径权重和：

```cpp
class Srate{
public:
	TreeNode* node;
	int depth;
	State(TreeNode* node,int depth):node(node),depth(depth){}
};

void levelOrderTraverse(TreeNode* root){
	if(root==nullptr) return;
	queue<State> q;
	q.push(State(root,1));//根节点的路径权重和是1
	while(!q.empty()){
		State cur = q.front();
		q.pop();
		std::cout<<"depth = "<<cur.depth<<", val = "<<cur.node->val<<std::endl;
		// 把 cur 的左右子节点加入队列
		if(cur->left!=nullptr) q.push(State(cur->left,cur.depth+1));
		if(cur->right!=nullptr) q.push(State(cur->right,cur->depth+1));
	}
}	
```

### 核心代码

```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    queue<TreeNode*> q;
    if (root) q.push(root);
    vector<vector<int>> result;
    while (!q.empty()) {
        int sz = q.size(); // 锁定当前层的节点数
        vector<int> vec;
        for (int i = 0; i < sz; i++) {
            TreeNode* node = q.front(); q.pop();
            vec.push_back(node->val);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
        result.push_back(vec);
    }
    return result;
}
```
# DFS 和 BFS 的适用场景

在实际算法中，**DFS 常常用于穷举所有路径，而 BFS 算法常常用于找寻最短路径**。

比如 [111. 二叉树的最小深度](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)就可以用 BFS 来做，一旦走到叶子节点就立马返回，保证深度最小的同时，尽可能的少遍历：

```cpp
class Solution {
public:
    int minDepth(TreeNode *root) {
        if (root == nullptr) return 0;
        
        std::queue<TreeNode *> q;
        q.push(root);
        // root 本身就是第一层，所以初始化为 1
        int depth = 1;

        while (!q.empty()) {
            int sz = q.size(); // 记录当前层的节点数量
            
            // 修正点：使用定义的变量名 sz
            for (int i = 0; i < sz; i++) {
                TreeNode *cur = q.front();
                q.pop();

                // BFS 的精髓：一旦发现叶子节点，立即返回当前深度
                if (cur->left == nullptr && cur->right == nullptr) {
                    return depth;
                }

                if (cur->left != nullptr) q.push(cur->left);
                if (cur->right != nullptr) q.push(cur->right);
            }
            // 这一层走完了，还没到叶子，深度加 1
            depth++;
        }
        return depth;
    }
};
```

DFS 遍历当然也可以用来寻找最短路径，但必须遍历完所有节点才能得到最短路径。

从时间复杂度的角度来看，两种算法在最坏情况下都会遍历所有节点，时间复杂度都是 $O(N)$，但在一般情况下，显然 BFS 算法的实际效率会更高。**所以在寻找最短路径的问题中，BFS 算法是首选**。

然而 DFS 适合于寻找所有路径，因为它本身就是一条树枝一条树枝的从左往右遍历，每一条树枝就是一条路径，递归遍历到叶子节点的时候递归路径就是一条树枝，所以 DFS 天然适合找寻所有路径。

# 多叉树的递归/层序遍历

首先明确：
- 多叉树就是二叉树的延伸，二叉树是特殊的多叉树
- 多叉树的遍历就是二叉树遍历的延伸
- 森林是指多个多叉树的集合，单独一棵多叉树是一个特殊的森林
- 森林在**并查集**算法会用到

二叉树节点：

```cpp
class TreeNode {
public:
    int val;
    TreeNode* left;
    TreeNode* right;

    TreeNode(int v) : val(v), left(nullptr), right(nullptr) {}
};
```

多叉树节点：

```cpp
class Node {
public:
    int val;
    vector<Node*> children; // 每个节点可以有任意多个子节点
};
```

森林就是多个多叉树的集合，可以这样表述：

```cpp
List<Node> forest;
```

只需要对每个根节点分别进行 DFS/BFS 遍历即可遍历森林的所有节点。

在并查集算法中，我们会同时持有多棵多叉树的根节点，那么这些根节点的集合就是一个森林。

## 多叉树的遍历

无法也就是递归遍历（DFS）和层序遍历（BFS）

### 递归遍历（DFS）

对比二叉树来看，无法我们手动写的 `left` 和 `right` 变成了一个 `for` 循环（因为不止两个孩子了嘛）。但是要注意一点的是，多叉树没有中序遍历的概念。

```cpp
// 二叉树的遍历框架
void traverse(TreeNode* root) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    traverse(root->left);
    // 中序位置
    traverse(root->right);
    // 后序位置
}

// N 叉树的遍历框架
void traverse(Node* root) {
    if (root == nullptr) {
        return;
    }
    // 前序位置
    for (Node* child : root->children) {
        traverse(child);
    }
    // 后序位置
}
```

### 层序遍历（BFS）

和二叉树的层序遍历一样，都是通过队列来实现，无非把二叉树的左右子节点换成了多叉树的所有子节点。所以类似二叉树，多叉树也是三种：

#### 第一种

> 无法记录节点深度

```cpp
void levelOrderTraverse(Node* root) {
    if (root == nullptr) {
        return;
    }
    std::queue<Node*> q;
    q.push(root);
    while (!q.empty()) {
        Node* cur = q.front();
        q.pop();
        // 访问 cur 节点
        std::cout << cur->val << std::endl;

        // 把 cur 的所有子节点加入队列
        for (Node* child : cur->children) {
            q.push(child);
        }
    }
}
```

#### 第二种

> 能够记录节点深度

```cpp
#include <iostream>
#include <queue>
#include <vector>

void levelOrderTraverse(Node* root) {
    if (root == nullptr) {
        return;
    }
    std::queue<Node*> q;
    q.push(root);
    // 记录当前遍历到的层数（根节点视为第 1 层）
    int depth = 1;

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            Node* cur = q.front();
            q.pop();
            // 访问 cur 节点，同时知道它所在的层数
            std::cout << "depth = " << depth << ", val = " << cur->val << std::endl;

            for (Node* child : cur->children) {
                q.push(child);
            }
        }
        depth++;
    }
}
```

#### 第三种

> 能够适配不同权重的路径

```cpp
// 多叉树的层序遍历
// 每个节点自行维护 State 类，记录深度等信息
class State {
public:
    Node* node;
    int depth;

    State(Node* node, int depth) : node(node), depth(depth) {}
};

void levelOrderTraverse(Node* root) {
    if (root == nullptr) {
        return;
    }
    std::queue<State> q;
    // 记录当前遍历到的层数（根节点视为第 1 层）
    q.push(State(root, 1));

    while (!q.empty()) {
        State state = q.front();
        q.pop();
        Node* cur = state.node;
        int depth = state.depth;
        // 访问 cur 节点，同时知道它所在的层数
        std::cout << "depth = " << depth << ", val = " << cur->val << std::endl;

        for (Node* child : cur->children) {
            q.push(State(child, depth + 1));
        }
    }
}
```


# 二叉树刷题总结 (2026-02-11)

## 第一部分：二叉树的基础遍历 (DFS)

**核心思想**：利用栈（系统栈或显式栈）进行深度优先搜索。

### 144. 二叉树的前序遍历 (Preorder Traversal)

**顺序**：中 -> 左 -> 右

#### 解法一：递归法 (Recursive)

```cpp
class Solution {
public:
    void traverse(TreeNode* cur, vector<int>& res) {
        if (cur == nullptr) return;
        res.push_back(cur->val);    // 中
        traverse(cur->left, res);   // 左
        traverse(cur->right, res);  // 右
    }
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        traverse(root, res);
        return res;
    }
};
```

#### 解法二：迭代法 (Iterative)

**思路**：利用栈。先压入右孩子，再压入左孩子（出栈顺序就是中-左-右）。

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> res;
        if (root == nullptr) return res;
        stack<TreeNode*> st;
        st.push(root);
        while (!st.empty()) {
            TreeNode* node = st.top();
            st.pop();
            res.push_back(node->val);
            if (node->right) st.push(node->right);
            if (node->left) st.push(node->left);
        }
        return res;
    }
};
```

---

### 145. 二叉树的后序遍历 (Postorder Traversal)

**顺序**：左 -> 右 -> 中

#### 解法一：递归法 (Recursive)

```cpp
class Solution {
public:
    void traverse(TreeNode* cur, vector<int>& res) {
        if (cur == nullptr) return;
        traverse(cur->left, res);
        traverse(cur->right, res);
        res.push_back(cur->val);    // 中
    }
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        traverse(root, res);
        return res;
    }
};
```

#### 解法二：迭代法 (Iterative - 巧解)

**思路**：前序是“中左右”，调整为“中右左”，然后翻转结果数组，得到“左右中”。

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
        vector<int> res;
        if (root == nullptr) return res;
        stack<TreeNode*> st;
        st.push(root);
        while (!st.empty()) {
            TreeNode* node = st.top();
            st.pop();
            res.push_back(node->val);
            if (node->left) st.push(node->left);   // 注意这里先压左
            if (node->right) st.push(node->right); // 后压右
        }
        reverse(res.begin(), res.end()); // 翻转
        return res;
    }
};
```

---

### 94. 二叉树的中序遍历 (Inorder Traversal)

**顺序**：左 -> 中 -> 右

#### 解法一：递归法 (Recursive)

```cpp
class Solution {
public:
    void traverse(TreeNode* cur, vector<int>& res) {
        if (cur == nullptr) return;
        traverse(cur->left, res);
        res.push_back(cur->val);    // 中
        traverse(cur->right, res);
    }
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        traverse(root, res);
        return res;
    }
};
```

#### 解法二：迭代法 (Iterative - 指针访问)

**思路**：利用指针一路向左钻到底，直到为空再弹栈处理。

```cpp
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> res;
        stack<TreeNode*> st;
        TreeNode* cur = root;
        while (cur != nullptr || !st.empty()) {
            if (cur != nullptr) {
                st.push(cur);
                cur = cur->left; // 一路向左
            } else {
                cur = st.top();
                st.pop();
                res.push_back(cur->val); // 处理中
                cur = cur->right;        // 转向右
            }
        }
        return res;
    }
};
```

---

## 第二部分：层序遍历 (BFS) - 队列模板

**核心思想**：利用 `std::queue` 和 `size` 变量控制每一层的遍历。

### 102. 二叉树的层序遍历 (Level Order Traversal)

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> res;
        if (!root) return res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            vector<int> level;
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front(); q.pop();
                level.push_back(node->val);
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            res.push_back(level);
        }
        return res;
    }
};
```

### 107. 二叉树的层序遍历 II (Bottom-up)

**思路**：在 102 的基础上，最后翻转结果数组。

```cpp
// 代码同上，只是在 return 前加一行：
reverse(res.begin(), res.end());
```

### 199. 二叉树的右视图 (Right Side View)

**思路**：BFS 遍历每一层，只记录该层的最后一个节点。

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> res;
        if (!root) return res;
        queue<TreeNode*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front(); q.pop();
                if (i == sz - 1) res.push_back(node->val); // 只记录最后一个
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
        }
        return res;
    }
};
```

### 637. 二叉树的层平均值 (Average of Levels)

**思路**：BFS 遍历每一层，求和取平均。

```cpp
// 核心循环内：
double sum = 0;
for (int i = 0; i < sz; i++) {
    // ... pop node ...
    sum += node->val;
    // ... push children ...
}
res.push_back(sum / sz);
```

### 515. 在每个树行中找最大值 (Largest Value in Each Tree Row)

**思路**：BFS 遍历每一层，用 `INT_MIN` 比较求最大值。

```cpp
// 核心循环内：
int maxVal = INT_MIN; // 或 node->val
for (int i = 0; i < sz; i++) {
    // ... pop node ...
    maxVal = max(maxVal, node->val);
    // ... push children ...
}
res.push_back(maxVal);
```

---

## 第三部分：树的深度 (DFS vs BFS)

### 104. 二叉树的最大深度 (Max Depth)

**推荐解法：DFS (递归)**

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) return 0;
        return 1 + max(maxDepth(root->left), maxDepth(root->right));
    }
};
```

### 111. 二叉树的最小深度 (Min Depth)

**推荐解法：BFS (迭代)** —— 遇到叶子节点立即返回。

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root) return 0;
        queue<TreeNode*> q;
        q.push(root);
        int depth = 1;
        while (!q.empty()) {
            int sz = q.size();
            for (int i = 0; i < sz; i++) {
                TreeNode* node = q.front(); q.pop();
                // 是叶子节点？直接返回！
                if (!node->left && !node->right) return depth;
                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }
            depth++;
        }
        return depth;
    }
};
```

---

## 第四部分：结构修改 (Next 指针)

### 116. 填充每个节点的下一个右侧节点指针 (Perfect Binary Tree)

### 117. 填充每个节点的下一个右侧节点指针 II (General Binary Tree)

**通用解法：BFS + pre 指针**

无论树是否完美，这个解法都适用。

```cpp
class Solution {
public:
    Node* connect(Node* root) {
        if (!root) return nullptr;
        queue<Node*> q;
        q.push(root);
        while (!q.empty()) {
            int sz = q.size();
            Node* pre = nullptr;
            for (int i = 0; i < sz; i++) {
                Node* cur = q.front(); q.pop();
                if (pre) pre->next = cur; // 连线
                pre = cur;                // 移动 pre
                if (cur->left) q.push(cur->left);
                if (cur->right) q.push(cur->right);
            }
        }
        return root;
    }
};
```