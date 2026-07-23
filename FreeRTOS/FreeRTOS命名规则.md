## 简介
在`HAL`库中的`FreeRTOS`中对应的库函数名和传统的裸机开发中的`HAL`库函数名类似，函数名都很长。不过好在`FreeRTOS`中的函数名也是有着对应的规则，因此我们可以尝试去理解它们，这样可以让我们的日常开发变得更加顺手

## 规则
`HAL`库中`FreeRTOS`对应的命名规则相较于原生的`FreeRTOS`更加统一，大致的命名规则如下：
`os + <模块/对象> + <具体动作>`

### 模块/对象名称
这部分内容跟在`os`后面，这部分内容代表我们要操作的具体系统资源，其首字母均为大写：
- `Kernel`：内核控制（系统启动、获取系统时间）
- `Thread`：任务线程（原生`FreeRTOS`的`Task`）
- `Semaphore`：信号量（用于同步）
- `Mutex`：互斥量（类似于互斥锁用于保护共享资源）
- `MessageQueue`：消息队列（用于任务之间的数据传输）
- `Timer`：软件定时器
- `EventFlags`：事件标志组（用于多事件同步）

### 具体动作
跟在对象后后面的单词代表我们要执行的操作，命名同样很规则

| 动作后缀         | 代表含义      | 代码示例                                         |
| ------------ | --------- | -------------------------------------------- |
| `New`        | 创建并初始化对象  | `osThreadNew()`<br>`osMessageQueueNew()`     |
| `Acquire`    | 获取/占用（资源） | `osMessageQueueNew()`<br>`osMutexAcquire`    |
| `Release`    | 释放/交出（资源） | `osSemaphoreRelease()`<br>`osMutexRelease()` |
| `Put / Set`  | 发送数据/设置标志 | `osMessageQueuePut()`<br>`osEventFlagSet()`  |
| `Get / Wait` | 接收数据/等待标志 | `osMessageQueueGet()`<br>`osEventFlagWait()` |
| `Delete`     | 删除对象回收内存  | `osThreadDelete()`                           |
|              |           |                                              |
