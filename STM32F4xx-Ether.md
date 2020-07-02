<head>
    <script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>
    <script type="text/x-mathjax-config">
        MathJax.Hub.Config({
            tex2jax: {
            skipTags: ['script', 'noscript', 'style', 'textarea', 'pre'],
            inlineMath: [['$','$']]
            }
        });
    </script>
</head>

**目录**

<a href="#Ethernet main features">Ethernet main features</a><br>
- <a href="#MAC core features">MAC core features</a>
- <a href="#DMA features">DMA features</a>
- <a href="#PTP features">PTP features</a>

<a href="#SMI MII and RMII">SMI, MII and RMII</a>
- <a href="#Station management interface">Station management interface: SMI</a>
- <a href="#Media-independent interface">Media-independent interface: MII</a>
- <a href="#Ethernet main features">Reduced media-independent interface: RMII</a>

<h1 id="Ethernet main features"> Ethernet main features </h1>

以太网外备使STM32F4xx能够根据IEEE 802.3-2002标准通过以太网发送和接收数据。它设提供了一个可配置的、灵活的外围设备，以满足各种应用和客户的需求。它支持到外部物理层（PHY）的两个行业标准接口：IEEE 802.3规范中定义的默认媒体独立接口（MII）和精简媒体独立接口（RMII）。它可以用于许多应用，如交换机、网络接口卡等。

<h3 id="MAC core features"> MAC core features </h3>

1. 有外部PHY接口并支持10/100mbit/s的数据传输速率。

2. 符合IEEE 802.3标准的MII接口，用于与外部快速以太网PHY通信。

3. 支持全双工和半双工操作：

- 支持用于半双工操作的CSMA/CD协议；

- 支持全双工操作的IEEE 802.3x流量控制；

- 在全双工操作中，可选地将接收到的暂停控制帧转发给用户应用程序；

- 背压支持半双工操作；

- 在全双工操作中，流量控制输入解除时，自动发送零量程暂停帧。

4. 在发送中插入前导码和帧起始数据（SFD），在接收路径中删除。

5. 在每帧的基础上实行自动CRC校验和生成PAD。（以太网帧长度最小64字节，长度不够的需要加PAD。）

6. 接收帧上的自动PAD/CRC剥离选项。

7. 可编程的帧长度，支持最大为16kb的标准帧。

8. 可编程帧间隙（40-96bit 时长）。

9. 支持多种灵活的地址筛选模式：

- 最多4个48位完美（DA）地址过滤器，每个字节都有掩码；

- 最多3个48位SA地址比较检查，每个字节都有掩码；

- 用于多播和单播（DA）地址的64位哈希过滤器（可选）；

- 通过所有多播寻址帧的选项；

- 支持混杂模式，无需任何过滤即可通过所有帧进行网络监控；

- 使用状态报告传递所有传入数据包（根据筛选器）。

10. 为发送和接收数据包返回单独的状态位（32bit）。

11. 支持接收帧的IEEE 802.1Q VLAN标记检测。

12. 独立的传输和接收以及应用控制接口。

13. 支持带有RMON/MIB计数器（RFC2819/RFC2665）的强制网络统计信息。

14. 用于PHY设备配置和管理的MDIO接口。

15. LAN唤醒帧和AMD Magic Packet帧检测。

16. 以太网帧封装的已接收IPv4和TCP数据包的校验和卸载接收功能。

17. 增强的接收功能，用于检查IPv4头校验和以及封装在IPv4或IPv6数据报中的TCP、UDP或ICMP的校验和。

18. 支持IEEE1588-2008中描述的以太网帧时间戳。在每帧的发送或接收状态下给出64位时间戳。

19. 两组FIFO：一个具有可编程阈值能力的2-KB传输FIFO，一个具有可配置阈值的2-KB接收FIFO（默认为64字节）

20. 在EOF传输后插入接收FIFO的接收状态向量允许在接收FIFO中存储多个帧，而不需要另一个FIFO来存储这些帧的接收状态。

21. 选项在接收时过滤所有错误帧，而不在存储和转发模式下将其转发到应用程序。

22. 选择转发尺寸不足的好帧。

23. 通过为接收队列中丢失或损坏的帧（由于溢出）生成脉冲来支持统计。

24. 支持向MAC核传输的存储转发机制。

25. 基于接收队列填充（阈值可配置）水平，自动生成暂停帧控制或返回到MAC核心的压力信号。

26. 处理碰撞帧的自动重新传输以进行传输。

27. 在后期碰撞、过度碰撞、过度延迟和欠载情况下丢弃帧。

28. 刷新Tx 队列的软件控制。

29. 存储和转发模式下，在传输的帧中计算并插入IPv4头校验和和和TCP、UDP或ICMP的校验和。

30. 支持MII上的内部环回以进行调试。

<h3 id="DMA features"> DMA features </h3>

1. 支持AHB从接口中的所有AHB突发类型。

2. 软件可在AHB主界面中选择AHB突发类型（固定或不定突发）。

3. 从AHB主端口选择地址对齐脉冲的选项。

4. 使用帧分隔符优化面向分组的DMA传输。

5. 字节对齐寻址，支持数据缓冲区。

6. 双缓冲区（环）或链表（链式）描述符链式。

7. 描述符架构，允许大数据块传输，CPU干预最少。

8. 每个描述符最多可以传输8 KB的数据。

9. 全面的正常运行和错误传输状态报告。

10. 用于发送和接收DMA引擎的单个可编程突发大小，以实现最佳主机总线利用率。

11. 不同操作条件下的可编程中断选项。

12. 每帧发送/接收完全中断控制。

13. 接收和发送引擎之间的循环或固定优先级仲裁。

14. 启动/停止模式。

15. 当前Tx/Rx缓冲指针作为状态寄存器。

16. 当前Tx/Rx描述符指针作为状态寄存器。

<h3 id="PTP features"> PTP features </h3>

1. 接收和发送帧时间戳。

2. 粗、精校正方法。

3. 当系统时间大于目标时间时触发中断。

4. 每秒脉冲输出（产品替代功能输出）。 

<h1 id="SMI MII and RMII"> SMI, MII and RMII </h1>

以太网外围设备由带有专用DMA控制器的MAC 802.3（媒体访问控制）组成。它通过一个选择位（参考SYSCFG_PMC寄存器）支持默认的媒体独立接口（MII）和精简的媒体独立接口（RMII）。

DMA控制器通过AHB主从接口与核心和存储器接口。AHB主接口控制数据传输，而AHB从接口访问控制和状态寄存器（CSR）空间。

发送FIFO（Tx FIFO）缓冲在MAC核发送前由DMA从系统存储器读取数据。类似地，接收FIFO（Rx FIFO）存储接收的以太网帧，直到它们被DMA传输到系统存储器。

以太网外围设备还包括SMI以与外部PHY通信。一组配置寄存器允许用户为MAC和DMA控制器选择所需的模式和特性。

注：使用以太网时，AHB时钟频率必须至少为25 MHz。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/ETH%20block%20diagram.jpg" width="50%" height="50%">

<h3 id="Station management interface"> Station management interface: SMI </h3>

<h3 id="Media-independent interface"> Media-independent interface: MII </h3>

<h3 id="Reduced media-independent interface"> Reduced media-independent interface: RMII </h3>

精简的媒体独立接口（RMII）规范以10/100mbit/s的速度减少了微控制器以太网外围设备和外部以太网之间的管脚数。

根据IEEE 802.3u标准，MII包含16个数据和控制管脚。RMII规范专门用于将管脚数减少到7个管脚（管脚数减少62.5%）。

RMII在MAC和PHY之间实例化。这有助于将MAC的MII转换为RMII。RMII块具有以下特征：

- 支持10 Mbit/s和100 Mbit/s的工作速率；

- 时钟基准必须加倍至50 MHz；

- 相同的时钟参考必须从外部来源到MAC和外部以太网PHY；

- 它提供独立的2位宽（dibit）的传输和接收数据路径。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/Reduced%20media-independent%20interface%20signals.png" width="50%" height="50%">

<h3 id="RMII clock sources"> RMII clock sources </h3>

从外部50MHz时钟对PHY进行时钟，或者使用带有嵌入式PLL的PHY来生成50MHz频率。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/RMII%20clock%20sources.png" width="50%" height="50%">

<h3 id="MII/RMII selection"> MII/RMII selection </h3>

模式MII或RMII是使用SYSCFG_PMC寄存器中的配置位23 MII_RMII_SEL来选择的。当以太网控制器处于重置状态或启用时钟之前，应用程序必须设置MII/RMII模式。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/Clock%20scheme.png" width="50%" height="50%">

为了节省一个引脚，两个输入时钟信号RMII_REF_CK和MII_RX_CLK被多路复用在同一个GPIO管脚上。

<h1 id="MAC 802.3"> MAC 802.3 </h1>

IEEE 802.3局域网（LANs）国际标准采用CSMA/CD（带冲突检测的载波感知多址）作为接入方式。以太网外设由独立接口（MII）的MAC 802.3（媒体访问控制）控制器和专用DMA控制器组成。

MAC区块为以下系列系统实现LAN-CSMA/CD子层：基带和宽带系统的10mbit/s和100mbit/s数据速率。支持半双工和全双工操作模式。碰撞检测访问方法仅适用于半双工操作模式。支持MAC控制帧子层。

MAC子层执行与数据链路控制过程相关联的以下功能：

1. 数据封装（发送和接收）

- 帧（帧边界定界、帧同步）

- 寻址（处理源地址和目标地址）

- 错误检测

2. 媒体访问管理

- 中等配置（避免碰撞）

- 争用解决（冲突处理）

基本上，MAC子层有两种工作模式：

1. 半双工模式：工作站使用CSMA/CD算法争夺物理介质的使用。

2. 全双工模式：当满足以下所有条件时，无需争用资源（不需要CSMA/CD算法）的同时传输和接收：

- 支持同时传输和接收的物理介质能力

- 正好有2个站点连接到LAN

- 两个站点均配置为全双工操作

<h3 id="MAC 802.3 frame format"> MAC 802.3 frame format </h3>

MAC块实现IEEE 802.3-2002标准指定的MAC子层和可选MAC控制子层（10/100mbit/s）。

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/MAC%20frame%20format.jpg" width="50%" height="50%">

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/Tagged%20MAC%20frame%20format.png" width="50%" height="50%">

为使用CSMA/CD MAC的数据通信系统指定了两种帧格式：

1. 基本MAC帧格式

2. 标记的MAC帧格式（基本MAC帧格式的扩展）

以上两张图片描述了框架结构（未标记和标记），包括以下字段：

1. 前导码3：用于同步的7字节字段（PLS电路）十六进制值：55-55-55-55-55-55-55-55-55位模式：01010101 01010101 01010101 01010101 01010101 01010101（从右到左位传输）。

2. 开始帧分隔符（SFD）：用于指示帧开始的1字节字段。十六进制值：D5位模式：11010101（从右到左位传输）。

3. 目的地和源地址字段：6字节字段，表示目的地和源站地址，如下所示）：

<img src="https://github.com/laneston/Pictures/blob/master/Post-STM32F4xxP_Ether/Address%20field%20format.png" width="50%" height="50%">

- 每个地址的长度为48位。

- 目标地址字段中的第一个LSB位（I/G）用于指示个人地址（I/G=0）或组地址（I/G=1）。一个组地址能识别无、1个、多个或所有连接到局域网的站点。在源地址中，第一位被保留并重置为0。

- 第二位（U/L）区分本地（U/L=1）或全局（U/L=0）受管地址。对于广播地址，该位也是1。

- 每个地址字段的每个字节必须首先传输最低有效位。

地址指定基于以下类型：

1. 单个地址：这是与网络上特定站点相关联的物理地址。

2. 组地址。与给定网络上的一个或多个站点关联的多目标地址。多播地址有两种：

- 多播组地址：与一组逻辑相关站点关联的地址。

- 广播地址：一个可分辨的预定义多播地址（目标地址字段中的所有1），它始终表示给定LAN上的所有站点。

3. QTag前缀：在源地址字段和MAC客户端长度/类型字段之间插入的4字节字段。此字段是基本帧（未标记）的扩展，用于获取标记的MAC帧。未标记的MAC帧不包括此字段。标记扩展如下：

- 与类型解释（大于0x0600）一致的2字节常量长度/类型字段值，等于802.1Q标记协议类型（0x8100十六进制）的值。此常量字段用于区分标记的和未标记的MAC帧。

- 包含标签控制信息字段的2字节字段，细分如下：3位用户优先级、规范格式指示符（CFI）位和12位VLAN标识符。标记的MAC帧的长度由QTag前缀扩展了4个字节。

4. MAC客户端长度/类型：2字节字段，含义不同（互斥），具体取决于其值：

- 如果该值小于或等于maxValidFrame（0d1500），则此字段指示802.3帧的后续数据字段中包含的MAC客户端数据字节数（长度解释）。

- 如果该值大于或等于MinTypeValue（0d1536 decimal，0x0600），则此字段指示与以太网帧相关的MAC客户端协议（类型解释）的性质。

无论对长度/类型字段的解释如何，如果数据字段的长度小于协议正常运行所需的最小值，则在数据字段之后但在FCS（帧检查序列）字段之前添加一个PAD字段。长度/类型字段首先用高阶字节发送和接收。

对于maxValidLength和minTypeValue（不包括边界）之间的长度/类型字段值，不指定MAC子层的行为：MAC子层可以传递它们，也可以不传递它们。

5. 数据和PAD字段：n字节数据字段。提供了完全的数据透明性，这意味着任何字节值的任意序列都可能出现在数据字段中。PAD的大小（如果有的话）由数据字段的大小决定。数据和PAD字段的最大和最小长度为：

- 最大长度=1500字节

- 未标记MAC帧的最小长度=46字节

- 标记MAC帧的最小长度=42字节

当数据字段长度小于所需的最小值时，将添加PAD字段以匹配最小长度（标记帧为42字节，未标记帧为46字节）。

6. 帧检查序列：包含循环冗余检查（CRC）值的4字节字段。CRC计算基于以下字段：源地址、目标地址、QTag前缀、长度/类型、LLC数据和PAD（即除前导码、SFD之外的所有字段）。生成多项式如下：

G(X)= $X^32$

帧的CRC值计算如下：

\1.   帧的前2位被补。

\2.     帧的n位是次数(n–1)的多项式M(x)的系数。目标地址的第一位对应于![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image018.png)项，数据字段的最后一位对应于![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image020.png)项。

\3.     M(x)与![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image022.png)相乘，再除以G(x)，产生一个小于31的余数。

\4.   R(x)的系数被视为32位序列。

\5.   位序列被补充，结果是CRC。

\6.     CRC值的32位放入帧检查序列中。![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image022.png)项是第一个传输，![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image020.png)项是最后一个传输。

MAC帧的每个字节，除FCS字段外，首先传输低阶位。无效的MAC帧由以下条件之一定义：

\1.   帧长度与长度/类型字段指定的预期值不一致。如果长度/类型字段包含类型值，则假定帧长度与此字段一致（没有无效帧）。

\2.   帧长度不是整数字节数（额外位）。

\3.   在传入帧上计算的CRC值与包含的FCS不匹配。

## MAC frame transmission

DMA控制传输路径的所有事务。从系统存储器读取的以太网帧被DMA推入FIFO。然后将这些帧弹出并传输到MAC核心。当帧结束被传输时，传输的状态从MAC核取出并传输回DMA。传输FIFO的深度为2Kbyte。FIFO填充级别指示给DMA，以便它可以使用AHB接口从系统内存启动所需的突发数据提取。来自AHB主接口的数据被推入FIFO。

当检测到SOF时，MAC接收数据并开始向MII传输。应用程序启动传输后，将帧数据传输到MII所需的时间是可变的，这取决于IFG延迟、发送前导码/SFD的时间以及半双工模式的任何退避延迟等延迟因素。EOF传输到MAC核后，核心完成正常传输，然后将传输状态返回给DMA。如果在传输过程中发生正常的冲突（半双工模式），MAC核将使传输状态有效，然后接受并丢弃所有进一步的数据，直到接收到下一个SOF。在观察到来自MAC的重试请求（处于状态）时，应该从SOF重新传输相同的帧。如果在传输期间没有连续地提供数据，MAC发出下溢状态。在帧的正常传输过程中，如果MAC接收到SOF而没有得到前一帧的EOF，则SOF被忽略，新帧被视为前一帧的继续。

向MAC核弹出数据有两种操作模式：

\1.   在阈值模式下，只要FIFO中的字节数超过配置的阈值级别（或在超过阈值之前写入帧结束时），数据就可以弹出并转发到MAC核心。使用ETH-DMABMR的TTC位配置阈值大小。

\2.   在存储和转发模式下，只有在FIFO中存储完整帧后，帧才会朝MAC核弹出。如果Tx FIFO的大小小于要传输的以太网帧，则当Tx FIFO几乎满时，该帧会向MAC核弹出。

应用程序可以通过设置FTF（ETH-DMAOMR register[20]）bit来刷新所有内容的传输FIFO。该位是自清除的，并将FIFO指针初始化为默认状态。如果在帧传输到MAC核期间设置了FTF位，则传输将停止，因为FIFO被认为是空的。因此，在MAC发射机处发生下溢事件，并且相应的状态字被转发到DMA。

### Automatic CRC and pad generation

当从应用程序接收到的字节数低于60（DA+SA+LT+Data）时，在发送帧中追加零，使数据长度正好为46字节，以满足IEEE 802.3的最小数据字段要求。MAC可以编程为不附加任何填充。计算帧检查序列（FCS）字段的循环冗余校验（CRC），并将其附加到正在发送的数据中。当MAC被编程为不将CRC值附加到以太网帧的末尾时，计算出的CRC不会被发送。此规则的一个例外是，当MAC被编程为对小于60字节的帧（DA+SA+LT+Data）附加PAD时，CRC将被附加到填充帧的末尾。

CRC生成器计算以太网帧的FCS字段的32位CRC。编码由以下多项式定义。

 

![img](file:///C:/Users/ADMINI~1/AppData/Local/Temp/msohtmlclip1/01/clip_image024.png)

### Transmit protocol

MAC控制以太网帧传输的操作。它执行以下功能以满足IEEE 802.3/802.3z规范。

\1.   生成前导码和SFD。

\2.   以半双工模式生成阻塞模式。

\3.   控制Jabber超时。

\4.   控制半双工模式的流量（背压式）。

\5.   生成传输帧状态。

\6.   包含符合IEEE1588的时间戳快照逻辑。

当请求新的帧传输时，MAC发送前导码和SFD，然后是数据。前导码被定义为0b10101010模式的7字节，SFD被定义为0b10101011模式的1字节。碰撞窗口定义为1时隙时间（10/100mbit/s以太网为512位时间）。干扰模式生成仅适用于半双工模式，而不适用于全双工模式。

在MII模式下，如果在从帧开始到CRC字段结束的任何时候发生冲突，MAC在MII上发送一个0x555555的32位阻塞模式，以通知所有其他站点发生了冲突。如果在前导码传输阶段发现冲突，MAC完成前导码和SFD的传输，然后发送干扰模式。

如果必须传输超过2048字节（默认值），则保持jabber定时器以切断以太网帧的传输。MAC在半双工模式下使用延迟机制进行流量控制（背压）。当应用程序请求停止接收帧时，MAC只要感测到帧的接收，就发送32字节的干扰模式，前提是发射流控制被启用。

这会导致碰撞，远程工作站会后退。应用程序通过在ETH-MACFCR寄存器中设置BPA位（位0）来请求流控制。如果应用程序请求传输帧，则即使在启动背压时也会对其进行调度和传输。注意，如果背压保持激活很长一段时间（并且发生超过16个连续的碰撞事件），则远程站会因过度碰撞而中止其传输。如果为传输帧启用IEEE1588时间戳，则当SFD被放到传输MII总线上时，此块将获取系统时间的快照。

### Transmit scheduler

MAC负责调度MII上的帧传输。它保持两个传输帧之间的帧间隔，并遵循半双工模式下的截断二元指数退避算法。MAC在满足IFG和退避延迟后启用传输。它保持任意两个发送帧之间配置的帧间间隔（ETH-MACCR寄存器中的IFG位）的空闲周期。如果要发送的帧早于配置的IFG时间到达，则MII在开始对其发送之前等待来自MAC的使能信号。一旦MII的载波信号变为非活动状态，MAC就会启动IFG计数器。在编程的IFG值结束时，MAC启用全双工模式的传输。

在半双工模式下，当IFG被配置为96位次时，MAC遵循IEEE 802.3规范第4.2.3.2.1节中规定的遵从规则。如果在IFG间隔的前三分之二（所有IFG值为64位倍）期间检测到载波，MAC重置其IFG计数器。如果在IFG间隔的最后三分之一期间检测到载波，MAC继续IFG计数并在IFG间隔之后启用发射机。MAC在半双工模式下运行时实现了截短的二进制指数退避算法。

### Transmit flow control

当发送流控制使能位（ETH-MACFCR中的TFE位）被设置时，MAC生成暂停帧，并在必要时以全双工模式发送它们。暂停帧被附加计算出的CRC，并被发送。暂停帧生成可以通过两种方式启动。

当应用程序在ETH-MACFCR寄存器中设置FCB位或当接收FIFO已满（包缓冲区）时，发送暂停帧。

如果应用程序已通过在ETH-MACFCR中设置FCB位来请求流控制，则MAC生成并发送单个暂停帧。生成帧中的暂停时间值包含ETH_MACFCR中的编程暂停时间值。要在先前发送的暂停帧中指定的时间之前延长暂停或结束暂停，应用程序必须在将暂停时间值（ETH_MACFCR寄存器中的PT）编程为适当的值之后请求另一个暂停帧传输。

如果应用程序在接收FIFO满时请求流控制，MAC生成并发送暂停帧。生成帧中的暂停时间的值是ETH-MACFCR中的编程暂停时间值。如果在该暂停时间耗尽之前，接收FIFO在可配置的时隙次数（ETH_MACFCR中的PLT比特）处保持满，则发送第二暂停帧。只要接收的FIFO保持满，则重复该过程。如果在采样时间之前不再满足该条件，则MAC发送具有零暂停时间的暂停帧，以指示远程端接收缓冲器准备好接收新的数据帧。

### Single-packet transmit operation

发送操作的一般事件顺序如下：

\1.   如果系统有要传输的数据，DMA控制器通过AHB主接口从内存中获取这些数据，并开始将它们转发到FIFO。它继续接收数据，直到传输完帧。

\2.   当超过阈值水平或接收到FIFO中的完整数据包时，帧数据被弹出并驱动到MAC核。DMA继续从FIFO传输数据，直到完整的数据包传输到MAC。当帧完成时，DMA控制器被来自MAC的状态通知。

### Transmit operation—Two packets in the buffer

\1.   因为DMA必须在将描述符释放到主机之前更新描述符状态，所以在传输FIFO中最多可以有两个帧。只有在设置了OSF（对第二帧操作）位的情况下，DMA才会获取第二帧并将其放入FIFO。如果未设置此位，则只有在MAC完全处理完该帧并且DMA释放了描述符之后，才从内存中获取下一帧。

\2.   如果设置了OSF位，DMA在完成第一帧到FIFO的传输后立即开始获取第二帧。它不会等待状态更新。同时，当第一帧被发送时，第二帧被接收到FIFO中。一旦第一帧被传输并且从MAC接收到状态，它就被推送到DMA。如果DMA已经完成将第二个包发送到FIFO，则第二个传输必须等待第一个包的状态，然后才能继续到下一帧。

### Retransmission during collision

当帧被传送到MAC时，在半双工模式下MAC线路接口上可能发生冲突事件。然后，MAC将通过在接收到帧结束之前，给出状态来指示重试尝试。然后重新传输被启用，帧从FIFO中再次弹出。在向MAC核弹出超过96个字节后，FIFO控制器释放出空间，并使DMA可以将更多数据推入。这意味着超过此阈值或当MAC核指示延迟碰撞事件时，无法重新传输。

### Transmit FIFO flush operation

MAC通过使用操作模式寄存器中的位20来控制软件刷新发送FIFO。刷新操作是立即的，即使Tx FIFO正在向MAC核传输帧，Tx FIFO和相应的指针也被清除到初始状态。这导致MAC发送器中发生下溢事件，并且帧传输被中止。这种帧的状态用下溢和帧刷新事件（TDES0位13和1）标记。在刷新操作期间，没有数据从应用程序（DMA）进入FIFO。传输传输状态字被传输到应用程序以获得刷新的帧数（包括部分帧数）。完全刷新的帧设置了帧刷新状态位（TDES0 13）。当应用程序（DMA）已接受刷新帧的所有状态字时，刷新操作完成。然后清除传输FIFO刷新控制寄存器位。此时，来自应用程序（DMA）的新帧被接受。所有在刷新操作后显示以供传输的数据都将被丢弃，除非它们以SOF标记开头。

### Transmit status word

在以太网帧传输到MAC核心的最后，在核心完成帧的传输之后，向应用程序给出传输状态。传输状态的详细描述与TDES0中的位[23:0]相同。如果启用IEEE1588时间戳，则返回特定帧的64位时间戳以及传输状态。

### Transmit checksum offload

TCP和UDP等通信协议实现校验和字段，这有助于确定通过网络传输的数据的完整性。由于以太网最广泛的用途是封装TCP和UDP over IP数据报，因此以太网控制器具有传输校验和卸载功能，支持在传输路径中进行校验和计算和插入，并在接收路径中进行错误检测。本节说明传输帧的校验和卸载功能的操作。

**TCP****、UDP或ICMP的校验和是在一个完整的帧上计算的，然后插入到其相应的头字段中。由于这一要求，仅当发送FIFO被配置为存储和转发模式时（即，当TSF位设置在ETH-ETH-DMAOMR寄存器中时），此功能才被启用。如果核心配置为阈值（穿透）模式，则绕过传输校验和卸载。**

**在将帧传输到MAC核心发射机之前，必须确保传输FIFO足够深，能够存储完整的帧。如果FIFO深度小于输入以太网帧大小，则绕过有效负载（TCP/UDP/ICMP）校验和插入功能，仅修改帧的IPv4头校验和，即使在存储和转发模式下也是如此。**

**传输校验和卸载支持两种校验和计算插入。这个可以通过设置CIC位（位28:27 in TDES1，在TDES1：****Transmit descriptor Word1****中描述）来控制每个帧的校验和。**
