```cpp
#include <iostream>
#include <algorithm> // 用于 std::max
#include <memory>    // 用于 std::unique_ptr

template<typename K, typename V>
class AVLMap {
private:
    struct AVLNode {
        K key;
        V value;
        std::unique_ptr<AVLNode> left;
        std::unique_ptr<AVLNode> right;
        int height;

        AVLNode(const K& k, const V& v)
            : key(k), value(v), left(nullptr), right(nullptr), height(1) {}
    };

    std::unique_ptr<AVLNode> root;
    size_t map_size;

    // 获取节点高度
    int getHeight(const std::unique_ptr<AVLNode>& node) {
        if (!node) return 0;
        return node->height;
    }

    // 获取平衡因子
    int getBalance(const std::unique_ptr<AVLNode>& node) {
        if (!node) return 0;
        return getHeight(node->left) - getHeight(node->right);
    }

    // 更新节点高度
    void updateHeight(std::unique_ptr<AVLNode>& node) {
        if (node) {
            node->height = 1 + std::max(getHeight(node->left), getHeight(node->right));
        }
    }
    
    // 右旋
    std::unique_ptr<AVLNode> rightRotate(std::unique_ptr<AVLNode> y) {
        std::unique_ptr<AVLNode> x = std::move(y->left);
        std::unique_ptr<AVLNode> T2 = std::move(x->right);

        x->right = std::move(y);
        x->right->left = std::move(T2);

        updateHeight(x->right);
        updateHeight(x);

        return x;
    }

    // 左旋
    std::unique_ptr<AVLNode> leftRotate(std::unique_ptr<AVLNode> x) {
        std::unique_ptr<AVLNode> y = std::move(x->right);
        std::unique_ptr<AVLNode> T2 = std::move(y->left);

        y->left = std::move(x);
        y->left->right = std::move(T2);

        updateHeight(y->left);
        updateHeight(y);

        return y;
    }

    // 插入节点的核心逻辑
    std::unique_ptr<AVLNode> insertNode(std::unique_ptr<AVLNode> node, const K& key, const V& value) {
        if (!node) {
            map_size++;
            return std::make_unique<AVLNode>(key, value);
        }

        if (key < node->key) {
            node->left = insertNode(std::move(node->left), key, value);
        } else if (key > node->key) {
            node->right = insertNode(std::move(node->right), key, value);
        } else { // 键已存在，更新值
            node->value = value;
            return node;
        }

        updateHeight(node);

        int balance = getBalance(node);

        // 左-左情况
        if (balance > 1 && key < node->left->key) {
            return rightRotate(std::move(node));
        }

        // 右-右情况
        if (balance < -1 && key > node->right->key) {
            return leftRotate(std::move(node));
        }

        // 左-右情况
        if (balance > 1 && key > node->left->key) {
            node->left = leftRotate(std::move(node->left));
            return rightRotate(std::move(node));
        }

        // 右-左情况
        if (balance < -1 && key < node->right->key) {
            node->right = rightRotate(std::move(node->right));
            return leftRotate(std::move(node));
        }

        return node;
    }

    // 查找具有最小键的节点
    AVLNode* minValueNode(AVLNode* node) {
        AVLNode* current = node;
        while (current && current->left) {
            current = current->left.get();
        }
        return current;
    }
    
    // 删除节点的核心逻辑
    std::unique_ptr<AVLNode> deleteNode(std::unique_ptr<AVLNode> node, const K& key) {
        if (!node) return nullptr;

        if (key < node->key) {
            node->left = deleteNode(std::move(node->left), key);
        } else if (key > node->key) {
            node->right = deleteNode(std::move(node->right), key);
        } else {
            // 找到要删除的节点
            if (!node->left || !node->right) {
                // 只有一个子节点或没有子节点
                auto temp = std::move(node->left ? node->left : node->right);
                map_size--;
                return temp;
            } else {
                // 有两个子节点
                AVLNode* temp = minValueNode(node->right.get());
                node->key = temp->key;
                node->value = temp->value;
                node->right = deleteNode(std::move(node->right), temp->key);
            }
        }
        
        if (!node) return nullptr; // 如果删除后树变为空

        updateHeight(node);

        int balance = getBalance(node);

        // 左-左情况
        if (balance > 1 && getBalance(node->left) >= 0) {
            return rightRotate(std::move(node));
        }

        // 左-右情况
        if (balance > 1 && getBalance(node->left) < 0) {
            node->left = leftRotate(std::move(node->left));
            return rightRotate(std::move(node));
        }

        // 右-右情况
        if (balance < -1 && getBalance(node->right) <= 0) {
            return leftRotate(std::move(node));
        }

        // 右-左情况
        if (balance < -1 && getBalance(node->right) > 0) {
            node->right = rightRotate(std::move(node->right));
            return leftRotate(std::move(node));
        }

        return node;
    }
    
    // 中序遍历辅助函数
    void inOrderTraversalHelper(const std::unique_ptr<AVLNode>& node) const {
        if (node) {
            inOrderTraversalHelper(node->left);
            std::cout << "Key: " << node->key << ", Value: " << node->value << std::endl;
            inOrderTraversalHelper(node->right);
        }
    }
    
    // 查找节点辅助函数
    AVLNode* findNode(AVLNode* node, const K& key) const {
        if (!node || node->key == key) {
            return node;
        }

        if (key < node->key) {
            return findNode(node->left.get(), key);
        } else {
            return findNode(node->right.get(), key);
        }
    }


public:
    AVLMap() : root(nullptr), map_size(0) {}

    // 插入键值对
    void insert(const K& key, const V& value) {
        root = insertNode(std::move(root), key, value);
    }
    
    // 删除键值对
    void erase(const K& key) {
        root = deleteNode(std::move(root), key);
    }
    
    // 查找键并返回值的指针
    V* find(const K& key) const {
        AVLNode* node = findNode(root.get(), key);
        if (node) {
            return &node->value;
        }
        return nullptr;
    }
    
    // 重载 [] 操作符
    V& operator[](const K& key) {
        AVLNode* node = findNode(root.get(), key);
        if (node) {
            return node->value;
        } else {
            // 如果键不存在，插入一个具有默认值的键
            insert(key, V{});
            // 再次查找以返回引用
            node = findNode(root.get(), key);
            return node->value;
        }
    }

    // 获取大小
    size_t size() const {
        return map_size;
    }

    // 检查是否为空
    bool empty() const {
        return map_size == 0;
    }
    
    // 中序遍历打印
    void in_order_traversal() const {
        inOrderTraversalHelper(root);
        std::cout << "--- End of Traversal ---" << std::endl;
    }
};

int main() {
    AVLMap<int, std::string> myMap;

    std::cout << "Is map empty? " << (myMap.empty() ? "Yes" : "No") << std::endl;

    std::cout << "\nInserting elements..." << std::endl;
    myMap.insert(10, "Apple");
    myMap.insert(20, "Banana");
    myMap.insert(30, "Cherry");
    myMap.insert(40, "Date");
    myMap.insert(50, "Elderberry");
    myMap.insert(25, "Fig");

    std::cout << "Map size: " << myMap.size() << std::endl;
    std::cout << "In-order traversal:" << std::endl;
    myMap.in_order_traversal();

    std::cout << "\nUsing operator[] to access and modify elements:" << std::endl;
    myMap[10] = "Avocado"; // 修改已存在的值
    myMap[60] = "Grape";   // 插入新值
    std::cout << "Value for key 10: " << myMap[10] << std::endl;
    std::cout << "Value for key 60: " << myMap[60] << std::endl;

    std::cout << "\nMap size after using operator[]: " << myMap.size() << std::endl;
    std::cout << "In-order traversal:" << std::endl;
    myMap.in_order_traversal();

    std::cout << "\nFinding elements:" << std::endl;
    std::string* val = myMap.find(20);
    if (val) {
        std::cout << "Found key 20 with value: " << *val << std::endl;
    } else {
        std::cout << "Key 20 not found." << std::endl;
    }

    val = myMap.find(100);
    if (val) {
        std::cout << "Found key 100 with value: " << *val << std::endl;
    } else {
        std::cout << "Key 100 not found." << std::endl;
    }
    
    std::cout << "\nDeleting elements..." << std::endl;
    myMap.erase(30); // 删除一个中间节点
    std::cout << "Deleted key 30. Map size: " << myMap.size() << std::endl;
    myMap.in_order_traversal();
    
    myMap.erase(10); // 删除一个节点
    std::cout << "Deleted key 10. Map size: " << myMap.size() << std::endl;
    myMap.in_order_traversal();

    myMap.erase(50); // 删除一个节点
    std::cout << "Deleted key 50. Map size: " << myMap.size() << std::endl;
    myMap.in_order_traversal();

    return 0;
}
```

`deleteNode` 部分可以这样写：
```cpp
// 在 deleteNode 函数的平衡部分
// ...
if (balance > 1) { // 左边重
    if (getBalance(node->left) < 0) { // 子节点右重 -> LR 情况
        node->left = leftRotate(node->left);
    }
    return rightRotate(node); // LL 或转化后的 LR 情况
}

if (balance < -1) { // 右边重
    if (getBalance(node->right) > 0) { // 子节点左重 -> RL 情况
        node->right = rightRotate(node->right);
    }
    return leftRotate(node); // RR 或转化后的 RL 情况
}
```

## unordermap

```cpp
#pragma once

#include <vector>
#include <functional>
#include <utility>
#include <iterator> // For std::forward_iterator_tag

template <typename K, typename V, typename Hasher = std::hash<K>>
class UnorderedMap {
private:
    // **[修改]** 节点结构现在存储 std::pair
    struct HashNode {
        std::pair<const K, V> data;
        HashNode* next;

        HashNode(const std::pair<const K, V>& p) : data(p), next(nullptr) {}
    };

    std::vector<HashNode*> buckets_;
    size_t size_;
    size_t bucket_count_;
    float max_load_factor_;
    Hasher hasher_;

    size_t get_bucket_index(const K& key) const {
        return hasher_(key) % bucket_count_;
    }

    void rehash(size_t new_bucket_count); // 声明 rehash

public:
    // --- [新增] 迭代器类 ---
    class iterator {
    private:
        UnorderedMap* map_ptr_;
        HashNode* current_node_;
        size_t bucket_index_;

    public:
        // C++ 迭代器标准类型定义
        using iterator_category = std::forward_iterator_tag;
        using value_type = std::pair<const K, V>;
        using difference_type = std::ptrdiff_t;
        using pointer = value_type*;
        using reference = value_type&;

        iterator(UnorderedMap* map, HashNode* node, size_t index)
            : map_ptr_(map), current_node_(node), bucket_index_(index) {}
        
        // 解引用
        reference operator*() const {
            return current_node_->data;
        }

        pointer operator->() const {
            return &(current_node_->data);
        }

        // 前缀自增 ++it
        iterator& operator++() {
            if (current_node_->next != nullptr) {
                // 如果当前链表还有节点，移动到下一个
                current_node_ = current_node_->next;
            } else {
                // 否则，寻找下一个非空的桶
                bucket_index_++;
                while (bucket_index_ < map_ptr_->bucket_count_ && map_ptr_->buckets_[bucket_index_] == nullptr) {
                    bucket_index_++;
                }

                if (bucket_index_ < map_ptr_->bucket_count_) {
                    current_node_ = map_ptr_->buckets_[bucket_index_];
                } else {
                    // 已经到达末尾
                    current_node_ = nullptr;
                }
            }
            return *this;
        }

        // 后缀自增 it++
        iterator operator++(int) {
            iterator old = *this;
            ++(*this);
            return old;
        }

        bool operator==(const iterator& other) const {
            return current_node_ == other.current_node_;
        }

        bool operator!=(const iterator& other) const {
            return !(*this == other);
        }
    };
    // --- 迭代器类结束 ---

    UnorderedMap(size_t initial_bucket_count = 10);
    ~UnorderedMap();

    // --- [新增] begin() 和 end() ---
    iterator begin() {
        if (size_ == 0) {
            return end();
        }
        size_t first_bucket = 0;
        while (first_bucket < bucket_count_ && buckets_[first_bucket] == nullptr) {
            first_bucket++;
        }
        return iterator(this, buckets_[first_bucket], first_bucket);
    }

    iterator end() {
        // end() 迭代器通常指向一个不存在的位置，用 nullptr 节点表示
        return iterator(this, nullptr, bucket_count_);
    }

    // --- [修改] 其他成员函数以适应新结构 ---
    void clear();
    iterator find(const K& key);
    bool insert(const K& key, const V& value);
    bool erase(const K& key);
    V& operator[](const K& key);

    size_t size() const { return size_; }
    bool empty() const { return size_ == 0; }
};

// --- 实现部分 ---

// 构造函数
template <typename K, typename V, typename Hasher>
UnorderedMap<K, V, Hasher>::UnorderedMap(size_t initial_bucket_count) 
    : size_(0), bucket_count_(initial_bucket_count), max_load_factor_(0.75f) {
    if (bucket_count_ == 0) bucket_count_ = 1;
    buckets_.resize(bucket_count_, nullptr);
}

// 析构函数
template <typename K, typename V, typename Hasher>
UnorderedMap<K, V, Hasher>::~UnorderedMap() {
    clear();
}

// clear
template <typename K, typename V, typename Hasher>
void UnorderedMap<K, V, Hasher>::clear() {
    for (size_t i = 0; i < bucket_count_; ++i) {
        HashNode* current = buckets_[i];
        while (current != nullptr) {
            HashNode* to_delete = current;
            current = current->next;
            delete to_delete;
        }
        buckets_[i] = nullptr;
    }
    size_ = 0;
}

// insert
template <typename K, typename V, typename Hasher>
bool UnorderedMap<K, V, Hasher>::insert(const K& key, const V& value) {
    if ((float)(size_ + 1) / bucket_count_ > max_load_factor_) {
        rehash(bucket_count_ * 2);
    }
    
    auto it = find(key);
    if (it != end()) {
        // 键已存在，更新值
        it->second = value;
        return false;
    }

    size_t index = get_bucket_index(key);
    // **[修改]** 使用 std::pair 构造节点
    HashNode* new_node = new HashNode({key, value});
    new_node->next = buckets_[index];
    buckets_[index] = new_node;
    size_++;
    return true;
}

// rehash
template <typename K, typename V, typename Hasher>
void UnorderedMap<K, V, Hasher>::rehash(size_t new_bucket_count) {
    std::vector<HashNode*> new_buckets(new_bucket_count, nullptr);

    for (size_t i = 0; i < bucket_count_; ++i) {
        HashNode* current = buckets_[i];
        while (current != nullptr) {
            HashNode* next_node = current->next;
            // **[修改]** 使用 data.first 获取键
            size_t new_index = hasher_(current->data.first) % new_bucket_count;
            current->next = new_buckets[new_index];
            new_buckets[new_index] = current;
            current = next_node;
        }
    }

    buckets_ = std::move(new_buckets);
    bucket_count_ = new_bucket_count;
}

// find
template <typename K, typename V, typename Hasher>
typename UnorderedMap<K, V, Hasher>::iterator UnorderedMap<K, V, Hasher>::find(const K& key) {
    size_t index = get_bucket_index(key);
    HashNode* current = buckets_[index];

    while (current != nullptr) {
        // **[修改]** 使用 data.first 比较键
        if (current->data.first == key) {
            return iterator(this, current, index);
        }
        current = current->next;
    }
    return end();
}

// erase
template <typename K, typename V, typename Hasher>
bool UnorderedMap<K, V, Hasher>::erase(const K& key) {
    size_t index = get_bucket_index(key);
    HashNode* current = buckets_[index];
    HashNode* prev = nullptr;

    while (current != nullptr) {
        // **[修改]** 使用 data.first 比较键
        if (current->data.first == key) {
            if (prev == nullptr) {
                buckets_[index] = current->next;
            } else {
                prev->next = current->next;
            }
            delete current;
            size_--;
            return true;
        }
        prev = current;
        current = current->next;
    }
    return false;
}

// operator[]
template <typename K, typename V, typename Hasher>
V& UnorderedMap<K, V, Hasher>::operator[](const K& key) {
    iterator it = find(key);
    if (it != end()) {
        // **[修改]** 迭代器指向 pair，所以用 ->second
        return it->second;
    } else {
        insert(key, V{});
        // **[修改]** 再次查找并返回
        return find(key)->second;
    }
}
```

**添加注释**
```cpp
#pragma once

#include <vector>
#include <functional>
#include <utility>
#include <iterator> // 提供 std::forward_iterator_tag

// ==========================
// 自定义哈希表类 UnorderedMap
// 使用单链表 + 拉链法解决冲突
// 提供类似 std::unordered_map 的接口
// ==========================
template <typename K, typename V, typename Hasher = std::hash<K>>
class UnorderedMap {
private:
    // 哈希表节点结构（链表节点）
    struct HashNode {
        std::pair<const K, V> data; // 存储键值对，key 是 const，保证不可修改
        HashNode* next;             // 指向下一个节点（链表）

        // 构造函数，接受键值对
        HashNode(const std::pair<const K, V>& p) : data(p), next(nullptr) {}
    };

    std::vector<HashNode*> buckets_; // 哈希桶数组，每个桶指向一个链表
    size_t size_;                    // 当前存储的元素个数
    size_t bucket_count_;            // 桶的数量
    float max_load_factor_;          // 最大负载因子 (size/bucket_count)
    Hasher hasher_;                  // 哈希函数对象

    // 根据 key 计算哈希桶索引
    size_t get_bucket_index(const K& key) const {
        return hasher_(key) % bucket_count_;
    }

    // 扩容与重哈希函数声明
    void rehash(size_t new_bucket_count);

public:
    // ======================
    // 内部迭代器类
    // ======================
    class iterator {
    private:
        UnorderedMap* map_ptr_;   // 指向所属的哈希表
        HashNode* current_node_;  // 当前链表节点
        size_t bucket_index_;     // 当前所在的桶索引

    public:
        // C++ 迭代器必须定义的类型
        using iterator_category = std::forward_iterator_tag;
        using value_type = std::pair<const K, V>;
        using difference_type = std::ptrdiff_t;
        using pointer = value_type*;
        using reference = value_type&;

        // 构造函数
        iterator(UnorderedMap* map, HashNode* node, size_t index)
            : map_ptr_(map), current_node_(node), bucket_index_(index) {}
        
        // 解引用运算符 *it
        reference operator*() const {
            return current_node_->data;
        }

        // 成员访问运算符 it->first / it->second
        pointer operator->() const {
            return &(current_node_->data);
        }

        // 前缀自增 ++it
        iterator& operator++() {
            if (current_node_->next != nullptr) {
                // 如果链表还有后继节点，则移动到下一个
                current_node_ = current_node_->next;
            } else {
                // 否则跳到下一个非空桶
                bucket_index_++;
                while (bucket_index_ < map_ptr_->bucket_count_ &&
                       map_ptr_->buckets_[bucket_index_] == nullptr) {
                    bucket_index_++;
                }

                if (bucket_index_ < map_ptr_->bucket_count_) {
                    current_node_ = map_ptr_->buckets_[bucket_index_];
                } else {
                    // 走到最后，设为 nullptr
                    current_node_ = nullptr;
                }
            }
            return *this;
        }

        // 后缀自增 it++
        iterator operator++(int) {
            iterator old = *this;
            ++(*this);
            return old;
        }

        // 比较运算符
        bool operator==(const iterator& other) const {
            return current_node_ == other.current_node_;
        }

        bool operator!=(const iterator& other) const {
            return !(*this == other);
        }
    };
    // --- 迭代器类结束 ---

    // 构造函数：初始化桶
    UnorderedMap(size_t initial_bucket_count = 10);
    // 析构函数：释放所有节点
    ~UnorderedMap();

    // 获取 begin 迭代器
    iterator begin() {
        if (size_ == 0) {
            return end();
        }
        size_t first_bucket = 0;
        while (first_bucket < bucket_count_ && buckets_[first_bucket] == nullptr) {
            first_bucket++;
        }
        return iterator(this, buckets_[first_bucket], first_bucket);
    }

    // 获取 end 迭代器（nullptr 表示末尾）
    iterator end() {
        return iterator(this, nullptr, bucket_count_);
    }

    // 主要接口函数
    void clear();                   // 清空哈希表
    iterator find(const K& key);    // 查找 key 对应元素
    bool insert(const K& key, const V& value); // 插入键值对（存在则更新）
    bool erase(const K& key);       // 删除指定 key
    V& operator[](const K& key);    // 下标运算符，若不存在则插入默认值

    // 辅助接口
    size_t size() const { return size_; }
    bool empty() const { return size_ == 0; }
};

// ==========================
// 成员函数实现部分
// ==========================

// 构造函数
template <typename K, typename V, typename Hasher>
UnorderedMap<K, V, Hasher>::UnorderedMap(size_t initial_bucket_count) 
    : size_(0), bucket_count_(initial_bucket_count), max_load_factor_(0.75f) {
    if (bucket_count_ == 0) bucket_count_ = 1;
    buckets_.resize(bucket_count_, nullptr);
}

// 析构函数
template <typename K, typename V, typename Hasher>
UnorderedMap<K, V, Hasher>::~UnorderedMap() {
    clear(); // 清理所有节点
}

// 清空哈希表clear
template <typename K, typename V, typename Hasher>
void UnorderedMap<K, V, Hasher>::clear() {
    for (size_t i = 0; i < bucket_count_; ++i) {
        HashNode* current = buckets_[i];
        while (current != nullptr) {
            HashNode* to_delete = current;
            current = current->next;
            delete to_delete;
        }
        buckets_[i] = nullptr;
    }
    size_ = 0;
}

// 插入元素insert
template <typename K, typename V, typename Hasher>
bool UnorderedMap<K, V, Hasher>::insert(const K& key, const V& value) {
    // 如果超过负载因子阈值，扩容
    if ((float)(size_ + 1) / bucket_count_ > max_load_factor_) {
        rehash(bucket_count_ * 2);
    }
    
    // 先检查是否已存在
    auto it = find(key);
    if (it != end()) {
        it->second = value; // 如果存在，则更新 value
        return false;
    }

    // 插入新节点（头插法）
    size_t index = get_bucket_index(key);
    HashNode* new_node = new HashNode({key, value});
    new_node->next = buckets_[index];
    buckets_[index] = new_node;
    size_++;
    return true;
}

// 重哈希（扩容）rehash
template <typename K, typename V, typename Hasher>
void UnorderedMap<K, V, Hasher>::rehash(size_t new_bucket_count) {
    std::vector<HashNode*> new_buckets(new_bucket_count, nullptr);

    // 遍历旧桶，把节点重新分配到新桶
    for (size_t i = 0; i < bucket_count_; ++i) {
        HashNode* current = buckets_[i];
        while (current != nullptr) {
            HashNode* next_node = current->next;
            size_t new_index = hasher_(current->data.first) % new_bucket_count;
            current->next = new_buckets[new_index];
            new_buckets[new_index] = current;
            current = next_node;
        }
    }

    buckets_ = std::move(new_buckets);
    bucket_count_ = new_bucket_count;
}

// 查找元素find
template <typename K, typename V, typename Hasher>
typename UnorderedMap<K, V, Hasher>::iterator UnorderedMap<K, V, Hasher>::find(const K& key) {
    size_t index = get_bucket_index(key);
    HashNode* current = buckets_[index];

    while (current != nullptr) {
        if (current->data.first == key) {
            return iterator(this, current, index);
        }
        current = current->next;
    }
    return end();
}

// 删除元素erase
template <typename K, typename V, typename Hasher>
bool UnorderedMap<K, V, Hasher>::erase(const K& key) {
    size_t index = get_bucket_index(key);
    HashNode* current = buckets_[index];
    HashNode* prev = nullptr;

    while (current != nullptr) {
        if (current->data.first == key) {
            if (prev == nullptr) {
                buckets_[index] = current->next;
            } else {
                prev->next = current->next;
            }
            delete current;
            size_--;
            return true;
        }
        prev = current;
        current = current->next;
    }
    return false;
}

// 下标运算符
template <typename K, typename V, typename Hasher>
V& UnorderedMap<K, V, Hasher>::operator[](const K& key) {
    iterator it = find(key);
    if (it != end()) {
        return it->second; // 已存在，返回引用
    } else {
        insert(key, V{});  // 不存在则插入默认值
        return find(key)->second;
    }
}

```