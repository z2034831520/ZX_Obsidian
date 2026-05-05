在`CMake`的所有指令和配置都写在一个名为`CMakeList.txt`的文本文件中，每个项目的根目录下往往会放置一个

每个配置文件都应该包含以下三句命令
```CMake
# 1. 指定项目需要的最低 CMake 版本要求
cmake_minimum_required(VERSION 3.10)

# 2. 声明项目名称和使用的语言（默认是 C 和 C++）
project(ProjectName)

# 3. 指定生成可执行文件
# 将 main.cpp 编译成名为 "hello_world" 的可执行程序
add_executable(hello_world main.cpp)
```
