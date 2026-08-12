---
source: ai-generated
style_id: zhou-writing-style-v1
style_version: "1.1"
ai_operation: generated
updated: 2026-08-12
---

# STM32无源蜂鸣器播放音乐——基于TIM1与FreeRTOS

最近在使用 STM32F103C8T6 做开发时，我需要通过 PA8 引脚控制一个低电平触发的无源蜂鸣器。最开始的目标只是让蜂鸣器发出声音，后面又继续尝试调整响度、播放简单旋律，并把另一个工程中的《晴天》乐谱迁移了过来。

这个过程看起来只是“输出一段 PWM”，但真正做起来会涉及定时器频率、音符时长、占空比、旧乐谱参数转换以及 FreeRTOS 延时等内容。本文就结合当前项目，记录一下无源蜂鸣器播放音乐的完整实现思路

## 一、有源蜂鸣器和无源蜂鸣器的区别

有源蜂鸣器内部自带振荡电路，只要给它有效电平就会按照固定频率发声。无源蜂鸣器内部没有振荡源，需要单片机持续输出方波才能发出声音。

因此，无源蜂鸣器除了可以发声，还可以通过改变方波频率产生不同音调。比如：

- 262 Hz 左右对应 C4，也就是常见的中音 Do；
- 294 Hz 左右对应 D4；
- 330 Hz 左右对应 E4；
- 392 Hz 左右对应 G4；
- 440 Hz 对应标准音 A4。

只要按一定顺序输出不同频率，并控制每个频率持续的时间，就可以组成一段旋律。

当前开发板使用的是低电平触发蜂鸣器模块，蜂鸣器的 I/O 引脚连接到 STM32F103C8T6 的 PA8。PA8 正好可以复用为 TIM1_CH1，因此这里使用 TIM1 的 PWM 输出控制蜂鸣器。

## 二、整体播放流程

当前项目中的调用过程可以整理为：

~~~text
main()
  ├─ MX_TIM1_Init()
  ├─ HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1)
  └─ osKernelStart()
       └─ StartDefaultTask()
            ├─ BuzzerMusic_Init()
            └─ BuzzerSongs_PlaySunny()
                 ├─ 读取当前音符参数
                 ├─ 将旧PSC换算成频率
                 ├─ 将时值换算成毫秒
                 ├─ BuzzerMusic_SetTone()
                 ├─ osDelay()
                 └─ BuzzerMusic_Stop()
~~~

其中，buzzer_music.c 负责底层蜂鸣器控制，包括设置音调、停止输出和播放单个音符；buzzer_songs.c 负责保存并播放具体歌曲。这样做以后，底层驱动和歌曲数据相互独立，后续增加新歌曲时不需要重复修改 TIM1 的控制代码。

## 三、TIM1如何产生蜂鸣器需要的方波

### 1. 定时器基础参数

当前 TIM1 的初始化参数为：

~~~c
htim1.Init.Prescaler = 72 - 1;
htim1.Init.Period = 250 - 1;
~~~

STM32F103C8T6 当前使用 72 MHz 定时器时钟，预分频器 PSC 设置为 71，因此 TIM1 的计数频率为：

~~~text
72 MHz ÷ (71 + 1) = 1 MHz
~~~

也就是说，定时器每计数一次需要 1 μs。

PWM 的输出频率由 PSC 和 ARR 共同决定：

~~~text
PWM频率 = 定时器时钟 ÷ (PSC + 1) ÷ (ARR + 1)
~~~

由于 PSC 已经固定为 71，计数频率固定为 1 MHz，所以程序只需要根据音符频率修改 ARR：

~~~text
ARR = 1,000,000 ÷ 音符频率 - 1
~~~

例如，希望蜂鸣器输出 4000 Hz 方波：

~~~text
ARR = 1,000,000 ÷ 4000 - 1 = 249
~~~

这也就是 TIM1 初始化时 Period 设置为 250 - 1 的原因。

### 2. 根据音符频率设置ARR

项目中的 BuzzerMusic_SetTone() 用于完成频率转换：

~~~c
period_counts = (BUZZER_COUNTER_CLOCK_HZ + (frequency_hz / 2U)) /
                frequency_hz;

__HAL_TIM_SET_AUTORELOAD(&htim1, period_counts - 1U);
__HAL_TIM_SET_COMPARE(&htim1,
                      TIM_CHANNEL_1,
                      (period_counts * BUZZER_VOLUME_PERCENT) / 100U);
__HAL_TIM_SET_COUNTER(&htim1, 0U);
HAL_TIM_GenerateEvent(&htim1, TIM_EVENTSOURCE_UPDATE);
~~~

这里先根据目标音符频率算出一个 PWM 周期需要多少个计数，再写入 ARR。公式中加入 frequency_hz / 2U，是为了在整数除法时进行四舍五入，减小频率误差。

修改 ARR 和 CCR 后，程序会把计数器清零，并产生一次更新事件，让新参数立即生效。如果只修改寄存器却没有正确启动 PWM，PA8 上仍然不会输出方波，因此 main() 中还需要执行：

~~~c
HAL_TIM_PWM_Start(&htim1, TIM_CHANNEL_1);
~~~

## 四、音调和节奏是如何协同的

一段音乐最基本的两个参数是音调和时值：

- 音调决定“唱哪个音”，在程序中表现为 PWM 频率；
- 时值决定“这个音唱多久”，在程序中表现为延时时间。

项目中定义了如下音符结构体：

~~~c
typedef struct
{
  uint16_t frequency_hz;
  uint16_t duration_ms;
} BuzzerMusicNote;
~~~

一个音符同时保存频率和持续时间。例如：

~~~c
{262U, 300U}
~~~

表示输出约 262 Hz 的方波并持续 300 ms。

播放单个音符时，BuzzerMusic_PlayTone() 先调用 BuzzerMusic_SetTone() 设置频率，再通过 osDelay() 保持这段时间，最后停止 PWM 有效输出：

~~~c
BuzzerMusic_SetTone(frequency_hz);
osDelay(note_on_ms);
BuzzerMusic_Stop();
osDelay(duration_ms - note_on_ms);
~~~

当前程序使用 BUZZER_NOTE_ON_PERCENT = 90，也就是一个音符总时长的前 90% 发声，后 10% 静音。这个很短的空隙可以把相邻的同音音符分开，避免它们听起来像一个连续长音。

因此，音调和节奏并不是由两个外设同时完成的。TIM1 持续负责产生当前频率的 PWM，FreeRTOS 任务通过 osDelay() 决定这个频率保持多长时间。延时结束后，程序再设置下一个音调。

## 五、低电平触发和静音处理

TIM1_CH1 当前配置为 PWM1 模式，并设置为低有效：

~~~c
sConfigOC.OCMode = TIM_OCMODE_PWM1;
sConfigOC.OCPolarity = TIM_OCPOLARITY_LOW;
~~~

低有效表示 PWM 的有效部分表现为低电平，这与当前蜂鸣器模块的低电平触发方式相匹配。

停止播放时，项目把 CCR 设置为 0：

~~~c
__HAL_TIM_SET_COMPARE(&htim1, TIM_CHANNEL_1, 0U);
~~~

在当前“PWM1 + 低极性”的配置下，CCR 为 0 会让 PA8 保持高电平，从而使低电平触发模块进入静音状态。这里不能脱离极性配置单独理解 CCR=0。如果 PWM 极性改成高有效，相同设置得到的引脚电平含义也会改变。

## 六、如何调节蜂鸣器响度

PWM 的频率主要决定音调，占空比会影响蜂鸣器接收到的平均驱动能量，因此可以在一定范围内改变听感响度。

当前工程使用：

~~~c
#define BUZZER_VOLUME_PERCENT 10U
~~~

设置比较值时，CCR 为整个周期计数值的 10%：

~~~c
CCR = period_counts × 10%
~~~

如果声音太大，可以继续把这个值调小，例如改为 5；如果声音太小，则可以适当调高。

不过，占空比与实际响度并不是严格的线性关系。蜂鸣器的谐振特性、模块上的驱动电路、供电电压和安装方式都会影响最终声音。同时，占空比过小可能使某些音调无法稳定发声。

如果需要更平滑、更明显的音量控制，最好增加三极管或 MOS 管驱动，并结合电阻、电源电压或硬件衰减进行调整。仅靠 PWM 占空比更适合做简单的软件音量控制。

## 七、《晴天》旧乐谱中的PSC参数如何转换

迁移过来的《晴天》工程没有直接保存音符频率，而是保存了原 TIM2 的 PSC 参数。旧工程的定时器时钟为 72 MHz，ARR 固定为 99，因此输出频率为：

~~~text
频率 = 72,000,000 ÷ (PSC + 1) ÷ (99 + 1)
     = 720,000 ÷ (PSC + 1)
~~~

当前项目使用 TIM1，并固定 PSC=71，通过修改 ARR 设置频率。因此不能直接把旧 PSC 写入当前 TIM1，而是要先把旧 PSC 还原成实际频率，再交给 BuzzerMusic_SetTone() 计算新的 ARR。

转换函数如下：

~~~c
divisor = (uint32_t)prescaler + 1U;
frequency_hz = (SUNNY_TIMER_FACTOR_HZ + (divisor / 2U)) / divisor;
frequency_hz <<= SUNNY_OCTAVE_SHIFT;
~~~

SUNNY_TIMER_FACTOR_HZ 的值为 720000，对应旧定时器中已经合并 ARR=99 后的换算系数。

SUNNY_OCTAVE_SHIFT 设置为 2，因此最终频率会左移两位，也就是乘以 4，相当于整体升高两个八度。这样处理是为了让迁移后的频率更接近当前蜂鸣器容易发声的范围。

例如旧 PSC 为 1635：

~~~text
原频率 = 720,000 ÷ (1635 + 1)
       ≈ 440 Hz

移高两个八度后：
新频率 ≈ 440 × 4
       ≈ 1760 Hz
~~~

1760 Hz 再传入 BuzzerMusic_SetTone()，程序会按照当前 1 MHz 计数频率重新计算 ARR：

~~~text
ARR ≈ 1,000,000 ÷ 1760 - 1
    ≈ 567
~~~

这样就完成了“旧 PSC 参数 → 实际音符频率 → 当前 TIM1 ARR”的转换。

## 八、乐谱中的时值如何转换为节奏

旧工程还保存了一组 sunny_time_units 数组，里面常见的数值有 25、50 等。项目使用下面的函数把它们换算为毫秒：

~~~c
return ((uint32_t)time_unit / 25U) * 180U;
~~~

按照这个公式：

- 25 对应 180 ms；
- 50 对应 360 ms；
- 75 对应 540 ms；
- 100 对应 720 ms。

这里保留了旧工程的整数运算方式。如果 time_unit 不是 25 的整数倍，小数部分会被直接舍去。

歌曲播放时，两组数组使用相同下标一一对应：

~~~c
frequency_hz = SunnyFrequencyFromPrescaler(sunny_prescalers[index]);
duration_ms = SunnyDurationMs(sunny_time_units[index]);
~~~

sunny_prescalers[index] 决定当前音符的音调，sunny_time_units[index] 决定当前音符持续多久。随后程序设置 PWM、执行 osDelay(duration_ms)，再停止输出并加入默认 5 ms 间隔。

PSC=30 在旧乐谱中被约定为休止符。程序识别到该数值后不会进行频率换算，而是返回 BUZZER_MUSIC_REST，也就是 0 Hz。播放函数收到休止符后直接调用 BuzzerMusic_Stop()，但仍保留对应的延时时间，因此节奏不会被打乱。

## 九、FreeRTOS任务中的播放方式

默认任务中的代码为：

~~~c
BuzzerMusic_Init();

for (;;)
{
  BuzzerSongs_PlaySunny();
}
~~~

BuzzerMusic_Init() 会先让蜂鸣器进入静音状态。之后默认任务不断调用 BuzzerSongs_PlaySunny()，所以一首歌曲播放结束后会立即重新开始。

osDelay() 只会阻塞当前任务，不会像裸机中的普通忙等待那样一直占用 CPU。等待音符结束期间，FreeRTOS 可以调度其他已就绪任务运行。

不过，当前播放接口仍然是“从任务角度阻塞”的：调用 BuzzerSongs_PlaySunny() 后，这个任务要等整首歌播放完成才能继续执行后面的代码。如果后续需要随时暂停、切歌或响应按键，可以把播放过程改成状态机，每次只处理一个音符，并使用软件定时器、消息队列或单独的音乐任务控制播放状态。

## 十、增加一首新歌曲的方法

如果歌曲直接使用频率和时长，可以定义 BuzzerMusicNote 数组：

~~~c
static const BuzzerMusicNote song[] =
{
  {262U, 300U},
  {294U, 300U},
  {330U, 300U},
  {0U,   150U},
  {392U, 600U}
};
~~~

然后调用：

~~~c
BuzzerMusic_Play(song, sizeof(song) / sizeof(song[0]));
~~~

如果乐谱采用频率表和索引数组，可以使用 BuzzerMusic_PlayIndexed()。这种写法适合大量重复音符，可以减少乐谱数据占用，并通过 octave_shift 参数整体移调。

编写乐谱时要注意三点：

1. 音符频率不能超过 uint16_t 的范围；
2. 根据当前 1 MHz 计数频率和 16 位 ARR 的限制，过低或过高的频率可能无法生成；
3. 每个音符之间最好留出几毫秒静音，否则连续相同音调不容易分辨。

## 十一、常见问题排查

### 1. 程序运行后完全没有声音

先确认 HAL_TIM_PWM_Start() 是否执行成功，再用示波器或逻辑分析仪检查 PA8 是否输出方波。同时检查 PA8 是否确实配置为 TIM1_CH1 复用推挽输出。

### 2. PA8有波形但蜂鸣器不响

检查蜂鸣器是否真的是无源蜂鸣器、模块是否共地、供电电压是否正确，以及模块的触发极性是否与 TIM1 输出极性一致。还要确认输出频率是否位于蜂鸣器能够明显发声的范围内。

### 3. 能发声但听不出旋律

重点检查音符频率换算和时值数组是否正确对应。迁移其他项目的 PSC 数据时，必须同时确认原工程的定时器时钟、PSC 和 ARR，不能只复制一个 PSC 数组。

### 4. 声音太大或太小

先调整 BUZZER_VOLUME_PERCENT，并观察不同音调下的表现。如果软件占空比调节效果不稳定，再从驱动电路、串联电阻或供电方式上处理。

### 5. 相邻音符粘在一起

在两个音符之间增加短暂静音，或者让单个音符只在总时长的 90% 内发声。当前通用播放函数已经使用这种方法，《晴天》播放函数则默认增加 5 ms 间隔。

## 十二、总结

无源蜂鸣器播放音乐，本质上是让定时器按照不同频率输出 PWM，并让每个频率保持指定时长。

当前项目使用 TIM1_CH1 从 PA8 输出低有效 PWM。TIM1 的 PSC 固定为 71，使计数频率保持在 1 MHz；播放音符时根据目标频率动态计算 ARR，通过 CCR 设置约 10% 的有效占空比，再由 FreeRTOS 的 osDelay() 控制音符时长。

《晴天》乐谱迁移时，旧工程保存的是 TIM2 的 PSC 参数，因此程序先根据 720000 ÷ (PSC + 1) 还原频率，再升高两个八度，最后交给当前 TIM1 驱动重新计算 ARR。时值数组则按照每 25 个单位对应 180 ms 的方式转换。

把频率生成、节奏控制和歌曲数据分开以后，整个结构会清晰很多：定时器只负责产生声音，音乐驱动负责播放音符，歌曲文件只保存旋律数据。后续无论是增加新歌曲、修改节奏，还是加入暂停和切歌功能，都可以在现有结构上继续扩展。
