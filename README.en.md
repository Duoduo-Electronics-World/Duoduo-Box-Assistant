# DuoDuo Box Assistant - V 1.2.6

- **If this project is useful to you, please give it a star.**
- **The project will continue to be updated. Engineers are welcome to report bugs and suggestions in Gitee Issues.**
- **The next update is expected around June 2026.**

Update notes:

1. Added TCP Client sessions.
2. Added TCP Server sessions.
3. Added UDP sessions.
4. Fixed known bugs.

## Introduction

DuoDuo Box Assistant is a multi-function embedded debugging tool. It supports serial debugging, J-Link RTT, generic debugger memory reading, TCP/UDP network debugging, real-time charts, common encoding tools, and data export.

Feature overview:

### Real-Time Charts And Data Analysis

- **10-channel real-time chart display**: waveform, histogram, and spectrum views for sensor data, control values, status values, and debug logs.
- **Chart interaction**: zoom, drag, auto-follow, pause drawing, clear data, and box selection.
- **Per-channel configuration**: independent channel name, color, visibility, gain, offset, line width, and point size.
- **Statistics**: real-time maximum, minimum, average value, and standard deviation.
- **Configurable chart buffer**: balance long-term observation with memory usage.

### Chart Parsing And High-Speed Logs

- **Custom chart frame prefix**: serial, J-Link RTT, TCP Client, TCP Server, and UDP sessions share the same chart-frame parser. The default prefix is `ch:`, and short prefixes such as `adc:` or `imu:` are also supported.
- **Simple CSV mode**: disable frame-prefix parsing to parse frames such as `1,2,3\n` directly.
- **Parsing switch**: disable chart parsing for high-speed plain log streams to reduce unnecessary parsing overhead.
- **Error-frame hints**: optional `PARSE 0x...` error codes help locate prefix mismatch, missing newline, invalid channel data, and other frame-format problems.

### Multi-Protocol Device Support

- **Serial debugging**: baud rate, data bits, parity, stop bits, flow control, text/hex send and receive, and selectable text encodings.
- **J-Link RTT**: high-speed embedded logs and chart data through RTT.
- **Generic Debugger**: read target memory through common debuggers such as ST-Link, J-Link, and CMSIS-DAP, then bind variables to chart channels.
- **TCP Client**: connect to a local, LAN, or remote TCP server, with local address/port binding and auto reconnect.
- **TCP Server**: listen on a selected address and port, show client count, accept multiple clients, send to all or a selected client, and disconnect a selected client.
- **UDP**: send to a fixed target or the last sender, suitable for connectionless packets, device discovery, broadcast/unicast testing, and lightweight protocol debugging.

### Independent Multi-Window Sessions

- **Parallel sessions**: open multiple serial, J-Link, debugger, TCP Client, TCP Server, and UDP windows at the same time.
- **Independent state**: each session has its own connection, receive/send area, counters, chart parser, and periodic sender.
- **Independent configuration**: chart parsing, frame prefix, error-frame hints, send cycle, and repeat count are saved by session type to avoid configuration leakage between sessions.

### Internationalization And Encoding

- **Chinese/English UI**: switch languages without restarting the workflow.
- **Multiple encodings**: UTF-8, GBK, ASCII, and other common text encodings.
- **Send/receive helpers**: text, hex, append newline, send history, file sending, and common utility tools.

### Efficient Data Processing

- **Large-stream receive optimization**: receive logs and chart buffers are managed separately to reduce UI pressure during high-speed streams.
- **Periodic sending**: configurable send interval and repeat count for command polling, stress testing, and protocol validation.
- **Data export**: save receive logs and export chart data for later analysis.
- **Clear error separation**: network socket messages are written to the log area, while chart parsing errors remain in the status bar for easier diagnosis.

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

Useful RTT API concepts:

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

## Network Sessions

The software supports three network session types: **TCP Client**, **TCP Server**, and **UDP**. Network sessions work like serial, J-Link, and debugger sessions: each session window has its own connection state, receive area, send area, send history, periodic sending settings, encoding settings, and chart parsing workflow.

Network sessions can be used for normal text/hex communication and for chart data acquisition. If received data follows the chart frame format, for example:

```text
ch:1,2,3\n
```

it can be displayed directly in waveform, histogram, or spectrum charts.

### Network Session Types

| Session Type | Behavior | Typical Use |
| --- | --- | --- |
| TCP Client | Actively connects to a TCP server | Connect to devices, local services, LAN servers, or Ethernet-to-serial modules |
| TCP Server | Listens on a local port and waits for clients | Simulate a server, receive device reports, or allow multiple clients to connect |
| UDP | Sends and receives connectionless packets | Device discovery, broadcast/unicast packets, lightweight protocols, or real-time data where occasional packet loss is acceptable |

TCP is connection-oriented. After a connection is established, both sides can send and receive continuously, and disconnection can be detected. UDP is connectionless. It does not establish a connection before sending, so it has lower overhead, but delivery and ordering are not guaranteed.

### Address And Port Rules

- **`127.0.0.1`**: loopback address. Use it only for testing within the same computer.
- **`0.0.0.0`**: commonly used for local binding. It lets the system choose an interface, or lets a server listen on all local interfaces.
- **LAN IP**: for example `192.168.2.82`. Use it when another computer or device on the same LAN needs to connect to this computer.
- **Port**: valid range is `1~65535`. A server listen port must be specified. A TCP client local port can usually stay `0`, so the system allocates it automatically.
- **Firewall**: if a LAN device cannot connect to this computer, first check whether Windows Firewall blocks the selected port.

Normal TCP/UDP connections to local or LAN devices do not need an HTTP proxy. The software uses regular socket connections.

### TCP Client

TCP Client actively connects to an existing TCP server. It is useful for testing a local service, LAN device, Ethernet-to-serial module, network relay, sensor gateway, or another PC-side TCP server.

Common settings:

- **Local address**: the local network interface address to bind. In most cases, keep `0.0.0.0` and let the system choose the proper interface. Enter a specific local IP only when you must use one network adapter.
- **Local port**: the local client port. In most cases, keep `0` and let the system allocate it automatically.
- **Remote address**: the TCP server IP address. Use `127.0.0.1` for same-PC testing, or a LAN IP such as `192.168.x.x` for a LAN device.
- **Remote port**: the server listen port, such as `8080`.
- **Auto reconnect**: reconnect automatically after disconnection at the configured interval. This is useful for long-running device tests, device reboot tests, and unstable network tests.

Workflow:

1. Confirm that the target TCP server is open and listening.
2. Enter the remote address and remote port in the TCP Client panel.
3. Keep the local address as `0.0.0.0` and the local port as `0` in most cases.
4. Open the TCP Client.
5. After connection succeeds, send text, hex data, or chart frame data from the send area.

For a local test, open a TCP Server session first and listen on `0.0.0.0:8080`, then open a TCP Client session and connect to `127.0.0.1:8080`. To connect to a server on another computer, enter that computer's LAN IP as the remote address.

### TCP Server

TCP Server listens on a selected local address and port, then waits for one or more TCP clients to connect. It is useful for simulating a device server, receiving Ethernet data from a target device, or testing with other network tools.

Common settings:

- **Listen address**: the local address to bind. `0.0.0.0` listens on all local network interfaces. To listen on only one interface, enter that interface IP address.
- **Listen port**: the port opened by the server, such as `8080`.
- **Send target**: send to all clients or to one selected client. This supports one-to-many broadcast-style testing and directed replies.
- **Client list**: shows connected clients as `IP:port`; the selected client can be disconnected manually.

Workflow:

1. Use `0.0.0.0` as the listen address in most cases.
2. Enter a free listen port, such as `8080`.
3. Open the TCP Server. The status will show that it is listening and how many clients are connected.
4. After clients connect, the client list will show each peer as `IP:port`.
5. When sending data, choose all clients or one selected client.
6. To kick a client, select it and disconnect the selected client.

If another PC or device on the LAN needs to connect to this server, use this computer's actual IP address, such as `192.168.2.82`, and the server listen port. If connection fails, check whether the server is listening, both devices are on the same network, and the firewall allows the port.

### UDP

UDP is connectionless. It does not establish a connection like TCP. It is suitable for short packets, broadcast-style data, device discovery, simple protocol tests, and real-time messages where occasional packet loss is acceptable.

Common settings:

- **Local address**: the local network interface to bind. In most cases, use `0.0.0.0`.
- **Local port**: the local port used to receive UDP packets.
- **Remote address/port**: the fixed send target.
- **Send mode**:
  - **Send to fixed target**: always send to the remote address and port shown in the panel.
  - **Reply to last sender**: after receiving a packet, send replies to the address and port of the last sender.

Workflow:

1. Use `0.0.0.0` as the local address in most cases.
2. Enter the local port used to receive UDP packets, such as `8081`.
3. If using **Send to fixed target**, enter the remote address and remote port.
4. If using **Reply to last sender**, receive a packet first so the software knows which address and port to reply to.
5. Open UDP and start sending or receiving.

UDP has no connected state. A successful send only means the packet was handed to the system network stack; it does not guarantee the peer received it. If you need reliable delivery, ordered data, disconnection detection, or multi-client connection management, use TCP first.

### Recommended Tests

**TCP same-PC test**

1. Create a TCP Server session. Listen on `0.0.0.0:8080`.
2. Create a TCP Client session. Connect to `127.0.0.1:8080`.
3. Open the TCP Server first, then open the TCP Client.
4. Send `hello` from the client and confirm the server receives it. Send `world` from the server and confirm the client receives it.
5. Send `ch:1,2,3\n` to verify chart parsing.

**TCP LAN test**

1. On the server computer, run `ipconfig` and find the IPv4 address, such as `192.168.2.82`.
2. Let the TCP Server listen on `0.0.0.0:8080`.
3. On another computer or device, connect the TCP Client to `192.168.2.82:8080`.
4. If connection fails, check whether both devices are on the same network, whether the port is already occupied, and whether the firewall allows the port.

**UDP two-session test**

1. Create UDP session A. Set local port to `8081`, remote address to `127.0.0.1`, and remote port to `8082`.
2. Create UDP session B. Set local port to `8082`, remote address to `127.0.0.1`, and remote port to `8081`.
3. Open both UDP sessions. Data sent by A should be received by B, and data sent by B should be received by A.

### Common Problems

| Symptom | Check First |
| --- | --- |
| TCP Client reports connection refused | Whether the server is open; whether remote IP and port are correct; whether the server listens on the expected interface |
| TCP Client cannot connect | Whether both sides are on the same network; whether the firewall blocks the port; whether `127.0.0.1` was used when the target is actually another device |
| TCP Server fails to listen | Whether the port is occupied; whether the listen address is a real local interface address |
| UDP receives no data | Whether the local port is correct; whether the peer sends to that local port; whether the firewall blocks UDP |
| No chart data appears | Whether the frame matches `ch:1,2,3\n`; whether each frame ends with newline; whether parse/error-frame hints are enabled |

## Charts

### Chart Data Parsing And Custom Frame Prefix

Serial, J-Link RTT, TCP Client, TCP Server, and UDP sessions can parse received text frames into chart data. The Generic Debugger does not use this text-frame parser; it reads target variables directly by memory address and data type.

The related options are in the right-side **Parameter Settings** panel:

- **Parse chart data**: when enabled, received data enters the chart parser and can update waveform, histogram, and spectrum charts. When disabled, received data is shown only as normal logs, which is better for high-speed plain log streams.
- **Use frame prefix**: when enabled, every chart frame must start with a prefix. The default prefix is `ch:`. Keep this enabled when normal logs and chart frames are mixed in the same receive stream.
- **Custom chart frame prefix**: accepts a 1 to 16 character prefix, such as `ch:`, `adc:`, or `imu:`. The prefix must not contain comma, carriage return, or newline characters. For high-speed streams, use a short ASCII prefix.
- **Error code/error-frame hint**: when enabled, parse failures show the latest `PARSE 0x...` error in the status bar `Errors` field, making frame-format problems easier to locate.

#### Prefix Frame Format

The recommended default format is:

```text
prefix + channel1,channel2,channel3 + \n
```

With the default `ch:` prefix:

```text
ch:12.5,18,30\n
ch:13.1,17.8,29.6\n
```

If the custom frame prefix is changed to `adc:`, the firmware or data source must send frames with the same prefix:

```text
adc:12.5,18,30\n
adc:13.1,17.8,29.6\n
```

Each frame supports up to 10 channels. Channels are separated by commas. Each channel value must be a valid number; integers, negative values, and decimals are supported. Every frame must end with `\n`, otherwise the software cannot reliably determine where the frame ends.

#### Simple CSV Format

If **Use frame prefix** is disabled, the software parses simple CSV frames:

```text
channel1,channel2,channel3\n
```

Examples:

```text
12.5,18,30\n
13.1,17.8,29.6\n
```

Simple CSV is suitable when the data source is clean and every received line contains only chart values. If the receive stream also contains normal logs, debug text, or other protocol messages, keep prefix mode enabled to avoid treating normal logs as chart data.

#### High-Speed Log Stream Suggestions

If a session is used only for normal logs and does not need chart updates, disable **Parse chart data**. Then the software will not run numeric chart parsing on every received line, reducing extra processing cost for high-speed log streams.

For high-speed chart acquisition, use a fixed, short, ASCII-only prefix such as `ch:` or `adc:`, and make the firmware always send complete frames:

```text
ch:1,2,3\n
```

Do not split one frame into multiple sends without a newline. When **Parse chart data + Error code/error-frame hint** are enabled, if the buffer waits too long for a newline and the content looks like a chart frame, the software reports a missing-newline parse error to help locate firmware output problems.

#### Configuration Storage

Chart parsing settings are saved in the configuration file, including whether chart parsing is enabled, whether a frame prefix is used, the custom prefix value, and whether parse error hints are enabled. Serial, J-Link RTT, TCP Client, TCP Server, and UDP sessions save their own settings independently.

### Chart Frame Format

The following examples use the default `ch:` prefix. A chart frame supports up to 10 channels:

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

Note: this software currently uses RTT channel 0. Use `SEGGER_RTT_printf(0, "test\n")` to send data.

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
| 0x0004 | Invalid frame prefix | The frame prefix does not match the current chart prefix setting | Wrong data source, protocol mismatch, or firmware prefix differs from the software setting |
| 0x0008 | Empty data after prefix | No channel data after the current chart frame prefix | Wrong frame format |
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
