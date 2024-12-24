每一个PCIe设备可以只有一个功能（Function），即Fun0。也可以拥有最多8个功能，即多功能设备（Multi-Fun）。需要注意的是，每个设备必须要有功能0（Fun0），其他的7个功能（Fun1~Fun7）都是可选的。不管这个PCIe设备拥有多少个功能，其每一个功能都有一个唯一独立的配置空间（Configuration Space）与之对应。
![b069916119a90a7e966c76104c2fe708](C:\Users\Administrator\AppData\Local\Temp\b069916119a90a7e966c76104c2fe708.png)

设备在系统的PCI地址空间里申请一段来用，所申请的空间基址和大小保存在BAR基址寄存器里。
BAR里的只是PCI域的地址空间，需要映射到IO地址空间里或者内存空间里之后软件才能使用。

映射到IO空间的话，用IO读写指令和函数去访问设备；
pcim_iomap bar地址IO空间映射，注意不能重复多次映射！！！
映射到内存空间的话，首先得到的是物理地址，映射到虚拟地址后就可以像用指针那样访问。
IO BAR和MEM BAR分别是映射到IO空间和内存空间的BAR；

每个BAR具体干嘛使设备自己定义的，要看手册。
一旦BAR的值确定了（Have been programmed），其指定范围内的当前设备中的内部寄存器（或内部存储空间）就可以被访问了。当该设备确认某一个请求（Request）中的地址在自己的BAR的范围内，便会接受这请求。
![aba9bf3cc2af8e30534847650990216e](C:\Users\Administrator\AppData\Local\Temp\aba9bf3cc2af8e30534847650990216e.png)

在PCIE配置空间里，0x10开始后面有6个32位的BAR寄存器，BAR寄存器中存储的数据是表示PCIE设备在PCIE地址空间中的基地址。
BAR寄存器的初始化是由内核进行初始化的，在系统开机时，内核会遍历查找各个PCIE设备，然后为PCIE设备分配对应的总线地址空间。
https://blog.csdn.net/kunkliu/article/details/108751452?utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.no_search_link&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~default-2.no_search_link

https://blog.csdn.net/wordwarwordwar/article/details/81194559

http://blog.chinaaet.com/justlxy/p/5100053328

TLP的概念
TLP(Transaction Layer Packet)应该算是PCIe中最重要的概念了。当处理器或其他PCIe设备访问PCIe设备时，所传送的数据报文首先通过事务层被封装成一个或多个TLP，之后才能通过PCIe总线的各个层次发送出去。可以说TLP是用户程序和PCIe设备交互的唯一渠道。
https://blog.csdn.net/abcamus/article/details/74157026

PCI BUS
一般指PCI总线。
与HOST主线直接连接的PCI总线通常被命名为PCI总线0。
![d66eebca1c99d996d56ae7b4c11fa0b5](C:\Users\Administrator\AppData\Local\Temp\d66eebca1c99d996d56ae7b4c11fa0b5.png)

和PCI总线一样，PCIe总线中的每一个功能（Function）都有一个唯一的标识符与之对应。这个标识符就是BDF（Bus，Device，Function），PCIe的配置软件（即Root的应用层，一般是PC）应当有能力识别整个PCIe总线系统的拓扑逻辑，以及其中的每一条总线（Bus），每一个设备（Device）和每一项功能（Function）。

在BDF中，Bus Number占用8位，Device Number占用5位，Function Number占用3位。显然，PCIe总线最多支持256个子总线，每个子总线最多支持32个设备，每个设备最多支持8个功能。

PCIe总线采用的是一种深度优先（Depth First Search）的拓扑算法，且Bus0总是分配给Root Complex。Root中包含有集成的Endpoint和多个端口（Port），每个端口内部都有一个虚拟的PCI-to-PCI桥（P2P），并且这个桥也应有设备号和功能号。

http://blog.chinaaet.com/justlxy/p/5100053262

BAR空间和TLP的关系
TLP能根据地址被路由到对应设备的BAR空间中去。比如说现在有一个mem read request，如果路由地址（地址信息包含在TLP中）是0x71000000，而有一个设备func0的mem空间范围是0x70000000~0x80000000，那么这个TLP就会被这个func处理。从func0的0x71000000对应的地址读取相应数据。这就是TLP中的地址字段和BAR空间的地址之间的关系
TLP中的地址哪里来？ATU(Address Translation Unit)转换过来的。这个问题就是这么的简单。ATU是什么？是一个地址转换单元，负责将一段存储器域的地址转换到PCIe总线域地址，除了地址转换外，还能提供访问类型等信息，这些信息都是ATU根据总线上的信号自己做的，数据都打包到TLP中，不用软件参与。软件需要做的是配置ATU，所以如果ATU配置完成，并且能正常工作，那么CPU访问PCIe空间就和访问本地存储器空间方法是一样的，只要读写即可。

原文链接：https://blog.csdn.net/abcamus/article/details/74157026

pci_find_capability
PCIe的capability代表PCIe设备是否具备某种功能.
https://blog.csdn.net/tiantao2012/article/details/65935333

pci_map_page
http://blog.chinaunix.net/uid-22237530-id-1781805.html
通过pci_map_page函数得到此页的总线地址（此地址写入DMA Channel 0/1 PCI Address寄存器）。

来自 http://blog.chinaunix.net/uid-22237530-id-1781805.html

pci_map_single 虚拟地址转换为PCI物理地址
https://blog.csdn.net/flyingleo1981/article/details/119949710

pci_resource_start(struct pci_dev *dev, int bar) Bar值的范围为0-5。
来自 https://blog.csdn.net/zz2862625432/article/details/104926925