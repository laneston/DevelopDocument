# NAND Flash operations

NAND闪存设备的命令锁存使能（CLE）和地址锁存使能（ALE）信号由FSMC控制器的一些地址信号驱动。这意味着要向NAND闪存发送命令或地址，CPU必须对其内存空间中的某个地址执行写入操作。

NAND闪存设备的典型页面读取操作如下：

## Part 1

根据NAND Flash的特点，通过配置FSMC_PCRx和FSMC_PMEMx寄存器，编程并启用相应的存储库。

一些NAND闪存设备要求在写入地址的最后一部分后，控制器等待R/NB信号变低，如图所示。

![Access to non ‘CE don’t care’ NAND-Flash](https://github.com/laneston/Pictures/blob/master/Post-STM32F4xx_NAND/Access to non 'CE don’t care' NAND-Flash.jpg)


1. CPU wrote byte 0x00 at address 0x7001 0000.
2. CPU wrote byte A7-A0 at address 0x7002 0000.
3. CPU wrote byte A15-A8 at address 0x7002 0000.
4. CPU wrote byte A23-A16 at address 0x7002 0000.
5. CPU wrote byte A25-A24 at address 0x7802 0000: FSMC performs a write access using FSMC_PATT2 timing definition, where ATTHOLD ≥7 (providing that (7+1) × HCLK = 112 ns > tWB max). This guarantees that NCE remains low until R/NB goes low and high again (only requested for NAND Flash memories where NCE is not don’t care).





