一个清晰的项目结构必须有一个能理解它的 `CMakeLists.txt` 来配套。

我们先回顾一下目标文件结构：

```
awesome_project/
├── CMakeLists.txt
├── src/
│   ├── database/
│   │   ├── database.cpp
│   │   ├── statement.cpp
│   │   └── result_set.cpp
│   └── main.cpp
└── include/
    └── awesome_project/
        ├── database/
        │   ├── database.hpp
        │   ├── statement.hpp
        │   └── result_set.hpp
```

下面就是为这个结构量身定制的 `CMakeLists.txt` 文件，我会先给出完整版，然后逐行进行详细讲解。

-----

### 完整的 `CMakeLists.txt`

```cmake
# 1. 项目基本设置 (和之前类似)
cmake_minimum_required(VERSION 3.10)
project(AwesomeProject VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 2. 查找所有的源文件 (.cpp)
# 递归地搜索 src 目录下的所有 .cpp 文件，并将列表存入 SOURCE_FILES 变量
file(GLOB_RECURSE SOURCE_FILES "${CMAKE_SOURCE_DIR}/src/*.cpp")

# 3. 定义可执行文件目标
# 使用我们刚刚找到的所有源文件来创建一个名为 "awesome_app" 的可执行文件
add_executable(awesome_app ${SOURCE_FILES})

# 4. 指定头文件搜索路径
# 这是关键一步！告诉 CMake (以及编译器) 在 "include" 目录下查找头文件。
# 这样，在代码中 #include <awesome_project/database/database.hpp> 才能被找到。
target_include_directories(awesome_app PUBLIC "${CMAKE_SOURCE_DIR}/include")

# 5. 查找并链接依赖库 (和之前类似)
# 查找 MariaDB 头文件
find_path(MARIADB_INCLUDE_DIR NAMES mysql.h PATHS /usr/include/mysql)
# 查找 MariaDB 库文件
find_library(MARIADB_LIBRARY NAMES mariadb)

# 检查是否找到
if (NOT MARIADB_INCLUDE_DIR OR NOT MARIADB_LIBRARY)
    message(FATAL_ERROR "MariaDB connector not found!")
endif()

# 将 MariaDB 的头文件目录也添加到搜索路径中
target_include_directories(awesome_app PRIVATE ${MARIADB_INCLUDE_DIR})
# 将 MariaDB 库链接到我们的程序
target_link_libraries(awesome_app PRIVATE ${MARIADB_LIBRARY})


# (可选, 但推荐) 设置输出目录
# 将最终生成的可执行文件 awesome_app 放到 build/bin 目录下，而不是 build/ 根目录
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
```

-----

### 逐行讲解

我们来分解一下这个脚本里的新知识点：

#### 1\. `file(GLOB_RECURSE SOURCE_FILES ...)`

  * `file(...)` 是 CMake 中用于操作文件的命令。
  * `GLOB_RECURSE` 是一种模式，它会**递归地**查找所有匹配的文件。
  * `SOURCE_FILES` 是我们自定义的变量名，用来存储找到的文件列表。
  * `"${CMAKE_SOURCE_DIR}/src/*.cpp"` 是匹配模式：
      * `${CMAKE_SOURCE_DIR}` 是 CMake 的内置变量，指向项目根目录（也就是顶层 `CMakeLists.txt` 所在的目录）。
      * `/src/*.cpp` 告诉它去 `src` 目录下查找所有以 `.cpp` 结尾的文件，`RECURSE` 选项让它也会进入 `src` 的所有子目录（比如 `database/`）去查找。

**效果**：执行完这行，`SOURCE_FILES` 变量的值就会是一个列表，包含 `src/main.cpp`, `src/database/database.cpp` 等所有源文件。

> **注意**: `GLOB` 非常方便，但有一个小缺点：如果你在项目中新建或删除了 `.cpp` 文件，CMake 可能不会自动检测到，你需要重新运行 `cmake ..` 来更新文件列表。在大型项目中，有些人会选择手动列出所有源文件，虽然繁琐但更精确。对于绝大多数项目，`GLOB` 足够好用。

#### 2\. `target_include_directories(awesome_app PUBLIC ...)`

这是处理 `include` 目录的核心命令。

  * `target_include_directories(...)` 的作用等同于在 `g++` 命令后加上 `-I` 参数。它告诉编译器：“除了默认位置，你还应该去这个目录里找头文件”。
  * `awesome_app`：指定这个命令是作用于我们 `add_executable` 创建的 `awesome_app` 这个目标上。
  * `PUBLIC`：这是一个关键字，表示这个头文件路径不仅 `awesome_app` 自己编译时需要，任何将来链接到 `awesome_app` 的其他库也需要这个路径（虽然对于可执行文件来说意义不大，但对于库开发很重要，是个好习惯）。其他选项有 `PRIVATE` (仅自己内部使用) 和 `INTERFACE` (仅给链接到我的用)。
  * `"${CMAKE_SOURCE_DIR}/include"`：我们指定的头文件目录。

**效果**：现在，当编译器处理 `#include <awesome_project/database/database.hpp>` 时，它会：

1.  在系统默认路径下找。
2.  在我们指定的 `${CMAKE_SOURCE_DIR}/include` 目录下找。它会拼接成路径 `/path/to/your/project/include/` + `awesome_project/database/database.hpp`，然后成功找到文件！

#### 3\. 链接依赖库

这部分和你之前学的类似，只不过我们现在是把找到的库链接到 `awesome_app` 这个目标上。这里我们为 `target_include_directories` 和 `target_link_libraries` 使用了 `PRIVATE` 关键字，因为 MariaDB 是我们程序的内部实现细节，其他程序不需要知道我们用了它。

### 构建流程

神奇的是，虽然 `CMakeLists.txt` 变复杂了，但你的**构建命令和之前完全一样**！

```bash
# 在项目根目录 awesome_project/ 下
rm -rf build  # 清理旧的构建
mkdir build
cd build

# CMake 会读取新的 CMakeLists.txt，自动查找所有 src 和 include 文件
cmake ..

# Make 命令会根据 CMake 生成的复杂 Makefile 进行编译和链接
make
```

编译成功后，因为我们设置了输出目录，你的可执行文件会在这里：
`build/bin/awesome_app`

运行它：

```bash
./bin/awesome_app
```

这个 `CMakeLists.txt` 脚本是一个非常好的模板，你可以把它作为你未来所有 C++ 项目的起点。它清晰、可扩展，并且遵循了现代 C++ 项目管理的最佳实践。



## A new CMakeLists

```CMake
cmake_minimum_required(VERSION 3.10)
project(AwesomeProject LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 查找所有 .cpp 文件（头文件不需要列出）
file(GLOB_RECURSE SRC "src/*.cpp")

# 查找 Boost
find_package(Boost REQUIRED COMPONENTS filesystem system)

# 定义可执行文件
add_executable(${PROJECT_NAME} ${SRC})

# 添加头文件路径（include 目录 + Boost）
target_include_directories(${PROJECT_NAME} PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${Boost_INCLUDE_DIRS}
)

# 链接 Boost 库
target_link_libraries(${PROJECT_NAME} PRIVATE ${Boost_LIBRARIES})

```