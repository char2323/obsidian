# 🧩 C++ 四大类型转换运算符详解

| 运算符 | 功能  | 安全性 | 用途/场景 | 注意事项 | 示例  |
| --- | --- | --- | ----- | ---- | --- |

---

## 1 `static_cast<T>(expr)`

### 功能

- **编译期类型转换**
    
- 可做 **数值类型、枚举类型、指针上/下行转换（非多态）**
    
- 避免隐式转换歧义
    

### 安全性

- 编译期检查，但**不做运行时类型检查**
    
- 不安全的下行指针转换可能产生未定义行为
    

### 用途

1. 数值类型转换（int ↔ double）
    
2. 枚举与整数转换
    
3. 非多态指针/引用上/下行转换
    
4. void* 与具体指针之间的转换
    

### 示例

```cpp
// 数值类型
int a = 10;
double b = static_cast<double>(a);

// 枚举类型
enum class Color : uint8_t { Red, Green, Blue };
Color c = Color::Green;
uint8_t id = static_cast<uint8_t>(c);

// 指针上行转换
struct Base {};
struct Derived : Base {};
Derived d;
Base* pb = static_cast<Base*>(&d); // ✅ 安全

// 指针下行转换（非多态，需保证安全）
Derived* pd = static_cast<Derived>(pb); // ⚠️ 可能不安全
```

---

## 2 `dynamic_cast<T>(expr)`

### 功能

- **运行时类型检查的安全指针/引用转换**
    
- 主要用于 **多态类型**（有虚函数的类）
    
- 向下转换时，运行时检查类型是否正确
    

### 安全性

- **向下转换安全**，失败返回：
    
    - 指针：`nullptr`
        
    - 引用：抛出 `std::bad_cast`
        

### 用途

1. 多态类的安全下行转换
    
2. 在继承树中动态判断对象类型
    

### 示例

```cpp
struct Base { virtual ~Base() = default; };
struct Derived : Base {};

Base* pb = new Derived;

// 安全下行转换
Derived* pd = dynamic_cast<Derived*>(pb); // ✅ 如果 pb 指向 Derived，则 pd 非空
if(pd) pd->foo();

// 失败时返回 nullptr
Base b;
Derived* pd2 = dynamic_cast<Derived*>(&b); // nullptr
```

> ⚠️ 注意：`dynamic_cast` 只对 **有虚函数的类** 有效。

---

## 3 `reinterpret_cast<T>(expr)`

### 功能

- **低层次的位级类型转换**
    
- 可以将指针/整数/引用转换成任意类型
    
- 用于访问底层内存布局
    

### 安全性

- **不安全**，容易产生未定义行为
    
- 只在必须处理底层数据、硬件寄存器或序列化时使用
    

### 用途

1. 整型 ↔ 指针转换（比如内存映射）
    
2. 不同类型指针之间强制转换
    
3. 访问二进制数据（慎用）
    

### 示例

```cpp
int a = 0x12345678;
char* p = reinterpret_cast<char*>(&a); // 查看 int 内存的每个字节

uintptr_t addr = reinterpret_cast<uintptr_t>(p); // 指针转整数

struct A { int x; };
struct B { int y; };
A a1{10};
B* b1 = reinterpret_cast<B*>(&a1); // ⚠️ 非法访问，可能 UB
```

> ⚠️ 使用 `reinterpret_cast` 时必须保证**你知道内存布局**，否则可能崩溃。

---

## 4 `const_cast<T>(expr)`

### 功能

- **去掉或添加 const/volatile**
    
- 只改变类型修饰符，不改变实际对象
    
- 常用于修改被误标为 const 的对象（慎用！）
    

### 安全性

- 对非 const 对象去 const ✅ 安全
    
- 对真正 const 对象去 const ❌ 不安全，修改会导致未定义行为
    

### 用途

1. 去掉函数参数 const 限定
    
2. 对 legacy API 或需要修改 const 数据的场景
    

### 示例

```cpp
void print(const int* p) {
    int* q = const_cast<int*>(p);
    *q = 100; // ⚠️ 如果原指针指向真正 const 数据，会 UB
}

int x = 42;
print(&x); // ✅ x 可修改

const int y = 10;
print(&y); // ❌ UB，不能修改 const 对象
```

---

## 🧠 5、总结表

|运算符|编译期/运行期|类型检查|安全性|典型用途|
|---|---|---|---|---|
|`static_cast`|编译期|编译期|较安全（非多态下行需谨慎）|数值、枚举、非多态指针、void*|
|`dynamic_cast`|运行期|运行期|安全|多态类型下行转换|
|`reinterpret_cast`|编译期|无|不安全|低层内存访问、指针 ↔ 整数|
|`const_cast`|编译期|无|有风险|去除/添加 const/volatile|

---

✅ 小结

- **现代 C++ 推荐**：尽量用 `static_cast` 和 `dynamic_cast`，尽量避免 `reinterpret_cast` 和 `const_cast` 除非真的必须操作底层。
    
- **思路**：
    
    1. 你想做“值类型转换”？ → `static_cast`
        
    2. 你想做“运行时安全下行多态转换”？ → `dynamic_cast`
        
    3. 你想做“内存/指针底层操作”？ → `reinterpret_cast`
        
    4. 你想去掉 const/volatile？ → `const_cast`
        

---

