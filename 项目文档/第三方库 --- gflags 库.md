
## 1. gflags 库
### 1.1 gflags 是什么

**gflags 是一个 C++ 命令行参数解析库。**

它最初由 Google 开发，主要用途是让 C++ 程序非常方便地定义、解析和使用命令行参数。现在它已经是社区维护的开源项目，不再由 Google 直接维护。官方仓库目前显示最新维护版本为 **gflags 2.3.0**，发布于 2025 年 12 月。[GitHub](https://github.com/gflags/gflags)

官方对它的定位其实很直接：它负责 **command-line flags processing**，并且一个比较有特色的地方是，flag 可以定义在真正使用它的源文件里，而不一定全部集中写在 `main()` 中。[GFlags](https://gflags.github.io/gflags/)

你可以先记住：

> **gflags = 帮你把 `argc/argv` 里的命令行配置转换成容易使用的 C++ 变量。**

### 1.2 什么叫“命令行参数”？

假设你有一个程序：
```bash
./server
```
现在你希望运行程序的时候指定：
```test
端口 = 8080
线程数 = 8
调试模式 = 开启
配置文件 = config.yaml
```
一种很常见的方式就是：
```
--port=8080
--threads=8
--debug=true
--config=config.yaml
```
就是所谓的 **command-line flags**。

gflags 的工作，就是帮助你的 C++ 程序理解这些内容。

官方文档也区分了 **flag** 和普通的 **argument**：像 `--port=8080` 这种带有选项含义的是 flag，而没有 `-` 前缀的普通字符串通常属于 positional arguments。

### 1.3 如果不用 gflags，会怎样？

例如你直接写：
```cpp
int main(int argc, char* argv[]) {
    // ...
}
```
那么运行：
```bash
./server --port=8080
```
实际上 C++ 给你的只是：
```cpp
argv[0] = "./server"
argv[1] = "--port=8080"
```
然后你得自己解析：
```cpp
std::string arg = argv[1];

if (arg.starts_with("--port=")) {
    // 截取 8080
    // string -> int
    // 判断是否合法
    // 保存结果
}
```
参数一多，就会出现大量类似代码。

比如：
```bash
./server \
    --port=8080 \
    --threads=8 \
    --debug=true \
    --host=127.0.0.1 \
    --config=config.yaml
```
你需要自己处理字符串切割、类型转换、默认值、非法输入、`--help` 等等。

gflags 就是为了帮你解决这些事情。

### 1.4 使用 gflags 后是什么样？

例如：
```cpp
#include <iostream>
#include <gflags/gflags.h>

DEFINE_int32(port, 8080, "Server port");
DEFINE_int32(threads, 4, "Worker thread count");
DEFINE_bool(debug, false, "Enable debug mode");
DEFINE_string(config, "config.yaml", "Config file");

int main(int argc, char** argv) {
    gflags::ParseCommandLineFlags(&argc, &argv, true);

    std::cout << "port: " << FLAGS_port << '\n';
    std::cout << "threads: " << FLAGS_threads << '\n';
    std::cout << "debug: " << FLAGS_debug << '\n';
    std::cout << "config: " << FLAGS_config << '\n';
}
```
运行：
```bash
./server --port=9000 --threads=16 --debug --config=test.yaml
```
那么程序中就会得到：
```cpp
FLAGS_port     == 9000
FLAGS_threads  == 16
FLAGS_debug    == true
FLAGS_config   == "test.yaml"
```
这就是 gflags 最核心的用法。

## 2. gflags 的基本使用

gflags 最基本的使用可以分成三步：

1. 定义参数
2. 解析参数
3. 使用参数

### 2.1 定义命令行参数

gflags 使用 `DEFINE_xxx` 来定义参数。

例如：
```cpp
DEFINE_int32(port, 8080, "Server port");
```

这表示定义了一个名为 `port` 的整数参数：
```
参数名：port
类型：int32
默认值：8080
说明：Server port
```

用户可以在运行程序时修改它：
```bash
./server --port=9000
```

如果用户没有传入 `--port`，那么它就使用默认值：
```
8080
```


### 2.2 常见的 DEFINE 类型

gflags 提供了几种常用的参数类型：
```cpp
DEFINE_bool(debug, false, "Enable debug mode");

DEFINE_int32(port, 8080, "Server port");

DEFINE_int64(max_size, 100000, "Max size");

DEFINE_double(threshold, 0.8, "Threshold");

DEFINE_string(name, "server", "Server name");
```

可以简单对应为：

| gflags 类型       | C++ 类型        |
| --------------- | ------------- |
| `DEFINE_bool`   | `bool`        |
| `DEFINE_int32`  | `int32`       |
| `DEFINE_int64`  | `int64`       |
| `DEFINE_double` | `double`      |
| `DEFINE_string` | `std::string` |

它们的基本格式都是：
```bash
DEFINE_类型(参数名, 默认值, "参数说明");
```

### 2.3 使用 `FLAGS_xxx` 访问参数

定义一个参数以后，gflags 会提供一个对应的`FLAGS_xxx`变量。

例如：
```cpp
DEFINE_int32(port, 8080, "Server port");
```
对应：
```
FLAGS_port
```

再比如：
```cpp
DEFINE_string(name, "server", "Server name");
```
对应：
```
FLAGS_name
```

所以规律很简单：
```
参数名：port

使用时：FLAGS_port
```
例如：
```cpp
std::cout << FLAGS_port << std::endl;
```

### 2.4 解析命令行参数

定义参数以后，还需要调用：
```cpp
gflags::ParseCommandLineFlags(&argc, &argv, true);
```
让 gflags 解析用户传入的命令行参数。

通常写在 `main()` 函数开始的位置：
```cpp
int main(int argc, char** argv) {
    gflags::ParseCommandLineFlags(&argc, &argv, true);

    // 后面就可以使用 FLAGS_xxx
}
```

例如用户运行：
```bash
./server --port=9000
```
解析之后：
```cpp
FLAGS_port == 9000
```

如果用户没有传：
```bash
./server
```
那么：
```cpp
FLAGS_port == 8080
```
也就是使用定义时的默认值。

### 2.5 一个完整的简单示例

```cpp
#include <iostream>
#include <gflags/gflags.h>

DEFINE_string(name, "world", "User name");
DEFINE_int32(age, 18, "User age");
DEFINE_bool(debug, false, "Enable debug mode");

int main(int argc, char** argv) {
    gflags::ParseCommandLineFlags(&argc, &argv, true);

    std::cout << "name: " << FLAGS_name << '\n';
    std::cout << "age: " << FLAGS_age << '\n';
    std::cout << "debug: " << FLAGS_debug << '\n';

    return 0;
}
```
直接运行：
```bash
./demo
```
结果大致为：
```
name: world
age: 18
debug: 0
```

传入参数：
```bash
./demo --name=Tom --age=25 --debug
```
结果：
```
name: Tom
age: 25
debug: 1
```

### 2.6 基本使用流程

所以 gflags 最基本的使用流程可以总结为：
```
DEFINE_xxx
    ↓
定义参数
    ↓
ParseCommandLineFlags()
    ↓
解析命令行参数
    ↓
FLAGS_xxx
    ↓
在程序中使用
```

## 3. 帮助系统与参数说明

### 3.1 参数说明

gflags 自带一套比较方便的帮助系统。

当我们定义参数时：
```cpp
DEFINE_int32(port, 8080, "Server port");
```

第三个参数：
```
"Server port"
```
就是这个参数的说明信息。

所以定义参数时，建议把说明写清楚。

### 3.2 使用 `--help` 查看帮助信息

gflags 支持直接通过命令行查看程序的参数说明。

例如：
```bash
./server --help
```

程序会输出类似：
```
Flags from main.cpp:

    -debug
        Enable debug mode
        type: bool
        default: false

    -port
        Server listening port
        type: int32
        default: 8080

    -config
        Config file path
        type: string
        default: "config.yaml"
```

这样用户就可以知道：
```
程序支持哪些参数
参数是什么类型
默认值是多少
参数的作用是什么
```

### 3.3 `--helpshort`

除了：
```
./server --help
```

还可以使用：
```
./server --helpshort
```

它只看当前可执行程序“自己”的参数，通常比完整帮助信息更加简洁。

举个例子。

假设你的项目有两个文件：
```cpp
// main.cpp
DEFINE_int32(port, 8080, "Server port");
```

```cpp
// database.cpp
DEFINE_string(db_host, "localhost", "Database host");
DEFINE_int32(db_port, 3306, "Database port");
```

最后它们一起编译成：
```bash
./server
```

那么：
```bash
./server --helpshort
```
主要只显示和 `server` 主程序对应文件中的参数，例如：
```
--port
    Server port
```

而：
```bash
./server --help
```
会看到所有链接进程序的 flags：
```
Flags from main.cpp:

--port
    Server port


Flags from database.cpp:

--db_host
    Database host

--db_port
    Database port
```


### 3.4 `--helpfull`

现在基本等同于 `--help`，只是语义上更明确地表示“显示全部参数”。

### 3.5 设置程序说明

gflags 还可以通过：
```cpp
gflags::SetUsageMessage();
```
设置程序的使用说明。

例如：
```cpp
int main(int argc, char** argv) {

    gflags::SetUsageMessage(
        "Usage: server [options]"
    );

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    return 0;
}
```
这样在查看帮助信息时，就可以看到：
```
Usage: server [options]
```

通常可以把它理解成：

> 告诉用户这个程序应该怎么使用。

### 3.6 设置版本信息

gflags 还可以设置程序版本：
```cpp
gflags::SetVersionString("1.0.0");
```

例如：
```cpp
int main(int argc, char** argv) {

    gflags::SetVersionString("1.0.0");

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    return 0;
}
```

之后可以通过：
```bash
./server --version
```

查看版本信息。

### 3.7 完整示例

```cpp
#include <iostream>
#include <gflags/gflags.h>

DEFINE_int32(port, 8080, "Server listening port");
DEFINE_bool(debug, false, "Enable debug mode");
DEFINE_string(config, "config.yaml", "Config file path");

int main(int argc, char** argv) {

    gflags::SetUsageMessage(
        "Usage: server [options]"
    );

    gflags::SetVersionString("1.0.0");

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    std::cout << "port: " << FLAGS_port << '\n';

    return 0;
}
```

运行：
```bash
./server --help
```

可以查看参数帮助。

运行：
```bash
./server --version
```

可以查看程序版本。

## 4. 在多个 `.cpp/.h` 文件中使用

在实际项目中，代码通常不会全部写在一个 `.cpp` 文件里。

例如：
```
project/
├── main.cpp
├── server.cpp
└── server.h
```

如果一个 gflags 参数需要在多个文件中使用，就需要区分：
```cpp
DEFINE_xxx
```
和：
```cpp
DECLARE_xxx
```

### 4.1 `DEFINE_xxx`：定义参数

`DEFINE_xxx` 用来真正定义一个 gflags 参数。

例如：
```cpp
DEFINE_int32(port, 8080, "Server port");
```

它表示：
```
定义一个名为 port 的参数
默认值为 8080
```

一个参数通常只能真正定义一次。

因此，不要在多个 `.cpp` 文件中重复写：
```cpp
DEFINE_int32(port, 8080, "Server port");
```

否则会出现重复定义的问题。

### 4.2 `DECLARE_xxx`：声明参数

如果一个参数已经在其他 `.cpp` 文件中定义，但是当前文件也想使用它，就可以使用：
```cpp
DECLARE_xxx
```

例如：
```cpp
DECLARE_int32(port);
```

它的意思是：

> `port` 这个参数已经在其他地方定义了，我这里只是要使用它。

之后就可以直接使用：
```cpp
FLAGS_port
```

### 4.3 一个简单例子

假设项目结构：
```
project/
├── main.cpp
├── server.cpp
└── server.h
```

我们希望在 `server.cpp` 中定义`port`参数，同时在 `main.cpp` 中也能使用它。

**server.cpp**：
```cpp
#include <gflags/gflags.h>

DEFINE_int32(port, 8080, "Server port");

void StartServer() {
    // 可以直接使用
    int port = FLAGS_port;
}
```
这里真正定义了：
```
FLAGS_port
```

**main.cpp**：
```cpp
#include <iostream>
#include <gflags/gflags.h>

DECLARE_int32(port);

int main(int argc, char** argv) {

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    std::cout << FLAGS_port << '\n';

    return 0;
}
```
这里没有再次定义 `port`。

而是：
```cpp
DECLARE_int32(port);
```
表示这个参数已经在其他文件中定义。

这样`FLAGS_port`就可以在 `main.cpp` 中正常使用。

### 4.4 在 `.h` 文件中声明

如果很多文件都需要使用同一个 flag，也可以把 `DECLARE_xxx` 写在头文件中。

例如：

**server.h**：
```cpp
#pragma once

#include <gflags/gflags.h>

DECLARE_int32(port);
```

**server.cpp**：
```cpp
#include "server.h"

DEFINE_int32(port, 8080, "Server port");
```

**main.cpp**：
```cpp
#include <iostream>
#include "server.h"

int main(int argc, char** argv) {

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    std::cout << FLAGS_port << '\n';

    return 0;
}
```

这样：
```
server.cpp
    ↓
DEFINE_int32
    ↓
真正定义参数

server.h
    ↓
DECLARE_int32
    ↓
声明参数

main.cpp
    ↓
#include "server.h"
    ↓
使用 FLAGS_port
```

## 5. 安装、编译与最小环境搭建

在使用 gflags 之前，首先需要把它安装到系统中，并确认编译器能够正常找到它。

### 5.1 环境准备

首先需要准备基本的 C++ 开发环境：
```
C++ 编译器
CMake
gflags
```

例如在 Linux 环境中，可以检查：
```bash
g++ --version
cmake --version
```

如果这些命令能够正常输出版本信息，就说明基本的编译环境已经存在。

### 5.2 直接安装 gflags

如果使用 Ubuntu / Debian，可以直接通过包管理器安装：
```bash
sudo apt update
sudo apt install libgflags-dev
```

安装完成后，系统中通常会包含：
```
gflags 头文件
gflags 库文件
CMake 配置文件
```

这样就可以直接在代码中使用：
```cpp
#include <gflags/gflags.h>
```

### 5.3 从源码编译安装

如果希望自己编译 gflags，也可以从官方源码构建。

首先下载源码：
```bash
git clone https://github.com/gflags/gflags.git
cd gflags
```

然后使用 CMake：
```bash
cmake -S . -B build
```

编译：
```bash
cmake --build build
```

最后安装：
```bash
sudo cmake --install build
```

整个过程可以简单理解为：
```
下载源码
    ↓
cmake 配置
    ↓
编译
    ↓
安装
```

当前 gflags 仍然使用 CMake 作为主要构建方式，因此这种方式也非常适合后面学习 CMake 集成。

### 5.4 创建一个最小测试程序

安装完成后，可以先创建一个简单的：
```
main.cpp
```

内容：
```cpp
#include <iostream>
#include <gflags/gflags.h>

DEFINE_string(name, "world", "User name");

int main(int argc, char** argv) {
    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    std::cout << "Hello, "
              << FLAGS_name
              << '\n';

    return 0;
}
```

### 5.5 直接使用 g++ 编译

暂时不使用 CMake，也可以直接使用 `g++` 编译：
```bash
g++ main.cpp -o demo -lgflags
```

这里：
```
main.cpp
    ↓
要编译的源文件

-o demo
    ↓
生成名为 demo 的程序

-lgflags
    ↓
链接 gflags 库
```

如果编译成功，就会生成：
```
demo
```

### 5.6 运行程序

直接运行：
```bash
./demo
```
输出：
```
Hello, world
```

如果传入参数：
```bash
./demo --name=Tom
```
输出：
```
Hello, Tom
```

这说明：
```
gflags 已安装成功
    ↓
头文件可以找到
    ↓
库可以正常链接
    ↓
命令行参数可以正常解析
```

最小环境就已经搭建完成了。

### 5.7 使用 CMake 编译最小项目

如果希望直接使用 CMake，可以建立：

```
gflags_demo/
├── CMakeLists.txt
└── main.cpp
```

**CMakeLists.txt**：
```
cmake_minimum_required(VERSION 3.15)

project(gflags_demo)

find_package(gflags REQUIRED)

add_executable(
    demo
    main.cpp
)

target_link_libraries(
    demo
    PRIVATE
    gflags::gflags
)
```
然后：
```bash
cmake -S . -B build
```
编译：
```bash
cmake --build build
```
运行：
```bash
./build/demo --name=Tom
```
输出：
```
Hello, Tom
```

这样就完成了一个最小的：
```
C++
+
CMake
+
gflags
```
开发环境。

## 6. 使用 CMake 集成到真实项目

前面我们已经知道如何在代码中使用 gflags。

但是在真实项目中，仅仅写：
```cpp
#include <gflags/gflags.h>
```
还不够。

我们还需要告诉 CMake：
```
gflags 在哪里
```

以及：
```
当前程序需要链接 gflags
```

### 6.1 一个简单的项目结构

假设我们的项目结构如下：
```
project/
├── CMakeLists.txt
└── main.cpp
```

**main.cpp**：
```cpp
#include <iostream>
#include <gflags/gflags.h>

DEFINE_int32(port, 8080, "Server port");

int main(int argc, char** argv) {

    gflags::ParseCommandLineFlags(
        &argc,
        &argv,
        true
    );

    std::cout << "port: "
              << FLAGS_port
              << '\n';

    return 0;
}
```

接下来需要配置 `CMakeLists.txt`。

### 6.2 使用 `find_package` 找到 gflags

如果系统中已经安装了 gflags，可以使用：
```
find_package(gflags REQUIRED)
```

它的作用就是：

> 让 CMake 在系统中查找已经安装好的 gflags。

例如：
```
cmake_minimum_required(VERSION 3.15)

project(gflags_demo)

find_package(gflags REQUIRED)

add_executable(demo main.cpp)
```

这里`find_package(gflags REQUIRED)`中的`gflags`表示要查找 gflags。

而`REQUIRED`表示：

> gflags 是当前项目必须依赖的库。

如果 CMake 找不到 gflags，就会直接报错并停止配置。

### 6.3 链接 gflags

找到 gflags 后，还需要把它链接到我们的程序中。

使用：
```
target_link_libraries()
```

例如：
```
target_link_libraries(
    demo
    PRIVATE
    gflags::gflags
)
```

完整的 `CMakeLists.txt`：
```
cmake_minimum_required(VERSION 3.15)

project(gflags_demo)

find_package(gflags REQUIRED)

add_executable(
    demo
    main.cpp
)

target_link_libraries(
    demo
    PRIVATE
    gflags::gflags
)
```

其中：
```
gflags::gflags
```

表示 gflags 提供的 CMake target。

官方推荐使用这种写法。

### 6.4 整个流程

可以把 CMake 的作用简单理解为：
```
find_package(gflags REQUIRED)
        ↓
找到 gflags

add_executable(...)
        ↓
创建我们的程序

target_link_libraries(...)
        ↓
把程序和 gflags 链接起来
```

也就是：
```
main.cpp
    +
gflags
    ↓
CMake
    ↓
编译和链接
    ↓
demo
```

### 6.5 编译项目

假设当前目录是：
```
project/
├── CMakeLists.txt
└── main.cpp
```

可以创建一个 `build` 目录：
```
mkdir build
cd build
```
然后运行：
```
cmake ..
```
生成构建文件。

接着编译：
```
cmake --build .
```
编译完成后运行：
```
./demo --port=9000
```
输出：
```
port: 9000
```
这说明 gflags 已经成功集成到了项目中。

### 6.6 如果 gflags 就放在项目里

有些项目不会提前把 gflags 安装到系统中，而是直接把 gflags 源码放到项目目录里。

例如：
```
project/
├── CMakeLists.txt
├── main.cpp
└── third_party/
    └── gflags/
```

这时可以使用：
```
add_subdirectory(third_party/gflags)
```

例如：
```
cmake_minimum_required(VERSION 3.15)

project(gflags_demo)

add_subdirectory(third_party/gflags)

add_executable(
    demo
    main.cpp
)

target_link_libraries(
    demo
    PRIVATE
    gflags::gflags
)
```

这里不再需要：
```
find_package(gflags REQUIRED)
```

因为 gflags 的源码本身就在当前项目中。

### 6.7 `find_package` 和 `add_subdirectory` 的区别

可以简单理解为：
```
find_package
    ↓
寻找已经安装好的 gflags
```

而：
```
add_subdirectory
    ↓
直接把 gflags 源码加入当前项目一起编译
```

例如：
```
gflags 已经安装在系统中
        ↓
find_package(gflags REQUIRED)
```

如果：
```
项目中直接包含 gflags 源码
        ↓
add_subdirectory(...)
```

两种方式最后都可以：
```
target_link_libraries(
    demo
    PRIVATE
    gflags::gflags
)
```

## 7. 常用补充

前面已经掌握了 gflags 的基本使用方式。

这一部分补充两个在实际项目中比较常见的功能：

1. bool 参数的常见写法
2. 参数校验 Validator

### 7.1 bool 参数的常见写法

假设定义了一个 bool 类型参数：
```cpp
DEFINE_bool(debug, false, "Enable debug mode");
```
默认值是：
```
false
```

运行程序时，可以有几种不同写法。

**==开启参数==**

可以写：
```bash
./server --debug
```
也可以写：
```bash
./server --debug=true
```
这两种写法都会得到：
```
FLAGS_debug == true
```

**==关闭参数==**

可以写：
```bash
./server --debug=false
```
也可以写：
```bash
./server --nodebug
```
这两种写法都会得到：
```
FLAGS_debug == false
```

所以可以简单记成：
```
--debug
--debug=true
    ↓
true


--nodebug
--debug=false
    ↓
false
```

其中 `--nodebug` 是 gflags 对 bool 参数提供的一种比较方便的写法。

> [!NOTE] 各类 flag 写法对比汇总：
> ==**1. DEFINE_bool（特殊）**==
> 
> ```cpp
> DEFINE_bool(debug, false, "demo");
> ```
> - `./app --debug` ✅ 合法 → FLAGS_debug = true
> - `./app --debug=true` ✅
> - `./server --debug=1`：1 等价 true
> - `./app --no-debug` ✅ → false
> - `./app --debug=false` ✅
> - `./server --debug=0` 0 等价 false
> - 不写 flag → 使用默认值 false
> - 注：bool flag**不能写 `--debug false`**（空格分开，会把 false 当成下一个程序参数，不是 flag 的值）。
> 
> ==**2. DEFINE_int32 / DEFINE_int64 / DEFINE_double（数值型）**==
> 
> ```cpp
> DEFINE_int32(port, 8080, "listen port");
> ```
> - `./app --port=9000` ✅
> - `./app --port 9000` ✅（空格形式也支持，但推荐`=`写法）
> - 不写 flag → 使用默认值 8080
> - `./app --port` ❌ **报错！缺少参数值**
> 
> ==**3. DEFINE_int32 / DEFINE_int64 / DEFINE_double（数值型）**==
> 
> ```cpp
> DEFINE_string(name, "test", "user name");
> ```
> - `./app --name=zhangsan` ✅
> - `./app --name zhangsan` ✅
> - 不写 flag → 默认值 `"test"`
> - `./app --name` ❌ 报错，缺少字符串值

### 7.2 参数校验 Validator

gflags 可以帮助我们判断参数的类型是否正确。

例如：
```bash
DEFINE_int32(port, 8080, "Server port");
```

如果用户输入：
```bash
./server --port=abc
```

gflags 会发现 `abc` 无法转换成整数。

但是还有另一种情况：
```bash
./server --port=-100
```

这里 `-100` 确实是一个整数，所以类型没有问题。

但是对于端口号来说 `-100` 显然是不合理的。

这时候就可以使用 Validator。

### 7.3 定义一个 Validator

例如：
```cpp
bool ValidatePort(
    const char* flagname,
    int32_t value
) {
    return value > 0 && value <= 65535;
}
```

这个函数的作用就是：
```
检查 port 的值
    ↓
如果合法
    ↓
返回 true

如果不合法
    ↓
返回 false
```

然后注册这个 Validator：
```cpp
DEFINE_int32(port, 8080, "Server port");

DEFINE_validator(
    port,
    &ValidatePort
);
```

这样，当用户输入：
```bash
./server --port=9000
```

因为 `9000` 满足条件，所以参数可以正常使用。

但是如果输入：
```bash
./server --port=-100
```

Validator 会返回 `false` 

gflags 就会认为这个参数不合法，并终止程序。
