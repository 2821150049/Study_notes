# 第14讲、IIC协议及EEPROM

## 一、存储器

### 1.常用存储器

<img src="img\1.png" alt="1" style="zoom:67%;" />

1. 易失性存储器

```
RAM：
SRAM-静态随机存储器（Static Random Access Memory）
DRAM-动态随机存储器（Dynamic Random Access Memory ）
所谓“随机存取”，指的是当存储器中的消息被读取或写入时，所需要的时间与这段信息所在的位置无关。
```

<img src="img\2.png" alt="2" style="zoom:67%;" />

<img src="img\3.png" alt="3" style="zoom:67%;" />

2. 非易失性存储器

```
ROM-只读存储器（ Read Only Memory ）：
MASK ROM
	OTP(One Time Programmable) ROM
	EPROM(Erasable Programmable)
	EEPROM (Electrically Erasable Programmable) 
FLASH-闪存：
	Nor Flash
	Nand Flash
磁盘、光盘等
```

## 二、IIC协议

I2C 通讯协议(Inter－Integrated Circuit)是由Phiilps公司开发的，由于它引脚少，硬件实现简单，可扩展性强，不需要USART、CAN等通讯协议的外部收发设备，现在被广泛地使用在系统内多个集成电路(IC)间的通讯。

### 1.物理特点

<img src="img\4.png" alt="4" style="zoom:67%;" />

```
它是一个支持多设备的总线。“总线”指多个设备共用的信号线。在一个I2C通讯总线中，可连接多个I2C通讯设备，支持多个通讯主机及多个通讯从机。
一个I2C总线只使用两条总线线路，一条双向串行数据线(SDA)，一条串行时钟线 (SCL)。数据线即用来表示数据，时钟线用于数据收发同步。
每个连接到总线的设备都有一个独立的地址，主机可以利用这个地址进行不同设备之间的访问。
总线通过上拉电阻接到电源。当I2C设备空闲时，会输出高阻态，而当所有设备都空闲，都输出高阻态时，由上拉电阻把总线拉成高电平。
多个主机同时使用总线时，为了防止数据冲突，会利用仲裁方式决定由哪个设备占用总线。
具有三种传输模式：标准模式传输速率为100kbit/s ，快速模式为400kbit/s ，高速模式下可达3.4Mbit/s，但目前大多I2C设备尚不支持高速模式。
连接到相同总线的IC数量受到总线的最大电容 400pF限制。
```

### 2.协议层

```
I2C的协议定义了通讯的起始和停止信号、数据有效性、响应、仲裁、时钟同步和地址广播等环节。
```

1. I2C基本读写过程

   1. 写数据

   <img src="img\5.png" alt="5"  />

   2. 读数据

   ![6](img\6.png)

2. 起始/停止信号（由主机发送）

<img src="img\7.png" alt="7" style="zoom:67%;" />

```C
// 当 SCL 线是高电平时 SDA 线从高电平向低电平切换，这个情况表示通讯的起始。
// 当 SCL 是高电平时 SDA 线由低电平向高电平切换，表示通讯的停止。
// 起始和停止信号一般由主机产生。
```

3. 数据有效性

<img src="img\8.png" alt="8" style="zoom:80%;" />

```c
// I2C使用SDA信号线来传输数据，使用SCL信号线进行数据同步。 SDA数据线在SCL的每个时钟周期传输一位数据。
// SCL为高电平的时候SDA表示的数据有效，即此时的SDA为高电平时表示数据“1”，为低电平时表示数据“0”。
// 当SCL为低电平时，SDA的数据无效，一般在这个时候SDA进行电平切换，为下一次表示数据做好准备。
```

4. 地址及数据方向

<img src="img\9.png" alt="9" style="zoom:67%;" />

```c
// I2C总线上的每个设备都有自己的独立地址，主机发起通讯时，通过SDA信号线发送设备地址(SLAVE_ADDRESS)来查找从机。设备地址可以是7位或10位。
// 紧跟设备地址的一个数据位R/W用来表示数据传输方向，数据方向位为“1”时表示主机由从机读数据，该位为“0”时表示主机向从机写数据。
```

5. 响应位`ACK`

<img src="img\10.png" alt="10" style="zoom:67%;" />

```c
//传输时主机产生时钟，在第9个时钟时，数据发送端会释放SDA的控制权，由数据接收端控制SDA，若SDA为高电平，表示非应答信号(NACK)，低电平表示应答信号(ACK)。
```

### 3.单片机硬件IIC介绍

1. 模式选择

```
1. 该接口在工作时可选用以下四种模式之一：
    从发送器
    从接收器
    主发送器
    主接收器
2. 默认情况下，它以从模式工作。接口在生成起始位后会自动由从模式切换为主模式，并在出现仲裁丢失或生成停止位时从主模式切换为从模式，从而实现多主模式功能。
3. 7 位/10 位寻址以及广播呼叫的生成和检测
4. 支持不同的通信速度：
     标准速度（高达 100 kHz）
     快速速度（高达 400 kHz）
5. 状态标志：
	发送/接收模式标志
	字节传输结束标志
	I2C 忙碌标志
6. 错误标志：
	主模式下的仲裁丢失情况
	地址/数据传输完成后的应答失败
	检测误放的起始位和停止位
	禁止时钟延长后出现的上溢/下溢
```



### 4.CubeMX配置硬件

![11](img\11.png)

### 5.细节补充

**为啥在发送起始信号后两根电平位都要拉低？**

```
I2C协议规定，CLK为高时，拉低SDA表示起始位，数据只允许在SCK为底时变化，CLK为高时，数据锁存；此时拉低CLK是开始写数据。
```

















