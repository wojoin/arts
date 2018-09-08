# 1. Algorithm: 一道leetcode的算法题
```
28. Implement strStr().
int strStr(string haystack, string needle) 
{
	if (needle.empty())
		return 0;
	if (haystack.empty() || haystack.size() < needle.size())
		return -1;

	int srclength = haystack.size();
	int length = needle.size();
	for (int i = 0;i < srclength;i++)
	{
		int index = i;
		int j = 0;
		while (haystack[index] == needle[j]){
			if (j == length - 1)
			{
				return i;
			}
			j++;
			index++;

		}
		//for (int j = 0;j < length;j++)	// Time Limit Exceeded 14737 more chars的情况下
		//{
		//	/*if (strncmp(haystack.substr(i, length).c_str(), needle.c_str(), length) == 0)
		//	{
		//		return i;
		//	}*/
		//}
	}
	return -1;
}
```

# 2. Review:阅读并点评一篇英文技术文章
## Building Microservices: Using an API Gateway
https://www.nginx.com/blog/building-microservices-using-an-api-gateway/

假设有个购物的App，现在需要过去产品的详细信息，展示产品详细信息的页面会涉及到很多信息，如
- 产品基本信息
- 物流信息
- 用户评论信息
- 产品推荐信息
对于单体应用Monolithic Application，客户端只需要发起一个REST请求(GET api.company.com/productdetails/productId) ，之后负载均衡把请求路由给后端程序，查询数据库表后返回结果给客户端。在微服务架构的应用程序中，不同的信息由不同的微服务展示，如
- 购物车服务展示产品的数量
- 物流服务展示用户物流信息，如收货地址，时间等
- 评论服务展示购买该产品的评论
- 推荐服务展示了相关的个性化推荐产品
这种情况下该如何访问这些服务获取信息呢

理论上，客户端可以通过URL向每个微服务发起请求，但是通过这种方式来访问的话会有不少困难和限制
- 客户端需要的信息与微服务返回结果之间的不匹配
- 调用微服务的协议可能并不是适用于web的(web-friendly)，如有些微服务使用Thrift binary RPC进行交互，而有这使用AMQP message protocol交互，客户端最好使用HTTP或者WebSocket进行交互
- 重构微服务很困难，在开发中发现合并两个微服务为一个或者拆分一个为两个，这时候如果客户端直接访问微服务的话会不够友好

# Using an API Gateway
这种情况下使用API Gateway是一种更好的方式,它是一个单入口端点,类似于OOD中的Facade设计模式,对外的每一类型客户端(Web, Mobile)提供合适的API. 这些API可能还有验证授权,监控,负载均衡,缓存等功能. API Gateway负责
- 路由转发(request routing),客户端的请求都被API Gateway路由到特定的微服务.
- 请求结果汇总(composition),调用多个微服务时需要讲结果汇总后发到客户端.
- 协议转换(protocol translation),可以将web-friendly的协议请求如HTTP,WebSocket的请求转换成用于微服务内部通信的web-unfriendly的协议,如Thrift binary RPC等.

API Gateway给客户端一个特定的API,客户端通过调用这个API即可获得客户端想要的信息,如API Gateway暴露一个产品详情的API,/productdetails?productid=xxx,而这个API内部可能调用了多个微服务将结果汇总后才能得到客户端想要的详情信息.

## API Gateway的优缺点
API Gateway的主要优点时封装了微服务的内部结构.客户端仅仅需要和API Gateway交互即可,这也简化了客户端代码.API Gateway为每一类型的客户端提供特定的API,这也减少了微服务和客户端之间的往返时间(round trips)[client and application vs. internal microservice].
缺点也很明显, 
- 它必须是一个高可用的组件,否则API Gateway出问题,服务也没法访问.
- 这也存在很大的风险,成为开发中的瓶颈.具体地说是开发了若干微服务,此时需要跟新API,这就要求API Gateway本身尽可能地轻量级.

# API Gateway的实现(实现一个API Gateway时需要考虑的因素)
- 性能和可扩展性
- 使用响应式编程模型
- 服务调用
    - 基于消息的异步机制,如AMQP,Zeromq,
    - 同步调用机制,如HTTP,Thrift
- 服务发现(IP address and port)
    - Client-Side Discovery, 如Service Registry
    - Server-Side Discovery
- Handling Partial Failures,这是所有分布式服务中都存在的问题,当一个服务调用另外一个服务时,相应慢或者没有响应时改如何处理.
    - 当服务响应时间超过阈值或者没有响应时,可以选择返回默认值或者从缓存中返回值.

# 3. Tip---学习至少一个技术技巧
https://github.com/wojoin/cpp11

学习了Callback的使用并实现一个Callback的demo


# 4. Share:分享一篇有观点和思考的技术文章
https://www.zhihu.com/question/19801131

这里面详细介绍了回调函数Callback的概念,对Callback的理解加深了,并自己实现了一个Callback的Demo.
Callback的概念:(1)起始函数(2)中间函数(注册回调函数的)(3)回调函数
在解释了回调函数的概念后,还进一步介绍了阻塞式回调和延迟式回调.
回调函数机制比普通函数的相互调用更为灵活,通过注册不同的回调函数,来决定、改变中间函数的行为.
