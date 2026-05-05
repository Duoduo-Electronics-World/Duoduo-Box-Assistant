# 调试器读取 STM32 变量说明

本文说明如何在 STM32 程序里定义一组测试变量，并通过本上位机的“通用调试器”页面读取内存、绑定到图表通道。

## 1. 基本原则

调试器读取的是芯片内存地址。上位机不知道变量名，只知道：

```text
地址 + 数据类型
```

所以单片机程序里需要做到两件事：

1. 变量必须真实存在于 RAM 中。
2. 上位机要拿到变量地址，并按正确类型解析。

推荐先用“全局变量 + 查看地址”的方式测试。这样不会和普通 RAM 分配冲突，也最容易验证。

## 2. STM32 测试结构体

在 STM32 工程里新建或加入下面代码，例如放到 `main.c` 或单独的 `debug_data.c/.h`。

```c
#include <stdint.h>
#include <stdbool.h>

typedef enum
{
    DEBUG_STATE_IDLE = 0,
    DEBUG_STATE_RUN  = 1,
    DEBUG_STATE_ERR  = 2
} DebugState_t;

typedef struct
{
    int32_t      ch0_i32;     // 上位机类型：int32
    uint32_t     ch1_u32;     // 上位机类型：uint32
    float        ch2_float;   // 上位机类型：float
    int16_t      ch3_i16;     // 上位机类型：int16
    uint16_t     ch4_u16;     // 上位机类型：uint16
    int8_t       ch5_i8;      // 上位机类型：int8
    uint8_t      ch6_u8;      // 上位机类型：uint8
    char         ch7_char;    // 上位机类型：int8 或 uint8
    bool         ch8_bool;    // 上位机类型：uint8
    DebugState_t ch9_state;   // 上位机类型：int32
} DebuggerData_t;

volatile DebuggerData_t g_debuggerData;
```

`volatile` 很重要。它告诉编译器：这个变量可能被外部调试器读取，不要随便优化掉。

## 3. 更新测试数据

可以写一个更新函数：

```c
void DebuggerData_Update(void)
{
    static uint32_t counter = 0;
    counter++;

    g_debuggerData.ch0_i32   = (int32_t)counter;
    g_debuggerData.ch1_u32   = counter * 2U;
    g_debuggerData.ch2_float = counter * 0.1f;
    g_debuggerData.ch3_i16   = (int16_t)(counter % 30000U);
    g_debuggerData.ch4_u16   = (uint16_t)(counter % 60000U);
    g_debuggerData.ch5_i8    = (int8_t)(counter % 120U);
    g_debuggerData.ch6_u8    = (uint8_t)(counter % 255U);
    g_debuggerData.ch7_char  = (char)('A' + (counter % 26U));
    g_debuggerData.ch8_bool  = (counter & 1U) ? true : false;
    g_debuggerData.ch9_state = (counter % 3U) == 0U ? DEBUG_STATE_IDLE :
                              (counter % 3U) == 1U ? DEBUG_STATE_RUN :
                                                     DEBUG_STATE_ERR;
}
```

在 `while (1)` 中调用：

```c
while (1)
{
    DebuggerData_Update();
    HAL_Delay(10);
}
```

如果不用 HAL，也可以放在自己的定时任务里更新。

## 4. 怎么获取变量地址

### 方法一：Keil MDK 查看地址

编译后进入 Debug，打开 Watch 窗口，输入：

```c
&g_debuggerData
&g_debuggerData.ch0_i32
&g_debuggerData.ch1_u32
&g_debuggerData.ch2_float
```

Keil 会显示每个变量的实际地址。

### 方法二：查看 map 文件

编译后打开工程输出目录里的 `.map` 文件，搜索：

```text
g_debuggerData
```

可以找到结构体起始地址。

如果只拿到结构体起始地址，也可以按偏移计算每个成员地址。

## 5. 上位机怎么填写

建议使用“独立地址模式”，每个通道单独填变量地址，最稳。

假设 Keil 查到：

```text
g_debuggerData 地址 = 0x20004410
```

这个结构体在常见 ARM 编译配置下通常偏移如下：

| 通道 | 成员 | 地址填写 | 类型 |
|---|---|---|---|
| 通道0 | ch0_i32 | 0x20004410 + 0 | int32 |
| 通道1 | ch1_u32 | 0x20004410 + 4 | uint32 |
| 通道2 | ch2_float | 0x20004410 + 8 | float |
| 通道3 | ch3_i16 | 0x20004410 + 12 | int16 |
| 通道4 | ch4_u16 | 0x20004410 + 14 | uint16 |
| 通道5 | ch5_i8 | 0x20004410 + 16 | int8 |
| 通道6 | ch6_u8 | 0x20004410 + 17 | uint8 |
| 通道7 | ch7_char | 0x20004410 + 18 | int8 |
| 通道8 | ch8_bool | 0x20004410 + 19 | uint8 |
| 通道9 | ch9_state | 0x20004410 + 20 | int32 |

也就是可以填：

```text
通道0：0x20004410    int32
通道1：0x20004414    uint32
通道2：0x20004418    float
通道3：0x2000441C    int16
通道4：0x2000441E    uint16
通道5：0x20004420    int8
通道6：0x20004421    uint8
通道7：0x20004422    int8
通道8：0x20004423    uint8
通道9：0x20004424    int32
```

注意：不同编译器、不同结构体顺序可能产生不同对齐和填充。最可靠的方法是直接在 Keil Watch 里查看每个成员的地址。

## 6. 连续地址模式怎么用

如果结构体成员是连续排列的，也可以用“连续地址模式”：

```text
起始地址：g_debuggerData 的地址
```

然后按通道顺序选择类型：

```text
通道0 int32
通道1 uint32
通道2 float
通道3 int16
通道4 uint16
通道5 int8
通道6 uint8
通道7 int8
通道8 uint8
通道9 int32
```

但只要结构体中间出现编译器填充字节，连续地址模式就可能读错。混合类型结构体更推荐“独立地址模式”。

## 7. 固定地址不变的做法

如果只是测试，直接使用全局变量 `g_debuggerData` 即可。每次编译后重新查看地址。

如果希望地址长期固定，例如永远固定在：

```text
0x20007000
```

不能只写：

```c
#define DEBUG_DATA_ADDR 0x20007000UL
```

因为这样只是强制往这个地址写，并没有告诉链接器这块 RAM 已经被占用，仍然可能和普通变量、堆、栈冲突。

正式做法是：在 linker/scatter 文件里专门预留一小段 RAM，然后把调试变量放进去。

### Keil ARMClang 示例

C 代码：

```c
__attribute__((section(".debugger_data")))
volatile DebuggerData_t g_debuggerData;
```

Scatter 文件中预留一段 RAM，例如：

```text
RW_IRAM1 0x20000000 0x7000  {
  .ANY (+RW +ZI)
}

RW_DEBUGGER_DATA 0x20007000 0x100  {
  *(.debugger_data)
}
```

这样 `g_debuggerData` 会被放到 `0x20007000` 附近，上位机就可以长期填这个地址。

注意：`0x20007000` 不是所有 STM32 都通用，必须确认你的芯片 RAM 覆盖这个地址。

## 8. 快速检查

连接调试器后，可以先用“内存读取”读结构体起始地址，例如：

```text
读取地址：0x20004410
读取长度：32 字节
```

如果能看到数据不断变化，再切到“图表采集”绑定通道。

如果读取失败，优先检查：

1. 地址是否在芯片 RAM 范围内。
2. 芯片型号 cfg 是否选对，例如 STM32G431 选择 `stm32g4x.cfg`。
3. 变量是否被定义成全局或静态变量。
4. 是否加了 `volatile`。
5. 程序是否正在运行并更新这个变量。
