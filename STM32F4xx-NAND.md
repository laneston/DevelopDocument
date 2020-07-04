**目录**

<a href="#STM32F4xx Part">STM32F4xx Part</a><br>
- <a href="#8-bit NAND Flash">8-bit NAND Flash</a>
- <a href="#NAND Flash operations">NAND Flash operations</a>
- <a href="#Timing diagrams for NAND">Timing diagrams for NAND</a>
- <a href="#Error Correction Code">Error Correction Code (ECC)</a>
- <a href="#Timing diagrams for NAND">Timing diagrams for NAND</a>
- <a href="#NAND Flash prewait functionality">NAND Flash prewait functionality</a>
- <a href="#NAND Flash Card control registers">NAND Flash Card control registers</a>
 1. <a href="#FSMC_PCR">PC Card/NAND Flash control registers 2..4 (FSMC_PCR2..4)</a>
 2. <a href="#FSMC_SR2">FIFO status and interrupt register 2..4 (FSMC_SR2..4)</a>

 <h1 id="STM32F4xx Part"> STM32F4xx Part</h1>

这一部分内容是关于STM32的灵活静态存储控制器（FSMC）在NAND Flash上的应用方式与注意事项。

 <h3 id="8-bit NAND Flash">8-bit NAND Flash</h3>
 
|  FSMC signal name   | I/O  |   Function  |
| :-----------------: | ---- | ----------- |
|A[17] |O  |NAND Flash address latch enable (ALE) signal|
|A[16] |O  |NAND Flash command latch enable (CLE) signal|
|D[7:0]|I/O|8-bit multiplexed, bidirectional address/data bus|
|NCE[x]| O | Chip select, x = 2, 3|
|NOE(= NRE)| O |Output enable (memory signal name: read enable, NRE)|
|NWE| O |Write enable|
|NWAIT/INT[3:2]| I |NAND Flash ready/busy input signal to the FSMC|

 <h3 id="NAND Flash operations">NAND Flash operations</h3>

NAND闪存设备的命令锁存使能（CLE）和地址锁存使能（ALE）信号由FSMC控制器的一些地址信号驱动。这意味着要向NAND闪存发送命令或地址，CPU必须对其内存空间中的某个地址执行写入操作。

NAND闪存设备的典型页面读取操作如下：

**Part 1：** 根据NAND Flash的特点，通过配置FSMC_PCRx和FSMC_PMEMx寄存器，编程并启用相应的存储库。

**Part 2：** CPU在公共内存空间执行字节写入，数据字节等于一个闪存命令字节（例如，对于Samsung NAND闪存设备，为0x00）。NAND闪存的CLE输入在写入选通（NWE上的低脉冲）期间处于活动状态，因此写入的字节被解释为NAND闪存的命令。一旦命令被NAND Flash设备锁定，就不需要为下面的页面读取操作写入该命令。

**Part 3：** CPU可以通过写入所需的字节来发送读操作的起始地址（STARTAD），例如，对于容量较小的设备，四个字节或三个字节。STARTAD[7:0]、STARTAD[15:8]、STARTAD[23:16]和TARTAD[25:24]，用于64 Mb x 8位NAND闪存。NAND闪存设备的ALE输入在写入选通（NWE上的低脉冲）期间处于活动状态，因此写入的字节被解释为读取操作的起始地址。使用属性内存空间可以使用FSMC的不同计时配置，该配置可用于实现一些NAND闪存所需的预等待功能。

**Part 4：** 控制器等待NAND闪存准备就绪（R/NB信号高）变为活动状态，然后开始新的访问（到同一个或另一个内存库）。等待时，控制器保持NCE信号激活（低）。

**Part 5：** 然后，CPU可以在公共内存空间中执行字节读取操作，逐字节读取NAND闪存页（数据字段+备用字段）。

**Part 6：** 下一个NAND闪存页可以在没有任何CPU命令或地址写入操作的情况下以三种不同的方式读取：

1. 只需执行步骤5中描述的操作；
2. 可以通过在步骤3重新启动操作来访问新的随机地址；
3. 通过在步骤2重新启动，可以向NAND闪存设备发送新命令。

<h3 id="Error Correction Code">Error Correction Code (ECC)</h3>

FSMC PC卡控制器包括两个纠错码计算硬件块，每个存储库一个。它们用于减少系统软件处理纠错码时主机CPU的工作量。这两个寄存器相同，分别与bank 2和bank 3相关联。因此，没有硬件ECC计算可用于连接到bank 4的存储器。

FSMC中实现的纠错码（ECC）算法可以对从NAND闪存读写的256、512、1024、2048、4096或8192个字节执行1位纠错和2位错误检测。它基于汉明编码算法，包括行和列奇偶校验的计算。

每次NAND闪存组激活时，ECC模块监测NAND闪存数据总线和读/写信号（NCE和NWE）。

功能操作包括：

- 当对bank 2和bank 3进行NAND闪存访问时，D[15:0]总线上的数据被锁定并用于ECC计算。
- 在任何地址访问NAND闪存时，ECC逻辑处于空闲状态，不执行任何操作。因此，定义命令或地址的写操作不考虑在ECC计算中。

一旦CPU从NAND闪存读取/写入所需的字节数，必须读取FSMC_ECCR2/3寄存器以检索计算值。一旦读取，应通过将ECCEN位重置为零来清除它们。要计算新的数据块，ECCEN位必须在FSMC_PCR2/3寄存器中设置为1。

执行ECC计算：

1.	在FSMC_PCR2/3寄存器中启用ECCEN位。
2.	将数据写入NAND闪存页。写入NAND页时，ECC块计算ECC值。
3.	读取FSMC_ECCR2/3寄存器中可用的ECC值，并将其存储在变量中。
4.	清除ECCEN位，然后在FSMC_PCR2/3寄存器中启用它，然后从NAND页读回写入的数据。在读取NAND页时，ECC块计算ECC值。
5.	ECC寄存器3中可用的新ECC/R2值。
6.	如果两个ECC值相同，则不需要进行校正，否则会出现ECC错误，并且软件校正例程返回有关错误是否可以校正的信息。

 <h3 id="Timing diagrams for NAND">Timing diagrams for NAND</h3>

每个PC卡/CompactFlash和NAND闪存库通过一组寄存器进行管理：

- 控制寄存器：FSMC_PCRx
- 中断状态寄存器：FSMC_SRx
- 纠错码（EEC）寄存器：FSMC_ECCRx
- 公共存储空间定时寄存器：FSMC_PMEMx
- 属性存储空间定时寄存器：FSMC_PATTx
- I/O空间定时寄存器：FSMC_PIOx

每个定时配置寄存器包含三个参数，用于定义NAND Flash访问的三个阶段的HCLK循环数，外加一个参数，用于定义在写入时开始驱动数据总线的定时。图表 2显示了公共内存访问的计时参数定义，知道了[属性存储空间]和[I/O存储空间]（仅适用于PC卡）内存空间访问时序是相似的。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xx_NAND/NAND%20controller%20timing%20for%20common%20memory%20access.jpg" width="50%" height="50%">

1.	在写入访问期间，NOE保持高（非活动）。在读取访问期间，NWE保持高（非活动）。
2.	对于写访问，保持相位延迟为（MEMHOLD）x HCLK周期，而对于读访问，保持相位延迟为（MEMHOLD+2）x HCLK周期。

 <h3 id="NAND Flash prewait functionality">NAND Flash prewait functionality</h3>

某些NAND闪存设备要求在写入地址的最后一部分后，控制器等待 R/NB 信号变低。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xx_NAND/Access%20to%20non%20%E2%80%98CE%20don%E2%80%99t%20care%E2%80%99%20NAND-Flash.jpg" width="50%" height="50%">

1. CPU在地址 0x7001 0000 写字节 0x00;
2. CPU在地址 0x7002 0000 写字节 A7-A0;
3. CPU在地址 0x7002 0000 写字节 A15-A8;
4. CPU在地址 0x7002 0000 写字节 A23-A16;
5. CPU在地址 0x7802 0000 写字节 A25-A24;

FSMC使用FSMC_PATT2定时定义执行写访问，其中ATTHOLD≥7（前提是（7+1）×HCLK=112 ns>tWB max）。这保证了在R/NB再次变低或变高之前，NCE保持在较低水平（仅对NCE不关心的NAND闪存请求）。

当需要此功能时，可以通过编程MEMHOLD值来满足tWB定时来保证。然而，对NAND闪存的CPU读取访问具有（MEMHOLD+2）x HCLK周期的保持延迟，而CPU写入访问的保持延迟为（MEMHOLD）x HCLK周期。

为了克服这种定时限制，可以使用属性内存空间，方法是用满足tWB定时的ATTHOLD值对其定时寄存器进行编程，并将MEMHOLD值保持在最小值。然后，CPU必须将公共内存空间用于所有NAND闪存读写访问，除非将最后一个地址字节写入NAND闪存设备，CPU必须写入属性内存空间。

<h3 id="NAND Flash Card control registers">NAND Flash Card control registers</h3>

NAND控制寄存器必须通过字（32位）访问。

<h4 id="FSMC_PCR">PC Card/NAND Flash control registers 2..4 (FSMC_PCR2..4)</h4>

Address offset: 0xA0000000 + 0x40 + 0x20 * (x – 1), x = 2..4

Reset value: 0x0000 0018

**Bit 8:7**

**Bit 8:7**

**Bit 12:9 TCLR[2:0]:** CLE 到 RE(read enable) 的延迟

跟据AHB时钟(HCLK)循环数量设置从CLE变低到RE变低的时间。

Time is t_clr = (TCLR + SET + 2) × THCLK (这里的THCLK是HCLK的周期时长)

- 0000: 1 HCLK cycle (default)
- 1111: 16 HCLK cycles

**Bit 8:7** 保留，必须保持在重置值

**Bit 6 ECCEN:** ECC计算逻辑使能位

- 0: ECC logic is disabled and reset (default after reset),
- 1: ECC logic is enabled.

**Bit 5:4 PWID[1:0]:** 数据总线宽度

定义外部内存设备宽度。

- 00: 8 bits
- 01: 16 bits (default after reset). This value is mandatory for PC Cards.
- 10: reserved, do not use
- 11: reserved, do not use

**Bit 3 PTYP:** Memory 类型

定义连接到相应memory bank的设备类型：

- 0: PC Card, CompactFlash, CF+ or PCMCIA
- 1: NAND Flash (default after reset)

**Bit 2 PBKEN:** PC Card/NAND Flash memory bank使能位。

使能memory bank。访问使能的memory bank会导致AHB总线上出现错误。
- 0：对应的memory bank被禁用（复位后默认）
- 1：启用相应的memory bank

**Bit 1 PWAITEN:**  等待功能使能位

使能PC Card/NAND Flash memory bank的等待功能：
- 0: disabled
- 1: enabled

对于PC卡，当启用等待功能时，MEMWAITx/ATTWAITx/IOWAITx位必须编程为如下值：
xxWAITx≥4+max_wait_assertion_time/HCLK

其中max_wait_assertion_time是nOE/nWE或nIORD/nIOWR低时NWAIT进入低位所用的最长时间。

**Bit 0** 保留，必须保持在重置值











<h4 id="FSMC_SR2">FIFO status and interrupt register 2..4 (FSMC_SR2..4)</h4>

Address offset: 0xA000 0000 + 0x44 + 0x20 * (x-1), x = 2..4

Reset value: 0x0000 0040

