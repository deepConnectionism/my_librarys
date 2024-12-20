## 问题1：

C++ 怎么把一些代码编译为共享库 以便写一些代码调用呢？

使用 CMake 来编译 C++ 代码为共享库是一个常见的做法，下面是使用 CMake 创建一个共享库的完整过程。

### 假设你有以下文件结构：

```
/my_project
    CMakeLists.txt
    src/
        my_library.cpp
        main.cpp
    include/
        my_library.h
```

### 1. 编写 C++ 代码和头文件

#### `src/my_library.cpp`

```cpp
#include "my_library.h"
#include <iostream>

void MyLibrary::say_hello() {
    std::cout << "Hello from MyLibrary!" << std::endl;
}
```

#### `include/my_library.h`

```cpp
#ifndef MY_LIBRARY_H
#define MY_LIBRARY_H

class MyLibrary {
public:
    void say_hello();
};

#endif
```

#### `src/main.cpp`

```cpp
#include "my_library.h"

int main() {
    MyLibrary lib;
    lib.say_hello();
    return 0;
}
```

### 2. 创建 `CMakeLists.txt`

在项目根目录（`/my_project`）下创建一个 `CMakeLists.txt` 文件，内容如下：

```cmake
# 设置 CMake 最低版本要求
cmake_minimum_required(VERSION 3.10)

# 设置项目名称
project(MyProject)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 指定头文件的路径
include_directories(include)

# 定义库的源文件
add_library(mylibrary SHARED src/my_library.cpp)

# 设置可执行文件
add_executable(main src/main.cpp)

# 链接共享库到可执行文件
target_link_libraries(main mylibrary)
```

### 3. 编译共享库和可执行文件

在项目根目录下（即 `CMakeLists.txt` 所在的目录）打开终端，然后执行以下命令来构建项目：

#### 创建构建目录并生成构建文件

```bash
mkdir build
cd build
cmake ..
```

这将生成用于构建项目的 Makefile 或其他构建工具。

#### 编译项目

```bash
make
```

编译完成后，你会得到以下文件：

- `libmylibrary.so`：共享库文件（在 `build` 目录下）。
- `main`：生成的可执行文件。

### 4. 运行可执行文件

由于共享库 `libmylibrary.so` 可能不在默认的库搜索路径中，你需要确保运行时能找到它。你可以使用 `LD_LIBRARY_PATH` 环境变量来指定共享库的路径。

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:./
./main
```

如果一切顺利，你应该看到如下输出：

```
Hello from MyLibrary!
```

### 总结

通过 CMake 编译共享库的关键步骤是：

- 使用 `add_library(mylibrary SHARED ...)` 来创建共享库。
- 使用 `target_link_libraries(main mylibrary)` 来链接共享库到你的可执行文件。
- 通过 `LD_LIBRARY_PATH` 或将共享库放到标准路径下来运行程序。

CMake 会帮助你自动化这些步骤，使得构建过程更加规范和易于管理。


## 问题2：

如果我想在另外的地方使用这个库呢？

如果你想在其他项目中使用你创建的共享库，你需要将库文件和头文件正确地引用到新项目中。以下是如何在 CMake 中链接外部库（例如你创建的 `libmylibrary.so`）的步骤。

假设你有一个新的项目需要使用这个库。

### 假设新的项目结构如下：

```
/new_project
    CMakeLists.txt
    src/
        app.cpp
```

并且你的共享库文件 `libmylibrary.so` 和头文件 `my_library.h` 存放在 `/my_project/build` 中（或者你也可以将它们安装到系统的库路径或其他位置）。

### 步骤 1：将库文件和头文件复制或链接到新的项目

你有几个选择：

#### 1.1. 直接复制共享库和头文件到新的项目
你可以将 `libmylibrary.so` 和 `my_library.h` 文件复制到新的项目中：

```
/new_project
    CMakeLists.txt
    src/
        app.cpp
    lib/
        libmylibrary.so
    include/
        my_library.h
```

app.cpp：

```cpp
#include "my_library.h"

int main() {
    MyLibrary lib;
    lib.say_hello();
    return 0;
}

```

#### 1.2. 使用外部路径引用
如果你不想复制文件，而是希望新项目直接引用现有项目中的库文件和头文件，你可以设置新的项目的 CMake 配置，指定共享库和头文件的路径。

### 步骤 2：修改新项目的 `CMakeLists.txt` 文件

#### 2.1. 使用复制的库和头文件（即 `lib/` 和 `include/` 目录）

如果你选择了直接复制的方式（即新项目中有 `libmylibrary.so` 和 `my_library.h`），修改新的项目的 `CMakeLists.txt` 文件如下：

```cmake
cmake_minimum_required(VERSION 3.10)

project(NewProject)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 指定库和头文件的路径
include_directories(${CMAKE_SOURCE_DIR}/include)  # 指定头文件路径
link_directories(${CMAKE_SOURCE_DIR}/lib)  # 指定库文件路径

# 定义可执行文件
add_executable(app src/app.cpp)

# 链接到共享库
target_link_libraries(app mylibrary)
```

#### 2.2. 使用外部路径引用（如果共享库和头文件存放在不同的目录）

如果你选择不复制文件，而是引用 `libmylibrary.so` 和 `my_library.h` 文件在其他地方，你可以这样修改 `CMakeLists.txt` 文件：

```cmake
cmake_minimum_required(VERSION 3.10)

project(NewProject)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 指定库和头文件的路径
include_directories(/path/to/my_project/include)  # 设置头文件路径
link_directories(/path/to/my_project/build)  # 设置共享库文件路径

# 定义可执行文件
add_executable(app src/app.cpp)

# 链接到共享库
target_link_libraries(app mylibrary)
```

- `/path/to/my_project/include` 是你共享库头文件（`my_library.h`）的路径。
- `/path/to/my_project/build` 是你编译生成的共享库文件（`libmylibrary.so`）的路径。

### 步骤 3：编译新项目

在新的项目根目录下，创建构建目录并编译项目：

```bash
mkdir build
cd build
cmake ..
make
```

### 步骤 4：运行程序

如果共享库不在标准的库路径下，你需要设置 `LD_LIBRARY_PATH`，让程序在运行时能够找到它：

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/my_project/build
./app
```

### 总结

1. **如果复制了共享库和头文件**：
   - 在新项目的 `CMakeLists.txt` 中使用 `include_directories` 和 `link_directories` 指定头文件和共享库的位置。
   - 使用 `target_link_libraries` 将共享库链接到可执行文件。

2. **如果不复制文件，而是引用已有的共享库和头文件**：
   - 在新项目的 `CMakeLists.txt` 中指定外部路径。

在这两种情况下，`target_link_libraries` 会确保新项目链接到你创建的共享库，并且 `LD_LIBRARY_PATH` 变量在运行时确保程序能找到该共享库。


## 问题3：

为了在两个 C++ 项目中共享同一个库，并避免它们之间的冲突，你可以采取几种不同的策略，具体取决于你如何管理和使用这个共享库。下面是几种常见的做法和文件结构示例。

### 假设的文件结构：

```
/shared_lib_project
    CMakeLists.txt
    src/
        my_library.cpp
    include/
        my_library.h

/project_1
    CMakeLists.txt
    src/
        main_1.cpp

/project_2
    CMakeLists.txt
    src/
        main_2.cpp
```

在这个结构中，`/shared_lib_project` 是包含共享库的项目，`/project_1` 和 `/project_2` 是依赖该共享库的两个项目。

### 方法 1：使用 **CMake 外部项目** 进行共享库的构建和管理

你可以将共享库项目作为外部依赖项引入到这两个项目中。这样，`project_1` 和 `project_2` 都能独立使用该库，而且不会有冲突。

#### 步骤 1：为共享库创建 `CMakeLists.txt`

在 `shared_lib_project` 中，定义共享库：

```cmake
# shared_lib_project/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(MyLibrary)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 添加共享库
add_library(mylibrary SHARED src/my_library.cpp)

# 设置头文件路径
target_include_directories(mylibrary PUBLIC ${CMAKE_SOURCE_DIR}/include)
```

#### 步骤 2：在 `project_1` 和 `project_2` 中引入共享库

在 `project_1` 和 `project_2` 的 `CMakeLists.txt` 中，你可以通过 `ExternalProject` 模块引入 `shared_lib_project`，并在构建过程中自动编译和链接共享库。

##### 1.1. `project_1/CMakeLists.txt` 示例

```cmake
# project_1/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(Project1)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 通过 ExternalProject 引入共享库
include_directories(/path/to/shared_lib_project/include)
link_directories(/path/to/shared_lib_project/build)

# 定义可执行文件
add_executable(main_1 src/main_1.cpp)

# 链接共享库
target_link_libraries(main_1 mylibrary)
```

##### 1.2. `project_2/CMakeLists.txt` 示例

```cmake
# project_2/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(Project2)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 通过 ExternalProject 引入共享库
include_directories(/path/to/shared_lib_project/include)
link_directories(/path/to/shared_lib_project/build)

# 定义可执行文件
add_executable(main_2 src/main_2.cpp)

# 链接共享库
target_link_libraries(main_2 mylibrary)
```

在这种方法中，两个项目都依赖于同一个共享库，且它们之间不会发生冲突。

#### 步骤 3：编译和运行

编译时，`project_1` 和 `project_2` 会依赖于同一个共享库，并且共享库的源代码和头文件在两个项目中都可以被访问，而没有冲突。

### 方法 2：将共享库安装到系统路径（避免路径冲突）

如果你不想在每个项目中重复构建共享库，可以选择将共享库安装到系统的标准路径，或者你自己的一个库路径中，这样所有依赖该库的项目都可以通过该路径来链接共享库。

#### 步骤 1：修改 `CMakeLists.txt` 安装共享库

在 `shared_lib_project` 的 `CMakeLists.txt` 中，添加一个安装步骤来将共享库安装到某个路径：

```cmake
# shared_lib_project/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(MyLibrary)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 创建共享库
add_library(mylibrary SHARED src/my_library.cpp)

# 设置头文件路径
target_include_directories(mylibrary PUBLIC ${CMAKE_SOURCE_DIR}/include)

# 安装共享库
install(TARGETS mylibrary DESTINATION /path/to/install/lib)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION /path/to/install/include)
```

#### 步骤 2：在 `project_1` 和 `project_2` 中使用共享库

在 `project_1` 和 `project_2` 的 `CMakeLists.txt` 中，指明共享库和头文件的安装路径：

```cmake
# project_1/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(Project1)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 设置库和头文件安装路径
include_directories(/path/to/install/include)
link_directories(/path/to/install/lib)

# 定义可执行文件
add_executable(main_1 src/main_1.cpp)

# 链接共享库
target_link_libraries(main_1 mylibrary)
```

```cmake
# project_2/CMakeLists.txt

cmake_minimum_required(VERSION 3.10)

project(Project2)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)

# 设置库和头文件安装路径
include_directories(/path/to/install/include)
link_directories(/path/to/install/lib)

# 定义可执行文件
add_executable(main_2 src/main_2.cpp)

# 链接共享库
target_link_libraries(main_2 mylibrary)
```

#### 步骤 3：安装并构建

1. 在 `shared_lib_project` 中构建并安装共享库：

   ```bash
   cd /path/to/shared_lib_project
   mkdir build
   cd build
   cmake ..
   make
   sudo make install
   ```

2. 在 `project_1` 和 `project_2` 中使用 `cmake` 编译和链接：

   ```bash
   mkdir build
   cd build
   cmake ..
   make
   ```

### 方法 3：使用 **静态库**（如果适用）

如果你不希望多个项目依赖于同一个共享库，可以考虑将共享库改为静态库（`.a`）。这样，你可以将静态库直接嵌入到每个项目中，避免了共享库的路径问题，但会增加最终可执行文件的大小。

#### 修改 CMake 配置以生成静态库

```cmake
# shared_lib_project/CMakeLists.txt

add_library(mylibrary STATIC src/my_library.cpp)
```

然后你可以在每个项目中链接这个静态库，类似于链接共享库。

### 总结

- **方法 1：通过 CMake 引入外部共享库**：在多个项目中共享构建过程和库，避免路径冲突。
- **方法 2：将库安装到系统路径**：将共享库安装到统一的路径，多个项目可以引用该库，避免路径问题。
- **方法 3：静态库**：将共享库编译为静态库，每个项目链接自己的副本，避免共享库的依赖冲突。

选择哪种方式取决于你如何管理这些库，以及是否希望在每个项目中独立管理依赖项。




