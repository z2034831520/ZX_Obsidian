## 引言
本期内容介绍一下如何让`stm32`单片机控制蜂鸣器播放音乐，本期教程中使用的无源蜂鸣器就是市面上最常见的三引脚无源蜂鸣器模块
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828601.png)
使用的单片机主控也是嵌入式开发工程师几乎人手一个`STM32F103C8T6`，相信你看完的本期教程之后也能够轻松的使用`STM32`单片机控制无源蜂鸣器播放音乐

## 硬件介绍

### 底层发声原理
我们首先需要了解一下无源蜂鸣器的发声原理，无源蜂鸣器相较于有源蜂鸣器内部没有振荡器，我们无法通过恒定的高低电平来控制其发出声音。我们需要高低变化的方波脉冲信号来驱动其发出声音，其实无源蜂鸣器的发声原理很简单，我们向`I/O`引脚输入不同的信号电平时内部的金属片就会产生不同方向的形变，因此当我们快速的改变电平信号就相当于在快速的改变金属片的形变方向，金属片快速的振动就会发出声音。具体的原理可以参照下图：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828628.png)
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828630.png)

了解了无源蜂鸣器发声的原理之后我们不难得出常规的`GPIO`输出模式是无法满足我们的需求的，我们需要对应的引脚能够输出快速变化的高低电平。因此我们不难得出，只有`PWM`输出模式才能满足我们的需求

### 输出控制
我们了解了最底层的硬件发声原理之后，我们接下来需要了解一下如何通过改变`PWM`模式中不同参数的数值，来达到我们输出不同音调和响度的需求
#### 音调控制
总所周知声音是由物体振动产生的，音调是由物体振动的频率决定的，我们想要改变无源蜂鸣器输出声音的频率就是改变输出`PWM`波的频率
这点并不难理解，我们如果输出`1000Hz`的方波，蜂鸣器的膜片每秒会振动`1000`次，声音会比较尖锐。我们输出一个`440Hz`的方波，声音听起来会比较低沉

想要改变方波的频率主要有两种方法，一是改变预分频器（`PSC`）的值，二是改变自动重装载器（`ARR`）的值，在后续的操作中我们主要会选择通过修改自动重装载器的值来控制音调高低
#### 响度控制
想要控制无源蜂鸣器发声的响度只有一种办法，那就是改变`PWM`的占空比，占空比可以控制无源蜂鸣器中发声片的振幅。我们在代码中通过修改捕获比较寄存器（`CCR`）中的值来改变`PWM`方波的占空比
## 配置PA8和TIM1
当前蜂鸣器的`I/O`引脚连接到开发板上的`PA8`引脚，`PA8`可以复用为`TIM`的输出通道一，我们在配置界面中点击`PA8`引脚就可以配置引脚模式信息了
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828600.png)
将`PA8`配置为`TIM1_CH1`之后就可以配置`TIM`对应的参数信息了
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828599.png)
由于蜂鸣器模块采用低电平触发，`TIM1`的输出通道需要配置为低有效：
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828602.png)

TIM1 当前的参数配置为：
`TIM1`的主频为`72 MHz`，我们将预分频值设置为 `72-1` ，对应的定时器计数频率为：`72 MHz ÷ (71 + 1) = 1 MHz`，因此后续定时器每 `1μs`增加一次计数值。当前的计数模式我们配置为向上计数模式，自动重装载值我们设置为`250-1`。按照默认的参数`PWM`的频率为
```txt
PWM频率 = TIM1时钟 ÷ (PSC + 1) ÷ (ARR + 1)
```

```txt
ARR = 249
PWM频率 = 1,000,000 ÷ 250
        = 4000 Hz
```
所以初始化参数对应的基础频率为`4Khz`


后续需要播放不同音调时根据音符频率动态修改自动重装载器的值：
~~~text
ARR = 1,000,000 ÷ 音符频率 - 1
~~~

配置的几个参数之间的关系如下图所示
![](https://cdn.jsdelivr.net/gh/z2034831520/obsidian-image-bed@main/images/file-20260813183828630%201.png)

## 代码讲解
接下来我会讲解一下部分核心代码是如何编写的，如果想要直接看完整的源代码可以跳转到源代码展示部分

播放音乐部分的代码我们分为两块进行编写，分别是底层的蜂鸣器驱动以及歌曲的曲谱播放，其中底层驱动代码位于`buzzer_music.c`和`buzzer_music.h`中，音乐播放部分代码位于`buzzer_songs.c`和`buzzer_songs.h`中
### 编写蜂鸣器底层驱动

首先我们需要编写蜂鸣器的底层驱动，即控制蜂鸣器发出不同音调，不同响度的声音。在该部分我们主要实现了`BuzzerMusic_SetTone`函数，该函数可以实现根据我们传入的频率信息进行计算，然后将计算出的结果写入指定的寄存器中，以此来实现输出指定的音调
```c
// 根据目标频率配置TIM1 PWM，以输出指定音调  
void BuzzerMusic_SetTone(uint16_t frequency_hz)  
{  
  uint32_t period_counts;  
  
  // 如果频率值为0那么直接停止输出  
  if (frequency_hz == BUZZER_MUSIC_REST)  
  {  
    BuzzerMusic_Stop();  
    return;  
  }  
  
  //计算自动重装载器的值  
  period_counts = BUZZER_COUNTER_CLOCK_HZ / frequency_hz;  
  
  // 越界检查  
  if ((period_counts < 2U) || (period_counts > BUZZER_MAX_PERIOD_COUNTS))  
  {  
    BuzzerMusic_Stop();  
    return;  
  }  
  
  // 将对应值写入重装载寄存器和比较寄存器中  
  __HAL_TIM_SET_AUTORELOAD(&htim1, period_counts - 1U);  
  __HAL_TIM_SET_COMPARE(&htim1, BUZZER_TIMER_CHANNEL, (period_counts * BUZZER_VOLUME_PERCENT) / 100U);  
}
```

### 迁移《晴天》乐谱
其实底层的驱动编写并不困难，真正让人感到头疼的是如何将音乐转换成可以被无源蜂鸣器播放的形式，在本次项目中我采取的是将歌曲拆分成一个个音符，然后通过结构体保存每个音符的信息
```c
// 音符结构体，保存一个音符的音调和持续时间信息  
typedef struct  
{  
  //音调  
  uint16_t frequency_hz;  
  //持续时间  
  uint16_t duration_ms;  
} SunnyNote;
```

然后创建一个结构体数组来保存每一个音符的信息方便我们后续遍历并播放
`static const SunnyNote sunny_score[]`
由于数组内容过长这里就先只展示部分信息
接下来在音乐播放函数中调用之前编写好的底层驱动函数来播放对应的音符信息
```c
// 开始循环遍历  
for (index = 0U; index < ARRAY_SIZE(sunny_score); index++)  
{  
  // 读取音符信息，准备播放  
  uint16_t frequency_hz = sunny_score[index].frequency_hz;  
  uint16_t duration_ms = sunny_score[index].duration_ms;  
  uint32_t gap_ms = SUNNY_DEFAULT_GAP_MS;  
  
  if (frequency_hz == BUZZER_MUSIC_REST)  
  {  
    BuzzerMusic_Stop();  
  }  
  else  
  {  
    BuzzerMusic_SetTone(frequency_hz);  
  }  
  
  if (duration_ms > 0U)  
  {  
    osDelay(duration_ms);  
  }  
  BuzzerMusic_Stop();  
  
  //特定音符之间取消间隔  
  if ((index == 49U) || (index == 67U) || (index == 178U))  
  {  
    gap_ms = 0U;  
  }  
  if (gap_ms > 0U)  
  {  
    osDelay(gap_ms);  
  }  
}
```
播放每个音符的时候我们只需要把握好音调、持续时间和与下一个音符之间的间隔即可

## 在main中启动PWM
最后记得在`main`函数中启动`PWM`输出
~~~c
MX_GPIO_Init();
MX_TIM1_Init();

if (HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1) != HAL_OK)
{
  Error_Handler();
}
~~~

## 调用音乐播放函数
我们可以自行选择音乐播放函数的调用位置，这里我选择在 `freertos.c` 中的任务里调用对应的音乐播放函数，首先包含音乐驱动头文件：
~~~c
#include "buzzer_music.h"
#include "buzzer_songs.h"
~~~
然后在默认任务中初始化蜂鸣器并播放《晴天》：
~~~c
void StartDefaultTask(void *argument)
{
  BuzzerMusic_Init();

  for (;;)
  {
    BuzzerSongs_PlaySunny();
  }
}
~~~
## 调整蜂鸣器响度

当前项目通过`buzzer_music.c`文件中的宏定义`BUZZER_VOLUME_PERCENT`来控制`PWM`的有效占空比：
~~~c
#define BUZZER_VOLUME_PERCENT 10U
~~~
声音太大时，可以把这个值调小，例如：
~~~c
#define BUZZER_VOLUME_PERCENT 5U
~~~

## 源代码展示
`buzzer_music.c`:
```c
#include "buzzer_music.h"  
  
#include "tim.h"  
  
// 定时器输出通道  
#define BUZZER_TIMER_CHANNEL          TIM_CHANNEL_1  
// TIM的计数频率  
#define BUZZER_COUNTER_CLOCK_HZ       1000000U  
//最大计数值  
#define BUZZER_MAX_PERIOD_COUNTS      65536U  
//播放音量大小值，百分比  
#define BUZZER_VOLUME_PERCENT         2U  
  
// 初始化蜂鸣器模块，默认为静音状态  
void BuzzerMusic_Init(void)  
{  
  BuzzerMusic_Stop();  
}  
  
// 根据目标频率配置TIM1 PWM，以输出指定音调  
void BuzzerMusic_SetTone(uint16_t frequency_hz)  
{  
  uint32_t period_counts;  
  
  // 如果频率值为0那么直接停止输出  
  if (frequency_hz == BUZZER_MUSIC_REST)  
  {  
    BuzzerMusic_Stop();  
    return;  
  }  
  
  //计算自动重装载器的值  
  period_counts = BUZZER_COUNTER_CLOCK_HZ / frequency_hz;  
  
  // 越界检查  
  if ((period_counts < 2U) || (period_counts > BUZZER_MAX_PERIOD_COUNTS))  
  {  
    BuzzerMusic_Stop();  
    return;  
  }  
  
  // 将对应值写入重装载寄存器和比较寄存器中  
  __HAL_TIM_SET_AUTORELOAD(&htim1, period_counts - 1U);  
  __HAL_TIM_SET_COMPARE(&htim1, BUZZER_TIMER_CHANNEL, (period_counts * BUZZER_VOLUME_PERCENT) / 100U);  
}  
  
// 将PWM有效占空比清零，使低电平触发蜂鸣器停止发声  
void BuzzerMusic_Stop(void)  
{  
  // 将捕获比较寄存器中的值设置为0，无源蜂鸣器停止发声  
  __HAL_TIM_SET_COMPARE(&htim1, BUZZER_TIMER_CHANNEL, 0U);  
}
```

`buzzer_music.h`:
```c
#ifndef BUZZER_MUSIC_H  
#define BUZZER_MUSIC_H  
  
#include <stdint.h>  
  
#define BUZZER_MUSIC_REST  0U  
  
void BuzzerMusic_Init(void);  
void BuzzerMusic_SetTone(uint16_t frequency_hz);  
void BuzzerMusic_Stop(void);  
  
#endif /* BUZZER_MUSIC_H */
```

`buzzer_songs.c`:
```c
#include "buzzer_songs.h"  
  
#include <stddef.h>  
#include <stdint.h>  
  
#include "buzzer_music.h"  
#include "cmsis_os2.h"  
  
// 宏函数作用是获取数组长度  
#define ARRAY_SIZE(array)              (sizeof(array) / sizeof((array)[0]))  
// 默认音符停顿时间  
#define SUNNY_DEFAULT_GAP_MS           5U  
  
// 音符结构体，保存一个音符的音调和持续时间信息  
typedef struct  
{  
  //音调  
  uint16_t frequency_hz;  
  //持续时间  
  uint16_t duration_ms;  
} SunnyNote;  
  
// 《晴天》乐谱：每项依次表示音调频率和持续时间，频率为0时表示休止  
static const SunnyNote sunny_score[] =  
{  
  {1760U, 360U}, {2092U, 360U}, {3136U, 360U}, {2092U, 360U}, {1396U, 360U},  
  {1568U, 180U}, {1760U, 180U}, {3136U, 360U}, {2092U, 360U}, {2092U, 360U},  
  {1568U, 360U}, {3136U, 360U}, {2092U, 360U}, {2092U, 360U}, {3136U, 360U},  
  {1976U, 360U}, {3136U, 360U}, {1760U, 360U}, {2092U, 360U}, {3136U, 360U},  
  {2092U, 360U}, {1396U, 360U}, {1568U, 180U}, {1760U, 180U}, {3136U, 360U},  
  {2092U, 360U}, {2092U, 360U}, {1568U, 360U}, {3136U, 360U}, {2092U, 360U},  
  {2092U, 360U}, {3136U, 360U}, {1568U, 180U}, {2092U, 180U}, {3136U, 360U},  
  {0U, 360U}, {3136U, 360U}, {3136U, 360U}, {2092U, 360U}, {2092U, 720U},  
  {2348U, 360U}, {2636U, 360U}, {0U, 360U}, {3136U, 360U}, {3136U, 360U},  
  {2092U, 360U}, {2092U, 360U}, {2348U, 180U}, {2636U, 180U}, {2348U, 180U},  
  {2092U, 180U}, {1568U, 360U}, {0U, 360U}, {3136U, 360U}, {3136U, 360U},  
  {2092U, 360U}, {2092U, 720U}, {2348U, 360U}, {2636U, 360U}, {0U, 360U},  
  {2636U, 720U}, {2348U, 180U}, {2636U, 180U}, {2792U, 180U}, {2636U, 180U},  
  {2348U, 180U}, {2792U, 180U}, {2636U, 180U}, {2348U, 180U}, {2092U, 360U},  
  {1568U, 360U}, {2092U, 360U}, {2092U, 360U}, {2636U, 360U}, {2792U, 360U},  
  {2636U, 360U}, {2348U, 360U}, {2092U, 180U}, {2348U, 180U}, {2636U, 360U},  
  {2636U, 360U}, {2636U, 360U}, {2636U, 360U}, {2348U, 180U}, {2636U, 180U},  
  {2348U, 360U}, {2092U, 720U}, {1568U, 360U}, {2092U, 360U}, {2348U, 360U},  
  {2636U, 360U}, {2792U, 360U}, {2636U, 360U}, {2348U, 360U}, {2092U, 180U},  
  {2348U, 180U}, {2636U, 360U}, {2636U, 360U}, {2636U, 360U}, {2636U, 360U},  
  {2348U, 180U}, {2636U, 180U}, {2348U, 360U}, {2092U, 540U}, {0U, 2880U},  
  {2092U, 180U}, {2092U, 180U}, {2092U, 180U}, {2092U, 180U}, {1760U, 360U},  
  {1976U, 360U}, {2092U, 360U}, {3136U, 360U}, {2792U, 360U}, {2636U, 360U},  
  {2092U, 360U}, {2092U, 900U}, {0U, 360U}, {2092U, 180U}, {2092U, 180U},  
  {2092U, 180U}, {2092U, 180U}, {2636U, 360U}, {2092U, 360U}, {1760U, 360U},  
  {1976U, 360U}, {2092U, 360U}, {3136U, 360U}, {2792U, 360U}, {2636U, 360U},  
  {2092U, 360U}, {2348U, 900U}, {2636U, 360U}, {2348U, 360U}, {2792U, 360U},  
  {2636U, 720U}, {2092U, 360U}, {3136U, 360U}, {3952U, 360U}, {4188U, 360U},  
  {3952U, 360U}, {3136U, 360U}, {2092U, 360U}, {0U, 360U}, {2092U, 360U},  
  {3520U, 360U}, {3520U, 360U}, {0U, 360U}, {3520U, 360U}, {3136U, 360U},  
  {3136U, 360U}, {0U, 360U}, {3136U, 360U}, {2792U, 360U}, {2636U, 360U},  
  {2348U, 360U}, {2636U, 360U}, {2792U, 360U}, {2636U, 900U}, {0U, 540U},  
  {2636U, 360U}, {2792U, 360U}, {3136U, 360U}, {2636U, 360U}, {0U, 360U},  
  {2792U, 360U}, {3136U, 360U}, {3952U, 360U}, {4700U, 360U}, {3952U, 360U},  
  {4188U, 360U}, {4188U, 900U}, {0U, 360U}, {4188U, 360U}, {4188U, 360U},  
  {3136U, 360U}, {3136U, 360U}, {3520U, 360U}, {3136U, 360U}, {2792U, 360U},  
  {2348U, 360U}, {2636U, 360U}, {2792U, 360U}, {3136U, 360U}, {3520U, 360U},  
  {2092U, 360U}, {3520U, 540U}, {3952U, 180U}, {3952U, 720U}, {2636U, 360U},  
  {2348U, 360U}, {2792U, 360U}, {2636U, 360U}, {0U, 360U}, {2092U, 360U},  
  {3136U, 360U}, {3952U, 360U}, {4188U, 360U}, {3952U, 360U}, {3136U, 360U},  
  {2092U, 360U}, {0U, 360U}, {2092U, 360U}, {3520U, 360U}, {3520U, 360U},  
  {0U, 360U}, {3520U, 360U}, {3136U, 360U}, {3136U, 360U}, {0U, 360U},  
  {3136U, 360U}, {2792U, 360U}, {2636U, 360U}, {2348U, 360U}, {2636U, 360U},  
  {2792U, 360U}, {2636U, 1080U}, {0U, 360U}, {2636U, 360U}, {2792U, 360U},  
  {3136U, 360U}, {2636U, 360U}, {0U, 360U}, {2792U, 360U}, {3136U, 360U},  
  {3952U, 360U}, {4700U, 360U}, {3952U, 360U}, {4188U, 360U}, {4188U, 900U},  
  {0U, 360U}, {2092U, 360U}, {4188U, 360U}, {3136U, 360U}, {3136U, 360U},  
  {3520U, 360U}, {3136U, 360U}, {2792U, 360U}, {1760U, 360U}, {1976U, 360U},  
  {2092U, 360U}, {2348U, 360U}, {2636U, 360U}, {2348U, 720U}, {0U, 0U},  
  {2636U, 360U}, {2092U, 1440U}  
};  
  
// 遍历迁移后的音调和时值数据，完整播放《晴天》旋律  
void BuzzerSongs_PlaySunny(void)  
{  
  size_t index;  
  
  // 开始循环遍历  
  for (index = 0U; index < ARRAY_SIZE(sunny_score); index++)  
  {  
    // 读取音符信息，准备播放  
    uint16_t frequency_hz = sunny_score[index].frequency_hz;  
    uint16_t duration_ms = sunny_score[index].duration_ms;  
    uint32_t gap_ms = SUNNY_DEFAULT_GAP_MS;  
  
    if (frequency_hz == BUZZER_MUSIC_REST)  
    {  
      BuzzerMusic_Stop();  
    }  
    else  
    {  
      BuzzerMusic_SetTone(frequency_hz);  
    }  
  
    if (duration_ms > 0U)  
    {  
      osDelay(duration_ms);  
    }  
    BuzzerMusic_Stop();  
  
    //特定音符之间取消间隔  
    if ((index == 49U) || (index == 67U) || (index == 178U))  
    {  
      gap_ms = 0U;  
    }  
    if (gap_ms > 0U)  
    {  
      osDelay(gap_ms);  
    }  
  }  
}
```

`buzzer_songs.h`:
```c
#ifndef BUZZER_SONGS_H  
#define BUZZER_SONGS_H  
  
void BuzzerSongs_PlaySunny(void);  
  
#endif /* BUZZER_SONGS_H */
```