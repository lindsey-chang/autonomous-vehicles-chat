**9.1 简介**

在这一章中，我们对 PerceptIn (普思英察)公司的 DragonFly 系列驾驶产品（包含DragonFly Pod 双座车以及 DragonFly Bus 公交车）进行了全面的案例分析。这两个产品正是采用本书[1,2]章中所介绍的模块化设计方法开发的。在本书[3, 4]章中，还可以找到一些关于自动驾驶车辆的视频演示。

双座的 DragonFly Pod（蜻蜓双座车） 是为私人自动驾驶运输服务而开发的，这样乘客就能够在旅途中享受隐私保护。DragonFly Pod 双座车的典型用途包括娱乐公园、工业园区、旅游景点和老年生活协会。八座的 DragonFly Bus（蜻蜓公交车）则是为距离通常小于5英里的公共自动驾驶交通解决方案而设计的。DragonFly Bus 公交车的典型用途包括大学校园、城市公交路线、机场和火车站的站内交通。

首先，我们介绍了这两种车辆的底盘规格，以便读者了解他们在物理结构上的差异。其次，我们揭秘了这两种底盘上的传感器配置，让读者能够了解感知和定位模块是如何部署的。第三，我们介绍了 DragonFly 系统的“解剖学”，即能够让这些车辆实现自动驾驶的软件架构。这能够让读者理解不同的模块是如何一起工作，进而形成一个系统。第四，我们介绍DragonFly系统的 "生理学"，即自动驾驶车辆的工作机制，让读者能够理解自动驾驶软件的生命周期。第五，通过便览不同模块之间进行通信所使用的数据结构，读者能够了解不同模块之间如何进行交互。最后，我们展示了用户如何通过一个简单的用户界面（UI）与自动驾驶汽车互动。

读完本章后，读者应该对如何从头开始搭建自己的自动驾驶车辆或机器人有一个基本的了解。

**9.2 底盘硬件规格** 

图 9.1 展示了一辆 DragonFly Pod 双座车，表 9.1 列出了底盘规格的详细信息。一辆 DragonFly Pod 双座车长2.18米，宽1.38米，高1.675米，重580公斤，最高行驶速度为每小时25公里。然而出于安全考虑，在自动驾驶模式下，我们通常将车速保持在每小时10公里左右。它的电池能够支撑车辆在正常速度（每小时10公里）下连续工作10小时。DragonFly Pod 双座车的底盘支持线控驱动，因此规划和控制模块可以通过发送控制命令来操纵车辆。此外，底盘还能够提供实时车辆状态信息，如角速度、线速度、制动压力等。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.001.png)

图 9.1 DragonFly Pod 双座车

||DragonFly Pod 双座车|
| :- | :- |
|悬架（前/后）|<p>麦弗逊式独立前悬架：螺旋弹簧+油缸液压减震</p><p>一体式后桥，速比12.49:1，螺旋弹簧阻尼+油缸液压减震</p>|
|车架材料|载体框架结构|
|驱动方式|前轮转向|
|制动器（前/后）|前盘式后轮毂，双管双回路液压制动器|
|停车系统|电动驻车制动器停车和手刹停车|
|控制方式|CAN总线|
|长度/宽度/高度|2180/1380/1675毫米|
|轴距|背面1190毫米，前面1170毫米|
|离地间隙|145毫米|
|最小转弯半径|≤4.4米|
|爬坡能力|15%（≥8.5°）|
|轮胎|155 65/R13 13|
|总重量|580公斤|
|最高速度|25公里每小时|
|电池类型|铅酸电池|
|电池容量|120安时|
|电池规格|12伏特/120安时|
|电池数量|4|
|充电器类型|外部充电器|
|充电电压|220伏特|
|车辆输出功率|12伏特，700瓦|
|城市范围 urban range？|100公里|
|充电时长|10小时，80%|
|通信方式|CAN 协议|
|底盘信息|角速度，线速度，制动压力，电压，电流，功率|
|座位数|2|
|驱动类型|中心电机|

表 9.1 DragonFly Pod 双座车 底盘规格

图 9.2 展示了一辆 DragonFly Bus 公交车，表 9.2 列出了底盘规格的细节。一辆 DragonFly Bus 公交车长3.93米，宽1.51米，高2.04米，重910公斤，最高时速为每小时30公里。在自动驾驶模式下，出于安全原因，我们通常将速度保持在每小时10公里左右。它的电池允许车辆以正常速度（每小时10公里）连续运行8小时。DragonFly Bus 公交车的底盘是支持线控驱动的，因此规划和控制模块可以通过发送控制命令来操纵车辆。此外，底盘还能够提供实时车辆状态信息，如角速度、线速度、制动压力等。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.002.png)

图9.2 DragonFly Bus 八座公交车


||DragonFly Bus 公交车|
| :- | :- |
|控制器|InBol 电控/AC MC3336|
|电池|Lvtong 免维护蓄电池 6V/170Ah \* 8 only(3h rate)|
|电动机|Lvtong 专用交流异步电动机 33V/5kW|
|充电器|智能充电器 48V/25A，充电时间小于10小时（放电率80%）|
|直流转换器|大功率隔离直流转换器 48V/12V, 400W|
|照明和报警系统|前灯，转向信号灯，雾灯，倒车灯，后方尾灯，蜗牛喇叭，反向语音喇叭|
|转向系统|双向齿轮齿条转向系统，自动间隙补偿功能：可选电源|
|制动系统|前盘后轮毂四轮液压制动器+手刹停车：可选配电动真空辅助制动系统|
|前悬架系统|麦弗逊式独立前悬架：螺旋弹簧+油缸液压阻尼|
|后悬架系统|一体式后桥，速比16:1，钢板弹簧+油缸液压减震|
|轮胎直径|165/70r13c 真空轮胎 （直径 560 毫米）： 13 钢环|
|长度/宽度/高度|3930/510/2040 mm|
|座位数|8|
|最高速度|30公里每小时|
|行驶里程公里数|75-95km（平坦道路）|
|每百公里能耗|9 kWh|
|最大灰度|0\.15|
|在斜坡上的性能|0\.2|
|最小转弯半径|5\.6m|
|整车重量|910kg|
|制动稳定性|1900 mm|
|最小离地间隙|155 mm|
|轮距|后轮 1330 mm, 前轮 1300 mm|
|通信方式|CAN 协议|

表 9.2 DragonFly Bus 公交车底盘规格

**9.3 传感器配置** 

在明白了底盘的功能之后，我们就需要弄清楚在自动驾驶车辆上如何部署传感器来进行感知和定位。图 9.3 展示了各类传感器在 DragonFly Pod 双座车上是怎么部署的。如图所示，我们主要使用四种类型的传感器，分别用不同颜色的图例进行表示：DragonFly 视觉模块（橙色圆角矩形），GPS接收器（深绿色大圆形），雷达（红色小矩形）和声纳（浅绿色小圆点）。另外，图中长度标注的单位为厘米。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.003.png)

图9.3 DragonFly Pod 双座车传感器配置

DragonFly 视觉模块可以用于定位和主动感知。它安装在车辆顶部的中心位置。这能够让车辆获得一个开放的视野，从而对周围的空间特征进行捕获。除此之外，将 DragonFly 视觉模块放置在车辆的中心，还有利于进行空间传感器校准。只需要将不同传感器的坐标与 DragonFly 视觉模块对齐，就能够很轻松地实现空间传感器校准。

我们沿着水平轴安排了两个 GPS 接收器。这两个GPS接收器形成一个差分对。这个差分对不仅能够提供精确的车辆实时位置，还可以提供准确的实时航向。这两个差分GPS接收器的中心也正好是 DragonFly 视觉模块的中心，从而简化了空间校准的过程。

在车辆的周围，我们还部署了六个雷达和八个声纳，以期能够最大化感知检测的覆盖范围。通过这种方式，雷达和声纳可以用来实现双层保护的被动感知。这种“双层”保护的被动感知主要体现在中近距离传感器的组合使用上：适用于中等距离（mid-range）的雷达，以及适用于近距离（close-range）的声纳。此外，雷达可以提供目标的距离信息，目标的速度信息，以及目标跟踪功能。这些信息与功能可以与视觉感知进行融合，用来实现更准确的主动感知。

图 9.4 展示了 DragonFly Bus 公交车上的传感器部署方式。图中所展示的四种传感器同样使用不同的图例进行标注：DragonFly 视觉模块（橙色圆角矩形），GPS接收器（深绿色大圆形），雷达（红色小矩形）和声纳（浅绿色小圆点）。图中长度标注的单位为厘米。整体上DragonFly Bus 公交车上的传感器部署方式与 DragonFly Pod 双座车的部署非常相似，除了以下这些要求：

● 考察 DragonFly Bus 公交车的长度，我们不再使用六个雷达和八个声纳，而是部署了八个雷达和八个声纳。

● 同样考虑到 DragonFly Bus 公交车的长度，DragonFly 视觉模块放置在车辆的前部而不再是车辆的中心。这样能够让车辆获得畅通无阻的视野，一览无余。

● 还是由于车辆的长度原因，差分GPS接收器是部署在沿车身的垂直方向而非水平方向。这样设计的目的是为了充分利用车辆的长度，从而获得更准确的实时航向。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.004.png)

图9.4 DragonFly Bus 八座公交车传感器配置

**9.4 软件架构** 

在了解了传感器的部署方案之后，我们就可以对 DrgonFly 系统的软件栈进行深入研究了。接下来，我们将介绍DragonFly系统的“解剖结构”，即软件架构。图 9.5展示了 DragonFly 系统软件架构的详细信息。需要注意的是，每个虚线框都代表一个独立的进程。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.005.png)

图9.5 DragonFly 系统软件架构

【译者注】如图中虚线框所示，图中共有6个独立进程，从上到下、从左到右依次为：GPS守护进程（GPS Daemon process）、图像与惯性测量单元调度进程（Image and IMU dispatcher process ）、用户界面前端进程（UI frontend process）、感知进程（Perception process）、定位进程（Localization process）、规划和控制进程（Planning and control process）。

接下来，我们先简要介绍上述图中每个独立进程分别包含哪些模块，便于读者对后续的系统软件工作流程进行理解。

1. GPS守护进程（GPS Daemon process），包含GPS Daemon（GPSD）模块，主要功能是获取GPS相关数据并发送到相应接口。
1. ` `图像与惯性测量单元调度进程（Image and IMU dispatcher process ），包含图像与惯性测量单元调度模块（Image and IMU dispatcher ）。主要功能是获取图像数据和IMU（inertial measurement unit - 惯性测量单元）数据，并发送到相应的数据接口。
1. 定位进程（Localization process）包含两个模块，分别是“图像、IMU、GPS接口（Image, IMU, GPS Interface）”和“普思英察-视觉惯性里程计（PI-VIO）”。这两个模块共同作用能够让系统得到准确的实时定位结果。
1. 感知进程（Perception process）则包含下列模块：立体视觉接口（Stereo Image Interface），深度学习模块（Deep Learning），立体匹配模块（Stereo Matching），视觉融合模块（Vision Fusing），视觉雷达融合模块（Vision Radar Fusion）。
1. 规划和控制进程（Planning and control process）包含众多模块，分别是：雷达和声纳 CAN 总线接口（Radar and Sonar CAN Bus Interface），障碍获取接口（Get Obstacle Interface），障碍决策模块（Obstacle Decision），姿态获取接口（Get Pose Interface），本地规划器（Local Planner），控制命令和底盘接口（Control Command and Chassis Interface），监控模块（Monitor），本地日志记录模块（Local Logging），用户界面后端模块（UI Backend）。
1. 用户界面前端进程（UI frontend process），包含用户界面前端（UI Frontend）模块，主要功能是让用户能够看到来自用户界面后端的各种状态信息，了解车辆状态。

我们首先回顾一下数据收集过程，这一阶段主要涉及到GPS守护进程（GPS Daemon process）和图像和惯性测量单元调度进程（Image and IMU dispatcher process ）。第一个进程是GPS守护进程（GPS Daemon process）。它会持续不断地去获取最新的 GNSS（ Global Navigation Satellite System - 全球导航卫星系统 ）数据并将数据发送到定位进程（Localization process）。类似地，图像和惯性测量单元调度进程（Image and IMU dispatcher process ）也会持续不断地去获取最新的图像和IMU数据，并将数据发送到定位进程和感知进程（Perception process）。

接下来，定位进程（Localization process）会接收图像、IMU 和 GNSS 数据并生成实时定位结果。需要注意的是，在我们的设计当中，是把 GNSS 数据当作地面真值的。但是，如果出现多路径问题或者一些其他问题，这些问题将导致 GNSS 数据不可用或者不准确。此时，视觉惯性里程计（Visual-Inertial Odometry (VIO) ）将接替GNSS生成准确的定位结果。

感知进程（Perception process）接收图像数据，以及来自底盘的雷达和声纳数据，然后结合这些信息生成实时感知结果。当接收到立体视觉（Steoro Image）数据时，感知进程利用深度学习（Deep Learning）模型来提取障碍物语义信息，并应用立体匹配（Stereo Mataching）技术生成障碍物深度信息。通过结合障碍物的语义与深度信息，感知进程可以准确地辨别障碍物的类型和距离信息。此外，雷达能够提供障碍物的速度信息，通过融合雷达和视觉结果，感知进程就能够得到障碍物的类型、距离和速度信息。

规划和控制进程（Planning and control process）是最为重要的进程，它就像是DragonFly自动驾驶系统的“大脑”一样。它消耗并利用来自感知进程（Perception process）和定位进程（Localization process）的输出，通过局部规划器（Local Planner）模块生成实时控制命令，并且将这些命令发送给底盘来执行。同时，监控（Monitor）模块会持续检测整个系统的健康状态，一旦任何模块出现故障，它都会让车辆停止。此外，用户界面后端（UI backend）也会不断向用户界面前端进程（UI frontend process）发送状态数据，从而让驾驶员/操作员能够持续了解系统的实时状态。同样，本地日志记录（Local logging）模块也会连续地记录系统信息，将其保存到本地磁盘或者远程云端当中，以便于进行调试。

**9.5系统机制**

理解DragonFly系统的“解剖结构”之后，这一节我们将深入研究DragonFly“生理学”，即系统的机制。图 9.6 说明了初始化DragonFly 系统的详细步骤。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.006.png)

图9.6 DragonFly 系统机制

【译者注】图9.6中左侧流程图，所涉及到的状态和动作描述如下：用户开机命令（User Power on Command），启动图像和惯性测量单元进程（Start Image and IMU Process），成功（Success），终止（Abort），启动感知进程（Start Perception Process），启动定位进程(Start Localization Process)， CAN 总线检验（CAN Bus Check），等待用户命令（Wait User Command）。

如图 9.6 的左侧所示，一旦用户让系统开机，系统首先在 DragonFly 视觉模块上启动图像和 IMU 采集流。如果启动不成功，那么系统中止并停止执行。否则，系统启动感知进程。接下来，系统启动定位进程。一旦这些都成功，系统会检查CAN （Control Area Network - 控制器局域网 ）总线，来确保底盘的通信是可接受的并且底盘的状态是合适的。当所有检查都通过后，系统会持续运转以等待用户命令。

【译者注】图9.6中右侧流程图，所涉及到的状态和动作描述如下：启动任务命令（Start Task Command），检查感知、定位、控制的运行时错误代码（Check Perception, Localization, Control Run Time Error Code,）成功（Success），停止（Stop），生成控制命令（Generate Control Command），站点（Station），终点（Terminal），恢复命令（Resume Command），任务完成（Task Complete）

如图 9.6 右侧所示，一旦启动了某个用户任务（例如，命令车辆从 A 点移动到 B 点），规划和控制模块就会进行检查，从而确保感知模块、定位模块和机箱都是正常运转的，并且车辆整体不会发出任何错误代码。如果所有检查返回的信息都是成功的，那么规划和控制模块会不断地发出控制命令，让车辆沿着指定路径行驶。如果车辆到达了某个车站，规划和控制模块会再次检查系统的健康状况，然后持续给车辆命令，让它继续沿着指定路径行驶。如果车辆到达了目的地，那么计划和控制进程会被终止，然后等待下一个任务命令。

**9.6 数据结构** 

理解了DragonFly系统的“解剖学”和“生理学”之后，在本节中，我们将介绍整个系统中使用的数据结构。这样，读者可以了解每个模块分别提供哪些信息，以及不同模块是如何相互作用的。需要注意的是，所有数据结构都使用Protobuf 方法（Protocol Buffers - 协议缓冲区 ） 进行编码，这是一种对结构化数据进行串行化的方法。Protobuf 方法在开发用于相互通信的程序中有着非常广泛的应用。Protobuf方法包含了一种接口性的描述语言，这种语言能够对某些数据的数据结构进行描述。Protobuf 方法还提供了能够将描述语言生成源代码的程序。该源代码主要用来生成或解析能够表示结构化数据的字节流。

**9.6.1常用数据结构** 

首先，我们要介绍的是在所有模块之间共享的通用数据结构。 Common.proto 文件中定义了在所有模块中共同使用的标志头。其中包括时间戳 timestamp，这个标志头的作用是同步所有数据的系统时间戳; 模块名称 module\_name，当前模块的名称; 序列号 sequence\_num，用于跟进和记录自系统启动以来发送的数据次数的计数器; 硬件时间戳 hardware\_timestamp，不同于系统时间戳，这是来自传感器的时间戳。通过同时跟踪传感器时间戳和系统时间戳，我们能够持续记录系统和传感器之间的时差。

Common.proto

syntax = "proto3";

package piauto.common;

message Header {

`  `// message publishing time in milliseconds since 1970.

`  `// 消息发布时间，用自1970以来的毫秒数表示

`  `uint64 timestamp = 1;



`  `// module name

`  `// 模块名称

`  `string module\_name = 2;



`  `// sequence number for each message: each module maintains its

`  `own counter for

`  `// sequence\_num, always starting from 1 on boot.

`  `// 每条消息的序列号：每一个模块维护各自的序列号计数器，计数器总是从1开始

`  `uint32 sequence\_num = 3;



`  `// hardware sensor timestamp in milliseconds since 1970

`  `// 传感器硬件时间戳，用自1970以来的毫秒数表示

`  `uint64 hardware\_timestamp = 4;

}

Geometry.proto 文件存储的数据结构包含了自动驾驶汽车在感知以及规划控制模块中经常用到的各种几何图形与概念。这些几何图形包括二维点（2D points）、三维点（3D points）、速度（velocity）和多边形（polygons）。一般来说，多边形通常被视为三维点的集合。

Geometry.proto

syntax = "proto3";

package piauto.common;

// A general 3D point, in meter

// 一般三维点，单位为米

message Point3D {

`  `double x = 1;

`  `double y = 2;

`  `double z = 3;

}

// A general 2D point, in meter

// 一般二维点，单位为米

message Point2D {

`  `double x = 1;

`  `double y = 2;

}

// General speed, in m/s

// 一般速度，单位为米每秒

message Velocity3D {

`  `double vel\_x = 1;

`  `double vel\_y = 2;

`  `double vel\_z = 3;

}

// A general polygon, points are counter clockwise

// 一般多边形，多边形顶点为逆时针方向计数

message Polygon {

`  `repeated Point3D point = 1;

}

**9.6.2底盘数据**

Chassis.proto 文件用来表示自动驾驶汽车中由底盘生成的数据格式。这些数据片段会被不断发送到规划控制模块。规划和控制模块可以将这些底盘数据与来自感知和定位模块的数据相结合，进而生成实时的控制命令。这些底盘数据也会被发送到UI模块，用来向乘客和驾驶员告知车辆的当前状态。在下面的底盘数据 protobuf 文件中，我们向读者展示了一些相对更常用的底盘数据包括错误代码（error code）、车速（vehicle speed）、车辆里程表（vehicle odometer）、燃油范围（fuel range）、转向角度（steering angle）和转向速度（steering velocity）。

Chassis.proto

syntax = "proto3";

package piauto.chassis;

import "header.proto";

// next id :31

message Chassis{

`    `enum DrivingMode{

`        `COMPLETE\_MANUAL = 0;     // manual mode 手动模式

`        `COMPLETE\_AUTO\_DRIVE = 1; // auto mode   自动模块

`        `AUTO\_STEER\_ONLY = 2;     // only steer  只使用转向

`        `AUTO\_SPEED\_ONLY = 3;     // include throttle and brake //包括油门和制动器



`        `// security mode when manual intervention happens, only response status

`        `// 发生手动干预时的安全模式，仅响应状态

`        `EMERGENCY\_MODE = 4;

`        `MANUAL\_INTERVENTION = 5; // human manual intervention 手动干预模式

`    `}

`    `enum ErrorCode{

`        `NO\_ERROR = 0;

`        `CMD\_NOT\_IN\_PERIOD = 1; // control cmd not in period 控制命令不在周期内

`        `// receive car chassis can frame not in period

`        `// 不在周期内接收车辆CAN帧



`        `CHASSIS\_CAN\_NOT\_IN\_PERIOD = 2;

`        `// car chassis report error, like steer, brake, throttle,gear fault

`        `//车辆底盘报错，例如转向、刹车、油门、齿轮故障

`        `CHASSIS\_ERROR = 3;

`        `// classify the types of the car chassis errors

`        `//对车辆底盘错误进行分类

`        `CHASSIS\_ERROR\_ON\_PARK = 4;

`        `CHASSIS\_ERROR\_ON\_LIGHT = 5;

`        `CHASSIS\_ERROR\_ON\_STEER = 6;

`        `CHASSIS\_ERROR\_ON\_BRAKE = 7;

`        `CHASSIS\_ERROR\_ON\_THROTTLE = 8;

`        `CHASSIS\_ERROR\_ON\_GEAR = 9;

`        `UNKNOWN\_ERROR = 10;

`    `}

`    `enum GearPosition{

`        `GEAR\_NEUTRAL = 0;

`        `GEAR\_DRIVE = 1;

`        `GEAR\_REVERSE = 2;

`        `GEAR\_PARKING = 3;

`        `GEAR\_LOW = 4;

`        `GEAR\_INVALID = 5;

`        `GEAR\_NONE = 6;

`    `}

`    `common.Header header = 1;

`    `bool engine\_started = 3;

`    `// engine speed in RPM. 

`    `// 用 RPM-每分钟转数 表示的引擎速度

`    `float engine\_rpm = 4;

`    `// vehicle speed in meters per second. 

`    `// 用米每秒表示的车辆速度

`    `float speed\_mps = 5;

`    `// vehicle odometer in meters.

`    `// 单位为米的车辆里程

`    `float odometer\_m = 6;

`    `// fuel range in meters.

`    `// 单位为米的燃油航程

`    `int32 fuel\_range\_m = 7;

`    `// real throttle location in [%], ranging from 0 to 100.

`    `// 以百分比表示的油门实际位置，范围从 0 到 100

`    `float throttle\_percentage = 8;

`    `// real brake location in [%], ranging from 0 to 100.

`    `// 以百分比表示的刹车实际位置，范围从 0 到 100

`    `float brake\_percentage = 9;

`    `// real steering location in degree, ranging from about -30 to 30.

`    `// clockwise: negative

`    `// counter clockwise: positive

`    `// 用度数表示的方向位置，范围从 -30 到 0

`    `// 顺时针：负数表示

`    `// 逆时针：正数表示

`    `float steering\_angle = 11;

`    `// applied steering velocity in [degree/second].

`    `// 应用转向速度，用度每秒表示

`    `float steering\_velocity = 12;

`    `// parking brake status.

`    `// 停车刹车状态

`    `bool parking\_state = 13;

`    `// battery voltage

`    `// 电池电压

`    `float battery\_voltage = 14;

`    `// battery power in [%], ranging from 0 to 100.

`    `// 以百分比表示的电池电量，范围从 0 到 100

`    `float battery\_power = 15;

`    `// signals.

`    `// 信号

`    `bool high\_beam\_signal = 16;

`    `bool low\_beam\_signal = 17;

`    `bool left\_turn\_signal = 18;

`    `bool right\_turn\_signal = 19;

`    `bool flash\_signal = 20;

`    `bool horn = 21;

`    `bool wiper = 22;

`    `bool disengage\_status = 23;

`    `DrivingMode driving\_mode = 24;

`    `ErrorCode error\_code = 25;

`    `GearPosition gear\_location = 26;

`    `// timestamp for steering module

`    `// 转向模块的时间戳

`    `double steering\_timestamp = 27; // In seconds, with 1e-6 accuracy 以秒为单位，精度为 1e-6

`    `WheelSpeed wheel\_speed = 30;

}

message WheelSpeed{

`    `enum WheelSpeedType{

`        `FORWARD = 0;

`        `BACKWARD = 1;

`        `STANDSTILL = 2;

`        `INVALID = 3;

`    `}

`    `bool is\_wheel\_spd\_rr\_valid = 1;

`    `WheelSpeedType wheel\_direction\_rr = 2;

`    `double wheel\_spd\_rr = 3;

`    `bool is\_wheel\_spd\_rl\_valid = 4;

`    `WheelSpeedType wheel\_direction\_rl = 5;

`    `double wheel\_spd\_rl = 6;

`    `bool is\_wheel\_spd\_fr\_valid = 7;

`    `WheelSpeedType wheel\_direction\_fr = 8;

`    `double wheel\_spd\_fr = 9;

`    `bool is\_wheel\_spd\_fl\_valid = 10;

`    `WheelSpeedType wheel\_direction\_fl = 11;

`    `double wheel\_spd\_fl = 12;

}

message License

{

`    `string vin = 1;

}

**9.6.3定位数据** 

Localization.proto文件格式是用来存储定位数据的，这些数据被不断地发送到规划和控制模块进行处理。在protobuf文件中，关键数据包括使用通用横轴墨卡托投影（Universal Transverse Mercator）格式表示的车辆位置（utm\_x，utm\_y）以及车辆航向。每条定位数据都包含关于当前定位数据的的详细信息。这些信息表明了当前定位数据是仅由 GPS 生成的还是由 GPS 和其他技术（例如 VIO）融合（fusion）生成的。

Localization.proto

syntax="proto3";

package piauto.localization;

import "header.proto";

enum LocalizationStatus{

`  `GPS = 0; // postion:GPS heading:GPS 位置：GPS 航向：GPS

`  `BOTH\_FUSION = 1; // postion:FUSION heading:FUSION 位置：融合 航向：融合

`  `GPS\_FUSION = 2; // postion:GPS heading:FUSION 位置：GPS 航向：融合

`  `FUSION\_GPS = 3; // postion:FUSION heading:GPS 位置：融合 航向：GPS

`  `INIT=4;

`  `ERROR=5;

}

enum FusionType{

`  `ORIGIN\_GPS = 0;

`  `FUSION = 1;

}

enum GPSStatus{

`  `FLOAT = 0;

`  `FIXED = 1;

}

enum ErrorType{

`  `// sensor failed

`  `IMAGE\_OPEN\_FAILED=0;

`  `IMU\_OPEN\_FAILED=1;

`  `GPS\_OPEN\_FAILED=2;



`  `// init failed

`  `GRAVITY\_INIT\_FAILED=11;

`  `GPS\_INIT\_FAILED=12;

`  `GPS\_INVALID\_DATA=13;



`  `// run error

`  `CONNECT\_FAILED=20;

}

message LocalizationData{

`  `// header

`  `common.Header header = 1;



`  `// position

`  `double utm\_x = 2;

`  `double utm\_y = 3;

`  `double utm\_x\_variance = 4;

`  `double utm\_y\_variance = 5;

`  `sint32 utm\_zone = 6;

`  `FusionType position\_type = 7;

`  `GPSStatus gps\_position\_status = 8;



`  `// heading

`  `double heading = 9;

`  `double heading\_variance = 10;

`  `FusionType heading\_mode = 11;

`  `GPSStatus gps\_heading\_status = 12;



`  `// system status

`  `LocalizationStatus localization\_status = 13;

`  `ErrorType error\_code = 14;

}

**9.6.4感知数据** 

Perception.proto 文件存储感知数据，这些数据被不断地发送到规划和控制模块来进行处理。在protobuf文件中关键的数据包括对象位置、对象速度和对象类型。每条感知数据都包含有关当前感知数据的详细信息，这些信息表明当前感知数据是由视觉（vision）、声纳（sonar）、雷达（radar）还是它们的融合（fusion）生成的。

Perception.proto

syntax = "proto3";

package piauto.perception;

import "geometry.proto";

import "header.proto";

message PerceptionObstacle{

`    `// timestamp

`    `// 时间戳

`    `common.Header header = 1;

`    `// we assume the basic sensors include radar, sonar, and stereo\_camera 

`    `// 我们假设基本传感器包括雷达，声纳和立体摄像机

`    `enum SensorType {

`        `UNKNOWN\_SENSOR = 0;

`        `RADAR = 1;

`        `VISION = 2;

`        `ULTRASONIC = 3;

`        `FUSION = 4;

`    `};

`    `SensorType sensor\_type = 2;

`    `// identify different sonar and radar

`    `// 标识不同的声纳和雷纳

`    `int32 sensor\_id = 3;

`    `// each obstacle has an unique id

`    `// 每个障碍物都有单独唯一的id

`    `int32 obstacle\_id = 4;

`    `common.Point3D position = 5;

`    `common.Velocity3D velocity = 6;

`    `// obstacle semantic type

`    `// 障碍物的语义类型

`    `enum ObstacleType{

`        `UNKNOWN\_OBSTACLE = 0;

`        `UNKNOWN\_MOVABLE = 1;

`        `UNKNOWN\_UNMOVABLE = 2;

`        `CAR = 3;

`        `VAN = 4;

`        `TRUCK = 5;

`        `BUS = 6;

`        `CYCLIST = 7;

`        `MOTORCYCLIST = 8;

`        `TRICYCLIST = 9;

`        `PEDESTRIAN = 10;

`        `TRAFFIC\_CONE = 11;

`        `TRAFFIC\_LIGHT = 12;

`    `};

`    `ObstacleType obstacle\_type = 7;

`    `// confidence level regarding the detection result

`    `// 检测结果的置信水平

`    `double confidence = 8;

`    `enum ConfidenceType{

`        `CONFIDENCE\_UNKNOWN = 0;

`        `CONFIDENCE\_CNN = 1;

`        `CONFIDENCE\_STEREO = 2;

`        `CONFIDENCE\_RADAR = 3;

`    `};

`    `ConfidenceType confidence\_type = 9;

`    `repeated common.Polygon polygons = 10;

`    `// traffic light detection result

`    `// 交通信号灯的检测结果

`    `enum TrafficLightColor{

`        `UNKNOWN = 0;

`        `RED = 1;

`        `YELLOW = 2;

`        `GREEN = 3;

`        `BLACK = 4;

`    `};

`    `TrafficLightColor traffic\_light\_color = 11;

`    `// historical points on the trajectory path

`    `// 轨迹路径上的历史点

`    `message TrajectoryPathPoint{

`        `common.Point3D path\_point = 1;

`        `// in milliseconds since 1970

`        `// 自1970年来的毫秒数

`        `uint64 timestamp = 2;

`        `// in millisecond by hardware sensor

`        `// 来自硬件传感器，用毫秒数表示

`        `uint64 hardware\_timestamp = 3;

`    `}

`    `repeated TrajectoryPathPoint trajectory\_points = 12;

`    `// confidence level of the trajectory prediction

`    `// 轨迹预测的置信水平

`    `double trajectory\_probability = 13;

`    `enum IntentType{

`        `UNKNOWN\_INTENT = 0;

`        `STOP = 1;

`        `STATIONARY = 2;

`        `MOVING = 3;

`        `CHANGE\_LANE = 4;

`        `LOW\_ACCELERATION = 5;

`        `HIGH\_ACCELERATION = 6;

`        `LOW\_DECELERATION = 7;

`        `HIGH\_DECELERATION = 8;

`    `}

`    `// estimated obstacle intent

`    `// 估计出的障碍物的意图

`    `IntentType intent\_type = 14;

`    `enum ErrorCode{

`        `OK = 0;

`        `IMAGE\_TIMEOUT\_ERROR = -1;

`    `}

`    `ErrorCode error\_code = 15;

}

message PerceptionObstacles{

`    `repeated PerceptionObstacle perception\_obstacle = 1;

}

**9.6.5规划数据** 

规划与控制模块使用来自感知模块、定位模块和底盘模块的数据，并生成实时控制命令。Decision.proto文件定义了车辆的行为，包括跟随前方车辆，给另一辆车让位，停止当前车辆以及避开障碍物等基础行为。此外当前车辆的停止原因同样也会在在Decision.proto中给出定义。

Decision.proto

syntax = "proto3";

package piauto.plannning;

import "geometry.proto";

message EStop{

`    `// is\_estop is true when emergency stop is required

`    `// 当需要紧急停止的时候， is\_estop 为真

`    `bool is\_estop = 1;

`    `string reason = 2;

}

message MainEmergencyStop

{

`    `// unexpected event happened, human driver is required to take over

`    `// 发生意外事件，需要人类驾驶员进行接管

`    `enum ReasonCode

`    `{

`        `ESTOP\_REASON\_INTERNAL\_ERR = 0;

`        `ESTOP\_REASON\_COLLISION = 1;

`        `ESTOP\_REASON\_SENSOR\_ERROR = 2;

`    `}

`    `ReasonCode reason\_code = 1;

}

enum StopReasonCode{

`    `STOP\_REASON\_HEAD\_VEHICLE = 0;

`    `STOP\_REASON\_DESTINATION = 1;

`    `STOP\_REASON\_PEDESTRIAN = 2;

`    `STOP\_REASON\_OBSTACLE = 3;

`    `STOP\_REASON\_PREPARKING = 4;

`    `STOP\_REASON\_SIGNAL = 5; // only for red light 只适用于红灯

`    `STOP\_REASON\_STOP\_SIGN = 6;

`    `STOP\_REASON\_YIELD\_SIGN = 7;

`    `STOP\_REASON\_CLEAR\_ZONE = 8;

`    `STOP\_REASON\_CROSSWALK = 9;

`    `STOP\_REASON\_CREEPER = 10;

`    `STOP\_REASON\_REFERENCE\_END = 11; // end of the reference line 参照线的末尾

`    `STOP\_REASON\_YELLOW\_SIGNAL = 12; // yellow light 黄灯

`    `STOP\_REASON\_LANE\_CHANGE\_URGENCY = 13;

}

message MainStop{

`    `StopReasonCode reason\_code = 1;

`    `string reason = 2;

`    `// when stopped, the front center of vehicle should be at this point.

`    `// 当车辆停止时，车辆的前部中心应该在这个位置。

`    `common.Point3D stop\_point = 3;

`    `// when stopped, the heading of the vehicle should be stop\_heading.

`    `// 当车辆停止时，车辆的航向应位于 stop\_heading。 

`    `double stop\_heading = 4;

}

// strategy to ignore objects

// 忽略对象时的策略

message ObjectIgnore{

`    `string ignore\_strategy = 1;

`    `double distance\_s = 2; // in meters 以米为单位

}

message ObjectStop{

`    `StopReasonCode reason\_code = 1;

`    `double distance\_s = 2; // in meters 以米为单位

`    `// when stopped, the front center of vehicle should be at this point.

`    `// 当车辆停止时，车辆的前部中心应该在这个位置。

`    `common.Point3D stop\_point = 3;

`    `// when stopped, the heading of the vehicle should be stop\_heading.

`    `// 当车辆停止时，车辆的航向应位于 stop\_heading。 

`    `double stop\_heading = 4;

`    `repeated string wait\_for\_obstacle = 5;

}

// strategy to follow objects

// 跟随对象时的策略

message ObjectFollow{

`    `string follow\_strategy = 1;

`    `double distance\_s = 2; // in meters 以米为单位

}

// strategy to yield objects

// 给对象让位的策略

message ObjectYield{

`    `string yield\_strategy = 1;

`    `double distance\_s = 2; // in meters 以米为单位

}

// strategy to avoidance objects, such as double-lane changing or floating - lane

// 避让对象的策略，例如由双车道变换或浮动车道

// avoidance

// 避让

message ObjectAvoid{

`    `string avoid\_strategy = 1;

`    `double distance\_s = 2; // in meters 以米为单位

}

message ObjectDecisionType

{

`    `oneof object\_tag

`    `{

`        `ObjectIgnore ignore = 1;

`        `ObjectStop stop = 2;

`        `ObjectFollow follow = 3;

`        `ObjectYield yield = 4;

`        `ObjectAvoid avoid = 5;

`    `}

}

message ObjectDecision{

`    `string id = 1;

`    `int32 perception\_id = 2;

`    `repeated ObjectDecisionType object\_decision = 3;

}

// decisions based on each object

// 基于每一个物体的决策

message ObjectDecisions{

`    `repeated ObjectDecision decision = 1;

}

message MainLaneKeeping{

`    `string lane\_id = 1;

`    `string sec\_id = 2;

}

message MainNotReady{

`    `// decision system is not ready. e.g. wait for routing data.

`    `// 决策系统还没有准备好，例如，在等待航路数据

`    `string reason = 1;

}

message MainParking{

`    `// parking\_lot

`    `// 停车场

`    `string parking\_lot = 1;

}

message MainMissionComplete{

`    `// arrived at routing destination

`    `// 达到航路目的地

`    `// when stopped, the front center of vehicle should be at this point.

`    `// 当车辆停止时，车辆的前部中心应该在这个位置。

`    `common.Point3D stop\_point = 1;

`    `// when stopped, the heading of the vehicle should be stop\_heading.

`    `// 当车辆停止时，车辆的航向应位于 stop\_heading。 

`    `double stop\_heading = 2;

}

message MainDecision{

`    `oneof task{

`        `MainLaneKeeping lane\_keeping = 1;

`        `MainStop stop = 2;

`        `MainEmergencyStop estop = 3;

`        `MainMissionComplete mission\_complete = 4;

`        `MainNotReady not\_ready = 5;

`        `MainParking parking = 6;

`    `}

}

message DecisionResult{

`    `// decisions based on task and motion planning

`    `// 基于任务和动作规划的决策

`    `MainDecision main\_decision = 1;

`    `// decisions based on each object

`    `// 基于每一个物体的决策

`    `ObjectDecisions object\_decision = 2;

}

Planning.proto 文件中封装了decision.proto 文件，在该文件中还包含了当前车辆的控制状态以及车辆状态。这些信息可以被发送到UI模块进行显示，也会被记录在日志当中以便于开发人员对车辆进行开发调试。

Planning.proto

syntax = "proto3";

package piauto.plannning;

import "decision.proto";

import "header.proto";

import "geometry.proto";

message Planning{

`    `common.Header header = 1;

`    `ControlState control\_state = 2; // control state 控制状态

`    `VehicleState state = 3;         // vehicle state 车辆状态



`    `// decision of the vehicle, lane follow, stop by obstacle and etc..

`    `// 车辆决策结果：车道跟随，障碍物停车等等

`    `DecisionResult decision = 4;

`    `repeated common.Point3D trajectory\_point = 5; // predict trajectory 预测轨迹



`    `// signal status of the current vehicle

`    `// 当前车辆的信号状态

`    `ADCSignals adc\_signals = 6;

`    `bool autonomous\_mode = 7;

}

enum ControlState{

`    `// attach to current lane

`    `// 附属到当前车道

`    `AttachLane = 0;

`    `// turning left and right state

`    `// 左转和右转状态

`    `Turnning = 1;

`    `// no attach lane, need slow down and try to attach lane

`    `// 没有附属到车道，需要减速并尝试连接车道

`    `CloserToLane = 2;

`    `// Stop

`    `// 停止

`    `Stop = 3;

}

message VehicleState{

`    `// Current pose of the vehicle

`    `// 车辆的当前姿态

`    `common.Point3D pose = 1;

`    `double body\_angle = 2;

`    `double front\_wheel\_angle = 3;

`    `double rear\_wheel\_speed = 4;

}

message ADCSignals{

`    `enum SignalType{

`        `LEFT\_TURN = 0;

`        `RIGHT\_TURN = 1;

`        `LOW\_BEAM\_LIGHT = 2;

`        `HIGH\_BEAM\_LIGHT = 3;

`        `FOG\_LIGHT = 4;

`        `EMERGENCY\_LIGHT = 5;

`        `HORN = 6;

`    `}

`    `repeated SignalType signal = 1;

}

**9.7用户界面** 

最后，我们展示了用户界面（UI），用户使用该用户界面来与自动驾驶车辆交互是非常容易的。图 9.7 显示了车辆静止时的用户界面。在用户界面的左侧，用户可以看到当前时间，车辆的当前速度，下一个站点以及到达目的地的预估时间。在用户界面的右侧，会显示部署环境的地图，还有沿着站点以及站点之间的固定路线。用户可以与地图进行交互。用户能够通过选择目的地，然后单击左侧的“开始”（START）来启动车辆，让车辆开始行进。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.007.jpeg)

图9.7 当车辆静止时的DragonFly 用户界面

图 9.8 展示了车辆移动时的用户界面。在用户界面的左侧，用户可以看到车辆的当前速度，检测到的障碍物也会被投影到用户界面上。用户还可以通过点击用户界面上的“停止”（STOP）按钮，来随时停止车辆。

![图片](Aspose.Words.41dc708e-b65d-45f3-8a6b-804222a26eac.008.png)

图9.8 当车辆移动时的DragonFly用户界面

**参考文献**

1 Kite‐Powell, J. (2018). This company says you can design your own autonomous vehicle.

<https://www.forbes.com/sites/jenniferhicks/2018/09/24/this>‐company‐says‐you‐can‐design‐your‐own‐autonomous‐vehicle/#33733dab2009 (accessed 1 October 2019).

2 PerceptIn (2018). Build your own autonomous vehicles with DragonFly technologies.

<https://www.perceptin.io/post/build>‐your‐own‐autonomous‐vehicle‐with‐dragonfly‐technologies (accessed 1 October 2019).

3 YouTube (2019). PerceptIn DragonFly Autonomous Pod in Extreme Traffic Environments.

<https://www.youtube.com/watch?v=KhzwnJ8ayYg&t=42s> (accessed 1 October 2019).

4 YouTube (2019). Autonomous Shuttle Service Provided by PerceptIn. <https://www.youtube.com/watch?v=6twW4EoiThk> (accessed 1 October 2019).


