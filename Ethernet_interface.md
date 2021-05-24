**目录**

<a href="#MII and RMII">MII与RMII</a><br>
<a href="#WIFI Standard">WIFI 标准</a><br>


<h3 id="MII and RMII">MII与RMII</h3>

MII即“媒体独立接口”，也叫“独立于介质的接口”。它是IEEE-802.3定义的以太网行业标准。它包括一个数据接口，以及一个MAC和PHY之间的管理接口。

RMII全称为“简化的媒体独立接口”，是IEEE-802.3u标准中除MII接口之外的另一种实现。 

**MII**

<img src="https://github.com/laneston/Pictures/blob/master/Ethernet_EVB_Hardware_Note/MII.jpg" width="50%" height="50%">

引脚说明如下：

1. MII_TX_CLK：发送数据使用的时钟信号，对于10M位/s的数据传输，此时钟为2.5MHz，对于100M位/s的数据传输，此时钟为25MHz。 
2. MII_RX_CLK：接收数据使用的时钟信号，对于10M位/s的数据传输，此时钟为2.5MHz，对于100M位/s的数据传输，此时钟为25MHz。
3. MII_TX_EN：传输使能信号，此信号必需与数据前导符的起始位同步出现，并在传输完毕前一直保持。
4. MII_TXD[3:0]：发送数据线，每次传输4位数据，数据在MII_TX_EN信号有效时有效。MII_TXD[0]是数据的最低位，MII_TXD[3]是最高位。当MII_TX_EN信号无效时，PHY忽略传输的数据。 
5. MII_CRS：载波侦听信号，仅工作在半双工模式下，由PHY控制，当发送或接收的介质非空闲时，使能此信号。 PHY必需保证MII_CRS信号在发生冲突的整个时间段内都保持有效，不需要此信号与发送/接收的时钟同步。 
6. MII_COL：冲突检测信号，仅工作在半双工模式下，由PHY控制，当检测到介质发生冲突时，使能此信号，并且在整个冲突的持续时间内，保持此信号有效。此信号不需要和发送/接收的时钟同步。 
7. MII_RXD[3:0]：接收数据线，每次接收4位数据，数据在MII_RX_DV信号有效时有效。MII_RXD[0]是数据的最低位，MII_RXD[3]是最高位。当MII_RX_EN无效，而MII_RX_ER有效时，MII_RXD[3:0]数据值代表特定的信息(请参考表194)。
8. MII_RX_DV：接收数据使能信号，由PHY控制，当PHY准备好数据供MAC接收时，使能该信号。此信号必需和帧数据的首位同步出现，并保持有效直到数据传输完成。在传送最后4位数据后的第一个时钟之前，此信号必需变为无效状态。为了正确的接收一个帧，有效电平不能滞后于数据线上的SFD位出现。 
9. MII_RX_ER：接收出错信号，保持一个或多个时钟周期(MII_RX_CLK)的有效状态，表明MAC在接收过程中检测到错误。具体错误原因需配合MII_RX_DV的状态及MII_RXD[3:0]的数据值。 
10. 为了产生TX_CLK和RX_CLK时钟信号，外接的PHY模块必需有来自外部的25MHz时钟驱动。该时钟不需要与MAC时钟相同。可以使用外部的25MHz晶体或者GD32F107xx微控制器的MCO引脚提供这一时钟。当时钟来源MCO引脚时需配置合适的PLL，保证MCO引脚输出的时钟为25MHZ。 

**RMII**

<img src="https://github.com/laneston/Pictures/blob/master/Ethernet_EVB_Hardware_Note/RMII.jpg" width="50%" height="50%">

通过将相同的时钟源接到MAC和以太网PHY的REF_CLK引脚保证两者时钟源的同步。可以通过外部的50MHZ信号或者GD32F107xx微控制器的MCO引脚提供这一时钟。当时钟来源MCO引脚时需配置合适的PLL，保证MCO引脚输出的时钟为50MHZ。

<h3 id="WIFI Standard"> WIFI 标准 </h3>

|标准|发布时间|工作频段|传输最高速率|
|:--:|:-----:|:-----:|:---------:|
|802.11a|1999|5GHz|54Mbps|
|802.11b|1999|2.4GHz|11mbps|
|802.11g|2003|2.4GHZ|54mbps|
|802.11n|2009|2.4G5GHz|300mbps|
|802.11ac|2012|5GHz|1Gbps|

以上各种标准的区别在于使用了不同的扩频方式、复用方式和调制方式。

- 扩频方式主要有DSSS，FHSS，THSS等。
- 复用和多址主要有TDMA，FDMA，CDMA，SDMA，OFDM等。
- 调制方式主要有BPSK，QPSK，QAM等。
- 每个子载波又采用BPSK，QAM，16QAM，64QAM的调制方式，每种调制方式都对应不同的数据速率。


