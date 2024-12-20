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
