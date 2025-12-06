# 多多盒子助手 -V 1.0.7  



- **如果这个项目对你有帮助，请给我们一个 ⭐ 星标！**  
- **短期内会继续更新请期待，欢迎广大工程师积极留言反馈BUG，任何问题都可在Issues留言反馈**  
- **预计下次更新在2026年2月左右**  

$\color{#0000FF}{
    更新预告：  
}$  

$\color{#0000FF}{
1、增加直方图表，可以显示那个数据出现的概率大和概率小的数据 
}$  

$\color{#0000FF}{
2、增加Jlink 一键擦除单片机内存，一般可以使用在单片机被锁定时，要强制解锁。
}$  

$\color{#0000FF}{
3、增加图表数据解析错误码提示状态
}$  

$\color{#0000FF}{
4、增加自动保存接收数据功能
}$  


## 介绍  

🌟 特性概览

 📊 **智能数据分析与可视化**
- **10通道实时波形显示** - 支持动态缩放和拖拽操作
- **自定义缓冲管理** - 可配置数据缓冲区大小，智能内存优化
- **实时统计分析** - 最大值、最小值、平均值计算显示
- **曲线增益调节** - 各通道独立增益和偏置调整

 🔌 **多协议多设备支持**
- **串口调试** - 完整串口参数配置（波特率、数据位、校验位等）
- **J-Link集成** - 内置SEGGER J-Link，支持RTT实时数据传输
- **多窗口并行** - 同时打开多个串口和J-Link设备窗口
- **设备独立管理** - 每个窗口独立运行，互不干扰

 🌍 **国际化与编码支持**
- **多语言界面** - 完整中英文界面切换
- **多编码解析** - UTF-8、GBK、ASCII等多种字符编码支持
- **智能编码识别** - 自动检测和转换不同编码格式

 ⚡ **高效数据处理**
- **智能内存管理** - 自动数据清理防止内存溢出
- **数据导出** - 支持TXT、CSV等多种格式导出
- **历史记录** - 完整的发送历史记录和快速重发
- **周期发送** - 可配置自动循环发送

### 界面  
![输入图片说明](%E5%9B%BE%E7%89%87/image.png)


## 安装教程
**1、打开文件夹：**  
![输入图片说明](%E5%9B%BE%E7%89%87/%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%E5%A4%B9image.png)  

**2、启动软件：**  
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%90%AF%E5%8A%A8%E8%BD%AF%E4%BB%B6image.png)  


## 使用说明  

#### J-link使用说明  

如果使用j-link 则安装J-link驱动，如果已经安装过J-link驱动了则不用安装，但是为了避免可能 $\color{#FF9933}{出现未知的错误问题}$ ，还是 $\color{#FF0000}{强烈}$ 建议安装此版本的J-link驱动程序。  

![输入图片说明](%E5%9B%BE%E7%89%87/%E5%AE%89%E8%A3%85J-link%E9%A9%B1%E5%8A%A811zon_created-GIF%20(4).gif)

![输入图片说明](%E5%9B%BE%E7%89%87/%E7%A7%BB%E6%A4%8D%E5%88%B0%E8%87%AA%E5%B7%B1%E7%9A%84keil%E5%B7%A5%E7%A8%8Bimage.png)  
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%A2%9E%E5%8A%A0%E5%A4%B4%E6%96%87%E4%BB%B6image.png)  
![输入图片说明](%E5%9B%BE%E7%89%87/Keil%E5%B7%A5%E7%A8%8B%E4%BB%A3%E7%A0%81%E9%85%8D%E7%BD%AEimage.png)  

#### 上行/下行缓冲区配置
> Note:  
>  如何手动配置RTT上行/下行缓冲区？  
>  **核心函数：**  
```
SEGGER_RTT_ConfigUpBuffer(unsigned BufferIndex, const char* sName, void* pBuffer, unsigned BufferSize, unsigned Flags)  
SEGGER_RTT_ConfigDownBuffer(unsigned BufferIndex, const char* sName, void* pBuffer, unsigned BufferSize, unsigned Flags)  
```
>  用途：用于创建自定义的上行（MCU到调试器）或下行（调试器到MCU）通道，替换默认的0号通道。  
>  **配置步骤：**  
>  定义缓冲区：声明静态字符数组作为数据缓冲区。  
>  调用配置函数：在初始化阶段配置缓冲区。  
>  **指定通道参数：**  
>  BufferIndex：通道索引（0为默认通道，1及以上为用户自定义通道）。  
>  sName：通道标识名。  
>  pBuffer：缓冲区地址。  
>  BufferSize：缓冲区大小。  
>  Mode：缓冲模式（如SEGGER_RTT_MODE_NO_BLOCK_SKIP非阻塞跳过模式）。  
>  **RTT缓冲区大小的根本限制：**  
>  **$\color{#FF0000}{RTT缓冲区直接占用MCU的RAM，调试器只是读取该内存区域。}$**  
>  **$\color{#FF0000}{因此，缓冲区最大尺寸的硬性上限是您愿意且能够从MCU总RAM中划拨出的空间。}$**  
>  MCU型号总RAM容量  
>  建议的RTT缓冲区大小  
>  STM32G473​ 128 KB SRAM  
>  可轻松配置 32KB 或 64KB  
>  STM32F407​ 192 KB RAM  
>  通常可用 64KB ~ 100KB  
>  STM32H743​ 512 KB DTCM RAM  
>  可配置高达几百KB  
>  **总结与使用建议**  
>  配置流程：先根据您的调试输出数据量需求。  
>  尺寸规划：在规划缓冲区大小时，必须首要考虑硬件限制，即根据您的MCU型号和应用程序已占用的RAM，评估剩余可用空间，并据此设置一个合理的缓冲区大小，避免导致系统内存不足。  



#### 波形图帧格式    
显示波形图发送数据格式,最大显示10个通道数据。  
"ch:1,2,3,4,5,6,7,8,9,10\n"  
数字1表示通道1的数据  
数字2表示通道2的数据  
数字3表示通道3的数据  
数字4表示通道4的数据  
数字5表示通道5的数据  
数字6表示通道6的数据  
数字7表示通道7的数据  
数字8表示通道8的数据  
数字9表示通道9的数据  
数字10表示通道10的数据  
一帧数据后面一定要加换行符‌，否则图标显示不出来。  
例如显示通道1波形图：  
"ch:65\n"  
"ch:71\n"  
"ch:80\n"  

```
int buffer[3]={65,71,80};
SEGGER_RTT_printf(0, "ch:%d\n",buffer[0]);  
SEGGER_RTT_printf(0, "ch:%d\n",buffer[1]);  
SEGGER_RTT_printf(0, "ch:%d\n",buffer[2]);  
```


例如显示通道1和通道2波形图：  
"ch:65,30\n"  
"ch:60,40\n"  
"ch:80,50\n"  


```
int buffer1[3]={65,60,80};
int buffer2[3]={30,40,50};
SEGGER_RTT_printf(0, "ch:%d,%d\n",buffer1[0],buffer2[0]);  
SEGGER_RTT_printf(0, "ch:%d,%d\n",buffer1[1],buffer2[1]);  
SEGGER_RTT_printf(0, "ch:%d,%d\n",buffer1[2],buffer2[2]);  
```
> Note:
> 1. **RTT目前只支持0通道，不支持其他通道**。这意味着你只能调用SEGGER_RTT_printf(0, "test\n")发送数据，而不能调用SEGGER_RTT_printf(1,"test\n")去发送数据。  

## 使用技巧  
![输入图片说明](%E5%9B%BE%E7%89%87/%E6%89%8B%E5%8A%A8%E8%BF%9E%E6%8E%A5image.png)
![输入图片说明](%E5%9B%BE%E7%89%87/%E7%82%B9%E5%87%BB%E6%8C%89%E9%92%AE%E7%BC%A9%E6%94%BEimage.png)
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%8F%AA%E6%98%BE%E7%A4%BA%E6%B3%A2%E5%BD%A2%E5%9B%BEimage.png)
## 参与贡献

**@多多盒子工作室**


## 获取最新版本
网址：https://gitee.com/momingchuan/duo-duo-box  


