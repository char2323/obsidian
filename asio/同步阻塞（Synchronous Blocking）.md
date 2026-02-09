# Boost.Asio TCP 同步通信 · 极简指南

## 1. 核心概念速查 (Cheat Sheet)

在写代码前，记住这 4 个核心组件的角色：

| **组件**       | **代码对象**        | **角色类比**   | **核心作用**                      |
| ------------ | --------------- | ---------- | ----------------------------- |
| **Context**  | `io_context`    | **发动机/老板** | 所有网络操作的驱动核心，必须最先创建。           |
| **Endpoint** | `tcp::endpoint` | **地址+门牌号** | `IP` 地址 + `Port` 端口号。         |
| **Acceptor** | `tcp::acceptor` | **门口迎宾**   | **服务端专用**。负责把新连接“转换”为 Socket。 |
| **Socket**   | `tcp::socket`   | **电话机**    | 真正用来读写数据的工具。                  |

---

## 2. 通信流程图解 (The Loop)

记住这个“你来我往”的闭环，就不会漏写 `read` 或 `write` 了：

1. **Client (写)**: `write(msg)` 📤 ----> **Server (读)**: `read(buf)` 📥
2. **Server (处理)**: ... 业务逻辑 ...
3. **Server (写)**: `write(reply)` 📤 ----> **Client (读)**: `read(buf)` 📥

---

## 3. 服务端代码模板 (Server)

**文件名**: `server.cpp`

**逻辑**: 准备 -> 监听 -> 等待连接 -> **先读后写**

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <string>

using namespace boost::asio;
using ip::tcp;

int main() {
    try {
        // 1. 【发动机】创建上下文
        io_context ioc;

        // 2. 【迎宾】创建 Acceptor
        // tcp::v4() 代表监听本机所有网卡，端口 8888
        tcp::acceptor acceptor(ioc, tcp::endpoint(tcp::v4(), 8888));

        std::cout << "✅ Server started on port 8888..." << std::endl;

        while (true) {
            // 3. 【电话】创建一个空的 socket 准备接客
            tcp::socket socket(ioc);

            // 4. 【阻塞】等待客户端连接
            // 程序会卡在这里，直到有人连上来
            acceptor.accept(socket);
            std::cout << "🔗 New connection established!" << std::endl;

            // 5. 【读】接收客户端发来的数据 (Client Write -> Server Read)
            char data[1024];
            boost::system::error_code error;
            
            // read_some: 只要收到 1 个字节就返回，不会死等填满 buffer
            size_t len = socket.read_some(buffer(data), error);

            if (error == error::eof) {
                std::cout << "👋 Connection closed by client." << std::endl;
            } else if (error) {
                throw boost::system::system_error(error);
            }

            std::cout << "📥 Received: " << std::string(data, len) << std::endl;

            // 6. 【写】发送回执 (Server Write -> Client Read)
            std::string message = "Server Echo: " + std::string(data, len);
            // write: 保证把 message 所有字节都发完才返回
            boost::asio::write(socket, buffer(message));
            std::cout << "📤 Reply sent." << std::endl;
        }
    } catch (std::exception& e) {
        std::cerr << "❌ Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

---

## 4. 客户端代码模板 (Client)

**文件名**: `client.cpp`

**逻辑**: 准备 -> 连接 -> **先写后读**

```cpp
#include <iostream>
#include <boost/asio.hpp>
#include <string>

using namespace boost::asio;
using ip::tcp;

int main() {
    try {
        // 1. 【发动机】创建上下文
        io_context ioc;

        // 2. 【电话】创建 Socket
        tcp::socket socket(ioc);

        // 3. 【目标】构建服务端地址 (注意使用 make_address)
        tcp::endpoint server_ep(ip::make_address("127.0.0.1"), 8888);

        // 4. 【连接】拨号
        std::cout << "⏳ Connecting to 127.0.0.1:8888..." << std::endl;
        socket.connect(server_ep);
        std::cout << "✅ Connected!" << std::endl;

        // 5. 【写】主动发送消息 (Client Write -> Server Read)
        std::string msg = "Hello from Client!";
        boost::asio::write(socket, buffer(msg));
        std::cout << "📤 Sent: " << msg << std::endl;

        // 6. 【读】等待服务端回信 (Server Write -> Client Read)
        char reply[1024];
        // 阻塞等待，直到收到回音
        size_t len = socket.read_some(buffer(reply));

        std::cout << "📥 Received Reply: ";
        std::cout.write(reply, len);
        std::cout << std::endl;

    } catch (std::exception& e) {
        std::cerr << "❌ Exception: " << e.what() << std::endl;
    }
    return 0;
}
```

---

## 5. 关键 API 释义

### `write` vs `read_some`

你可能注意到了我在代码中混用了这两个：

1. **`boost::asio::write(sock, buf)` (推荐发送用)**
    - **含义**：它是“强迫症”。如果你让它发 100 字节，它会一直发，直到 100 字节全部进入网卡缓存才结束。
    - **场景**：发送完整的消息。
2. **`sock.read_some(buf)` (推荐接收不定长数据用)**
    - **含义**：它是“有多少拿多少”。只要网线上传过来哪怕 1 个字节，它就立刻返回。
    - **场景**：通用的接收。如果你用全局的 `read` 去读，如果客户端发的字节数不够填满 buffer，程序就会**永久卡死**。

### `buffer(...)`

- 它不申请内存！它只是把你的数组、字符串包装成 Asio 能看懂的格式 `(地址, 长度)`。

---

## 6. 如何编译 (CMake)

复制这份 `CMakeLists.txt` 到根目录：

```CMake
cmake_minimum_required(VERSION 3.16)
project(SimpleEcho LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
find_package(Boost REQUIRED COMPONENTS system) # 有些版本可能需要链接 system

add_executable(server server.cpp)
# 如果是 Linux/Mac 通常只需要 pthread，但在 Asio 中有时不需要显式链接
# 如果报错链接错误，尝试加上 target_link_libraries(server PRIVATE Boost::system)

add_executable(client client.cpp)
```

**运行命令:**

```Bash
cmake -B build
cmake --build build
# 先运行服务端
./build/server
# 再开一个窗口运行客户端
./build/client
```

---

