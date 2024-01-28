**3 智能机器人和自动驾驶车辆的底盘技术**

**3.1简介**

`      `如图3.1所示，底盘执行指令是由规划和控制模块决策后并通过CAN总线发出的，这也是智能机器人和自动驾驶车辆与外界直接交互的“物理实体”。在本章节中我们将介绍智能机器人和自动驾驶车辆的底盘技术。整个章节会分为三个部分：首先，我们会简要的介绍构建自动驾驶车辆和智能机器人所需要的基本底盘技术，特别是线控驱动技术。线控驱动技术指的是取代传统机械控制的电子系统[1]，线控技术不使用电缆、液压或者其他方式来为驾驶员就车辆速度或方向提供直接的、物理层面的控制，而是使用电子控制来激活刹车、控制转向和对其他机械系统进行操作。通常来说，现代的智能汽车中三个主要的车辆控制系统已经被电子控制所取代：电子油门控制系统、线控刹车系统和线控转向系统。

`      `其次，我们介绍两个开源项目， the Open Source Car Control(OSCC)和OpenCaret[2,3]。OSCC作为一个兼具软件和硬件功能的开源项目，它能够以更加智能的方法完成对汽车的控制，从而进一步促进自动驾驶汽车技术的发展。该项目中采用了模块化的设计思路，从而保证了项目的稳定运行，并通过使用软件将汽车的通信网络和控制系统相连接。OpenCaret项目是建立在上述OSCC项目上的，该项目能够实现现代智能汽车L3级别的自动驾驶系统。后续的章节中，本书将会向读者展示一个将起亚 Soul EV型号车辆底盘改装为自动驾驶车辆底盘的详细信息。

`       `在本章节的最后，我们将会向读者展示一个详细的自动驾驶底盘改造案例——基于PerceptIn自动驾驶车辆的底盘软件适配改造。在该项目中，作者为不同的底盘提供了一个抽象层，因此底盘制造商可以轻松将PerceptIn的自动驾驶技术栈与他们自身的底盘结合，从而快速的将传统车辆底盘转变成自动驾驶汽车的车辆底盘。

![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.001.jpeg)

图 3.1 模块化设计架构（Chassis:汽车底盘 Sonars:声呐 Radars:毫米波雷达 CAN Bus:CAN总线 Plannning and Control:控制和规划模块 Map:地图 Computer Vision:计算机视觉 GNSS:全球定位系统）

**3.2 线控节流阀**

`      `与传统的油门控制系统不同，现代的汽车控制系统使用了一系列电子传感器和执行器，而传统的油门控制系统是通过机械电缆将油门踏板与油门连接在一起。如图3.2所示

`  `![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.002.png)

图3.2电子节气门控制（Drive-By-Wire Throttle Pedal：线控油门踏板；Electronic Throttle Control Module：电子油门控制模块；Drive-By-Wire Throttle Body：线控油门体；Feedback：反馈；TPS：Throttle Position Sensor油门踏板位置传感器；Engine Control Unit：发动机控制单元；Vehicle Existing ECM (With no DBW Capability)：车辆上现存的发动机控制模块（但是不具备线控的功能）；IAC Signal：空气怠速控制；Brake Signal：制动信号；）

`     `在使用电子油门控制(ETC)的车辆中，会通过油门踏板发出信号，使机电执行器打开油门。一般的典型ETC系统会由加速踏板模块、可由电子油门体(ETB)开启和关闭的节流阀、动力系统控制模块或发动机控制模块(PCM或ECM)组成。ECM是一种电子控制单元(Electronic Control Unit)，它是一种嵌入式系统，通过计算其他传感器(包括加速踏板位置传感器、发动机速度传感器、车辆速度传感器和巡航控制开关)测量到的数据，利用软件来确定所需的油门位置。然后通过ECM内部的闭环控制算法，利用电机将节流阀打开到所需的角度。节流阀是ETB的一部分，在配备节流阀控制器传感器的车辆上，节流阀的开度是根据油门踏板被踩的远近来确定的。

**3.3 线控制动技术**

线控制动系统主要分为以下两类系统：电子液压制动系统和电子机械制动系统。这两个系统在设计的时候都会考虑到的故障安全问题。传统的液压制动系统由一个主缸和几个副缸构成。当司机踩下制动踏板时，司机对踏板施加的压力会被传递到主缸上。在大多数情况下，这一压力会通过真空或液压制动助力器进行放大，然后该压力会通过制动管路被传递到制动卡钳（brake calipers）或车轮制动分泵缸（wheel cylinders）上以实现汽车的制动。

防抱死制动系统（Anti-lock brake systems，ABS）是现代线控制动系统技术的雏形，因为该技术可以让汽车在没有驾驶员控制的情况下，自动控制车辆制动器所需要力的大小。汽车通过一个电子执行器来控制液压制动器（hydraulic brakes）以实现防抱死功能，汽车的一些其他的安全技术也是基于这一原理。

电子稳定控制系统（Electronic stability control systems）、牵引力控制系统（traction control systems）和自动制动系统（automatic braking systems）都依赖于防抱死制动系统，同时上述的这些系统均属于线性制动系统的外部功能。使用液压制动技术的车辆，其车轮刹车卡钳仍然是通过液压驱动的，但是这些刹车卡钳并不与主缸直接相连接（主缸需要由制动踏板驱动）。当车辆需要制动时，操作者踩下制动踏板就会驱动一个或一系列传感器完成车辆的制动。

制动时，控制单元会根据传感器的信号来决定每个车轮所需的制动力，并根据需要来激活液压刹车卡钳。在机电制动系统中，根本没有液压元件。这类线性制动系统仍然使用传感器来确定车轮所需要的制动力的大小，但是这种制动力并不是通过液压系统来传递的，而是通过电子执行器来驱动每个车轮上的制动器，从而来提供合适的制动力。

**3.4 线控转向技术**

`      `目前，绝大多数汽车都配备了齿条装置或者蜗杆齿扇式转换器，这些装置与方向盘之间直接相连。当操作者转动方向盘时，齿轮齿条装置或转向器也会随之转动。其中齿轮齿条装置可以通过拉杆向球形接头施加扭矩，而转向器可以通过转向臂来移动转向连杆，进而控制着汽车的转向。然而，在很多配备了线控转向技术的现代汽车中，方向盘与轮胎之间并没有实质性的物理结构将两者连接。事实上从技术层面上来讲，线控转向系统即使不使用方向盘也可以对汽车进行转向控制。在实际使用时，方向盘更多的是通过转向路感模拟器向驾驶员提供一个转向反馈。驾驶员对方向盘的动作被ECUs检测到后，ECUs用检测到的动作信息去驱动电动马达的转速，进而达到控制车辆转向的目的。

**3.5 Open Source Car Control项目**

`      `为了更好地了解和学习智能机器人和自动驾驶汽车中的线性驱动技术，OSCC(Open Source Car Control)项目可以说是一个不错的出发点。OSCC项目作为一个兼具软件和硬件功能的开源项目，能够使用计算机去控制现代智能汽车的运行。可以说，该项目很大程度上促进了自动驾驶汽车技术的快速发展。该项目中采用了模块化的设计思路，从而保证了项目的稳定性以及可扩展性，并通过软件将汽车的通信网络和控制系统相连接。

**3.5.1 OSCC APIs**

CAN通道的控制

oscc\_result\_t oscc\_open( uint channel );

oscc\_result\_t oscc\_close( uint channel );

这段代码分别是OSCC项目的开始和结束语句。其中oscc\_open()语句会在指定的CAN通道中打开一个套接字，这样OSCC就可以快速的与固件模块进行通信。而oscc\_close()则可以关闭指定通道中的连接。

OSCC模块的启动和关闭

oscc\_result\_t oscc\_enable( void );

oscc\_result\_t oscc\_disable( void );

在初始化CAN和固件模块之间的连接后，可以使用如上所示的方法来开启和关闭该系统。这些方法可以让应用程序决定向固件发送命令的时机。实际上，应用程序只能在系统启动时发送命令，但是可以随时接收来自固件的信息。

对应模块控制命令的发布

oscc\_result\_t publish\_brake\_position( double normalized\_position );

oscc\_result\_t publish\_steering\_torque( double normalized\_torque );

oscc\_result\_t publish\_throttle\_position( double normalized\_position );

上述命令会向指定固件模块发送一个双重值。比如刹车和油门模块接收到的双重值可能为[0.0,1.0]，其中0.0表示关闭刹车，1.0表示油门打开；转向的双重值[-1.0,1.0]，其中-1.0表示逆时针转向，1.0是顺时针转向。有时为了让汽车达到需要的状态，API还会构建合适的值作为“欺骗电压值”发送到汽车固件模块中。出于安全性考虑，API还会对值进行安全性检查，以保证发送出的电压值不会超过汽车电压范围。

OSSC报告和车载诊断信息（OBD）的处理——回调函数

oscc\_result\_t subscribe\_to\_brake\_reports( void(\*callback)(oscc\_brake\_report\_s \*report) );

oscc\_result\_t subscribe\_to\_steering\_reports( void(\*callback)(oscc\_steering\_report\_s \*report) );

oscc\_result\_t subscribe\_to\_throttle\_reports( void(\*callback)(oscc\_throttle\_report\_s \*report) );

oscc\_result\_t subscribe\_to\_fault\_reports( void(\*callback)(oscc\_fault\_report\_s \*report) );

oscc\_result\_t subscribe\_to\_obd\_messages( void(\*callback)(struct can\_frame \*frame) );

应用程序会需要在OSCC的API中注册一个回调函数以接收各个模块的信息。当该回调函数接收到来自API套接字连接中的消息，它就会将该消息发送给应用程序。

**3.5.2 硬件机构**

OSCC以2014年的Kia Soul作为标准基础改装车型，该车辆配备有线控转向技术和线控油门技术。因此该车辆的转向和油门系统的执行器是配备了电子控制系统的，从而我们可以通过对这些系统进行电子控制，从而获得车辆执行器的完全控制。

但是，该车辆并没有配备电控的刹车，因此我们必须再增加一个线控刹车系统。为了有效控制刹车时的力度，我们可以在起亚车辆的刹车系统中内嵌一个刹车执行器。为了实现Kia Soul车辆有效的横纵向的控制，我们需要对这三个系统进行控制（转向系统、油门系统、刹车系统），将这三个控制系统与现有的汽车CAN总线相连接，并为额外的微处理器和新的执行器件进行供电。本节中给车辆构建的新的控制模块都是以Arduino控制器为微处理器来完成设计和构建的。下面我们来介绍一些主要的汽车硬件：

硬件网关（*Hardware gateway*）：Kia Soul汽车上存在着很多不同的CAN总线。其中OBD-II CAN总线包含了车辆状况信息，例如方向盘旋转的角度、汽车轮胎的转速和制动压力。这类消息经常被作为比例-积分-微分控制（PID）和路径规划算法的输入。不同于传统车辆的共享车辆OBD-II总线，并可能产生干扰车辆的本地信息等问题，OSCC(Open Source Car Control)项目选择拥有一条独立的CAN总线，该总线一般被称为控制CAN总线，汽车的控制命令与状态报告的发送和接收操作都会在操作CAN总线中进行。CAN网关在车辆本地OBD-II总线和控制CAN总线之间架起了一座桥梁，并将和相关的OBD-II信息从OBD-II总线转发到控制CAN总线。这样一来，订阅了OBD信息的应用程序就可以使用这些信息。虽然CAN网关连接着控制CAN总线和OBD-II CAN总线，但是CAN消息的发布流向却是单向的，即只能从OBD-II CAN总线流向控制CAN总线。

转向机构（*Hardware steering*）：Kia Soul汽车的转向系统是一个电动辅助转向(EPAS)系统。整个转向杆由一个大电流直流电机和一个扭矩传感器构成。其中扭矩传感器会测量方向盘上所受力的大小和方向，并向EPAS微处理器输出模拟信号。最终由微处理器对电机进行控制从而“协助”车辆完成转向。

油门机构（*Hardware throttle*）：Kia Soul的油门系统是一个ETC系统，油门体（throttle body）与油门之间并不是通过机械电缆连接的，而是通过一个位置传感器，将油门与一个电动节流阀相连接。我们可以通过移除加速位置传感器（APS）的输入并将虚假的位置值传入到ETC微处理器中来实现ETC系统的控制，因为踏板位置传感器使用了冗余位置传感器（一般踏板都包含着两个位置传感器并采用冗余控制），且这两个传感器输出的都是模拟信号。

刹车机构（*Hardware brake*）：遗憾的是，起亚Soul的制动系统仍是采用的传统的机械系统。按照工厂制造标准，起亚Soul是不具备电子控制制动的能力的。虽然之后有一些型号的车辆配备了电控制动系统，例如，以2004年-2009年见丰田出产的Prius车型为代表。这种车型配备了电子控制的执行器，但是其缺少配套的微处理器，而是由汽车的ECU进行控制。我们所设计的电子刹车制动装置中配有七个压力传感器(pressure sensors)，十个比例螺线管(proportional solenoids)，一个蓄能器(accumulator)，一个刹车泵(pump)，一个诊断部件(diagnostics components)和一个泄压阀。这些元件可以在汽车废品回收站购买，并且可以直接安装到现有的Kia制动系统中，而不用担心会对原有的制动系统带来不良影响。这样就可以为整辆车增加线控制动系统。

**3.5.3 固件**

制动固件的功能如下：读取制动踏板传感器中的数值并发送其状态报告，另外还可以从控制CAN中线中接收制动命令和故障报告。当制动固件收到制动命令时，制动固件会向ECU发送一个虚假的高低位信号以表示制动请求。当制动固件接收到故障报告时，其将禁用制动模块。

转向固件的功能如下：读取扭矩传感器中的数值并发送其状态报告，另外还可以从控制CAN总线中接收转向命令和故障报告。当转向固件收到转向命令信息时，转向固件会向EAPC ECU发送一个虚假的高低位信号以表示转向请求。当转向固件接收到故障报告时，将会禁用转向模块。

油门固件的功能如下：读取APS中的数值并发送其状态报告，另外还可以从控制CAN总线中接收油门命令和故障报告。当油门固件收到油门控制命令信息时，油门固件会向ECU发送一个虚假的高低位信号以表示节流阀开度请求（汽油发动机是通过进气量的多少来控制发动机喷油量 进而控制动力输出。油门踏板踩下的深度决定了节流阀的开度，节流阀开度影响着发动机的进气量，进气越多，喷油量越多，动力越强）。当油门阀固件接收到故障报告时，其将禁用节流阀模块。

**3.6 OpenCaret**

OpenCaret构建于OSCC(Open Source Car Control)项目之上的，为现代智能汽车[3]实现L3级的高速公路自动驾驶系统提供了便捷的平台。尤其在这个项目当中包含了如何将Kia Soul EV型号的传统车辆转换为线控驾驶底盘的详细操作步骤，相关的内容将会在本节中详细介绍。

**3.6.1 OSCC 节流阀**

踏板位置传感器使用两个输出模拟信号的冗余位置传感器。传感器位置值的量程与从“闭合节流阀”到“打开节流阀”所对应的节流阀位置范围相关。通过注入两个虚假的位置传感器值，就可以实现对油门的控制。Kia ECU通过检测来自传感器的模拟信号中的不连续点来实现对油门踏板位置传感器的故障检测。如果检测到的模拟信号出现任何不连续，那么汽车将进入故障状态，此状态下汽车加速踏板对节流阀的映射将大大减少。为了克服这个问题，新的节流阀微处理器将在感知虚假的位置值之前，在传感器位置值和虚假位置值之间进行插值。我们使用一个继电器来切换ETC微处理器从踏板位置传感器和虚假位置感知到的输入。我们对此的改造步骤如下所示：

●第一步:找到油门踏板位置传感器。

●第二步:断开踏板位置传感器，连接节流阀电缆。

●第三步:将动力单元与紧急刹车电源总线相连接

●第四步:将模块与网关模块控制CAN总线相连接。

**3.6.2 OSCC制动**

Kia Soul EV电动汽车的刹车模块由两部分组成。第一部分是信号欺骗，第二部分是刹车灯开关。为了修改刹车踏板，我们需要断开：(i)刹车踏板位置传感器和(ii)刹车灯开关之间的连接。

`       `安装:车辆控制模块(VCM)可以控制位置传感器的开关状态（该传感器用于控制汽车的刹车）和刹车灯开关继电器的常开（NO）和常闭（NC）。

●第一步:拆卸制动踏板位置传感器和刹车灯开关。

●第二步:为踏板位置传感器和刹车灯开关安装VCM连接器。

**3.6.3 OSCC转向**

我们可以通过移除输入到EPAS微处理器的扭矩传感器并注入虚假的扭矩信息来控制EPAS电机。Kia Soul汽车的ECU通过检测来自传感器的模拟信号中的不连续点来实现对扭矩传感器的故障检测。如果检测的模拟信号出现任何不连续现象，汽车就会进入故障状态，此状态下动力转向系统会被禁用。为了克服这一问题，新的扭矩欺骗微处理器将在感知到虚假信号之前，会在扭矩传感器值和虚假扭矩值之间进行插值。我们使用一个继电器来切换EPAS微处理器从原始扭矩传感器和虚假的扭矩感知到的输入。图3.3所示为Kia Soul线控驱动转向系统。

![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.003.png)

图3.3 Kia Soul线控驱动转向系统（To Control CAN Bus：控制CAN总线；To Emergency Stop Power：紧急刹车电；CAN H：高速CAN信号；CAN L：低速CAN 信号；VIN：电压输入端 ；GND：接地（公共端）；OSCC Steering Module：OSCC转向模块； SIG IN A：A端信号输入； SIG IN B：B端信号输入 ；SIG OUT A：A端信号输出；SIG OUT B：B端信号输出；high signal：高速信号；low signal：低速信号；Torque sensor：扭矩传感器；EPAS ECU: EPAS电子控制单元；Final Steering Module Wiring：最终的线控驱动转向系统）



**3.7 以Perceptln自动驾驶汽车底盘为例的软件适配层**

我们将在本章节以作者公司——Perceptln的底盘软件适配层为例，深入讨论Perceptln的智能汽车中是如何管理不同车辆底盘的适配以及不同底盘之间的信息交互的适配[4]。Perceptln自动驾驶汽车底盘的软件适配层具体架构如图3.4所示。需要注意的是，在图中显示，规划和控制模块所生成的控制命令会直接与底盘进行交互，并直接发送到底盘模块去执行。此外还有一些被动感知模块也可以直接与底盘模块进行双向的交互，例如声呐传感器和雷达传感器。

![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.004.jpeg)

图3.4 Perceptln底盘软件适配层的交互结构（Chassis hardware: 底盘硬件平台；DFPod: 蜻蜓自动驾驶汽车；VendingCar: 自动售卖车；Others:其他功能的智能汽车 DFPod VehicleControlUnit: DragonFly Pod自动驾驶汽车的车辆控制单元接口；  VendingCar VehicleControlUnit: 自动售卖车的车辆控制单元接口；Others VehicleControlUnit: 对应功能的车辆控制单元接口；Derived: 派；VehicleControlUnit:车辆控制单元接口；output: 输出 PassiveSafety: 被动安全接口；Sensors: 传感器接口；trigger: 触发；Interfaces: 接口；ActivatePassiveSafety(): 被动安全激活方法；InactivatePassiveSafety(): 被动安全抑制方法；Chassis: 底盘构成；Planning and Control: 控制和规划模块；Perception: 车辆感知）

从上图中，我们可以看到该底盘模块的核心由三个部分组成：

车辆控制单元接口（*VehicleControlUnit*）：该接口为不同的车辆底盘平台提供了抽象类。因此，在开发过程中，开发人员没有必要完全理解CAN通讯协议的具体细节。实际上，当开发者想要构建一个新的车辆底盘平台时，他/她只需要从车辆控制单元虚拟接口中派生出一个新的类可以实现其核心功能。

传感器接口（*Sensors*）：该接口为连接到CAN总线上的传感器提供了抽象类。该接口主要面向的是雷达传感器、声呐传感器这类被动式的感知传感器。所以，开发人员可以不需要深入了解这些传感器的具体工作原理即可通过该接口轻松的获取各类被动式的传感器数据。

被动安全接口（*PassiveSafety*）：开发人员可以在接口中实现并调整汽车被动感知的逻辑。例如，当雷达或者声呐探测到前方两米处存在障碍物时，开发者需要让车辆自动停止。这种情况下，开发人员就需要从传感器接口中及时获取被动感知传感器的数据，并在被动安全接口中实现避障或停车的功能。

`       `传感器的硬件连接配置如图3.5所示。图中将复杂的各类线路简化成只有两条CAN总线的配置，其中CAN1总线用于连接底盘平台，而CAN2总线则与被动感知传感器相连接。这两条CAN总线通过一个CAN接口卡与控制计算机相连接。如果设计者想要将所有的传感器和底盘都通过同一条CAN总线相连接也是可行的，但是设计者必须和底盘供应商就CAN接口的使用达成一致。

![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.005.jpeg)

图 3.5 CAN总线连接配置（Chassis CanBus: 底盘CAN总线；Sensor CanBus: 传感器CAN总线

Can 1: 1号CAN总线；Can 2: 2号CAN总线；Can Card: CAN接口卡 ）

DragonFly自动驾驶汽车的软件接口配置如图3.6所示。每当设计人员需要实现一个新的底盘平台时，我们都需要实现车辆控制单元接口及其内部的一些基本功能， 例如：速度设置（SetSpeed）、制动请求（SetBrake）、角度设置（SetAngle）、速度获取（GetSpeed）、制动状态获取（GetBrake）、角度获取（GetAngle）等。需要注意的是，在规划和控制模块与底盘进行交互时，以下这些函数承担着最基本的功能。

![图片](Aspose.Words.a4c26921-7d5e-4e94-a3f6-5ec596177647.006.jpeg)

图3.6 DragonFly Pod的软件接口(DFPod VehicleControlUnit: DragonFly Pod自动驾驶汽车的车辆控制单元接口； CanBus: CAN总线；Buffer: 缓冲区；GetSpeed()；GetBrake()；GetAngle()；SetSpeed()；SetBrake()；SetAngle())

软件接口的定义如下所示

// error code definitions enum ErrCode {

OK = 1,

CAN\_ERR = -1,

BOUND\_ERR = -2,

TIME\_OUT =-3

}



ErrCode SetSpeed (float val)

// this function sets the current speed of the chassis



ErrCode SetBrake (float val)

// this function sets the brake value, ranging from 0 to 100, with 100 being the strongest brake.

ErrCode SetSteeringAngle (float val)

// this function sets the steering angle, with positive being left, and negative being right.

float GetSpeed ()

// this function gets the current speed of the chassis.



float GetBrake ()

// this function gets the current brake value



float GetSteeringAngle ()

// this function gets the current steering angle



// the following data structure defines how we store chassis status

ChassisData { float speed; float angle; float brake;

}



boost::signal2::connection SubscribeToChassisData(void(ChassisData&) call\_back)

// this function subscribes chassis data so that every time there is an update from the chassis, the subscriber will receive a notification

**参考文献**

**1**   YouTube (2012). Nissan drive by wire. ht[tps://www](http://www.youtube.com/watch?v).y[outube.com/w](http://www.youtube.com/watch?v)at[ch?v=](http://www.youtube.com/watch?v) MH7e5aUDWYY&feature=youtu.be (accessed 1 November  2018).

**2**   GitHub (2017). Open Source Car Control. <https://github.com/PolySync/oscc> (accessed

1 October 2018).

**3**   GitHub (2018). Drive by Wire Installation. <https://github.com/frk2/opencaret/wiki/> Drive-by-wire-Installation (accessed 1 October   2018).

**4**   PerceptIn (2017).  PerceptIn DragonFly Pod. ht[tps://www](http://www.perceptin.io/products).per[ceptin.io/products](http://www.perceptin.io/products)  (accessed

1 November 2018).

**5**   YouTube (2018). DragonFly 77 GHz millimeter Wave Radar. ht[tps://www](http://www.youtube.com/).y[outube.com/](http://www.youtube.com/) watch?v=ZLOgcc7GUiQ (accessed 1 December 2018).

**6**   YouTube (2018). DragonFly Sonar. ht[tps://www](http://www.youtube.com/watch?v=-H3YdC-xSgQ).y[outube.com/w](http://www.youtube.com/watch?v=-H3YdC-xSgQ)at[ch?v=-H3YdC-xSgQ](http://www.youtube.com/watch?v=-H3YdC-xSgQ)

(accessed 1 December 2018).


