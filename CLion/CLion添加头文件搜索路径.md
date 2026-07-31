## 简介
在`CLion`中我们在项目中添加了新的源文件或者新的文件夹之后，我们需要在项目中添加对应的文件信息，这步操作就是在给新建的文件和文件夹**上户口**，否则在编译的时候编译器就无法找到对应的文件，这也必然会导致报错，接下来我会介绍一下如何在`CLion`中添加对应的头文件搜索路径

## 实操
如果我们新建了一个包含头文件的文件夹，且我们还在程序中引用了该文件夹中的头文件，那么我们就需要告诉编译器去哪里找这些`.h`文件，对应部分的代码如下：

```cmake
# Add include paths  
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE  
    # Add user defined include paths  
)
```
假设我们创建了一个`App`文件夹，并在该文件夹下又创建了一个`Inc`文件夹，我们将程序中引用的部分头文件放入了改文件夹中，那么我们就需要使用`${CMAKE_CURRENT_SOURCE_DIR}/App/Inc`语句将改文件夹添加到程序的头文件搜索路径中，修改后的代码部分如下：
```cmake
# 添加头文件搜索路径
target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE
    # Add user defined include paths
    ${CMAKE_CURRENT_SOURCE_DIR}/App/Inc
)
```
该命令和添加源文件的命令形式极为相似，我们可以类比的去记忆
