**4 声呐和毫米波雷达的被动感知**

**4.1 简介**

在自动驾驶系统中，通常存在两种不同的感知系统，分别为：主动感知以及被动感知。在主动感知的定义中，检测到的障碍物会被主动发送给规划和控制模块用于辅助决策，然后规划和控制模块会依据感知到的环境来给出一系列指令以控制无人驾驶车辆。而作为被动感知，该方案意味着当检测到障碍物后，原始数据不会被送入规划和控制模块进行决策。相反，原始数据会通过控制器区域网络(CAN)总线的形式直接发送到底盘，以便快速地对突发情况做出决策。被动感知通过 CAN 总线直接连接到底盘，这其实只需要在底盘上添加一个简单的决策模块，当车辆检测到安全距离内存在有障碍物时停止车辆。这也符合我们自动驾驶的初衷，即在发生紧急情况时，尽快制动车辆，而不是通过完整的决策流程才将信息发送给底盘，这也是保证乘客以及行人安全的最佳方式。

因此，在我们的模块化设计架构中，有三层保护：首先是计算机视觉（主动感知），主要用于远距离障碍物探测；毫米波（mmWave）雷达，主要用于中距离障碍物探测；声呐传感器，主要用于短距离障碍物探测。这些传感器的作用其实取决于你如何设计自动驾驶感知系统，比如说毫米波雷达也可以帮助主动感知。如图 4.1 所示，可以采用毫米波雷达和声呐传感器进行被动感知。在本章中，我们首先介绍毫米波雷达技术的基本原理，然后解释如何利用毫米波雷达和声呐传感器来部署自动驾驶中的被动感知。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.001.jpeg)

图 4.1 模块化设计架构

在上面的 4.1 图中，最下方分别为底盘(Chassis)，声呐(Sonars)，毫米波雷达(Radar)；通过中间的控制器区域网络(CAN bus)将底层传感器与上面的规划与控制(Planning and Control)联系在一起，然后规划控制与计算机视觉(Computer Vision)、地图(Map)和全球定位系统(GNSS)关联在一起。

**4.2 毫米波雷达的基本原理**

毫米波是一类特殊的雷达技术，其通过短波长的电磁波完成对物体的检测[1]。在这一小节中，作者将介绍毫米波雷达的基本原理；如果想要了解更完整和全面的信息，可见参考文献[2]。

简而言之，雷达传感器通过向视场角范围内发射电磁波信号，并在碰撞到路径上的物体后反射回来。雷达传感器通过捕捉反射的信号，即可以确定物体的范围、速度以及物体的角度。

顾名思义，我们这一节讲到的毫米波雷达其信号的波长会在毫米范围内，这个尺度的波长在电磁波谱中通常被认为是短波长。短波长的传感器在实际应用中有着比较明显的优势，第一个优点是，短波长可以相应的将系统组件的尺寸做的很小，例如处理毫米波信号所需的天线；另一个优点是精确度高，一个工作在 76-81GHz 的毫米波系统，其相应的波长约为 4 毫米，这也代表这该传感器有能力探测到几分之一毫米的运动，大大提升了检测的精度。

一个完整的毫米波雷达系统会包含有发射(TX)、接收(RX)和射频(RF)这三种通用基础组件；能够处理模拟信号的组件，如时钟；以及能够转换处理数字信号的组件，如模数转换器(ADCs)、微控制器单元(MCUs)和数字信号处理器(DSPs)。

此外还有一类特殊的毫米波技术被称为调频连续波(FMCW)。顾名思义，使用调频连续波技术的雷达可以连续发射频率调制信号以测量范围内物体的距离以及角度和速度等信息[3]。

**4.2.1 距离测量**

对于雷达系统而言，其基本原理就是雷达传感器会主动发射一束电磁信号，在碰撞到物体后电磁信号会沿着原路发生反射。对于调频连续波雷达，其信号频率随时间线性增加。这种类型的信号被称为 "线性调频(chirp)"。

调频连续波雷达(FMCW)系统会发射一个线性调频信号，并捕捉碰撞到物体后原路反射回来的电磁信号。图 4.2 代表了一个调频连续波雷达的射频(RF)部件的简化框图， 具体工作原理如下。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.002.jpeg)

图 4.2 FMCW 雷达框图。1 为合成器(synthesizer)；2 为 TX 天线(TX)；3 为 RX 天线(RX)；4 为混频器(mixer)并最终输出中频信号(IF Signal)

接下来我们需要注意几个细节：

① 每一个合成器（synthesizer）都会产生一个线性调频信号(chirp)。

② 该线性调频(chirp)会由发射天线（TX ant）发射。

③ 当线性调频信号在碰撞到一个物体，并发生反射后，会产生一个反射的线性调频信号，该信号由接收天线（RX ant）捕获。

④ 混频器(mixer)会将RX和TX信号结合起来，产生一个中频信号(IF Signal)。其中混频器是一个电子元件，它的作用是将两个信号结合起来并输出一个新频率的新信号。

其中，混频器的输出处会产生一个瞬时频率，其频率值等同于 TX 线性调频信号(chirp)和 RX 线性调频信号(chirp)的瞬时频率之差；其输出的相位值也等于 TX 线性调频信号(chirp)和 RX 线性调频信号(chirp)的相位之差。因此，混频器输出的初始相位即为发送线性调频信号(chirp)对应时刻的 TX 线性调频信号(chirp)相位与 RX 线性调频信号(chirp)相位之差。通过混频器输出的相位，我们可以快速的推导出被探测物体的距离信息。

**4.2.2 速度测量**

对于速度的测量，调频连续波(FMCW)雷达会发射两个时间相隔tC的线性调频信号(chirp)。每个反射的线性调频信号(chirp)会通过 FFT(快速傅里叶变换)处理，以检测物体可能存在的范围，这种技术被称为范围快速傅里叶变换(range‐FFT)。其中，每个线性调频信号(chirp)对应的范围快速傅里叶变换(range‐FFT)在相同的位置处存在有峰值，但两者的相位是不同的。因此，测量两者的相位差可以计算出被探测物体的速度，即vc。

值得一提的是，在测量时，如果存在有多个具有不同速度的运动物体，且它们与雷达之间的距离相同，那么基于两个线性调频信号(chirp)的速度测量方法就失效了。其原因是因为这些待测量物体均处与雷达有着相同的距离，这就导致它们将反射回具有相同中频频率的信号。因此，范围快速傅里叶变换(range‐FFT)将只会产生一个单一的峰值，反映到现实中就代表这些物体在相同距离上的均产生一个信号并叠加的结果。如果在这种情况下，雷达系统必须发射两个以上的线性调频信号(chirp)来测量速度，例如发射一组等间隔的N个线性调频信号(chirp)。通常这一组线性调频信号(chirp)被称为——线性调频帧。

**4.2.3 角度测量**

在上文中，我们提到调频连续波(FMCW)雷达系统也是可以根据反射信号来估算出物体与水平面的夹角的。角度估计的原理如下：物体的距离微小变化会导致范围快速傅里叶变换(range‐FFT)峰值的相位变化，该结果可以在雷达存在有两个RX天线的情况下进行角度估计。因为物体到每个RX天线的距离不同，这会导致FFT峰值的相位产生差异。而这个相位差异可以使我们能够估计反射信号的角度。

**4.3 毫米波雷达部署**

图 4.3 向读者展示了如何将毫米波雷达传感器安装在车辆上。一般来说，毫米波雷达这类设备会被放在车辆的正前方位置，使其能够捕捉到车辆前方 15-20 米的范围（图 4.4）。当有物体闯入到探测范围内，毫米波雷达可以立即检测到该物体，并将检测结果直接发送给底盘用于被动感知(passive perception)，或者发送到主计算单元用于主动感知，在经过处理后再发送给底盘用于规划控制。对于大多数低速的自动驾驶车辆而言，一般的制动距离小于 1 米。这使得可供主动感知的探测范围在车辆的 5 米以外，而可供被动感知使用的探测范围一般则在 5 米以内。当然，这个阈值不是固定的，可以使用软件 UI 界面进行动态配置。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.003.jpeg)

图 4.3 毫米波雷达在车辆当中的一般安装位置

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.004.jpeg)

图 4.4 毫米波雷达的探测范围

图 4.5 向读者展示了雷达的硬件链接关系。在默认情况下，雷达是直接连接到 CAN 总线上的。但是如果主计算单元没有 CAN 总线，就会需要一个 USB-CAN 的转换装置（CAN 卡）来将雷达的信息传输到 USB 设备上。此外还需要一个电源来为雷达传感器供电。一旦雷达传感器被连接到 CAN 卡上，CAN 卡就可以将 CAN 信息转化为 USB 串口信息发送到计算机上，并开始读取探测数据。这部分难度不大，基本上在 5 分钟以内可以顺利配置完成。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.005.jpeg)

图 4.5 硬件组成，其中包含有供电电源(Power supply)，USB 接口(USB interface)，CAN 卡(CAN card)以及 77G 频率雷达(77G Radar)

图 4.6 向读者显示了毫米波雷达配置显示的用户界面(UI)，该用户界面将探测到的物体以鸟瞰的形式投影到界面中。在界面上，读者可以看到清晰地显示了探测到的障碍物的距离以及方向。这些结果将会作为被动感知的结果发送给底盘用于对自动驾驶车辆进行规划和控制，并促使模块能够做出智能的运动控制决策。该款 77G 频率毫米波雷达的操作演示视频可以在[4]链接中找到。此外，图 4.7 展示了该设备的硬件规格[5]。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.006.jpeg)

图 4.6 毫米波雷达的用户界面(UI)

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.007.jpeg)

图 4.7 雷达的硬件规格

这里我们来具体看一下图 4.7 的规格参数，首先是分成了三大块：覆盖范围(Coverage)、准确度(Accuracy)、其他参数(Others)。其中在覆盖范围(Coverage)中存在有以下几个比较重要的指标：水平视场角(Horizontal Angle)、垂直视场角(Vertical Angle)、最大探测范围(Maximum Range)、雷达刷新频率(Update Interval)、最大跟踪目标(Maximum Tracking Targets)。准确度(Accuracy)中存在有两个指标：探测范围的精度(Range)以及角度分辨率(Angle Resolution)。对于其他参数(Others)而言，存在有 DC 输入电压(DC Voltage)、DC 功率(DC Power)、毫米波雷达尺寸(Size)、输出接口(Output Interface)。

下面的这个代码片段向读者展示了毫米波雷达简单的数据结构，其中包含物体索引、物体的范围或距离、物体的径向速度、物体的径向加速度、方位角以及信号强度或者功率。同时为了更加方便的调用雷达传感器，作者还提供一个非常简单的软件应用程序接口（API）来捕捉雷达数据，通过直接调用软件应用程序接口（API）可以使读者快速建立自己的被动或主动感知逻辑。

struct MWR\_Data {

int index; // object index (range: 0 – 63)

float Range; // usually the range is within 30 meters.

float RadialVelocity; // radial velocity

float RadialAcc; // radial acceleration

float Azimuth; // azimuth angle with clockwise direction.

float Power; // detection signal strength

MWR\_Data(int i, float Rg, float RV, float RA, float Az, float Pw)

: index(i), Range(Rg), RadialVelocity(RV), RadialAcc(RA), Azimuth(Az),

Power(Pw) {}

};

MWRadar mwr\_radar;

std::vector<MWR\_Data> data; // data means the radar's data

// Read the latest 10 frame.

mwr\_radar.Read(data,10)

// Read the latest frame

mwr\_radar.Read(data,1)

**4.4 声呐传感器部署**

声呐传感器的原理是主动发射人类无法听到的超声波，并接收被障碍物反射回来的声波，由于在空气中声波的传播速度是恒定的，所以根据声波发出到接收到声波所需时间，可以快速地计算出距离。这有点类似于上文讲的毫米波雷达如何测量无线电波击中物体后返回所间隔的时间。

由于原理上的不同，声呐可以探测到雷达和激光雷达(LiDAR)无法探测到的某些物体。例如，雷达或者说基于光特性的传感器，都很难正确检测出塑料、玻璃等透明的物体，但声呐传感器却可以很好地对塑料、玻璃等进行探测。而且，声呐传感器的原理只是超声波，它不会受到待检测材料颜色的影响。当然，声呐传感器也存在问题，如果一个物体是由吸音的材料制成的，或者其形状能够将声波反射到远离接收器的位置，使声呐接收不到返回的超声波，那么声呐传感器的读数将是不可靠的。

可以这么说，在我们自动驾驶场景中，声呐传感器是车辆安全的最后一道防线，通过不间断的检测，以保证汽车周围 3 米范围内的安全，从而确保底盘能够及时处理突发危险。

图 4.8 显示了在自动驾驶车辆上声呐的安装位置。一般来说会将声呐设备放在车辆前放的中间两侧。由于是扇形区域，声呐传感器能够快速有效地捕捉到车辆前方 3-5 米范围内的障碍物(图 4.9)。当有物体进入到声呐的探测范围内，声呐传感器可以快速地探测到物体，并将检测结果直接发送给底盘用于被动感知(passive perception)，这部分的声呐传感器演示视频可以在链接[6]中找到。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.008.jpeg)

图 4.8 声呐传感器在车辆当中通常安装的位置

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.009.jpeg)

图 4.9 声呐传感器的探测范围

图 4.10 向读者展示了声呐的硬件测试装置。同样的，在默认情况下声呐传感器也是连接到 CAN 总线上的，我们需要一个 USB-CAN 的转换装置（CAN 卡）来将声呐的信息传输到 USB 设备上。此外还需要一个电源来为声呐传感器供电。一旦声呐传感器被连接到 CAN 卡上，CAN 卡就可以将 CAN 信息转化为 USB 串口信息发送到计算机上，并开始读取探测数据。这部分难度不大，基本上在 5 分钟以内可以顺利配置完成。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.010.jpeg)

图 4.10 硬件组成，其中包含有供电电源(Power supply)，USB 接口(USB interface)，CAN 卡(CAN card)，声呐控制器(sonar controller)，以及声呐探测头(sonar detector)

图 4.11 向读者显示了声呐传感器的用户界面(UI)，用户可以通过该界面查看车辆前方探测到的障碍物。该传感器配备了两个声呐探测头，可以将车辆前方的障碍物准确地探测出来。声呐传感器会将探测到的物体以鸟瞰图的形式投影到用户界面中。在界面上清晰地向用户显示了车辆与探测到的障碍物之间的距离。图 4.12 展示了该设备的硬件规格[5]。

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.011.jpeg)

图 4.11 声呐传感器的用户界面(UI)

![图片](Aspose.Words.49793cf6-d3c6-4362-98d7-e0e1eefc9ddf.012.jpeg)

图 4.12 声呐传感器规格

这里我们来具体看一下图 4.12 的规格参数，主要参数有探头光束角度(Probe Beam Angle)，当中含有两个参数：水平方向上的视场角(Horizontal Direction)、垂直方向上的视场角(Vertical Direction)，其他则是超声波频率(Ultrasonic Frequency)、防水等级(Waterproof Leve)、有效检测距离(Detection Distance)、传感器盲区(Blind Zone)、工作电压(Operating Voltage)、通信方式(Communication Type)。

下面的这个代码片段向读者展示了声呐传感器简单的数据结构，其中包含两个探测距离(一个用于左探测单元，另一个用于右探测单元)。同时为了更加方便的调用雷达传感器，作者还提供了一个非常简单的软件应用程序接口（API）来捕捉声呐数据，通过直接调用软件应用程序接口（API）可以使读者快速建立自己的被动感知逻辑。

struct USR\_Data {

unsigned short left\_front; // object range detected by "FB" channel,

unit in millimeter.

unsigned short right\_front; // object range detected by "FC" channel,

unit in millimeter.

};

USR sonar;

USR\_Data usr\_data;

sonar.Read(usr\_data);

**参考文献**

1. Hasch, J., Topak, E., Schnabel, R. et al. (2012). Millimeter‐wave technology for automotive radar sensors in the 77 GHz frequency band. IEEE Transactions on Microwave Theory and Techniques 60 (3): 845–860. 
1. Iovescu, C. and Rao, S. (2017). The fundamentals of millimeter wave sensors. <http://www.> ti.com/lit/wp/spyy005/spyy005.pdf (accessed 1 February 2019). 
1. Hymans, A.J. and Lait, J. (1960). Analysis of a frequency‐modulated continuous‐wave ranging system. Proceedings of the IEE‐Part B: Electronic and Communication Engineering 107 (34): 365–372. 
1. YouTube (2018). DragonFly 77 GHz millimeter Wave Radar. <https://www.youtube.com/> watch?v=ZLOgcc7GUiQ (accessed 1 September 2019). 
1. PerceptIn (2018). Products. <https://www.perceptin.io/products> (accessed 1 September 2019). 
1. YouTube (2018). DragonFly Sonar. <https://www.youtube.com/watch?v=> ‐H3YdC‐xSgQ (accessed 1 September 2019).


