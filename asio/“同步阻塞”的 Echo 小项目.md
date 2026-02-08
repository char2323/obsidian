我们将流程分为两个阶段：

1. **连接建立阶段 (The Handshake)**：双方如何“勾搭”上的。
2. **数据收发阶段 (The Echo Loop)**：连接好后，数据怎么流动的。

---

### 核心角色回顾

在看图之前，请再次确认舞台上的四个核心演员：

- **`io_context` (ioc)**：幕后的大老板，所有人都得依靠它（图中省略，但默认存在）。
- **`tcp::acceptor` (迎宾)**：**服务端独有**，专门负责在门口等人。
- **`tcp::endpoint` (地址)**：门牌号（IP + 端口）。
- **`tcp::socket` (电话)**：真正用来干活（说话/听音）的工具。

---

### 一：连接建立流程 (Connection Establishment)

这个阶段的目标是：让客户端和服务端各自手里的“空” `socket` 变成“通”的状态。

**同步模式特点**：API 会**阻塞 (Block)**，直到操作完成。图中用红色竖线 `|` 表示阻塞等待状态。

代码段

```
sequenceDiagram
    participant C as 🔵 Client (客户端)
    participant N as 🌐 Network (操作系统内核/网络)
    participant S as 🟢 Server (服务端)

    Note over S: 1️⃣ 准备阶段
    S->>S: tcp::acceptor acceptor(ioc, ep)
    Note right of S: ✅ 绑定端口(bind)并开始监听(listen)<br/>迎宾员就位

    Note over S: 2️⃣ 等待连接
    S->>+S: acceptor.accept(server_socket) 🔒阻塞!
    Note right of S: ⏳ 服务端卡在这里死等...<br/>(传入一个空的 server_socket 用于接客)

    Note over C: 3️⃣ 发起连接
    C->>C: tcp::socket client_socket(ioc)
    C->>C: tcp::endpoint server_ep(...)
    C->>+C: client_socket.connect(server_ep) 🔒阻塞!
    Note left of C: ⏳ 客户端卡住，正在拨号...

    C->>N: TCP 三次握手 (SYN)
    N->>S: 收到连接请求

    Note right of S: 🔔 内核通知 Acceptor 有人来了!
    S->>-S: accept 返回!

    Note left of C: ✅ 握手成功
    C->>-C: connect 返回!

    Note over C, S: 🎉 连接建立成功!
    Note left of C: client_socket 现在是通的
    Note right of S: server_socket 现在是通的
```

#### 📝 API 关键信息标注

| **步骤** | **所在端** | **API 调用**                                           | **关键特性说明 (同步模式)**                                                     |
| ------ | ------- | ---------------------------------------------------- | --------------------------------------------------------------------- |
| 一      | Server  | **`tcp::acceptor(ioc, ep)`**<br><br>  <br><br>(构造函数) | **全自动**：一行代码完成了 socket()->bind()->listen() 三个底层操作。它告诉操作系统：“这个端口归我管了”。 |
| 二      | Server  | **`acceptor.accept(sock)`**<br><br>  <br><br>(成员函数)  | **🔴 阻塞点**：程序运行到这就停住不动了，直到有客户端成功连上来。它成功返回后，传入的那个空 `sock` 就变活了。        |
| 三      | Client  | **`sock.connect(ep)`**<br><br>  <br><br>(成员函数)       | **🔴 阻塞点**：程序停住，底层的 TCP 三次握手正在进行。如果连不上（比如目标没开机或被防火墙挡住），它会抛出异常。        |

---

### 二：数据收发流程 (Data Transmission - Echo)

连接建立后，`acceptor` 的任务完成了（在那个简单示例中）。现在舞台交给两个已经连接好的 `socket`。

**场景**：客户端发一句，服务端收到了回一句。

**注意**：图中的 `buffer(...)` 是核心适配器，它把你的数组/字符串包装成 Asio 认识的格式。

代码段

```
sequenceDiagram
    participant C as 🔵 Client Socket
    participant N as 🌐 Network (缓冲区)
    participant S as 🟢 Server Socket

    note over C, S: 前提：双方 Socket 已连接

    %% --- 阶段 1: Client 发送 -> Server 接收 ---
    Note over C: 1 客户端发送 (Write)
    C->>+C: boost::asio::write(sock, buffer(msg)) 🔒
    C->>N: 发送数据 "Hello" ->>
    Note left of C: ⏳ 阻塞直到 msg 全部写卡网卡
    C->>-C: write 返回

    Note over S: 2 服务端接收 (Read)
    S->>+S: sock.read_some(buffer(data)) 🔒
    Note right of S: ⏳ 阻塞，如果网卡没数据就死等
    N->>S: ->> 收到数据 "Hello"
    Note right of S: 🔔 有数据了! 哪怕只有1字节也立刻返回
    S->>-S: read_some 返回 (len=5)

    %% --- 阶段 2: Server 处理并回传 ---
    Note over S: (服务端处理数据，拼接回信...)

    Note over S: 3 服务端回传 (Write)
    S->>+S: boost::asio::write(sock, buffer(reply)) 🔒
    S->>N: 发送数据 "I heard: Hello" ->>
    S->>-S: write 返回

    Note over C: 4 客户端接收回信 (Read)
    C->>+C: sock.read_some(buffer(buf)) 🔒
    Note left of C: ⏳ 阻塞等待...
    N->>C: ->> 收到数据 "I heard: Hello"
    C->>-C: read_some 返回
```

#### API 关键信息标注

| **步骤** | **动作** | **API 调用**                                                          | **关键特性说明 (同步模式)**                                                                             |
| ------ | ------ | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| 一、三    | **发送** | **`boost::asio::write(sock, buffer(...))`**<br><br>  <br><br>(全局函数) | **🔴 强迫症阻塞**：这个函数是“使命必达”。如果你给它 100字节，它保证全部发送出去后才返回。适合发送定长的、完整的消息包。                            |
| 二、四    | **接收** | **`sock.read_some(buffer(...))`**<br><br>  <br><br>(成员函数)           | **🔴 敏捷阻塞**：这个函数是“有多少拿多少”。只要网络对面传来**至少 1 个字节**，它就立刻带着数据返回，不会傻等到把你的 buffer 填满。这是处理未知长度数据的标准方式。 |
| 全程     | **适配** | **`boost::asio::buffer(你的变量)`**<br><br>  <br><br>(工厂函数)             | **灵魂适配器**：它不拷贝内存，只是给你现有的数组、vector、string 贴个标签，告诉 Asio 地址在哪里，长度是多少。收发数据**必用**。                 |

### 总结心法

只需记住这两个“套路”：

1. **连接套路**：服务端 `accept()` 卡住等人，客户端 `connect()` 发起拨号。这俩碰上头了，连接就通了。
2. **通信套路**：**有写必有读**。
    - 想把消息完整发出去？用全局的 `asio::write`。
    - 不知道对方发多少，来了就处理？用成员函数 `read_some`。
    - 无论读写，都要用 `buffer()` 把你的数据包一下。