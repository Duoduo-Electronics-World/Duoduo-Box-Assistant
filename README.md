# 多多盒子

#### 介绍

![输入图片说明](%E5%9B%BE%E7%89%87/J%20link%20RTT%E4%B8%BB%E9%A1%B5%E9%9D%A2image.png)


#### 软件架构
软件架构说明



#### 安装教程
1、打开文件夹：  
![输入图片说明](%E5%9B%BE%E7%89%87/%E6%89%93%E5%BC%80%E6%96%87%E4%BB%B6%E5%A4%B9image.png)

2、启动软件：  
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%90%AF%E5%8A%A8%E8%BD%AF%E4%BB%B6image.png)


#### 使用说明

#### J-link使用说明

如果使用j-link 则安装J-link驱动  
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%AE%89%E8%A3%85j-link%E9%A9%B1%E5%8A%A8image.png)  
![输入图片说明](%E5%9B%BE%E7%89%87/%E6%89%A7%E8%A1%8C%E5%AE%89%E8%A3%85j-link%E9%A9%B1%E5%8A%A8image.png)  
![输入图片说明](%E5%9B%BE%E7%89%87/%E7%A7%BB%E6%A4%8DRTT%E6%BA%90%E6%96%87%E4%BB%B6image.png)  
![输入图片说明](%E5%9B%BE%E7%89%87/keil%20%E5%B7%A5%E7%A8%8B%E4%BD%BF%E7%94%A8RTT%20%E6%89%93%E5%8D%B0%E8%BE%93%E5%87%BA%E4%BF%A1%E6%81%AFimage.png)
![输入图片说明](%E5%9B%BE%E7%89%87/%E5%A2%9E%E5%8A%A0%E5%A4%B4%E6%96%87%E4%BB%B6image.png)  

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
"ch:70\n"  
"ch:70\n"  

```
SEGGER_RTT_WriteString(0, "ch:65\n");  
SEGGER_RTT_WriteString(0, "ch:70\n");  
SEGGER_RTT_WriteString(0, "ch:80\n");
```


例如显示通道1和通道2波形图：  
"ch:65,30\n"  
"ch:60,40\n"  
"ch:80,50\n"  


```
SEGGER_RTT_WriteString(0, "ch:65,30\n");  
SEGGER_RTT_WriteString(0, "ch:60,40\n");  
SEGGER_RTT_WriteString(0, "ch:80,50\n");
```


#### 使用技巧  
![输入图片说明](%E5%9B%BE%E7%89%87/%E4%BD%BF%E7%94%A8%E6%8A%80%E5%B7%A7image.png)  

#### 参与贡献




#### 获取最新版本
网址：https://gitee.com/momingchuan/duo-duo-box  


