# wifi控制智能设备

##### 局域网发现设备

SSDP:Simple Sever Discovery Protocol,简单服务发现协议，此协议为网络客户提供一种无需任何配置、管理和维护网络设备服务的机制。此协议采用基于通知和发现路由的多播发现方式实现。协议客户端在保留的多播地址：239.255.255.250：1900（IPV4）发现服务，（IPv6 是：FF0x::C）同时每个设备服务也在此地址上上监听服务发现请求。如果服务监听到的发现请求与此服务相匹配，此服务会使用单播方式响应。

常见的协议请求消息有两种类型，第一种是服务通知，设备和服务使用此类通知消息声明自己存在；第二种是查询请求，协议客户端用此请求查询某种类型的设备和服务。请求消息中包含设备的特定信息或者某项服务的信息，例如设备类型、标识符和指向设备描述文档的URL地址。下图显示这两类通知消息和HTTP协议的关系：

 

![img](https://images.cnblogs.com/cnblogs_com/debin/188363/1.gif)

设备发现过程允许控制点使用一个设备类型或标识，或者是服务类型进行查询。这要求标准设备或服务类型，或者设备特定实例的发现和广告消息基于一个独一无二的标识，UPnP设备和服务类型的定义是UPnP论坛工作委员会的责任。从设备获得响应的内容基本上与多址传送的设备广播相同，只是采用单址传送方式。

SSDP 协议消息

1、设备查询消息

  当一个控制点加入到网络中时，设备发现过程允许控制点寻找网络上感兴趣的设备。发现消息包括设备的一些特定信息或者某项服务的信息，例如它的类型、标识符、和指向XML设备描述文档的指针。从设备获得响应从本质上说，内容与多址传送的设备广播相同，只是采用单址传送方式。设备查询通过HTTP协议扩展M-SEARCH方法实现的。典型的设备查询请求消息格式：



```
`M-SEARCH * HTTP/1.1 HOST: 239.255.255.250:1900 MAN: "ssdp:discover" MX: seconds to delay response ST: search target `
```



各HTTP协议头的含义简介：

HOST：设置为协议保留多播地址和端口，必须是：239.255.255.250:1900（IPv4）或FF0x::C(IPv6)

MAN：设置协议查询的类型，必须是：ssdp:discover

MX：设置设备响应最长等待时间，设备响应在0和这个值之间随机选择响应延迟的值。这样可以为控制点响应平衡网络负载。

ST：设置服务查询的目标，它必须是下面的类型：

　　ssdp:all 搜索所有设备和服务
　　upnp:rootdevice 仅搜索网络中的根设备
　　uuid:device-UUID 查询UUID标识的设备
　　urn:schemas-upnp-org:device:device-Type:version 查询device-Type字段指定的设备类型，设备类型和版本由UPNP组织定义。
　　urn:schemas-upnp-org:service:service-Type:version 查询service-Type字段指定的服务类型，服务类型和版本由UPNP组织定义。

 

 

在设备接收到查询请求并且查询类型（ST字段值）与此设备匹配时，设备必须向多播地址239.255.255.250:1900回应响应消息。典型：



```
`HTTP/1.1 200 OK CACHE-CONTROL: max-age = seconds until advertisement expires DATE: when reponse was generated EXT: LOCATION: URL for UPnP description for root device SERVER: OS/Version UPNP/1.0 product/version ST: search target USN: advertisement UUID `
```



 

各HTTP协议头的含义简介：



| CACHE-CONTROL | max-age指定通知消息存活时间，如果超过此时间间隔，控制点可以认为设备不存在 |
| ------------- | ------------------------------------------------------------ |
| DATE          | 指定响应生成的时间                                           |
| EXT           | 向控制点确认MAN头域已经被设备理解                            |
| LOCATION      | 包含根设备描述得URL地址                                      |
| SERVER        | 饱含操作系统名，版本，产品名和产品版本信息                   |
| ST            | 内容和意义与查询请求的相应字段相同                           |
| USN           | 表示不同服务的统一服务名，它提供了一种标识出相同类型服务的能力。 |



 

2、设备通知消息

 

在设备加入网络，UPnP发现协议允许设备向控制点广告它的服务。它使用向一个标准地址和端口多址传送发现消息来实现。控制点在此端口上侦听是否有新服务加入系统。为了通知所有设备，一个设备为每个其上的嵌入设备和服务发送一系列相应的发现消息。每个消息也包含它表征设备或服务的特定信息。

**3.1.1 ssdp:alive消息**

在设备加入系统时，它采用多播传送方式发送发现消息，包括告知设备包含的根设备信息，所有嵌入设备以及它包含的服务。每个发现消息包含四个主要对象：

1. 在NT头中包含的潜在搜索目标。
2. 在USN头中包含的复合发现标识
3. 在LOCATION头中关于设备信息的URL地址
4. 在CACHE-CONTROL头中表示的广告消息的合法存在时间。

对于根设备，存在三种发现消息：



| NT                 | USN                                     |
| ------------------ | --------------------------------------- |
| 根设备的UUID       | 根设备的UUID                            |
| 设备类型：设备版本 | 根设备的UUID，设备类型：设备版本        |
| upnp:rootdevice    | 根设备的UUID，设备类型和upnp:rootdevice |



对于根设备，存在两种发现消息：



| NT                 | USN                                |
| ------------------ | ---------------------------------- |
| 嵌入设备的UUID     | 嵌入设备的UUID                     |
| 设备类型：设备版本 | 嵌入设备的UUID，设备类型和设备版本 |



对于每个服务：



| NT                 | USN                                |
| ------------------ | ---------------------------------- |
| 服务类型：服务版本 | 相关设备的UUID，服务类型和服务版本 |



如果一个根设备有n个嵌入设备，m个嵌入服务，而且包含k个不同的服务类型，这将会发出3 + 2n + k次请求。这些广告消息像控制点描述了设备的所有信息。这些消息必须作为一系列一起发出，发送的顺序无关紧要，但是不能对单个消息进行刷新或取消的操作。选择一个适当的持续期是在最小化网络通讯和最大化设备状态及时更新之间求得一个平衡，相对较短的持续时间可以保证控制点在牺牲网络流量的前提下获得设备的当前状态；持续期越长可以大大减少设备刷新造成的网络流量。一般而言，设备制造商应该选择一个适当的持续时间值。

由于UDP协议是不可信的，设备应该发送多次设备发现消息。而且为了降低控制点无法收到设备或服务广告消息的可能性，设备应该定期发送它的广告消息。在设备加入网络时，它必须用NOTIFY方法发送一个多播传送请求。NOTIFY方法发送的请求没有回应消息，典型的设备通知消息格式如下：



```
`NOTIFY * HTTP/1.1 HOST: 239.255.255.250:1900CACHE-CONTROL: max-age = seconds until advertisement expires LOCATION: URL for UPnP description for root device NT: search target NTS: ssdp:alive USN: advertisement UUID `
```





各HTTP协议头的含义简介：



| HOST          | 设置为协议保留多播地址和端口，必须是239.255.255.250:1900。   |
| ------------- | ------------------------------------------------------------ |
| CACHE-CONTROL | max-age指定通知消息存活时间，如果超过此时间间隔，控制点可以认为设备不存在 |
| LOCATION      | 包含根设备描述得URL地址                                      |
| NT            | 在此消息中，NT头必须为服务的服务类型。                       |
| NTS           | 表示通知消息的子类型，必须为ssdp:alive                       |
| USN           | 表示不同服务的统一服务名，它提供了一种标识出相同类型服务的能力。 |



一个发现响应可以包含0个、1个或者多个服务类型实例。为了做出分辨，每个服务发现响应包括一个USN：根设备的标识。在同样的设备里，一个服务类型的多个实例必须用包含USN:ID的服务标识符标识出来。例如，一个灯和电源共用一个开关设备，对于开关服务的查询可能无法分辨出这是用于灯的。UPNP论坛工作组通过定义适当的设备层次以及设备和服务的类型标识分辨出服务的应用程序场景。这么做的缺点是需要依赖设备的描述URL。

**3.1.2 ssdp:byebye消息**

在设备和它的服务将要从网络中卸载时，设备应该对于每个未超期的ssdp:alive消息多播方式传送ssdp:byebye消息。但如果设备突然从网络卸载，它可能来不及发出这个通知消息。因此，发现消息必须在CACHE-CONTROL包含超时值，如果不重新发出广告消息，发现消息最后超时并从控制点的缓存中除去。典型的设备卸载消息格式如下：



```
`NOTIFY * HTTP/1.1 HOST: 239.255.255.250:1900NT: search target NTS: ssdp:byebye USN: advertisement UUID各HTTP协议头的含义简介： HOST	设置为协议保留多播地址和端口，必须是239.255.255.250:1900 NT	在此消息中，NT头必须为服务的服务类型。 NTS	表示通知消息的子类型，必须为ssdp:alive USN	表示不同服务的统一服务名，它提供了一种标识出相同类型服务的能力`
```





专有设备或服务可以不遵循标准的UPNP模版。但如果设备或服务提供UPNP发现、描述、控制和事件过程的所有对象，它的行为就像一个标准的UPNP设备或服务。为了避免命名冲突，使用专有设备命名时除了UPNP域之外必须包含一个前缀"urn:schemas-upnp-org"。在与标准模版相同时，应该使用整数版本号。但如果与标准模版不同，不可以使用设备复用和徽标。

简单设备发现协议不提供高级的查询功能，也就是说，不能完成某个具有某种服务的设备这样的复合查询。在完成设备或者服务发现之后，控制点可以通过设备或服务描述的URL地址完成更为精确的信息查询。



##### 局域网连接设备

根据ssdp协议发现设备后，建立socket连接，剩下的就是数据传输交互，自己可以定协议。

socket处理粘包的问题也比较常见，心跳维持，重连等。



##### 智能硬件模组

一般的智能硬件是能够运行rtos的设备，类似树莓派 阿蒂诺等，如今的智能硬件是类似乐鑫等常见生产的嵌入固定计算单元的硬件，常见做法如嵌入固定的其他模组，比如通信模块，在通信模块封装好指令，屏蔽掉云平台的差异，可以大批量生产。

cpu轮训IO模组，处理下行命令，通过串口给IO模组实现上行通信。



## RTOS 概念

RTOS （ Real-time operating system , 实时操作系统），又称实时操作系统，是管理系统硬件和软件资源的系统软件，以方便开发者使用，操作系统管理的资源包括处理器、存储器、外设、甚至包括文件系统等等。

实时操作系统最大的特色就是其“实时性”。也就是说，如果有任务需要执行，实时操作系统会立即（在较短时间内）执行该任务，保证了任务在指定时间内完成。

实时操作系统根据任务执行的实时性，分为“硬实时”操作系统和“软实时”操作系统，“硬实时”操作系统比“软实时”操作系统响应更快、实时性更高，“硬实时”操作系统大多应用于工业领域。

- “硬实时”操作系统必须使任务在确定的时间内完成。
- “软实时”操作系统能让绝大多数任务在确定时间内完成。

Huawei LiteOS 是一款“软实时”操作系统。

实时操作系统通常都会有最基础的内核（ Kernel ），以及扩展模块，例如文件系统、网络协议栈和应用、设备驱动程序等模块。

传统的单片机开发和部分物联网硬件开发，直接裸跑代码，主要采用下面两种编程模式：

- 轮询模式：main函数死循环，不断的查询状态位（如寄存器），满足条件就去执行相应的函数，完成后继续执行main函数剩下的逻辑。
- 中断模式：main作为为主任务死循环，外部信号触发中断，打断主任务，去处理中断任务，中断处理完自动回到主任务。

## 为什么使用 RTOS

随着物联网和人工智能技术快速发展，人们对身边的各种设备要求也越来越高。家里的台灯不仅要能远程开关，还能够通过感知周围环境和记录用户使用习惯自动进行调节；为了随时掌握身体健康状况，各种可穿戴智能手环推陈出新，能够定位，测步，记录心跳等等。

程序的复杂性也在指数级暴增。RTOS（嵌入式实时操作系统）就好比一座“大厦”的地基，只有构筑在坚固可靠的基石上，我们的物联网产品才能应对各种考验。

在8位或16位嵌入式系统应用中，由于CPU能力有限，往往采用单片机开发模式。但是，当嵌入式系统比较复杂、采用32位CPU时，由于处理能力强大，单线程的编程方式不但代码逻辑复杂、容易出错，同时也很难发挥出32位CPU的处理能力。而引入操作系统后，最主要的变化就在于”多线程“，让多任务并行，充分发挥系统资源的能力。

## 使用 RTOS带来的好处

- 降低开发难度，直接使用系统API，即可完成系统资源的申请、多任务的配合（基于优先级的实时抢占调度，同优先级的时间片调度)，以及任务间的通信等（如锁、事件等机制）。
- 增加代码可读性，易于维护和管理。
- 提升可移植性，对接不同芯片的工作由操作系统完成，应用开发者只需要关注 OS 层接口。

对于物联网端侧设备厂家来说，其产品是否需要使用操作系统，建议从以下这几个角度进行考虑和分析，相信就会有答案了。

- 开发效率
- 代码可维护性
- 可移植性
- 提升硬件资源效率
- 应用复杂性/是否需要多线程调度/联网/文件系统/加密/安全等

