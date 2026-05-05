# DuoDuo Box Assistant - V 1.2.2

- **If this project is useful to you, please give it a star.**
- **The project will continue to be updated. Engineers are welcome to report bugs and suggestions in Gitee Issues.**
- **The next update is expected around June 2026.**

Update notes:

1. Added the Generic Debugger window. It can read target RAM through a debugger and bind memory values to chart channels.
2. Optimized the data acquisition workflow for waveform charts, histograms, spectrum charts, and related views.
3. Fixed known bugs.

## Introduction

DuoDuo Box Assistant is a multi-function embedded debugging tool. It supports serial debugging, J-Link RTT, generic debugger memory reading, real-time charts, common encoding tools, and data export.

Feature overview:

### Data Analysis And Visualization

- **10-channel real-time waveform display**: dynamic zooming and dragging.
- **Custom buffer management**: configurable chart buffer size.
- **Real-time statistics**: maximum, minimum, average value, and standard deviation.
- **Per-channel gain control**: independent gain and offset adjustment for every channel.

### Multi-Protocol Device Support

- **Serial debugging**: baud rate, data bits, parity, stop bits, flow control, and other parameters.
- **J-Link RTT**: real-time data transmission through SEGGER RTT.
- **Generic Debugger**: supports common debuggers such as ST-Link, J-Link, CMSIS-DAP, and compatible adapters for reading target memory.
- **Memory-to-chart acquisition**: bind RAM variables directly to waveform, histogram, or spectrum channels.
- **Multiple windows**: open multiple serial, J-Link, and debugger windows at the same time.
- **Independent device sessions**: each window works independently.

### Internationalization And Encoding

- **Chinese/English UI**: switch languages in the software.
- **Multiple encodings**: UTF-8, GBK, ASCII, and other common text encodings.
- **Encoding conversion**: receive and send data with selectable encodings.

### Efficient Data Processing

- **Memory management**: automatic cleanup to avoid excessive memory usage.
- **Data export**: TXT, CSV, and other formats.
- **Send history**: keep and quickly reuse sent commands.
- **Periodic sending**: configurable loop sending.

### Main Window

![Main window](%E5%9B%BE%E7%89%87/%E4%B8%BB%E9%A1%B5%E9%9D%A2.png)

## Installation

Start the software:

![输入图片说明](%E5%9B%BE%E7%89%87/%E5%90%AF%E5%8A%A8%E8%BD%AF%E4%BB%B6.gif)

## J-Link RTT Guide

### J-Link Driver

If you use J-Link, install the J-Link driver first. If the driver has already been installed, you do not need to install it again. To avoid compatibility problems, use the driver version recommended by this manual.

![输入图片说明](%E5%9B%BE%E7%89%87/%E5%AE%89%E8%A3%85J-link%E9%A9%B1%E5%8A%A8.gif)

How fast can J-Link RTT transfer data?

![输入图片说明](%E5%9B%BE%E7%89%87/Jlink%E9%80%9F%E5%BA%A6%E5%9B%BE.jpg)  

In this test, RTT can transmit 82 characters in about 1 us. This is fast enough for many embedded logging and waveform use cases.

Test condition: STM32F407 Cortex-M4, 168 MHz clock.

Another RTT speed comparison curve is shown below:

![输入图片说明](%E5%9B%BE%E7%89%87/J-link%E5%92%8C%E5%85%B6%E4%BB%96%E6%8E%A5%E5%8F%A3%E6%AF%94%E8%BE%83%E9%80%9F%E5%BA%A6%E5%9B%BE.png)  

### Port RTT Into Your Project

![Port RTT to Keil project](%E5%9B%BE%E7%89%87/%E7%A7%BB%E6%A4%8D%E5%88%B0%E8%87%AA%E5%B7%B1%E7%9A%84keil%E5%B7%A5%E7%A8%8Bimage.png)
![Add header files](%E5%9B%BE%E7%89%87/%E5%A2%9E%E5%8A%A0%E5%A4%B4%E6%96%87%E4%BB%B6image.png)
![Keil code configuration](%E5%9B%BE%E7%89%87/Keil%E5%B7%A5%E7%A8%8B%E4%BB%A3%E7%A0%81%E9%85%8D%E7%BD%AEimage.png)

### RTT Up/Down Buffer Configuration

Useful SEGGER RTT APIs:

```c
SEGGER_RTT_ConfigUpBuffer(unsigned BufferIndex, const char* sName, void* pBuffer, unsigned BufferSize, unsigned Flags);
SEGGER_RTT_ConfigDownBuffer(unsigned BufferIndex, const char* sName, void* pBuffer, unsigned BufferSize, unsigned Flags);
unsigned SEGGER_RTT_Read(unsigned BufferIndex, void* pBuffer, unsigned BufferSize);
```

These functions are used to create or replace RTT up buffers and down buffers.

Common parameters:

- `BufferIndex`: RTT channel index. `0` is the default channel. `1` and above can be used for custom channels.
- `sName`: channel name.
- `pBuffer`: buffer address.
- `BufferSize`: buffer size.
- `Flags`: buffer mode, such as non-blocking skip mode.

RTT buffers are allocated from MCU RAM. The maximum buffer size depends on the target MCU RAM capacity and the RAM already used by your application.

General suggestions:

| MCU | RAM | Suggested RTT Buffer |
| --- | --- | --- |
| STM32G473 | 128 KB SRAM | 32 KB or 64 KB |
| STM32F407 | 192 KB RAM | 64 KB to 100 KB |
| STM32H743 | 512 KB DTCM RAM | Hundreds of KB if the project allows it |

Start with the amount of debug data you need, then choose a buffer size that leaves enough RAM for the application stack, heap, and global variables.

## Generic Debugger

The Generic Debugger window can read target memory directly and bind the values to chart channels. This is useful for observing variables, structure members, status values, slow trend data, and simple waveform verification.

Supported common debugger categories include:

- ST-Link
- J-Link
- CMSIS-DAP
- FTDI-based debuggers
- Raspberry Pi GPIO debugging methods
- Bus Pirate
- JTAGkey / Amontec
- HLA-style adapters
- Debug adapters from chip vendors or development boards

The debugger reads by **address + data type**. The PC software does not know the variable names inside the MCU. You need to define variables in the firmware first, then obtain their addresses from Keil, a map file, or another debug view.

### STM32 Test Structure Example

You can add the following code to an STM32 project, for example in `main.c`, or split it into `debugger_data.c/.h`.

```c
#include <stdint.h>
#include <stdbool.h>

typedef enum
{
    DEBUGGER_STATE_IDLE = 0,
    DEBUGGER_STATE_RUN  = 1,
    DEBUGGER_STATE_ERR  = 2
} DebuggerState_t;

typedef struct
{
    int32_t       ch0_i32;     // PC type: int32
    uint32_t      ch1_u32;     // PC type: uint32
    float         ch2_float;   // PC type: float
    int16_t       ch3_i16;     // PC type: int16
    uint16_t      ch4_u16;     // PC type: uint16
    int8_t        ch5_i8;      // PC type: int8
    uint8_t       ch6_u8;      // PC type: uint8
    char          ch7_char;    // PC type: int8 or uint8
    bool          ch8_bool;    // PC type: uint8
    DebuggerState_t ch9_state; // PC type: int32
} DebuggerData_t;

volatile DebuggerData_t g_debuggerData;
```

Keep `volatile`. It tells the compiler that the variable may be accessed externally and should not be optimized away casually.

### Update Test Data

Use a simple update function so each field changes continuously. This makes it easy to check whether the PC-side acquisition is working.

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
    g_debuggerData.ch9_state = (counter % 3U) == 0U ? DEBUGGER_STATE_IDLE :
                              (counter % 3U) == 1U ? DEBUGGER_STATE_RUN :
                                                     DEBUGGER_STATE_ERR;
}
```

Call it from the main loop, a timer task, or an RTOS task:

```c
while (1)
{
    DebuggerData_Update();
    HAL_Delay(10);
}
```

### Get Variable Addresses

Method 1: Use Keil MDK Watch.

After compilation, enter debug mode and add these expressions to Watch:

```c
&g_debuggerData
&g_debuggerData.ch0_i32
&g_debuggerData.ch1_u32
&g_debuggerData.ch2_float
```

Keil will show the actual addresses of the structure and its members.

![输入图片说明](%E5%9B%BE%E7%89%87/STM32%20%E8%8E%B7%E5%8F%96%E9%80%9A%E9%81%93%E6%95%B0%E6%8D%AE%E7%9A%84%E5%9C%B0%E5%9D%80.gif)

Method 2: Check the map file.

After compilation, open the `.map` file in the output directory and search for:

```text
g_debuggerData
```

You can find the start address of the structure. If you only know the structure start address, calculate member addresses by offset, or check each member directly in Keil Watch.

### Independent Address Mode

Independent address mode is recommended for structures that contain mixed data types. Each channel uses its own address and type. This is the most reliable and easiest mode to debug.

Assume:

```text
g_debuggerData address = 0x20004410
```

With a common ARM compiler layout, the members may be arranged like this:

| Channel | Member | Address | Type |
| --- | --- | --- | --- |
| Channel 0 | ch0_i32 | 0x20004410 | int32 |
| Channel 1 | ch1_u32 | 0x20004414 | uint32 |
| Channel 2 | ch2_float | 0x20004418 | float |
| Channel 3 | ch3_i16 | 0x2000441C | int16 |
| Channel 4 | ch4_u16 | 0x2000441E | uint16 |
| Channel 5 | ch5_i8 | 0x20004420 | int8 |
| Channel 6 | ch6_u8 | 0x20004421 | uint8 |
| Channel 7 | ch7_char | 0x20004422 | int8 |
| Channel 8 | ch8_bool | 0x20004423 | uint8 |
| Channel 9 | ch9_state | 0x20004424 | int32 |

Different compilers, structure order, and alignment settings may add padding bytes. The most reliable method is to check each member address in Keil Watch.

### Continuous Address Mode

If structure members are laid out continuously, you can use continuous address mode:

```text
Start address: address of g_debuggerData
```

Then select the channel types in order:

```text
Channel 0: int32
Channel 1: uint32
Channel 2: float
Channel 3: int16
Channel 4: uint16
Channel 5: int8
Channel 6: uint8
Channel 7: int8
Channel 8: uint8
Channel 9: int32
```

If padding bytes exist inside the structure, continuous address mode may read the wrong values. For mixed-type structures, prefer independent address mode.

![输入图片说明](%E5%9B%BE%E7%89%87/%E8%B0%83%E5%BC%8F%E5%99%A8%E7%BB%91%E5%AE%9A%E6%95%B0%E6%8D%AE%E9%80%9A%E9%81%93%E6%98%BE%E7%A4%BA%E5%9B%BE%E8%A1%A8%E6%95%B0%E6%8D%AE.gif)


### Keep The Address Fixed

For quick tests, a global variable such as `g_debuggerData` is enough. Check its address again after each build.

If you want the address to stay fixed for a long time, for example:

```text
0x20007000
```

Do not only write:

```c
#define DEBUGGER_DATA_ADDR 0x20007000UL
```

That only forces the program to write to that address. It does not tell the linker that the RAM area is occupied, so it may conflict with normal variables, heap, or stack.

A more reliable method is to reserve a small RAM region in the linker/scatter file and place the debugger variable there.

Keil ARMClang example:

```c
__attribute__((section(".debugger_data")))
volatile DebuggerData_t g_debuggerData;
```

Scatter file example:

```text
RW_IRAM1 0x20000000 0x7000  {
  .ANY (+RW +ZI)
}

RW_DEBUGGER_DATA 0x20007000 0x100  {
  *(.debugger_data)
}
```

Then `g_debuggerData` will be placed near `0x20007000`, and the PC software can keep using that address.

Note: `0x20007000` is not universal for all STM32 chips. Always confirm that the target RAM actually covers this address.

### Quick Check

After connecting the debugger, first use Memory Read to read the structure start address, for example:

```text
Read address: 0x20004410
Read length: 32 bytes
```

If the data changes continuously, switch to Chart Acquisition and bind the channels.

If reading fails, check:

1. Whether the address is inside the target RAM range.
2. Whether the chip series is correct, for example STM32G431 should use `stm32g4x.cfg`.
3. Whether the variable is global or static.
4. Whether `volatile` is used.
5. Whether the firmware is running and updating the variable.

### Generic Debugger Vs J-Link RTT

Both Generic Debugger and J-Link RTT can send MCU data to the PC-side charts, but their working methods and suitable scenarios are different.

**Generic Debugger** actively reads target memory by address. You only need to know the variable address and type, then bind RAM variables to chart channels. It is suitable for observing global variables, structure members, status values, slow trends, and cases where you do not want to add communication code to the firmware.

With Generic Debugger, the MCU does not actively send data. The PC reads specified addresses at the configured period. Because every read goes through the debug interface, it is not ideal as a replacement for high-speed continuous data streams. For high-speed ADC waveforms, lower the sampling expectation, buffer data on the MCU side, or read processed/downsampled data.

**J-Link RTT** uses an RTT buffer in the MCU firmware. The MCU actively writes data to the RTT buffer, and the PC reads the RTT buffer. It is better for real-time logs, high-speed continuous data, `printf` debugging, and complete sampled data frames. If an ADC samples at a high rate and sends frames to the PC, RTT is usually more suitable than polling addresses through the Generic Debugger.

Simple selection guide:

| Requirement | Recommended Method |
| --- | --- |
| View a variable, structure member, or status value | Generic Debugger |
| Bind 1 to 10 RAM variables to chart channels | Generic Debugger |
| Avoid adding sending code to firmware | Generic Debugger |
| High-speed continuous log or waveform output | J-Link RTT |
| `printf`-style debug output | J-Link RTT |
| MCU actively outputs complete frames at a fixed sample rate | J-Link RTT |

If you only want to quickly inspect variable changes, use Generic Debugger. If you need a high-speed real-time data stream, prefer J-Link RTT.

## Charts

### Chart Frame Format

The waveform chart supports up to 10 channels. The recommended frame format is:

```text
ch:1,2,3,4,5,6,7,8,9,10\n
```

Each number corresponds to one channel:

- Number 1: channel 1 data.
- Number 2: channel 2 data.
- Number 3: channel 3 data.
- Number 4: channel 4 data.
- Number 5: channel 5 data.
- Number 6: channel 6 data.
- Number 7: channel 7 data.
- Number 8: channel 8 data.
- Number 9: channel 9 data.
- Number 10: channel 10 data.

Every frame must end with `\n`, otherwise the chart may not parse the data.

Example: show one channel:

```text
ch:65\n
ch:71\n
ch:80\n
```

```c
int buffer[3] = {65, 71, 80};
SEGGER_RTT_printf(0, "ch:%d\n", buffer[0]);
SEGGER_RTT_printf(0, "ch:%d\n", buffer[1]);
SEGGER_RTT_printf(0, "ch:%d\n", buffer[2]);
```

Example: show channel 1 and channel 2:

```text
ch:65,30\n
ch:60,40\n
ch:80,50\n
```

```c
int buffer1[3] = {65, 60, 80};
int buffer2[3] = {30, 40, 50};
SEGGER_RTT_printf(0, "ch:%d,%d\n", buffer1[0], buffer2[0]);
SEGGER_RTT_printf(0, "ch:%d,%d\n", buffer1[1], buffer2[1]);
SEGGER_RTT_printf(0, "ch:%d,%d\n", buffer1[2], buffer2[2]);
```

Note: RTT currently uses channel 0 for this software. Use `SEGGER_RTT_printf(0, "test\n")` to send data.

### Waveform Chart

The waveform chart displays changing data over time. It supports up to 10 channels, per-channel visibility, color, line width, scatter display, gain, and offset. It also calculates key statistics in real time.

![Waveform chart](%E5%9B%BE%E7%89%87/%E6%B3%A2%E5%BD%A2%E5%9B%BE%E5%8A%A8%E5%9B%BE.gif)

#### Measurement Cursor

Double-click a data point to use the measurement cursor. One measurement group can select up to 3 points, and up to 10 measurement groups can be created. Right-click to cancel measurement labels.

![Measurement cursor](%E5%9B%BE%E7%89%87/%E6%B5%8B%E9%87%8F%E5%8D%A1%E5%B0%BA.png)

#### Box Selection

Use the box selection feature to select a data range and calculate statistics for that range.

![Box selection](%E5%9B%BE%E7%89%87/%E6%95%B0%E6%8D%AE%E6%A1%86%E9%80%89.gif)

### Histogram

The histogram shows data distribution. It divides the value range into continuous bins and counts how many samples fall into each bin. It is useful for observing probability distribution, concentration, dispersion, and stability.

Use the right-side channel panel to control color, bin width, and visibility. The software calculates maximum, minimum, average value, and standard deviation for each channel.

Suggested usage:

- Observe the overall shape of the histogram.
- A bell-like shape may indicate an approximately normal distribution.
- Adjust the X-axis range and bin width to focus on a specific value range.
- Use average value and standard deviation to evaluate stability and consistency.

#### Core Logic

Example configuration:

```text
Bin width: 5
X-axis lower limit: 0
X-axis upper limit: 1000
Total bins: 200
```

Input frame:

```text
ch:10,32,18
```

Calculation:

- `10 / 5 = 2`: bin 2 count + 1.
- `32 / 5 = 6.4`: bin 6 count + 1.
- `18 / 5 = 3.6`: bin 3 count + 1.

If the data buffer is full, the earliest frame is removed and the corresponding bin counts are decreased.

![Histogram](%E5%9B%BE%E7%89%87/%E7%9B%B4%E6%96%B9%E5%9B%BE%E5%8A%A8%E5%9B%BE.gif)

### Spectrum Chart

The spectrum chart converts a time-domain signal into the frequency domain for analysis. A waveform chart shows how a signal changes over time. A spectrum chart shows which frequency components are inside the signal and how strong they are.

Common use cases:

1. View the main frequency of a signal.
2. Analyze periodic signal components.
3. Observe high-frequency noise and spurious components.
4. Compare frequency distribution across channels.
5. Check mechanical vibration, motor speed, and periodic sensor output.
6. Locate periodic interference in serial or J-Link acquired data.

Sampling interval matters. The software uses the configured per-frame time interval `dt` to calculate the sample rate:

```text
sample rate (Hz) = 1 / dt(s)
```

Examples:

- If the signal sample rate is 1000 Hz, then `dt = 1 ms`.
- If the signal sample rate is 20 kHz, then `dt = 0.05 ms`.

If `dt` is wrong, the frequency axis and peak positions will also be wrong.

![Spectrum chart](%E5%9B%BE%E7%89%87/%E9%A2%91%E8%B0%B1%E5%9B%BE.gif)

### Move Chart Out Of The Main Window

Hold the left mouse button on a chart tab and drag it out to create a floating chart window.

![Move chart out](%E5%9B%BE%E7%89%87/%E5%9B%BE%E8%A1%A8%E7%A7%BB%E5%8A%A8%E5%87%BA%E6%82%AC%E6%B5%AE%E7%AA%97%E5%8F%A3%E6%93%8D%E4%BD%9C.gif)

## Tools

### CRC Calculator

![输入图片说明](%E5%9B%BE%E7%89%87/CRC%E8%AE%A1%E7%AE%97%E5%99%A8.gif)

### ASCII Table

![输入图片说明](%E5%9B%BE%E7%89%87/ASCLL%E7%A0%81.gif)

### RGB Color Converter

![输入图片说明](%E5%9B%BE%E7%89%87/RGB%E9%A2%9C%E8%89%B2%E7%A0%81%E4%BA%92%E6%8D%A2.gif)

### Base Converter

![输入图片说明](%E5%9B%BE%E7%89%87/%E8%BF%9B%E5%88%B6%E8%AE%A1%E7%AE%97%E5%99%A8.gif)

## Errors

### PARSE 0x0000 Data Parse Error

`PARSE` means data parsing error. `0x0000` is a hexadecimal error code used to identify the specific error type.

| Error Code | Name | Description | Common Cause |
| --- | --- | --- | --- |
| 0x0001 | Data too short | The received data length is not enough | Data was truncated or transmission was incomplete |
| 0x0002 | Missing newline | The data frame does not end with `\n` | Wrong frame format |
| 0x0004 | Invalid prefix | The prefix is not `ch:` | Wrong data source or protocol mismatch |
| 0x0008 | Empty data after prefix | No data after `ch:` | Wrong frame format |
| 0x0010 | Data too long | Data part is longer than 200 characters | Abnormal data or buffer overflow |
| 0x0020 | Empty channel data | A channel value is empty | Wrong frame format |
| 0x0040 | Invalid number format | Channel data is not a valid floating-point number | Non-numeric characters |
| 0x0080 | Conversion failed | String-to-number conversion failed | Abnormal data format |
| 0x0100 | Value out of range | Numeric value is outside the allowed range | Sensor abnormality or bad data |

### SEND 0x0000 Send Error

`SEND` means send-data error. `0x0000` is a hexadecimal error code.

| Error Code | Name | Description | Common Cause |
| --- | --- | --- | --- |
| 0x0001 | Send data error | When using J-Link to send data, the target may not read data in time. The status bar shows the successfully sent byte count. | The PC sends too fast, or the target does not process the received data in time |

### Error Code Calculation

Single error code:

```text
0x0001 = data too short
0x0004 = invalid prefix
0x0040 = invalid number format
```

Combined error code:

When multiple errors occur at the same time, the displayed code is the sum of all related error codes.

Examples:

```text
0x0001 + 0x0002 = 0x0003
0x0004 + 0x0040 = 0x0044
0x0001 + 0x0002 + 0x0040 = 0x0043
```

## Tips

### Software Operation

![Manual connection](%E5%9B%BE%E7%89%87/%E6%89%8B%E5%8A%A8%E8%BF%9E%E6%8E%A5image.png)
![Button zoom](%E5%9B%BE%E7%89%87/%E7%82%B9%E5%87%BB%E6%8C%89%E9%92%AE%E7%BC%A9%E6%94%BEimage.png)
![Waveform only](%E5%9B%BE%E7%89%87/%E5%8F%AA%E6%98%BE%E7%A4%BA%E6%B3%A2%E5%BD%A2%E5%9B%BEimage.png)

### Data Save And Processing

![Save data](%E5%9B%BE%E7%89%87/%E6%95%B0%E6%8D%AE%E4%BF%9D%E5%AD%98.gif)

Use WPS or other spreadsheet tools to split invalid characters and save pure numeric data.

![WPS data split](%E5%9B%BE%E7%89%87/WPS%E6%95%B0%E6%8D%AE%E5%88%86%E5%89%B2%E4%BF%9D%E5%AD%98.gif)

## Contribution

**DuoDuo Box Studio**

## Get The Latest Version

Website: https://gitee.com/momingchuan/duo-duo-box
