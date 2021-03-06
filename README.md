# hirpc-java
hirpc是一个跨语言的服务治理rpc框架，hirpc-java是其java版本，其拥有很多创新性的设计。

长久以来，跨语言rpc与服务治理一直不能有效结合，比如谷歌的grpc支持跨语言，但是其并不支持服务治理，仍要对其进行二次开发，
并且grpc也并不简洁易用，其对Spring没有做任何支持。在java中阿里的dubbo鼎鼎有名，支持服务治理，但是其并不支持跨语言。所以本框架由此而生，
将会尝试将跨语言rpc与服务治理进行结合，提供出一个具备服务治理功能的跨语言rpc框架，并且简洁易用。
# 总体流程
![image](https://github.com/mwq0106/hirpc-java/blob/master/assert/QQ%E6%88%AA%E5%9B%BE20191218202933.png)
# 特性
- 支持跨语言的rpc，基于ProtoBuf实现，底层协议对跨语言提供了支持，
在进行跨语言rpc时只需编写方法参数对应的.proto文件，并且编译成对应语言的实体类即可进行rpc调用，
而不需要像grpc一样还需安装对应语言的插件然后再通过插件进行编译，同时编写.proto文件的方式也更简单。
相比grpc，此种方式更加的简单易懂，学习成本也更低。
- 同语言内的rpc无需编写.proto文件，同语言内的rpc并没有涉及到跨语言，所以并不需要ProtoBuf的支持，
同语言内方法参数可以灵活定义，只有在有跨语言的需求时才需要编写.proto文件
- 服务注册，服务发现与服务治理，基于ZooKeeper实现了服务治理的核心功能，当提供服务的机器上线时，
会向服务注册中心（即ZooKeeper）注册服务信息，服务消费方可以在服务注册中心进行服务发现，
获取提供服务的机器列表，当提供服务的机器下线时，服务注册中心会删除该机器的相关信息
- 负载均衡机制，框架实现了轮询，随机，一致性哈希等负载均衡机制
- 失败机制，消费端如果发生了服务调用失败的情况，则会触发相应的失败机制，包括快速失败，故障重试，故障转移
- 注解式服务注册与服务发现，在java版本中与spring深度结合，可在springBoot项目中使用注解的方式完成服务注册与服务注入，
全面的提高开发的效率与便捷性，抛弃传统的xml配置。
- 多种序列化方式支持，在进行跨语言rpc时，将会使用ProtoBuf作为序列化框架，当进行非跨语言rpc时，
在java中将会使用kryo作为序列化框架
- 自定义rpc协议，能实现跨语言的rpc，除了使用了ProtoBuf提供支持，还有另外一个关键因素就是自定义传输层的rpc协议，
协议的设计充分考虑到了跨语言的需求以及未来的拓展，随着未来的发展协议可能还会作出相应变化
- TCP长连接及心跳机制
# 使用
- 在sample模块中已经给出使用示例，本地只需运行zookeeper，然后运行consumer与provider模块即可跑起本项目
- 开发的基本流程：

1.在api中定义一个接口
```java
public interface DemoService {
    Test2.Person hello(Test2.Person person1, Test2.Person person2);
    String hello2(String name);
}
```
2.在provider中实现该接口并且加上注解@RpcService
```java
@RpcService
public class DemoServiceImpl implements DemoService{
    @Override
    public Test2.Person hello(Test2.Person person1,Test2.Person person2){
        return "hi" + xxx1 + xxx2;
    }
    @Override
    public String hello2(String name){
        return "hi," + name;
    }
}
```
3.在consumer中使用注解@RpcReference即可完成依赖注入及服务引用
```java
@RestController
public class DemoController {
    @RpcReference
    private DemoService demoService;

    @RequestMapping("/sayHello")
    public String sayHello(@RequestParam String name) {
        Test2.Person person = demoService.hello(xx,xxx);
        return person.getName();
    }
    @RequestMapping("/sayHello2")
    public String sayHello2(@RequestParam String name) {
        return demoService.hello2(name);
    }
}
```
4.运行zookeeper环境

5.运行consumer与provider模块

6.本地访问http://localhost:8082/sayHello?name=aa
