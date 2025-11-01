在 **Qt** 中，`connect` 是用于**信号（signal）和槽（slot）之间建立连接的函数，它是 Qt 的“事件驱动机制”的核心。**
当信号发出时，连接到它的槽函数就会被自动调用。

---

## 🧩 一、`connect` 的语法格式

Qt 目前有 **两种主要语法风格**：

---

### ✅ **1. 新语法（推荐，自 Qt 5 起）**

```cpp
connect(sender, &SenderClass::signalName,
        receiver, &ReceiverClass::slotName);
```

#### 示例：

```cpp
connect(button, &QPushButton::clicked,
        this, &MainWindow::onButtonClicked);
```

🔹 特点：

- 类型安全（编译期检查信号与槽是否匹配）
    
- 支持 lambda 表达式
    
- 更容易阅读与重构
    

#### 也可以连接到 lambda：

```cpp
connect(button, &QPushButton::clicked, this, [=]() {
    qDebug() << "Button clicked!";
});
```

---

### ⚙️ **2. 旧语法（Qt 4 及以前）**

```cpp
connect(sender, SIGNAL(signalName(parameters)),
        receiver, SLOT(slotName(parameters)));
```

#### 示例：

```cpp
connect(button, SIGNAL(clicked()),
        this, SLOT(onButtonClicked()));
```

🔹 特点：

- 参数写在宏 `SIGNAL()` 和 `SLOT()` 里，编译时不会检查类型。
    
- 书写繁琐，容易出错。
    
- **仍然兼容，但不推荐新项目使用。**
    

---

## 🧭 二、参数匹配规则

- **槽函数的参数数量 ≤ 信号的参数数量**。
    
- 槽函数的参数类型应与信号对应参数类型兼容。
    

例如：

```cpp
connect(slider, &QSlider::valueChanged,
        this, &MainWindow::updateValue);  // updateValue(int)
```

可以匹配：

```cpp
void MainWindow::updateValue(int value);
```

---

## 🔄 三、断开连接（disconnect）

```cpp
disconnect(sender, &SenderClass::signalName,
           receiver, &ReceiverClass::slotName);
```

或简单断开全部：

```cpp
disconnect(sender, nullptr, receiver, nullptr);
```

---

## 💡 四、信号连接信号、信号连接多个槽

也可以：

```cpp
connect(button1, &QPushButton::clicked, this, &MainWindow::saveFile);
connect(button1, &QPushButton::clicked, this, &MainWindow::logClick);
```

甚至：

```cpp
connect(button1, &QPushButton::clicked, button2, &QPushButton::click);
```

👉 这表示点击 `button1` 会自动触发 `button2` 的点击。

---

## 🧱 五、总结表格

|语法类型|写法|检查方式|支持 lambda|推荐度|
|---|---|---|---|---|
|旧语法|`connect(obj, SIGNAL(...), obj, SLOT(...))`|运行时检查|否|❌|
|新语法|`connect(obj, &Class::signal, obj, &Class::slot)`|编译时类型检查|✅|✅✅✅|

---

