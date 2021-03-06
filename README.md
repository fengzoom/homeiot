﻿# homeiot
物联网安卓端开发经验

从上年的6月份开始，接手公司的智能家居开发项目，负责安卓端的开发，整个智能家居项目从零开始，一年过后，把这一年的经验写下来，做经验交流。

项目大致的架构
第一阶段是通过网关去管理子设备，网关承担管理子设备，承担子设备跟云通讯功能，而子设备跟网关的通讯是采用国外比较流行的zwave协议，没有采用国内比较流行zigbee协议，没做过zigbee的开发，但是zwave信号方面是挺强的，指令的控制成功率非常高。网关通过有线或者wifi的方式接入家庭的路由器，实现与云端通讯。

云端，第一阶段使用的公有云，由于没相关的物联网经验，使用的公有云给整个开发过程带来很多困难，很多特色功能，公有云无法实现，只能去app和网关去承担这部分工作，绕开云端。以后开发iot时，找云端一定要慎重，不然会给整个项目带来无尽的困难，如果公司资金充足，可以考虑开发私有云，基于阿里的云，以及智能家居的iot套件的基础上开发属于自己的私有云，很多功能定制就不用受到公有云的限制。如果项目定制化程度不高，一般公有云都能解决，使用公有云价格比较划算，适合公司起步阶段和创业团队。

app端，安卓和IOS都属于自己开发，由于自己，以及公司新成立的团队，都未有智能家居方向上的项目经验，所以在整个，app的交互，设计方面都花了很多时间，以及走了很多弯路，即使现在app已经上架，产品已经成型，都只是达到顺利使用，很多功能暂时无法实现，交互方面也未能达到如期目标。
很多原本应该由云端实现的功能，交由app端是完成，或者是与网关配合完成，这样不多不少给整个系统带来了很多不稳定的因素，或者导致整个数据不可管理。只能折中选择，暂时满足功能使用。

下面根据功能分部谈谈整个项目

1，子设备与网关的通讯
该部分不是我开发的，代码级的了解谈不上，但是在调试时，大致的流程、功能、业务逻辑基本清晰。

子设备跟网关采用zwave协议，国外比较流行，但是在国内使用该协议的厂商比较少，优点可以自行百度一下，zwave的准入门槛还是挺高的，相关的认证也非常严格，稍有不符合规范，都会被打回。我说说自己这一年跟它打交道过程中发现的有确点，可能比较片面，毕竟第一次开发智能家居，也未接触过zigbee，无法根据两者的使用情况做对比。
优点，开发的网关，以及子设备，因为都是根据zwave的标准协议开发，网关可以无阻地接入其他zwave协议的子设备，网关能知道加进来的是一块什么类型的产品，能根据不同类型的产品进行获取数据以及控制指令下发，而开发的子设备，也能无阻地接入到其他厂商开发的zwave网关，并实现指令下发，这样能让公司产品可以通过单品的形式售卖，增加初期的销售额，接纳你产品变得没那么高，融入其他的生态也变得容易，不需要全家桶才能使用。

缺点：更多是坑，对zwave不熟悉而导致的。1，zwave对特定的设备都有一套特定的强制命令集，例如温控器，它有规定了很多基本功能的指控指令的格式，内容。例如：温控器硬件有一套自有的通讯协议，1002这个地址设置数值26，为给温控器设置温度值，而zwave对温控器的协议就是需要这样的一个json字符串来设定温度{"value":"26", "type":1,"unit":0,"use_default":22}，这样带来的问题，硬件每次上报自己的数据，都得按照zwave标准协议转换一次，而app获取数据都要解析一次，获取里面的内容值。那个type也是一个小小的坑，设置温度是分设置制热温度还是制冷温度的。2,这个强制的指令集还有一个更为坑的地方，就是硬件的部分模式跟开关机是定义在同一个指令内容里面，它的一个小小的规定，导致app做多了很多工作以及部分交互无法完成，而且也导致当初我们绕了很久。他的模式控制指令{"value":8,"use_default":0}，这个指令很神奇的把制热，制冷，复位（开机），关机，除湿这个功能合在一起，制冷制热除湿放在一起这个可以理解，但是把开关机也放置到这个指令中，实在令人抓狂。app操作温控器时，就要考虑很多问题了。例如，app给温控器发送一个开机指令5，其实不叫开机，叫复位，回复到原来来状态，温控器开机了，但是app不知道温控器复位后是什么模式，需要温控器另外上报一次自己的状态，才能让app知道设备是什么状态。如果温控器关机状态0，app如果直接给温控器设置制冷这个模式，温控器无法执行开机并做制冷，需要app先给一个复位指令5，然后再给一个制冷指令1才能正确执行，耐人寻味。标准协议还有很多奇奇怪怪的坑，不一一列举，反正严重影响到app的交互。

2，app与网关的通讯问题
在zwave网络中，网关中的zwave模块是有更多的功能以及权限的，例如分配子设备的nodeid以及homeid，以及对zwave网络拓扑的管理，所以在实际的使用中，需要app能对网关的zwave模组进行对于用户不需要关心的指令操作。这个功能叫透传功能，云端提供这个通道，部分公有云不提供，需要商务洽谈，增加才有。有了这个透传，app就能绕开公有云干涉的过多业务层限制了，当时也是受制于公有云，迫于无奈才选择这样做的，有了透传app能直接发送zwave模组的操作指令给网关，对网关进行更多的操作。而且指令掌握在自己手上，调试起来也方便很多，为什么这么说。我们选择的公有云他没有对接zwave经验，还是按照对接zigbee的经验，让我们踩了不少坑，例如，app触发添加指令时，按照云端来下发指令，云端以每5秒下发一下添加指令，也许zigbee能承受，但是zwave模组就会疯狂的进入添加模式，这样会直接导致子设备添加时出现异常，信息不全或者失败等问题，即使添加上也会出现控制不到，因为添加到一半，zwave又重新进入添加模式了。通过app直接给网关发送透传指令时，就变得可控了。

3，网关
网关是由一家开发zwave网关的公司开发的，硬件用他们的标准品，程序是根据我们的需求定制。整个开发调试测试过程中，也是充满无尽的坑，但是经过一年的摸索，负责跟网关的同事也对网关有代码级的认识了。我写写我遇到跟安卓这边调试时遇到的问题。
1，网关的zwave模组进入including模式后，会把其他的信息都丢掉的。当including指令发给网关后，网关进入添加模式，搜索时间我们设定大约60秒，在这个时间内，子设备发过来的信息更新，或者app下发的其他设备的控制指令是会丢失的，这个也是找了很才找到。删除设备的时候也是，所以在取消添加以及取消删除操作时，app要让网关放弃执行添加或者删除。
2，




