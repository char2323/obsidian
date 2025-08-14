## Linux重要指令

- **pwd-Print current working directory**
	- **作用：** 打开当前终端所在的目录
	- **用法：** pwd

- **ls-List directory contents
	- **作用：** 列出当前工作目录下的所有文件/文件夹的名称
	- **用法1：** ls
	- **用法2：** ls \[路径]
		- 含义：列出指定路径下的所有文件/文件夹的名称
		- 绝对路径：相对于根目录的路径
		- 相对路径：相对于当前目录的路径
	```bash
#ls 相对路径
ls ./ # 表示当前目录下
ls ../ # 上一级目录下
#ls 绝对路径
ls /home
```
	- **用法3：** ls 选项
		- 含义：在列出指定路径下的文件/文件夹的名称，并以指定的格式进行显示
	```bash
ls -lah /home
#选项解释
-l 表示list，表示以详细列表的形式进行展示
-a 表示显示所有的文件/文件夹
-h 表示以可读性较高的形式显示
#ls -l 中‘-’表示该行对应的文档类型为文件，‘d'表示该行文档类型为文件夹
	```

- **cd-change directory**
	- **作用：** 切换当前的工作目录
	- **用法1：** cd；cd~ （表示进入home目录）
	- **用法2：** cd \[相对路径]
	- **用法3：** cd \[绝对路径]

- **mkdir-make directories**
	- **作用：** 创建目录
	- **用法1：** mkdir 路径
	- **用法2：** mkdir -p 路径
		- 含义：一次性创建多层不存在的目录
	- **用法3：** mkdir 路径1 路径2
		- 含义：一次性创建多个目录

- **touch-change file timesamps**
	- **作用：** 创建文件
		- touch的作用本来不是创建文件，而是将指定文件的修改时间设置为当前时间。就是假装碰(touch)一下这个文件，假装这个文件被修改了，于是文件的修改时间就被设置为了当前时间。但这带来了一个副作用，就是touch一个不存在的文件的时候，他就会创建这个文件。于是touch就可以承当创建文件的功能。
	- **用法1：** touch \[路径]
	- **用法2：** touch 路径1 路径2
```bash
# 在当前目录下创建文件
touch linux.txt

# 在上级目录下创建linux文件
touch ../linux

# 在/home/charmmy下创建文件
touch /home/charmmy/linux.txt

# 在当前目录下创建file file.txt两个文件
touch file file.txt
```


- **rm-remove file or directories**
	- **作用：** 删除文件/文件夹
	- **用法1：** rm \[选项] 需要移除的文件路径
	- **用法2：** rm -rf 需要移除的目录 -r表示递归
```bash
# 删除当前路径下的myfolder文件
rm -rf myfolder

# 删除/use路径下的myfolder文件
rm -rf /usr/myfolder
```

- **cp-copy files and directories**
	- **作用：** 复制文件/文件夹到指定的位置
	- **用法1：** cp 被复制的文件路径 文件被复制到的路径
	- **用法2：** cp -r 被复制的文件夹路径 文件被复制到的路径 -r表示递归

- **mv-move(rename) files**
	- **作用：** 移动文件到新的位置，或者重命名文件
	- **用法：** mv 需要移动的文件路径 需要保存的位置路径

- **man-an interface to the system reference manuals**
	- **作用：** 包含了linux中全部命令手册
	- **用法：** man 命令，按q退出

- **shutdown -power-off the machine**
	- **作用：** 关机
	- **用法：** shut -h now（立即关机）


## Vim文件编辑
### Vim模式

主要的四种模式：
- Normal模式：默认进入的模式，也是最常用的模式
- Insert模式：插入模式，像正常的文本编辑器一样输入
- Command模式：命令模式，在底部输入命令
- Visual模式：可视模式，对文本进行选择

#### Nomal模式：基本移动
- `hjkl`-上下左右
- `gg`-跳到第一行
- `G`-跳到最后一行
- `Ctrl-u/Ctrl-b`-往上翻半页/一页
- `Ctrl-d/Ctrl-f`-往下翻半页/一页
- `{lineno}gg`-跳到第`lineno`行
- `zz/zt/zb`-光标行设置为屏幕居中/屏幕第一行屏幕最后一行

#### Insert模式
- `i`-代表insert，当前光标之前开始输入
- `a`-代表append，当前光标之后开始输入
- `o`-下方插入新的一行，然后开始输入
- `s`-删除当前光标的字符，然后开始输入
- `I`-在本行的开头开始输入
- `A`-在本行的末尾开始输入
- `O`-上方插入新的一行，然后开始输入
- `S`-删除当前行，然后开始输入

#### Command模式
Normal模式下输入`:`进入Command模式
- `:w`-保存当前文件
- `:q`-退出
- `:q!`-放弃当前更改，然后退出
- `:wq`-保存当前更改，然后退出
- `:h {command}`-显示关于命令的帮助
- `Esc`回到Normal模式


#### VIsual模式
- Normal模式下按`v`进入可视模式
- 进入可视模式后可以用Normal模式下的命令选择文本
- Normal模式下按`V`进入行可视模式，一次选中一行，在需要选中多行时很方便
- 剪切：选中后按`x`
- 复制：选中后按`y`
- 粘贴：`p`
- 撤销：`u`
- 重做：`Ctrl-r`
- `Esc`回到Normal模式

### 移动与编辑

#### 基于单词的移动
- `w` -代表 word，跳转到下一处单词的开头
- `b` -代表 back，跳转到上一处单词的开头
- `e` -代表 end，跳转到下一处单词的结尾
- `ge` - `e` 的反向版本，跳转到上一处单词的结尾
- `WEB` 对应的单词的连续的非空字符

#### 基于搜索的移动
行内搜索：
- `f{char}/t{char}` -跳转到本行下一个 `char` 字符出现处/前
- `;/,` -快速向前/向后重复 `ft` 查找
- `F{char}/T{char}` -往前搜索而非往后
文件中搜索：
- `/{pattern}` -跳转到本文件中下一个 `pattern` 出现的地方
- `?{pattern}` -跳转到本文件中上一个 `pattern` 出现的地方
- `pattern` 可以是正则表达式
- `*` -等价于 `/{pattern}`，`pattern` 是当前光标下的单词
- `nN` -快速重复 `/` 查找

#### 基于标记的移动
- `m{mark}` -把当前位置标记为 `mark`
- `‵mark` -跳转到名为 `mark` 的标记位置
`mark` 是 `a-z` 的字符
常用场景：当需要临时离开当前光标处，做一些事情再**快速的**回来

内置标记：
- ` `` ` -上次跳转前的位置
- `` `. `` -上次修改的位置
- `` `^ `` -上次插入的位置

实用跳转：
- `^/$` -跳转到本行的开始/末尾
- `%` -跳到匹配的配对符（括号等）处

#### Operator+Motion=Action
`{operator}{motion}` -一次编辑动作
常见操作符：
- `c` -代表 change，修改，删除内容并进入插入模式
- `d` -代表 delete，删除
- `y` -代表 yank，复制
- `v` -代表 visual，选中文本，进入可视模式

```bash
dgg # 删除到第一行 
ye  # 复制到单词结尾
d$  # 删除到行尾
dt; # 删除直到分号为止的内容
```

大部分操作符连续按两次 `cc/dd/yy` 可以表示为作用在这一行上
- `dd` -删除这一行

## GCC编译器
### 编译过程

1. **预处理-Pre-Processing    //.i文件**
```bash
# -E选项指示编译器仅对输入文件进行预处理
g++ -E test.cpp -o test.i  //.i 文件
```
2. **编译-Compiling    // .s文件**  
```bash
# -S 编译选项告诉g++在为C++代码产生类汇编语言文件后停止编译
# g++产生的汇编语言文件缺省扩展名是 .s
g++ -S test.i -o test.s
```
3. **汇编-Assembling    //.o文件**
```bash
# -c 选项告诉g++把源代码编译为机器语言的目标代码
#缺省时g++建立的目标代码文件有一个.o的扩展名
g++ -c test.s -o test.o
```
4. **链接-Linking    //.bin文件**
```bash
# -o 编译选项来为将产生的可执行文件用指定的文件名
g++ test.o -o test
```

`g++ test.cpp -o test`其实就可以把上面的操作汇总

### g++重要编译参数

1. **-g** 编译带调试信息的可执行文件 `g++ -g test.cpp -o test`
2. **-O\[n]** 优化源代码(一般用O2)并输出可执行文件 `g++ test.cpp -O2 -o test `或者 `g++ -O2 test.cpp`
3. **-l 和 -L 指定库文件|指定库文件路径**
```bash
# -l参数就是用来指定程序要链接的库，-l参数紧接着就是库名
# 在 /lin 和 /usr/lin 以及 /usr/local/lib 里的库直接用-l参数就可以链接
# 链接glog库
g++ -lglog test.cpp

# 如果库文件没有放在上面三个目录里，需要使用-L参数指定库文件所在目录
# -L参数跟着的是库文件所在的目录名
# 链接mytest库，libmytest.so在/home/charmmy/mytestlibfolder目录下
g++ -L/home/charmmy/mytestlibfolder -lmytest test.cpp
```
4. **-I (Include)** 指定头文件搜索目录
```bash
# /usr/include目录一般是不用指定的，gcc知道从哪里去找，但是如果头文件不在/usr/include里，我们就要用-I参数指定了，比如头文件放在/myinclude里，那么编译命令行就要加上-I/myinclude了。-I可以用相对路径，比如头文件在当前目录，可以用 -I. 来指定
g++ -I/myinclude test.cpp
```
5. **Wall** 打印警告信息 `g++ -Wall test.cpp`
6. **-w** 关闭警告信息 `g++ -w test.cpp`
7. **-std=c++11** 设置编译标准 `g++ -std=c++11 test.cpp`
8. **-o** 指定输出文件名 `g++ test.cpp -o test`
9. **-D** 定义宏 

## GDB调试器

### 常用调试命令参数

调试开始：执行**gdb\[exefilename]**，进入gdb调试程序，其中exefilename为要调试的可执行文件名
```bash
# 以下命令后括号内为命令的简化使用，如run(r)，直接输入命令r就代表命令run

$(gdb)help(h) # 查看命令帮助，具体命令查询在gdb中输入help+命令

$(gdb)run(r) # 重新开始运行文件(run-text：加载文本文件，run-bin：加载二进制文件)

$(gdb)start # 单步执行，运行程序，停在第一行执行语句

$(gdb)list(l) # 查看源代码(list-n)从第n行开始查看代码，list+函数名：查看具体函数

$(gdb)set # 设置变量的值

$(gdb)next(n) # 单步调试(逐过程，函数直接执行)

$(gdb)step(s) # 单步调试(逐语句：跳入自定义函数内部执行)

$(gdb)backtrace(bt) # 查看函数的调用的栈帧和层级关系

$(gdb)frame(f) # 切换函数的栈帧

$(gdb)info(i) # 查看函数内部局部变量的数值

$(gdb)finish # 结束当前函数，返回到 函数调用点

$(gdb)continue(c) # 继续运行

$(gdb)print(p) # 打印值及地址

$(gdb)quit(q) # 推出gdb

$(gdb)break+num(b) # 在第num行设置断点

$(gdb)info breakpoints # 查看当前设置的所有断点

$(gdb)delete breakpoints num(d) # 删除第num个断点

$(gdb)display # 追踪查看具体变量值

$(gdb)undisplay # 取消追踪查看变量

$(gdb)watch # 被设置观察点的变量发生修改时，打印显示

$(gdb)i watch # 显示观察点

$(gdb)enable breakpoints # 启用断点

$(gdb)disable breakpoints # 禁用断点

$(gdb)x # 查看内存x/20xw 显示20个单元，16进制，4字节每单元

$(gdb)run argv[1] argv[2] # 调试时命令行传参

$(gdb)set follow-fork-mode child # makefile项目管理，选择跟踪父子进程(fork())
```

**Tips**
	1. 编译程序时需要加上-g之后才能用gdb进行调试：`gcc -g main.c -o main`
	2. 回车键：重复上一条命令

```C++
#include <iostream>
using namespace std;
int main(int argc,char** argv){
	int N=100;
	int sum=0;
	int i=1;
	while(i<=N){
	sum+=i;
	i++;
	}
	cout<<"sum="<<sum<<endl;
	cout<<"The program is over."<<endl;
	return 0;
}
```

## VSCode常用快捷键

| 功能          | 快捷键            | 功能       | 快捷键           |
| ----------- | -------------- | -------- | ------------- |
| 转到文件/其他常用操作 | `ctrl+p`       | 关闭当前文件   | `ctrl+w`      |
| 打开命令面板      | `ctrl+shift+p` | 当前行上移/下移 | `Alt+up/down` |
| 打开终端        | `ctrl+`        | 变量统一命名   | `F2`          |
| 关闭侧边栏       | `ctrl+b`       | 转到定义处    | `F12`         |

## CMake
### 语法特性介绍

- **基本语法格式：指令（参数1 参数2）**
	- 参数使用括弧括起
	- 参数之间使用空格或者分号隔开
- **指令是大小写无关的，参数和变量是大小写相关的**
- **变量使用`${}`方式取值，但是在`IF`控制语句中是直接使用变量名**
```cmake
set(HELLO hello.cpp)
add_executable(hello main.cpp hello.cpp)
ADD_EXECUTABLE(hello main.cpp ${HELLO})
```

### 重要指令

- `cmake_minimum_required`-指定CMake的最小版本要求
	- 语法：`cmake_minimum_required(VERSION versionNumber[FATAL_ERROR])`
```cmake
# CMake最小版本要求为2.8.3
cmake_minimum_required(VERSION 2.8.3)
```

- `project`-定义工程名称，并可指定工程支持的语言
	- 语法：`project(projectname[CXX][X][Java]`
```cmake
# 指定工程名为helloworld
project(helloworld)
```

- `set`-显示的定义变量
	- 语法：`set(VAR [VALUE][CACHE TYPR DOCSTRING [FORCE]])`
```CMake
# 定义SRC变量，其值为sayhello.cpp hello.cpp
set(SRC sayhello.cpp hello.cpp)
```

- `include_directories`-向工程添加多个特定的头文件搜索路径-->相当于指定g++编译器的-I参数
	- 语法：`include_directories([AFTER|BEFORE][SYSTEM] dir1 dir2...)`
```cmake
# 将/usr/include/myincludefolder 和 ./include 添加到头文件搜索路径
include_directories(/usr/include/myincludefolder ./include)
```

- `link_directories`-向工程添加多个特定的库文件搜索路径——>相当于指定g++编译器的-L参数
	- 语法：`link_directories(dir1 dir2...)`
```cmake
# 将/usr/lib/mylibfolder 和 ./lib 添加到库文件搜索路径
link_directories(/usr/lib/mylibfolder ./lib)
```

- `add_library`-生成库文件
	- 语法：`add_library(libname [SHARED|STATIC|MODULE][EXCLUDE_FROM_ALL] source1 source2...sourceN)`
```cmake
# 通过变量SRC生成libhello.so共享库
add_library(hello SHARED ${SRC})
```

- `add_compile_options`-添加编译参数
	- 语法：`add_compile_options(<option>...)`
```cmake
# 添加编译参数 -wall -std=c++11 -o2
add_compile_options(-Wall -std=c++11 -o2)
```

- `add_executable`-生成可执行文件
	- 语法：`add_executable(exename source1 source2 ... sourceN)`
```cmake
# 编译main.cpp生成可执行文件main
add_executable(main main.cpp)
```

- `target_link_libraries`-为target添加需要链接的共享库-->相当于指定g++编译器-l参数
	- 语法：`target_link_libraries(target library1<debug|optimized> library2...`
```cmake
# 将hello动态库链接到可执行文件main
target_link_libraries(main hello)
```

- `add_subdirectory`-向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存放的位置
	- 语法：`add_subdirectory(source_dir[binary_dir][EXCLUDE_FORM_ALL])`
```cmake
# 添加src子目录，src中需要有一个CMakeLists.txt
add_subdirectory(src)
```

- `aux_source_directory`-发现一个目录下所有源代码文件并将列表存储在一个变量中，这个指令临时被用来自动构建源文件列表
	- 语法：`aux_source_directory(dir VARIABLE)`
```cmake
# 定义SRC变量，其值是当前目录下所有的源代码文件
auc_source_directory(. SRC)
# 编译SRC变量所代表的源代码文件，生成main可执行文件
add_executable(main ${SRC})
```

### CMake常用变量

- `CMAKE_C_FLAGS`-gcc编译选项
- `CMAKE_CXX_FLAGS`-g++编译选项
```cmake
# 在CMAKE_CXX_FLAGS编译选项后追加-std=c++11
set(CMAKE_CXX_FLAGS"${CMAKE_CXX_FLAGS} -std=c++11")
```

- `CMAKE_BUILD_TYPE`-编译类型(Debug,Release)
```cmake
# 设定编译类型为debug，调试时需要选择debug
set(CMAKE_BUILD_TYPE Debug)
# 设定编译类型为release，发布时需要选择release
set(CMAKE_BUILD_TYPE Release)
```

- `CMAKE_BINARY_DIR` `PROJECT_BINARY_DIR` `<projectname>_BINARY_DIR` 
	1. 这三个变量代指的内容是一样的
	2. 如果是`in source build`，指的就是工程顶层目录
	3. 如果是`out-of-source`编译，指的是工程编译发生的目录
	4. `PROJECT_BINARY_DIR`与其他指令稍有不同，不过可以暂且理解为一致的

- `CMAKE_SOURCE_DIR` `PROJECT_SOURCE_DIR` `<projectname>_SOURCE_DIR` 
	1. 这三个变量代指的内容是一致的，不论采用何种编译方式，都是工程顶层目录
	2. 也就是在`in source bulid`时，与`MAKE_BINARY_DIR`等变量一致
	3. `PROJECT_SOURCE_DIR`与其他指令稍有不同，不过可以暂且理解为一致的

- `CMAKE_C_COMPILER`-指定C编译器
- `CMAKE_CXX_COMPILER`-指定C++编译器
- `EXECUTABLE_OUTPUT_PATH`-可执行文件输出的存放路径
- `LIBRARY_OUTPUT_PATH`-库文件输出的存放路径

### CMake编译工程

CMake目录结构：项目主目录存在一个CMakeLists.txt文件

**两种方式设置编译规则：**
	1. 包含源文件的子文件夹包含CMakeList.txt文件，主目录的CMakeList.txt通过`add_subdirectory`添加子目录即可；
	2. 包含源文件的自文件夹未包含CMakeList.txt文件，子目录编译规则体现在主目录的CMakeList.txt中

**编译流程：**
	1. 手动编写CMakeList.txt
	2. 执行命令`cmake PATH`生成Makefile(PATH是顶层CMakeList.txt所在的目录)
	3. 指定命令`make`进行编译

```bash
# important tips
.   # 表示当前目录
./  # 表示当前目录

..  # 表示上级目录
../ # 表示上级目录
```

**内部构建：**
```Cmake
# 在当前目录下，编译本目录的CMakeLists.txt，生成Makefile和其他文件
cmake .

#执行make命令，生成target
make
```

推荐使用**外部构建(out-of-cource build)**，将编译输出文件与源文件放到不同目录中
```cmake
## 外部构建

#1.在当前目录下，创建build文件夹
mkdir build

#2.进入到build文件夹
cd build

#3.编译上级目录的CMakeLists.txt，生成Makefile和其他文件
cmake ..

#4.执行make命令，生成target
make
```

![[Pasted image 20250429000752.png]]

![[Pasted image 20250429001704.png]]
