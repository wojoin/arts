# 1. Algorithm: 一道leetcode的算法题
```
167. Two Sum II
vector<int> twoSum2(vector<int>& numbers, int target)
{
	vector<int> result;

	int low = 0;
	int high = numbers.size() - 1;

	for (int index = 0;index < numbers.size();index++)
	{
		if (numbers[low] + numbers[high] == target)
		{
			result.push_back(low + 1);
			result.push_back(high + 1);
			break;
		}
		else if (numbers[low] + numbers[high] < target)
			low++;
		else
			high--;
	}

	return result;
}

```

# 2. Review:阅读并点评一篇英文技术文章
### 3. Building Microservices: Inter-Process Communication in a Microservices Architecture

https://www.nginx.com/blog/building-microservices-inter-process-communication/

单体应用中一个功能调用另外一个功能是通过方法调用,这通常实在语言级别. 而微服务应用是个分布式系统,每个服务实例通常是个进程,那么这些服务之间的交互需要用到Inter-process communication(IPC)机制.

再看IPC之前,先考虑下设计中存在问题.
### 交互方式
C/S之间的交互方式有很多种,总体上来说可以分为两类
一类是
- one-to-one一对一的交互,一个客户端请求被某个服务处理
- one-to-many一对多的交互,一个客户端请求被多个服务处理
另外一个分类是
- 同步方式,客户端发起请求后能及时收到服务端的响应,客户端发起请求后处于阻塞状态,等待服务端结果
- 异步方式,客户端在等待服务端响应时不阻塞,通过一定的机制将响应通知客户端,常用的消息机制

 category  | one-to-one | one-to-many
---|---|---
Synchronous  | request/response | --
Asynchronous | notification | publish/subscribe
Asynchronous | request/ async response| publish/subscribe

one-to-one的交互方式
- request/response,客户端发起请求后等待服务器响应
- notification,客户端发请求后,响应由服务的通知客户端
- request/async response, 客户端发起请求后不阻塞,响应由服务端通知

one-to-many的交互方式
- publish/subscribe,客户端发起一个通知消息, 由感兴趣的服务端对消息进行处理
- Publish/async responses,客户端发起一个请求消息,之后在timneout内等待服务端响应

服务之间的交互通常会使用多种方式.

**API的定义**
在进行开发具体服务之前,最好先定义好服务端与客户端之间的接口API,这使得服务端能更好的满足客户端的请求.
API的定义本质上依赖与你使用的是哪一种IPC机制,如果使用基于消息的IPC,那么API由消息通道和消息类型构成;如果使用HTTPS,那么API由URL请求和相应格式构成.

**API的演化**
当服务升级时,最好采用增量式部署服务,这样新版本和旧版本可以同时运行.
对于需要改动接口以增加程序健壮性,新加的属性要有默认值以便客户端可以忽略这些新加的属性.
在做了一个主版本升级时,可能会存在一些接口没有向后兼容,比如使用基于HTTP机制的REST,这时候可以在URL中嵌入服务的版本号访问服务.

**部分失败的处理**Handling Partial Failure
客户端访问服务端正常情况下会得到响应,有时候可能由于服务挂了或者请求量太多导致服务处理变慢,响应也变得慢了,或者在服务端一个服务在调用另外一个服务时出现失败. 当服务不可用时,结果该如何返回?
下面列出一些原因和相应的方案
- Network timeouts,等待响应式不要无限制地等待下去,要设置一个timeouts,
- Limiting the number of outstanding requests限制未完成的请求量,对特定的服务需要限制客户端的未完成请求量. 达到请求限制上限时,告知客户端失败原因
- Circuit breaker pattern,断路器模式,统计成功和失败的请求数. 当错误率达到配置文件中的阈值时,触发断路器使后续的访问立即返回失败(此时可能服务有问题了或者请求无意义),超过一定的timeout时间后,客户端应该重新尝试访问,如果成功了需要关系断路器.
- Provide fallbacks,预备方案,请求失败后执行一个预备方案,如请求一个产品相信列表中的推荐信息时,如果推荐信息为空,此时可以返回默认值或者缓存数据.

**Netflix Hystrixs**是一个实现这些方案的一个开源库

### IPC技术
synchronous,request/response‑based: REST, Thrift
asynchronous,message‑based: AMQP, STOMP
消息格式: JSON, XML,Protocol Buffers

**基于消息的异步通信**
消息由消息头(元信息)和消息体构成.
消息在通道上传输,在通道中发送消息的一端称为producer,接收消息的一端称为comsumer. 有两种类型的通道.
point-to-point和publish-subscribe. point-to-point用于one-to-one的交互方式,某个特定的消费者处理消息.
publish-subscribe则用于one-to-many的交互方式,由感兴趣的消费者处理消息.
有很多这样的消息系统,如RabbitMQ, Kafka

使用消息系统优点
- 将客户端从服务端中解耦, 客户端只需要发送消息给消息系统,而无须关心服务端服务实例的存在.
- 消息缓存,使用队列对消息进行缓存,而不用保证像单体应用中保持客户端与服务端都可用
- 客服端服务端交互的灵活性,多种方式交互,one-to-one和one-to-many.

使用消息系统的不足
- 增加了复杂性,因为消息系统是个独立的系统,需要而外地安装,配置等,也需要保证消息系统高可用,否则系统的可靠性会受到影响
- Complexity of implementing request/response‑based interaction, 每个请求消息包含了通道ID和相关联的(消息)ID,客户端用消息ID查找响应消息.

**请求响应的同步IPC**
在请求响应的同步IPC中,客户端等待服务端的响应. 不过也有使用事件驱动的异步模式.
这样的主流协议有REST和Thrift.

### 消息格式
主要有两类XML和JSON

# 3. Tip---学习至少一个技术技巧
学习了两种高效的事件处理模式:Reactor和Proactor模式.
其中涉及到I/O模型(主要用I/O复用),队列,回调函数,多线程知识
http://blog.jobbole.com/59676/

https://www.zhenchao.org/2017/10/23/reactor-pattern/

# 4. Share:分享一篇有观点和思考的技术文章
看了极客专栏关于微服务的文章,关于服务追踪的相关问题

https://time.geekbang.org/column/article/15273