##  虫洞 · 技术栈 | 沉淀、分享、成长，让自己和他人都能有所收获！

> **作者：** 小傅哥，Java Developer，[CSDN 博客专家](https://bugstack.blog.csdn.net)

> 本文档是作者小傅哥多年从事一线互联网```Java```开发的学习历程技术汇总，旨在为大家提供一个较清晰详细的学习教程，侧重点更倾向编写Java核心内容。如果本文能为您提供帮助，请给予支持(关注、点赞、分享)！

<div align="center">
<a href="https://github.com/MyGitBooks/itstack.github.io"><img src="https://badgen.net/github/stars/MyGitBooks/itstack.github.io?icon=github&color=4ab8a1"></a>
<a href="https://itstack.org/_media/qrcode.png?x-oss-process=style/may"><img src="https://badgen.net/github/forks/MyGitBooks/itstack.github.io?icon=github&color=4ab8a1"></a>
<a href="https://itstack.org" target="_blank"><img src="https://itstack.org/_media/onlinebook.svg"></a>
<a href="https://itstack.org/_media/qrcode.png?x-oss-process=style/may"><img src="https://itstack.org/_media/wxbugstack.svg"></a>
</div>
<br/>

| Java基础 | JVM虚拟机| Spring源码 | Netty4.x专题 | 领域驱动设计 | 中间件开发 | JavaAgent | 架构框架搭建 | 
| :---: |  :---: |  :---: |  :---: |  :---: |  :---: |  :---: |  :---: | 
| [:coffee:](#coffee-Java基础编程) | [:computer:](#用Java实现jvm虚拟机) | [:pencil2:](#pencil2-Spring系列源码解读) | [:sound:](#sound-Netty4.x专题) | [:triangular_ruler:](#triangular_ruler-DDD领域驱动设计) | [:electric_plug:](#electric_plug-中间件开发) | [:ghost:](#ghost-JavaAgent全链路监控) | [:art:](#art-架构框架搭建) |

<br/>
<div align="center">
    <a href="https://itstack.org" style="text-decoration:none"><img src="https://itstack.org/_media/icon.svg" width="128px"></a>
</div>
<br/>  

## :coffee: Java基础编程

* [`在windows环境下安装Elasticsearch 6.2.2`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2019-08-12-windows环境下安装elasticsearch6.2.2.md)
* [`elasticsearch-head插件安装`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2019-08-13-elasticsearch-head插件安装.md)
* [`并不想吹牛皮，但！为了把Github博客粉丝转移到公众号，我干了！`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2019-11-23-并不想吹牛皮，但！为了把Github博客粉丝转移到公众号，我干了！.md)
* [`有点干货 | Jdk1.8新特性实战篇(41个案例)`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2019-12-10-[有点干货]Jdk1.8新特性实战篇(41个案例).md)
* [`有点干货 | JDK、CGLIB动态代理使用以及源码分析`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2019-12-21-[有点干货]JDK、CGLIB动态代理使用以及源码分析.md)
* [`似乎你总也记不住，byte取值范围是 -127~128 还是 -128~127`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-any/2020-01-18-似乎你总也记不住，byte的取值范围是-127~128还是-128~127.md)

## :computer: 用Java实现jvm虚拟机

* [`用Java实现JVM第一章《命令行工具》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-01-用Java实现JVM第一章《命令行工具》.md)
* [`用Java实现JVM第二章《搜索class文件》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-02-用Java实现JVM第二章《搜索class文件》.md)
* [`用Java实现JVM第三章《解析class文件》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-03-用Java实现JVM第三章《解析class文件》.md)
* [`用Java实现JVM第三章《解析class文件》附[classReader拆解]`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-04-用Java实现JVM第三章《解析class文件》附[classReader拆解].md)
* [`用Java实现JVM第四章《运行时数据区》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-05-用Java实现JVM第四章《运行时数据区》.md)
* [`用Java实现JVM第五章《指令集和解释器》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-06-用Java实现JVM第五章《指令集和解释器》.md)
* [`用Java实现JVM第六章《类和对象》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-07-用Java实现JVM第六章《类和对象》.md)
* [`用Java实现JVM第七章《方法调用和返回》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-08-用Java实现JVM第七章《方法调用和返回》.md)
* [`用Java实现JVM第八章《数组和字符串》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-09-用Java实现JVM第八章《数组和字符串》.md)
* [`用Java实现JVM第九章《本地方法调用》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-10-用Java实现JVM第九章《本地方法调用》.md)
* [`用Java实现JVM第十章《异常处理》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-jvm/2019-05-11-用Java实现JVM第十章《异常处理》.md)

## :pencil2: Spring系列源码解读

* [`源码分析 | Mybatis接口没有实现类为什么可以执行增删改查`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2019-12-25-[源码分析]Mybatis接口没有实现类为什么可以执行增删改查.md)
* [`源码分析 | Spring定时任务Quartz执行全过程源码解读`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2020-01-01-[源码解析]Spring定时任务Quartz执行全过程源码解读.md)
* [`源码分析 | 咋嘞？你的IDEA过期了吧！加个Jar包就破解了，为什么？`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2020-01-06-[源码分析]咋嘞？你的IDEA过期了吧！加个Jar包就破解了，为什么？.md)
* [`源码分析 | 像盗墓一样分析Spring是怎么初始化xml并注册bean的`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2020-01-08-[源码分析]像盗墓一样分析Spring是怎么初始化xml并注册bean的.md)
* [`源码分析 | 基于jdbc实现一个Demo版的Mybatis`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2020-01-13-[源码分析]基于jdbc实现一个Demo版的Mybatis.md)
* [`源码分析 | 手写mybait-spring核心功能(干货好文一次学会工厂bean、类代理、bean注册的使用)`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-code/2020-01-20-[源码分析]手写mybait-spring核心功能(干货好文一次学会工厂bean、类代理、bean注册的使用).md)

## :sound: Netty4.x专题

- 基础入门篇
    * [`netty案例，netty4.1基础入门篇零《初入JavaIO之门BIO、NIO、AIO实战练习》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-07-30-netty案例，netty4.1基础入门篇零《初入JavaIO之门BIO、NIO、AIO实战练习》.md)
    * [`netty案例，netty4.1基础入门篇一《嗨！NettyServer》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-01-netty案例，netty4.1基础入门篇一《嗨！NettyServer》.md)
    * [`netty案例，netty4.1基础入门篇二《NettyServer接收数据》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-05-netty案例，netty4.1基础入门篇二《NettyServer接收数据》.md)
    * [`netty案例，netty4.1基础入门篇三《NettyServer字符串解码器》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-06-netty案例，netty4.1基础入门篇三《NettyServer字符串解码器》.md)
    * [`netty案例，netty4.1基础入门篇四《NettyServer收发数据》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-07-netty案例，netty4.1基础入门篇四《NettyServer收发数据》.md)
    * [`netty案例，netty4.1基础入门篇五《NettyServer字符串编码器》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-08-netty案例，netty4.1基础入门篇五《NettyServer字符串编码器》.md)
    * [`netty案例，netty4.1基础入门篇六《NettyServer群发消息》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-09-netty案例，netty4.1基础入门篇六《NettyServer群发消息》.md)
    * [`netty案例，netty4.1基础入门篇七《嗨！NettyClient》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-10-netty案例，netty4.1基础入门篇七《嗨！NettyClient》.md)
    * [`netty案例，netty4.1基础入门篇八《NettyClient半包粘包处理、编码解码处理、收发数据方式》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-11-netty案例，netty4.1基础入门篇八《NettyClient半包粘包处理、编码解码处理、收发数据方式》.md)
    * [`netty案例，netty4.1基础入门篇九《自定义编码解码器，处理半包、粘包数据》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-12-netty案例，netty4.1基础入门篇九《自定义编码解码器，处理半包、粘包数据》.md)
    * [`netty案例，netty4.1基础入门篇十《关于ChannelOutboundHandlerAdapter简单使用》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-13-netty案例，netty4.1基础入门篇十《关于ChannelOutboundHandlerAdapter简单使用》.md)
    * [`netty案例，netty4.1基础入门篇十一《netty udp通信方式案例Demo》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-14-netty案例，netty4.1基础入门篇十一《nettyudp通信方式案例Demo》.md)
    * [`netty案例，netty4.1基础入门篇十二《简单实现一个基于Netty搭建的Http服务》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-1/2019-08-15-netty案例，netty4.1基础入门篇十二《简单实现一个基于Netty搭建的Http服务》.md)
  
 - 中级拓展篇
    * [`netty案例，netty4.1中级拓展篇一《Netty与SpringBoot整合》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-16-netty案例，netty4.1中级拓展篇一《Netty与SpringBoot整合》.md)
    * [`netty案例，netty4.1中级拓展篇二《Netty使用Protobuf传输数据》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-17-netty案例，netty4.1中级拓展篇二《Netty使用Protobuf传输数据》.md)
    * [`netty案例，netty4.1中级拓展篇三《Netty传输Java对象》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-18-netty案例，netty4.1中级拓展篇三《Netty传输Java对象》.md)
    * [`netty案例，netty4.1中级拓展篇四《Netty传输文件、分片发送、断点续传》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-19-netty案例，netty4.1中级拓展篇四《Netty传输文件、分片发送、断点续传》.md)
    * [`netty案例，netty4.1中级拓展篇五《基于Netty搭建WebSocket，模仿微信聊天页面》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-20-netty案例，netty4.1中级拓展篇五《基于Netty搭建WebSocket，模仿微信聊天页面》.md)
    * [`netty案例，netty4.1中级拓展篇六《SpringBoot+Netty+Elasticsearch收集日志信息数据存储》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-21-netty案例，netty4.1中级拓展篇六《SpringBoot+Netty+Elasticsearch收集日志信息数据存储》.md)
    * [`netty案例，netty4.1中级拓展篇七《Netty请求响应同步通信》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-22-netty案例，netty4.1中级拓展篇七《Netty请求响应同步通信》.md)
    * [`netty案例，netty4.1中级拓展篇八《Netty心跳服务与断线重连》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-23-netty案例，netty4.1中级拓展篇八《Netty心跳服务与断线重连》.md)
    * [`netty案例，netty4.1中级拓展篇九《Netty集群部署实现跨服务端通信的落地方案》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-24-netty案例，netty4.1中级拓展篇九《Netty集群部署实现跨服务端通信的落地方案》.md)
    * [`netty案例，netty4.1中级拓展篇十《Netty接收发送多种协议消息类型的通信处理方案》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-25-netty案例，netty4.1中级拓展篇十《Netty接收发送多种协议消息类型的通信处理方案》.md)
    * [`netty案例，netty4.1中级拓展篇十一《Netty基于ChunkedStream数据流切块传输》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-26-netty案例，netty4.1中级拓展篇十一《Netty基于ChunkedStream数据流切块传输》.md)
    * [`netty案例，netty4.1中级拓展篇十二《Netty流量整形数据流速率控制分析与实战》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-27-netty案例，netty4.1中级拓展篇十二《Netty流量整形数据流速率控制分析与实战》.md)
    * [`netty案例，netty4.1中级拓展篇十三《Netty基于SSL实现信息传输过程中双向加密验证》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-2/2019-08-28-netty案例，netty4.1中级拓展篇十三《Netty基于SSL实现信息传输过程中双向加密验证》.md)
 
 - 高级应用篇
    * [`手写RPC框架第一章《自定义配置xml》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-3/2019-09-01-手写RPC框架第一章《自定义配置xml》.md)
    * [`手写RPC框架第二章《netty通信》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-3/2019-09-02-手写RPC框架第二章《netty通信》.md)
    * [`手写RPC框架第三章《RPC中间件》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-3/2019-09-03-手写RPC框架第三章《RPC中间件》.md)
    * [`websocket与下位机通过netty方式通信传输行为信息`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-3/2019-12-01-websocket与下位机通过netty方式通信传输行为信息.md)
   
 - 源码分析篇
    * [`netty案例，netty4.1源码分析篇一《NioEventLoopGroup源码分析》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-10-netty案例，netty4.1源码分析篇一《NioEventLoopGroup源码分析》.md)
    * [`netty案例，netty4.1源码分析篇二《ServerBootstrap配置与绑定启动》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-11-netty案例，netty4.1源码分析篇二《ServerBootstrap配置与绑定启动》.md)
    * [`netty案例，netty4.1源码分析篇三《Netty服务端初始化过程以及反射工厂的作用》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-12-netty案例，netty4.1源码分析篇三《Netty服务端初始化过程以及反射工厂的作用》.md)
    * [`netty案例，netty4.1源码分析篇四《ByteBuf的数据结构在使用方式中的剖析》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-13-netty案例，netty4.1源码分析篇四《ByteBuf的数据结构在使用方式中的剖析》.md)
    * [`netty案例，netty4.1源码分析篇五《一行简单的writeAndFlush都做了哪些事》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-14-netty案例，netty4.1源码分析篇五《一行简单的writeAndFlush都做了哪些事》.md)
    * [`netty案例，netty4.1源码分析篇六《Netty异步架构监听类Promise源码分析》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-netty/itstack-demo-netty-4/2019-09-15-netty案例，netty4.1源码分析篇六《Netty异步架构监听类Promise源码分析》.md)    

## :triangular_ruler: DDD领域驱动设计

* [`DDD专题案例一《初识领域驱动设计DDD落地》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-ddd/2019-10-15-DDD专题案例一《初识领域驱动设计DDD落地》.md)
* [`DDD专题案例二《领域层决策规则树服务设计》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-ddd/2019-10-16-DDD专题案例二《领域层决策规则树服务设计》.md)
* [`DDD专题案例三《领域驱动设计架构基于SpringCloud搭建微服务》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-ddd/2019-10-17-DDD专题案例三《领域驱动设计架构基于SpringCloud搭建微服务》.md)

## :electric_plug: 中间件开发

* [`Spring Boot 中间件开发(一)《服务治理中间件之统一白名单验证》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-middleware/2019-12-02-SpringBoot中间件开发(一)《服务治理中间件之统一白名单验证》.md)
* [`开发基于SpringBoot的分布式任务中间件DcsSchedule(为开源贡献力量)`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-middleware/2019-12-08-开发基于SpringBoot的分布式任务中间件DcsSchedule(为开源贡献力量).md)

## :ghost: JavaAgent全链路监控

* [`基于JavaAgent的全链路监控一《嗨！JavaAgent》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-10-基于JavaAgent的全链路监控一《嗨！JavaAgent》.md)
* [`基于JavaAgent的全链路监控二《通过字节码增加监控执行耗时》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-11-基于JavaAgent的全链路监控二《通过字节码增加监控执行耗时》.md)
* [`基于JavaAgent的全链路监控三《ByteBuddy操作监控方法字节码》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-12-基于JavaAgent的全链路监控三《ByteBuddy操作监控方法字节码》.md)
* [`基于JavaAgent的全链路监控四《JVM内存与GC信息》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-13-基于JavaAgent的全链路监控四《JVM内存与GC信息》.md)
* [`基于JavaAgent的全链路监控五《ThreadLocal链路追踪》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-14-基于JavaAgent的全链路监控五《ThreadLocal链路追踪》.md)
* [`基于JavaAgent的全链路监控六《开发应用级监控》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-agent/2019-07-15-基于JavaAgent的全链路监控六《开发应用级监控》.md)

## :art: 架构框架搭建

* [`发布Jar包到Maven中央仓库(为开发开源中间件做准备)`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-frame/2019-12-07-发布Jar包到Maven中央仓库(为开发开源中间件做准备).md)
* [`架构框架搭建(一)《单体应用服务之SSM整合：Spring4 + SpringMvc + Mybatis》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-frame/2019-12-22-架构框架搭建(一)《单体应用服务之SSM整合：Spring4+SpringMvc+Mybatis》.md)
* [`架构框架搭建(二)《Dubbo分布式领域驱动设计架构框体》`](https://github.com/fuzhengwei/itstack/blob/master/docs/notes/itstack-demo-frame/2019-12-31-架构框架搭建(二)《Dubbo分布式领域驱动设计架构框体》.md)

---

##  转载分享

建立本开源项目的初衷是基于个人学习与工作中对 Java 相关技术栈的总结记录，在这里也希望能帮助一些在学习 Java 过程中遇到问题的小伙伴，如果您需要转载本仓库的一些文章到自己的博客，请按照以下格式注明出处，谢谢合作。

```
作者：小傅哥
链接：https://bugstack.cn
来源：bugstack虫洞栈
```

## 与我联系

- **加群交流**
    本群的宗旨是给大家提供一个良好的技术学习交流平台，所以杜绝一切广告！由于微信群人满 100 之后无法加入，请扫描下方二维码先添加作者 “小傅哥” 微信(fustack)，备注：加群。
    
    <img src="https://itstack.org/_media/fustack.png?x-oss-process=style/may" width="180" height="180"/>

- **公众号(bugstack虫洞栈)**
    沉淀、分享、成长，专注于原创专题案例，以最易学习编程的方式分享知识，让自己和他人都能有所收获。目前已完成的专题有；Netty4.x实战专题案例、用Java实现JVM、基于JavaAgent的全链路监控、手写RPC框架、DDD专题案例、源码分析等。
    
    <img src="https://itstack.org/_media/qrcode.png?x-oss-process=style/may" width="180" height="180"/>

## 参与贡献

1. 如果您对本项目有任何建议或发现文中内容有误的，欢迎提交 issues 进行指正。
2. 对于文中我没有涉及到知识点，欢迎提交 PR。

## 致谢

感谢以下人员对本仓库做出的贡献，当然不仅仅只有这些贡献者，这里就不一一列举了。如果你希望被添加到这个名单中，并且提交过 Issue 或者 PR，请与我联系。

<a href="https://github.com/linw7">
    <img src="https://avatars0.githubusercontent.com/u/3761578?s=460&v=4" style="border-radius:5px" width="50px">
</a> 
<a href="https://github.com/g10guang">
    <img src="https://avatars0.githubusercontent.com/u/30902679?s=400&v=4" style="border-radius:5px" width="50px">
</a> 
<a href="https://github.com/g10guang">
    <img src="https://avatars1.githubusercontent.com/u/15908832?s=180&v=4" style="border-radius:5px" width="50px">
</a>
<a href="https://github.com/g10guang">
    <img src="https://avatars2.githubusercontent.com/u/24491006?s=460&v=4" style="border-radius:5px" width="50px">
</a> 
<a href="https://github.com/g10guang">
    <img src="https://avatars1.githubusercontent.com/u/57205940?s=180&v=4" style="border-radius:5px" width="50px">
</a>
