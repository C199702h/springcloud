# springcloud
springcloud
单体架构： 将业务的所有功能集中在一个项目中开发，打成一个包部署
优点： 架构简单、部署成本低 ； 缺点： 耦合度高
项目打包部署到Tomcat，用户直接访问。用户量增加后就多部署几台服务器形成集群。
![image](https://github.com/chenghang0202/springcloud/assets/134860202/80855267-8d7d-41f4-9c76-7384db30779a)

分布式架构： 根据业务功能对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务。
优点： 降低服务耦合、有利于服务升级拓展 ；

由于分布式架构的特性我们也出现了一些思考：
服务拆分粒度如何?
服务集群地址如何维护?
服务之间如何实现远程调用?
服务健康状态如何感知?
为了解决分布式带来的问题，微服务出现了

微服务是一种经过良好架构设计的分布式架构方案，微服务架构特征:
单一职责： 微服务拆分粒度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
面向服务： 微服务对外暴露业务接口
(由于不同模块部署在不同服务器、无法直接调用)
自治： 队独立、技术独立、数据独立、部署独立
隔离性强： 服务调用做好隔离、容错、降级，避免出现级联问题
(避免某个模块宕机造成影响)


单体架构特点?
简单方便，高度耦合，扩展性差，适合小型项目。例如: 学生管理系统

分布式架构特点?
松耦合，扩展性好，但架构复杂，难度大。适合大型互联网项目，例如:京东、淘宝

微服务:一种良好的分布式架构方案
优点: 拆分粒度更小、服务更独立、耦合度更低缺点:架构非常复杂，运维、监控、部署难度提高


在微服务架构中，配置中心和注册中心是两个重要的组件。
配置中心用来统一管理项目中所有配置，各种参数、各种开关，全部都放到一个集中的地方进行统一管理，并提供一套标准的接口。当各个服务需要获取配置的时候，就来「配置中心」的接口拉取。
注册中心则是用来管理服务实例的注册和发现的。各个服务实例在启动时会向注册中心注册自己的信息（如IP地址和端口号），其他服务实例可以通过注册中心来发现并调用这些服务12。

![image](https://github.com/chenghang0202/springcloud/assets/134860202/efb07970-1dc8-419f-af3b-daba572ce0e6)

![image](https://github.com/chenghang0202/springcloud/assets/134860202/42d4878a-2a47-4639-82a9-5cb70e8a1e3b)

![image](https://github.com/chenghang0202/springcloud/assets/134860202/186f52fd-1b80-4c5f-9573-721062df9f02)

1.不同微服务，不要重复开发相同业务
2.微服务数据独立，不要访问其它微服务的数据库
3.微服务可以将自己的业务暴露为接口，供其它微服务调用

1.基于RestTemplate发起的http请求实现远程调用
2.http请求做远程调用是与语言无关的调用，只要知道对方的ip、端口、接口路径、请求参数即可。


服务提供者： 一次业务中，被其它微服务调用的服务。(提供接口给其它微服务)
服务消费者： 一次业务中，调用其它微服务的服务。(调用其它微服务提供的接口)
一个服务既可以是提供者也可以是消费者，要根据具体的业务和情况来判断

![image](https://github.com/chenghang0202/springcloud/assets/134860202/cdf55957-f9f1-4d56-bc3f-d716fdc11061)

消费者该如何获取服务提供者具体信息?
①：服务提供者启动时向eureka注册自己的信息
②：eureka保存这些信息
③：消费者根据服务名称向eureka拉取提供者信息

如果有多个服务提供者，消费者该如何选择?
①：服务消费者利用负载均衡算法，从服务列表中挑选一个

消费者如何感知服务提供者健康状态?
①：服务提供者会每隔30秒向EurekaServer发送心跳请求，报告健康状态
②：eureka会更新记录服务列表信息，心跳不正常会被剔除
③：消费者就可以拉取到最新的信息

在Eureka架构中，微服务角色有两类
EurekaServer：服务端，注册中心
记录服务信息
心跳监控

EurekaClient:客户端
Provider: 服务提供者，例如案例中的 user-servicea
注册自己的信息到EurekaServer
每隔30秒向EurekaServer发送心跳
consumer:服务消费者，例如案例中的 order-service根据服务名称从EurekaServer拉取服务列表
基于服务列表做负载均衡，选中一个微服务后发起远程调用


1.搭建EurekaServer
引入eureka-server依赖
添加@EnableEurekaServer注解
在application.yml中配置eureka地址
2.服务注册
引入eureka-client依赖
在application.yml中配置eureka地址
3.服务发现
引入eureka-client依赖
在application.yml中配置eureka地址
给RestTemplate添加@LoadBalanced注解
用服务提供者的服务名称远程调用

整个过程当中我们只需要配置好Eureka，就自动做到了负载均衡、拉取服务，那么这些事情到底是怎么完成的呢？
![image](https://github.com/chenghang0202/springcloud/assets/134860202/f3aa3703-f8dc-4e32-9dc6-a90b4e72a187)
Ribbon的负载均衡规则是一个叫做IRule的接口来定义的，每一个子接口都是一种规则

1.Ribbon负载均衡规则
规则接口是IRule
默认实现是ZoneAvoidanceRule，根据zone选择服务列表，然后轮询

2.负载均衡自定义方式
代码方式:配置灵活，但修改时需要重新打包
配置方式:直观，方便，无需重新打包发布但是无法做全局配置

3.饥饿加载
开启饥饿加载
指定饥饿加载的微服务名称

根据权重负载均衡
实际部署中会出现这样的场景
服务器设备性能有差异，部分实例所在机器性能较好，另一些较差，我们希望性能好的机器承担更多的用户请求
Nacos提供了权重配置来控制访问频率，权重越大则访问频率越高

包括服务器的升级、我们可以使用权重来使得某个微服务无人访问，然后停机升级。这样也不会影响正在使用的用户

实例的权重控制
Nacos控制台可以设置实例的权重值，0~1之间
同集群内的多个实例，权重越高被访问的频率越高
权重设置为0则完全不会被访问

acos中服务存储和数据存储的最外层都是一个名为namespace的东西，用来做最外层隔离


Nacos环境隔离
namespace用来做环境隔离
每个namespace都有唯一id
不同namespace下的服务不可见
![image](https://github.com/chenghang0202/springcloud/assets/134860202/cc4c4037-2ffc-427f-a828-68e0f7801b0f)

![image](https://github.com/chenghang0202/springcloud/assets/134860202/8716d626-f09d-423d-9d50-8d54465e6621)

临时实例的情况下，如果你终止程序，过30s，到nacos中查看就会发现爆红然后直接消失(被nacos踢出)
非临时实例终止程序，nacos中查看该服务爆红，但不会踢出。重新启动非临时实力即可

1.Nacos与eureka的共同点
   都支持服务注册和服务拉取
   都支持服务提供者心跳方式做健康检测
2.Nacos与Eureka的区别
   Nacos支持服务端主动检测提供者状态:临时实例采用心跳模式，非临时实例采用主动检测模式
   临时实例心跳不正常会被剔除，非临时实例则不会被剔除
   Nacos支持服务列表变更的消息推送模式，服务列表更新更及时
   Nacos集群默认采用AP方式，当集群中存在非临时实例时，采用CP模式 ； Eureka采用AP方式

1.搭建MySQL集群并初始化数据库表
2.下载解压nacos
3.修改集群配置(节点信息)、数据库配置
4.分别启动多个nacos节点
5.nginx反向代理

方式一是配置文件，feign.client.config.xxx.loggerLevel
如果xxx是default则代表全局
如果xxx是服务名称，例如userservice则代表某服务
方式二是java代码配置Logger.Level这个Bean
如果在@EnableFeignClients注解声明则代表全局
如果在@FeignClient注解中声明则代表某服务

Feign性能调优
Feign底层的客户端实现
URLConnection： 默认实现，不支持连接池
Apache HttpClient： 支持连接池
OKHttp： 支持连接池
我们知道没有连接池的情况下，需要每次都重连和断开，影响性能

因此优化Feign的性能主要包括:
使用连接池代替默认的URLConnection
日志级别，最好用basic或none

![image](https://github.com/chenghang0202/springcloud/assets/134860202/7e157592-6a4d-4b08-af55-f16124473070)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/c60c75c4-e628-4f42-8090-2eed18d41db3)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/43042f08-a785-445f-ad19-4728e8dffa68)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/67af23e5-7fc4-4c13-873c-90de7ade3355)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/ac43d1d0-8e46-476e-8dbf-3e0810fa479e)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/de8b4702-02e2-4b90-8551-0eeebb534cd6)
断言工厂是用来匹配请求的，比方说有很多微服务交由网关管理。
每个微服务都有不同的断言 工厂配置，有的微服务必须几点之前、有的微服务必须什么IP
当前端发来URL，请求的时候，URL会跟配置中比较
满足断言工厂配置条件的才能找到对应的服务并响应
![image](https://github.com/chenghang0202/springcloud/assets/134860202/71ced1fe-34db-445b-851d-280bf9bf5917)
![image](https://github.com/chenghang0202/springcloud/assets/134860202/5d752377-2055-4bc4-a17f-021d9c1957a8)
请求进入网关会碰到三类过滤器： 当前路由的过滤器、DefaultFilter、GlobalFilter
请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链 (集合)中，排序后依次执行每个过滤器

1、 每一个过滤器都必须指定一个int类型的order值，order值越小，优先级越高，执行顺序越靠前
2、 GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
3、 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增
4、 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器>GlobalFilter的顺序执行

跨域问题处理
**跨域：**域名不一致就是跨域，主要包括：
域名不同： www.taobao.com 、www.taobao.org
域名相同： 端口不同： localhost:8080、 localhost:8081
跨域问题： 浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题
解决方案： CORS
![image](https://github.com/chenghang0202/springcloud/assets/134860202/210c47c6-64e9-48bd-867e-8c6a49e7119f)


初始Docker
Docker是一个快速交付应用、运行应用的技术
1.可以将程序及其依赖、运行环境一起打包为一个镜像
可以迁移到任意Linux操作系统
2.运行时利用沙箱机制形成隔离容器，各个应用互不干扰
3.启动、移除都可以通过一行命令完成，方便快捷
![image](https://github.com/chenghang0202/springcloud/assets/134860202/0d2fe71f-c267-4e33-979c-3d68ef907114)

Docker与虚拟机
Docker和虚拟机的差异
docker是一个系统进程 ； 虚拟机是在操作系统中的操作系统
docker体积小、启动速度快、性能好 ； 虚拟机体积大、启动速度慢、性能一般
镜像和容器
镜像(Image)： Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。
容器(Container)： 镜像中的应用程序运行后形成的进程就是容器，只是Docker会给容器做隔离，对外不可见

镜像和容器
Docker是一个CS架构的程序，由两部分组成:
服务端(server)： Docker守护进程，负责处理Docker指令，管理镜像、容器等
客户端(client)： 通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令








https://blog.csdn.net/u014685437/article/details/130919452?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522171214894316800185896085%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=171214894316800185896085&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-130919452-null-null.142^v100^pc_search_result_base8&utm_term=spring%20cloud&spm=1018.2226.3001.4187
