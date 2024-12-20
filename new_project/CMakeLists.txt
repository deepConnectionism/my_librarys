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
