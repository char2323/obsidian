### 项目目标

1. **多线程并发**：可以同时处理多个客户端（Client A 和 Client B 可以同时说话，互不干扰）。
2. **持久连接**：客户端连上后，可以一直说话（死循环），直到输入 "exit" 才断开。
3. **简单明了**：使用 **“Thread-Per-Connection” (一客一线程)** 模式。这是最符合直觉的多线程网络模型。
    

---

### 核心架构图解

在这个模型中，主线程（Main Thread）只做一件事：**当门童（Accept）**。

一旦客人进来了，主线程立刻雇佣一个 **服务员（新线程）** 专门陪这位客人聊天，然后主线程继续回门口等人。

代码段

```
sequenceDiagram
    participant M as 🟢 主线程 (Acceptor)
    participant T1 as 🟠 线程 A
    participant T2 as 🟠 线程 B
    participant C1 as 🔵 客户端 A
    participant C2 as 🔵 客户端 B

    Note over M: 监听端口 8888...
    
    C1->>M: 请求连接 (Connect)
    M->>T1: ✅ 接受连接! 创建线程 A 专门服务 C1
    Note right of T1: 线程 A 启动<br/>进入收发循环
    
    C2->>M: 请求连接 (Connect)
    M->>T2: ✅ 接受连接! 创建线程 B 专门服务 C2
    Note right of T2: 线程 B 启动<br/>进入收发循环

    C1->>T1: 发送 "Hello"
    T1->>C1: 回复 "Echo: Hello"
    
    C2->>T2: 发送 "Hi"
    T2->>C2: 回复 "Echo: Hi"
```

---

### 1. 服务端代码 (`server.cpp`)

这里有一个关键点：我们需要把 `socket` 的所有权转移给新线程。最简单的办法是使用 `std::shared_ptr`。

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <thread>
#include <memory> // for std::shared_ptr

using namespace boost::asio;
using ip::tcp;

// -----------------------------------------------------------
// 👨‍🍳 服务员逻辑：专门处理某一个客户端的聊天
// -----------------------------------------------------------
void session(std::shared_ptr<tcp::socket> sock) {
    try {
        // 获取客户端 IP，方便打印日志
        std::string remote_ip = sock->remote_endpoint().address().to_string();
        std::cout << "Thread[" << std::this_thread::get_id() << "]: Serving " << remote_ip << std::endl;

        char data[1024];
        
        // 循环：只要客户端不挂断，就一直聊
        while (true) {
            boost::system::error_code error;
            
            // 1. 读数据 (阻塞等待)
            size_t len = sock->read_some(buffer(data), error);

            if (error == error::eof) {
                std::cout << "Thread[" << std::this_thread::get_id() << "]: Client disconnected." << std::endl;
                break; // 对方挂断了，退出循环
            } else if (error) {
                throw boost::system::system_error(error);
            }

            // 2. 打印收到的消息
            std::string msg(data, len);
            // 去掉末尾可能的换行符，为了打印好看
            if (!msg.empty() && msg.back() == '\n') msg.pop_back(); 
            std::cout << "Received from " << remote_ip << ": " << msg << std::endl;

            // 3. 原样回传 (Echo)
            std::string reply = "Server Echo: " + msg + "\n";
            write(*sock, buffer(reply));
        }

    } catch (std::exception& e) {
        std::cerr << "Session Exception: " << e.what() << std::endl;
    }
}

// -----------------------------------------------------------
// 🚪 门童逻辑：只负责 Accept，然后派人去干活
// -----------------------------------------------------------
int main() {
    try {
        io_context ioc;
        // 监听 8888 端口
        tcp::acceptor acceptor(ioc, tcp::endpoint(tcp::v4(), 8888));

        std::cout << "Server is running on port 8888..." << std::endl;

        while (true) {
            // 1. 准备一个智能指针管理的 Socket
            // 为什么用 shared_ptr？因为我们要把它传给另一个线程，
            // 如果用栈变量，main 函数循环一结束，socket 就析构了，新线程会崩溃。
            auto socket = std::make_shared<tcp::socket>(ioc);

            // 2. 等待连接 (阻塞)
            acceptor.accept(*socket);

            std::cout << "Main Thread: New connection accepted!" << std::endl;

            // 3. 【关键】创建一个新线程，把 socket 传过去
            // std::thread(函数名, 参数1, 参数2...)
            std::thread t(session, socket);

            // 4. 分离线程 (Fire and Forget)
            // 我们不想在 main 线程里 join() 等它，那样会阻塞 main 线程。
            // detach 表示：“你去跑吧，跑完自己结束，不用向我汇报。”
            t.detach();
        }

    } catch (std::exception& e) {
        std::cerr << "Main Exception: " << e.what() << std::endl;
    }

    return 0;
}
```

---

### 2. 客户端代码 (`client.cpp`)

客户端我们需要稍微升级一下，让它支持 **“用户输入 -> 发送 -> 接收 -> 再输入”** 的循环。

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <string>

using namespace boost::asio;
using ip::tcp;

int main() {
    try {
        io_context ioc;
        tcp::socket socket(ioc);
        
        // 连接到本机 8888
        socket.connect(tcp::endpoint(ip::make_address("127.0.0.1"), 8888));
        std::cout << "Connected to server! Type message and press Enter (type 'exit' to quit)." << std::endl;

        while (true) {
            // 1. 读取用户键盘输入
            std::cout << "You: ";
            std::string request;
            std::getline(std::cin, request); // 读取整行

            if (request == "exit") {
                break; // 输入 exit 退出
            }
            
            // 加上换行符发过去，显得正式一点
            request += "\n";

            // 2. 发送给服务器
            write(socket, buffer(request));

            // 3. 接收服务器的回显
            char reply[1024];
            size_t len = socket.read_some(buffer(reply));

            std::cout << std::string(reply, len); // 打印回显
        }

    } catch (std::exception& e) {
        std::cerr << "Client Exception: " << e.what() << std::endl;
    }

    return 0;
}
```

---

### 核心知识点总结

1. **`std::shared_ptr<tcp::socket>`**：
    - 这是多线程网络编程的神器。因为 Socket 是不可复制的（non-copyable），但我们需要把它从 Main 线程“搬运”到 Worker 线程。用智能指针包裹它是最简单的方案，还能保证 Socket 在线程结束前不被销毁。
2. **`t.detach()`**：
    - 这就是“多线程服务器”的精髓。主线程像一个发牌员，发完牌（创建线程）就不管了，立刻回到 Acceptor 等待下一位客人。
3. **并发模型**：
    - 这叫 **Connection-Per-Thread** 模型。优点是逻辑极其简单（写同步代码就行）；缺点是如果有一万个用户连进来，就要开一万个线程，内存会爆。但对于学习和小型项目，这是完美的起步！

### server 的另一种写法

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <thread>
#include <memory>
#include <string>

using namespace boost::asio;
using ip::tcp;

// 使用 using 替代 typedef (现代 C++ 风格)
using socket_ptr = std::shared_ptr<tcp::socket>;

void session(socket_ptr sock) {
    try {
        //以此打印线程ID，方便调试
        std::cout << "[Thread " << std::this_thread::get_id() << "] Client connected: " 
                  << sock->remote_endpoint() << std::endl;

        char data[1024];

        for (;;) {
            boost::system::error_code error;
            
            // 1. 读取数据
            size_t length = sock->read_some(buffer(data), error);

            if (error == error::eof) {
                std::cout << "[Thread " << std::this_thread::get_id() << "] Client disconnected." << std::endl;
                break; // 正常退出
            }
            else if (error) {
                throw boost::system::system_error(error); // 异常退出
            }

            // 2. 处理数据 (无需 memset，直接用长度构造 string)
            // 这样既快又安全，不会读到上次残留的垃圾数据
            std::string msg(data, length);
            std::cout << "Received: " << msg << std::endl;

            // 3. 回传数据
            // 使用 boost::asio::write 保证发完
            boost::asio::write(*sock, buffer(data, length));
        }
    }
    catch (std::exception& e) {
        std::cerr << "Exception in thread: " << e.what() << std::endl;
    }
}

void server(io_context& ioc, unsigned short port) {
    // 现代写法：make_address 更通用
    tcp::acceptor acceptor(ioc, tcp::endpoint(tcp::v4(), port));
    
    std::cout << "Server running on port " << port << "..." << std::endl;

    for (;;) {
        // 1. 使用 make_shared 替代 new，性能更好
        auto socket = std::make_shared<tcp::socket>(ioc);

        // 2. 阻塞等待连接
        acceptor.accept(*socket);

        // 3. 创建线程并分离
        // 这里不使用全局 set，避免内存泄露。
        // detach() 意味着线程结束后，操作系统会自动回收资源。
        std::thread(session, socket).detach();
    }
}

int main() {
    try {
        io_context ioc;
        server(ioc, 10086);
    }
    catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

