# 中压配电终端通用平台

## 目录

[TOC]



## 1  系统架构

在统一中压配电终端通用平台上选用HPM6750/6450作为保护板主控芯片，根据层次化设计原则，在系统结构方面将系统分为**应用层**、**软件包**、**标准接口层**、**驱动层**、**硬件层**和**内核层**六层结构，各层次结构如下图所示：

![中压配电终端通用平台系统结构图](Images/中压配电终端通用平台系统结构图.png)

**内核层**是HPM6750/6450的内核，是芯片的架构，和CPU中的命令执行方式和中断处理方式相关，与操作系统内核移植有关。

**硬件层**分为总线和设备，总线包括SPI、IIC、USAR、USB等与通信有关的总线，一般是HPM6750/6450的外设；设备是实现不同功能的硬件，包括HPM6750/6450的部分外设（如定时器、RTC、ADC等）和一些外置芯片（如ADC芯片、看门狗芯片、存储芯片等）。

**驱动层**包括操作系统内核和设备驱动框架，系统内核负责上层任务之间的调度管理并接管芯片的中断管理；驱动框架层定义了与硬件操作相关的软件结构，一方面为标准化接口层实现统一的设备操作接口提供了支持，另一方面提供了同类硬件的标准化操作接口，为驱动硬件的实现提供了模板。

**标准化接口层**提供了一套统一的，标准化的硬件操作接口，所有的硬件不论种类如何，在应用程序层面使用的接口是相同的。该层次与**驱动层**的设备驱动框架提供了一套该平台的标准化设备驱动框架。

**USB和网络协议栈作**为一类特殊的框架的横跨了标准化接口层，因为他提供了一套标准的USB和网络协议的底层实现了上层接口。

**软件包**是为实现一定功能和业务的同类程序的包装，在该系统中，一般包含加解密软件包、GUI软件包、数学算法软件包等。文件系统是一类特殊的软件包，提供了一套对文件操作的包装。

**应用层**包含与业务相关的程序，包括交采、保护、通信、告警、录波、存储等，是FTU/DTU的业务核心。

## 2  操作系统和文件系统的移植

​	内容...	

















## 3  标准化设备驱动框架

标准化设备驱动框架在驱动中引入I/O设备模型框架，硬件和应用程序之间，共分成三层，从上到下分别是 I/O 设备管理层、设备驱动框架层、设备驱动层。驱动框架如下图所示：

![设备驱动框架](Images/设备驱动框架.png)

最终实现平台驱动采用统一接口方案，系统使用人员（应用程序设计人员）拿到系统后，不管是什么样的设备，最终操作的接口是统一的。设计过程中，采用模块化设计思路，按照自顶向下的设计原则，采用整体抽象局部细化的设计方式，先对顶层功能进行抽象建模，再对每一个模块详细设计。

### 3.1  I/O设备管理层

I/O设备管理层是用户实际使用的层次，在该层次，是对所有设备功能的抽象，抽取不同设备的共性，屏蔽不同设备的差异，不论用户访问的是GPIO、串口、SPI还是其他任何设备，都采用统一接口。我们把接口抽象为*open*，*close*，*read*，*write*，*control*，*rx_indicate*，*tx_complete*这7类。（其中*rx_indicate*与*tx_complete*两类抽象的是中断的接收完成和发送完成）

|   序 号   |     接口名称     | 功能描述                                                     |
| :--: | ------------------------ | :----------------------------------------------------------- |
| 1    | device_open(uint hand, ) | 通过设备句柄，应用程序可以打开和关闭设备，打开设备时，会检测设备是否已经初始化，没有初始化则会默认调用初始化接口初始化设备。 |
| 2    | device_close() | 关闭已经打开的设备。 |
| 3    | device_write() | 写数据。 |
| 4    | device_read() | 读数据 |
| 5    | device_control(hand，flag，void* data) | 设备控制。(在control里设置GPIO，里边需要设备列表) |
| 6    | device_set_rx_indicate(uint hand, ) | 设置接收完成回调函数。 |
| 7    | device_set_tx_complete(uint hand, ) | 设置发送完成回调函数。 |

### 3.2  设备驱动框架层

#### 3.2.1  设备分类

##### 3.2.1.1  总线

| 序号  | 总线类型 | 驱动框架    | 数量  | 备注             |
| ----- | -------- | ----------- | ----- | ---------------- |
| 1     | UART     | UART        | 15路  |                  |
| 2     | SPI      | SPI/SPI_DEV | 4     |                  |
| 3     | IIC      | IIC         | 2     |                  |
| 4     | CAN      | CAN         | 1     |                  |
| ==5== | ==RMII== | ==RMII==    | ==2== | ==不用驱动框架== |
| ==6== | ==USB==  | ==USB==     | ==1== | ==不用驱动框架== |
| ==7== | ==FEMC== | ==FEMC==    | ==1== | ==不用驱动框架== |
| 8     | GPIO     | GPIO        |       |                  |
| ==9== | ==SDIO== | ==SDIO==    | ==1== | ==不用驱动框架== |

##### 3.2.1.2  设备

<table> 
	<thead> 
        <tr> 
            <th>序号</th>
            <th>功能</th>
            <th>类型</th>
            <th>型号</th>
            <th>总线</th>
            <th>驱动框架</th>
            <th>备注</th>
        </tr> 
    </thead>
    <tbody> 
        <tr> 
            <td>1</td> 
            <td rowspan="4">存储</td>
            <td>SDRAM</td>
            <td>SCB33S256160AE</td>
            <td>FEMC</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>2</td> 
            <td>EEPROM/铁电</td>
            <td>FM24C128D</td>
            <td>SPI</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>3</td> 
            <td>SD卡</td>
            <td>XTSDG04GWSIGA</td>
            <td>SPI/SD</td>
            <td> </td>
            <td>文件系统</td>
        </tr>
        <tr> 
            <td>4</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>5</td> 
            <td>时钟</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>6</td> 
            <td rowspan="3">交采</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>7</td> 
            <td rowspan="7">传感器</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>8</td> 
            <td>定时器</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>9</td> 
            <td rowspan="2">串口转发</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>10</td> 
            <td>串口转发</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>11</td> 
            <td>遥控</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>12</td> 
            <td>遥信</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>13</td> 
            <td>遥控</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>14</td> 
            <td>遥控</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>15</td> 
            <td>网口</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
        <tr> 
            <td>16</td> 
            <td>加密</td> 
            <td>NOR_flash</td>
            <td>SCB33S256160AE</td>
            <td>XPI接口</td>
            <td> </td>
            <td> </td>
        </tr>
    </tbody>
</table>

#### 3.2.2  驱动框架实现

##### 3.2.2.1  总线驱动框架实现

（1）USART

<table> 
	<thead> 
        <tr> 
            <th>属性组</th> 
            <th colspan="2">配置</th>
        </tr> 
    </thead>
    <thead> 
        <tr> 
            <th>名称</th> 
            <th>类型</th>
            <th>描述</th>
        </tr> 
    </thead> 
    <tbody> 
        <tr> 
            <td>baud_rate</td> 
            <td> </td>
            <td>波特率</td> 
        </tr>
        <tr> 
            <td>data_bits</td> 
            <td> </td>
            <td>数据位</td> 
        </tr>
        <tr> 
            <td>stop_bits</td> 
            <td> </td>
            <td>停止位</td> 
        </tr>
        <tr> 
            <td>parity</td> 
            <td> </td>
            <td>校验位</td> 
        </tr>
        <tr> 
            <td>bit_order</td> 
            <td> </td>
            <td>字节序</td> 
        </tr>
        	<thead>
        		<tr> 
            		<th>属性组</th> 
            		<th colspan="2">缓冲区</th>
        		</tr> 
    		</thead>
        	<tr> 
            <td>serial_rx</td> 
            <td> </td>
            <td>读缓冲区</td> 
        </tr>
        <tr> 
            <td>serial_tx</td> 
            <td> </td>
            <td>写缓冲区</td> 
        </tr>
        <tr> 
            <td>bufsz</td> 
            <td> </td>
            <td>缓冲区长度</td> 
        </tr>
        <thead>
        		<tr> 
            		<th>对下接口</th> 
            		<th colspan="2">描述</th>
        		</tr> 
    	</thead>
        <tr> 
            <td>configure()</td> 
            <td colspan="2">配置串口的配置属性组内容</td> 
        </tr>
        <tr> 
            <td>control()</td> 
            <td colspan="2">控制串口</td> 
        </tr>
        <tr> 
            <td>putc()</td> 
            <td colspan="2">写字节</td> 
        </tr>
        <tr> 
            <td>getc()</td> 
            <td colspan="2">读字节</td> 
        </tr>
        <tr> 
            <td>dma_transmit()</td> 
            <td colspan="2">DMA传输</td> 
        </tr>
    </tbody> 
</table>

（2）SPI

<table> 
	<thead> 
        <tr> 
            <th>属性组</th> 
            <th colspan="2">配置</th>
        </tr> 
    </thead>
    <thead> 
        <tr> 
            <th>名称</th> 
            <th>类型</th>
            <th>描述</th>
        </tr> 
    </thead> 
    <tbody> 
        <tr> 
            <td>mode</td> 
            <td width="100px">SPI_BUS_MODE_SPI
				SPI_BUS_MODE_QSPI
				SPI_MODE_0
				SPI_MODE_1
				SPI_MODE_2
				SPI_MODE_3
				SPI_MODE_MASK
			</td>
            <td>模式</td> 
        </tr>
        <tr> 
            <td>data_width</td> 
            <td> </td>
            <td>数据位宽</td> 
        </tr>
        <tr> 
            <td>reserved</td> 
            <td> </td>
            <td>寄存器地址</td> 
        </tr>
        <tr> 
            <td>max_hz</td> 
            <td> </td>
            <td>最大频率</td> 
        </tr>
        	<thead>
        		<tr> 
            		<th>属性组</th> 
            		<th colspan="2">缓冲区</th>
        		</tr> 
    		</thead>
        	<tr> 
            <td>user_data</td> 
            <td> </td>
            <td>用户数据缓冲区</td> 
        </tr>
        </tr>
        <thead>
        		<tr> 
            		<th>对下接口</th> 
            		<th colspan="2">描述</th>
        		</tr> 
    	</thead>
        <tr> 
            <td>configure</td> 
            <td colspan="2">配置SPI的属性组内容</td> 
        </tr>
        <tr> 
            <td>xfer</td> 
            <td colspan="2">配置SPI的属性组内容</td> 
        </tr>
    </tbody> 
</table>

（3）IIC

<table> 
	<thead> 
        <tr> 
            <th>属性组</th> 
            <th colspan="2">配置</th>
        </tr> 
    </thead>
    <thead> 
        <tr> 
            <th>名称</th> 
            <th>类型</th>
            <th>描述</th>
        </tr> 
    </thead> 
    <tbody> 
        <tr> 
            <td>flags</td> 
            <td width="100px"></td>
            <td>标识</td> 
        </tr>
        <tr> 
            <td>addr</td> 
            <td> </td>
            <td>地址</td> 
        </tr>
        <tr> 
            <td>timeout</td> 
            <td> </td>
            <td>超时时间</td> 
        </tr>
        <tr> 
            <td>retries</td> 
            <td> </td>
            <td>重发间隔/重发次数</td> 
        </tr>
        	<thead>
        		<tr> 
            		<th>属性组</th> 
            		<th colspan="2">缓冲区</th>
        		</tr> 
    		</thead>
        	<tr> 
            <td>user_data</td> 
            <td> </td>
            <td>用户数据缓冲区</td> 
        </tr>
        </tr>
        <thead>
        		<tr> 
            		<th>对下方法</th> 
            		<th colspan="2">描述</th>
        		</tr> 
    	</thead>
        <tr> 
            <td>master_xfer</td> 
            <td colspan="2">主机数据传输</td> 
        </tr>
        <tr> 
            <td>slave_xfer</td> 
            <td colspan="2">从机数据传输</td> 
        </tr>
		<tr> 
            <td>i2c_bus_control</td> 
            <td colspan="2">I2C配置控制</td> 
        </tr>
    </tbody> 
</table>

（4）GPIO

<table> 
	<thead> 
        <tr> 
            <th>对下方法</th> 
            <th>描述</th>
        </tr> 
    </thead>
    <tbody> 
        <tr> 
            <td>pin_mode()</td> 
            <td>设置引脚模式</td> 
        </tr>
        <tr> 
            <td>pin_write()</td> 
            <td>设置引脚电平</td> 
        </tr>
        <tr> 
            <td>pin_read()</td> 
            <td>读取引脚电平</td> 
        </tr>
        <tr> 
            <td>pin_attach_irq()</td> 
            <td>绑定引脚中断回调函数</td> 
        </tr>
        <tr> 
            <td>pin_detach_irq()</td> 
            <td>解绑引脚中断回调函数</td> 
        </tr>
        <tr> 
            <td>pin_irq_enable()</td> 
            <td>使能引脚中断</td> 
        </tr>
    </tbody> 
</table>

## 4  工作计划

