# 1. Algorithm: 一道leetcode的算法题
```
int removeDuplicates(vector<int>& nums) 
{
        if(nums.size() < 2)
            return nums.size();
        
        int n = 1;
        for(int i = 1;i < nums.size();i++)
            if(nums[i] != nums[i-1])
                nums[n++] = nums[i];
        return n;
}
```
# 2. Review:阅读并点评一篇英文技术文章
https://www.nginx.com/blog/introduction-to-microservices/
这篇文章中介绍了微服务,从传统的单体应用(Monolithic Applications)存在的问题以及微服务是如何解决的以及微服务的优缺点.
单体应用的好处是适合于初创企业的快速迭代,有
1. 易于开发,IDEs的支持
2. 易于测试,有很多工具支持,如Selenium
3. 易于部署,以一个WAR文件打包,之后运行在容器中,如Tomcat或者一些JARs,Node.js打包成一个目录结构
4. 易于扩展,可以运行多个实例,然后使用负载均衡进行路由
业务快速发展之后,单体应用会变得很大很复杂,各个模块之间相互调用,相互耦合,这严重违反了低耦合高内聚原则.
单体应用变得很大很复杂之后存在的问题
1. 对任何一个开发者, 应用太过复杂而难以理解
2. 修复bug和添加新功能很难,也很耗时,这会形成一个恶心循环,打击团队士气
3. 降低开发速度,单体应用很大之后,IDEs的启动时间会更长,而且便宜时间也会更久
4. 持续部署难度很大,每次因为一个小小的改动就需要重新部署整个应用,这对单体应用异常困难
5. 难于扩展,一些模块如CPU密集型应用可能需要部署在EC2 Compute Optimized instances上,而另外一些如内存数据库(NoSQL)之类的会需要部署在 EC2 Memory‑optimized instances上,然而这些模需要部署在同一台机器上,你会选择一个折中的方案.
6. 可靠性低. 如果某个模块因为内存泄露或者一个运行时的bug会导致整个应用不可访问.
7. 难以使用新框架新语言.

Microservice解决了复杂性问题,不过本身也存在一些不足
微服务是把应用拆分为一些更小粒度,相互协调的服务,服务之间通过API/REST API进行交互.
微服务的一个重要影响是应用程序和数据库之间的关系,单体应用通常是共享数据库的,而微服务为了确保松耦合,一些微服务都有自己的数据库. 
1. 微服务解决了单体应用的复杂度, 它将单体应用拆分成若干服务,这些服务以RPC或者消息驱动的API的形式定义,这样每个服务就可以由不同的团队来维护,也更易于开发,理解和维护.
2. 每个服务独立地部署,这使得持续部署成为现实
3. 每个服务可以独立地扩展,可以根据服务的具体需求进行独立地扩展,比如可以在EC2 Compute Optimized上部署CPU密集型的服务,可以在EC2 Memory‑optimized上部署内存数据库服务.

微服务的不足
1. 微服务的大小问题,什么样的服务算是微服务(记住微服务的目标是使agile的开发和部署更容易.)
2. 微服务的复杂性,这是由于微服务也是分布式系统的原因.
3. 使用不同数据库架构带来的挑战.有的服务使用关系型数据库,而有些使用内存数据库
4. 微服务的测试变得复杂了
5. 微服务的部署变得复杂了

# 3. Tip---学习至少一个技术技巧
Meyers 令operator=返回一直reference to *this,这在重在操作符=,+=时需要考虑到Meyers 令operator=返回一直reference to *this.
# 4. Share:分享一篇有观点和思考的技术文章
https://www.gartner.com/en/research/methodologies/gartner-hype-cycle
我觉得大家有必要了解下Gartner Hype Cycle技术成熟度曲线,这对于后期的个人发展会有些好处