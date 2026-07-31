## 引言
在前面的[CLion添加头文件搜索路径](CLion添加头文件搜索路径.md)和[CLion添加源文件搜索路径](CLion添加源文件搜索路径.md)文章中我们学习了如何在`CLion`中添加对应的文件搜索路径，本次我们会学习如何进行一些进阶管理，让`CMakeLists.txt`文件中的内部不至于太冗长

## 实操
随着项目变得越来越复杂，我们需要控制的外设也越来越多，业务裸机也变得越来越复杂，也必然会创建更多的文件，如果我们将所有的文件都一股脑的放到`target_sources` 下，那么过不了多久该部分的代码就会变得无比臃肿，因此我们可以想办法通过定义变量来解决这一问题，我们可以将文件的路径信息保存到变量中，然后再将变量导入到`target_sources` 下，示例代码如下：
```cmake
# 1. 定义模块变量 
set(APP_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/App/main_task.c ${CMAKE_CURRENT_SOURCE_DIR}/App/motor_control.c ) 

set(DRIVER_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/Custom/imu_sensor.c ${CMAKE_CURRENT_SOURCE_DIR}/Drivers/Custom/can_bsp.c )
```
我们可以定义两个变量，一个用于存放业务逻辑相关的文件信息，另一个用于存放驱动文件的文件信息，然后我们再将这两个变量导入到`target_sources`中
```cmake
# 2. 将变量展开并添加到工程源文件中 
target_sources(${CMAKE_PROJECT_NAME} PRIVATE 
	${APP_SOURCES} 
	${DRIVER_SOURCES} 
)
```

对于初学者来说可能会感觉到疑惑的事是，为什么`CMakeLists.txt`文件中默认什么源文件路径都没有？按理说这样不会引发报错吗？这种怀疑是正确的，我们可以在`CMakeLists.txt`文件中看到这样一句话
![](assets/CLion嵌入式开发基操——文件管理/file-20260729153955379%201.png)
红色方框内的语句作用是添加其它文件夹下的`CMakeLists.txt`文件来导入编译信息，在程序中括号内的文件夹内刚好有一个`CMakeLists.txt`文件
![](assets/CLion嵌入式开发基操——文件管理/file-20260729153955379.png)
该文件夹中就包含了所有由`CubeMX`自动生成的源文件、`HAL`库外设驱动文件、以及系统启动文件
