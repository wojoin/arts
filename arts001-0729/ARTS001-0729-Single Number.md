## 1. Algorithm: 一道leetcode的算法题
本周算法题是137.Single Number II
一个非空整形数组，其中每个元素出现三次和唯一一次的，找出那个出现唯一一次的元素，如[3,3,5,3]找出那个5.

### 思路
使用一个map进行key-value的统计，如果key存在则增加value的值，不存在就插入。
```
int singleNumber2(const vector<int>& v)
{
	map<int, int> m;
	for (int i = 0; i < v.size(); i++)
	{
		if (m.find(v[0]) == m.end())
		{
			m.insert(std::pair<int, int>(v[i], 1));
		}
		else
		{
			m[v[i]]++;
		}
	}

	map<int,int>::iterator it = m.begin();
	for (;it != m.end();it++)
	{
		if ( it->second % 3 == 1)
			return it->first;
	}
}
```

后来看了网上有另外以一种解法，是关于位操作的解法.
因为是整数，我们可以建立数组来统计每一位上出现1的次数，因为整数要么出现三次，要么一次，所以第i位上出现1的次数对3取余要么是0要么是1，1的则是那个出现一次的整数

```
int singleNumber(int A[], int length)
	{
		int count[32] = { 0 };
		int result = 0;
		// thinking from bit.
		for (int i = 0; i < 32; i++)
		{
			for (int j = 0; j < length; j++)
			{
				// 统计每一个位上的1个数
				if ((A[j] >> i) & 0x1)
					count[i]++;
			}
			result |= ((count[i] % 3) << i); //结果需要异或，需要记录出现一次的整数每一个位上的1
		}
		return result;
	}
```

## 2. Review:阅读并点评一篇英文技术文章
本周阅读的英文技术文章是关于Deep Learning中的一篇
[link](https://medium.com/ub-women-data-scholars/introduction-to-deep-learning-bcae684488e0)
文章中想给出了Deep Learning的定义以及在AI中所处的一个位置，是属于Meachine Learning 的子集。随后给出一个最简单的感知器perceptron的工作方式，由此引入了sigmoid的分类函数，这是分类中的一个最基本也是很重要的一个分类函数。由此引入了目前Neural Network(NN)的架构图，一共三层，Input Layer,Hidden Layer, Output Layer,这三层也和平常计算机处理数据的模型一直，Input，Process,Output,不同的地方是在这个Process的过程.在Deep Learning 重要也在Hidden Layer这一层，在这一层里隐藏了很多处理方法,Feedforward NN, Back Propagation , Full Connection(FC), Convolutional NN(CNN),这些隐藏层很多都是这些不同NN的组合以及一些优化方法。
之后引入了一些Neural Network的类型，有Feedforward NN, CNN, Recurrent NN,其中CNN主要用于计算机视觉CV方向，RNN用于语音识别text to speech,文本转语音，之后介绍了一些Deep Learning的一些开源库，TensorFlow, Caffe, Torch等，理解原理后用好一个库之后再用其他库很容易，一开始可以着重使用其中一个，推荐使用TensorFlow, Google的一个开源库，Trust it 就好了。

## 3. Tip
wait(status) & exit(status)中status的分析

父进程如何知道子进程是以何种方式退出的呢? 子进程的退出方式有正常的退出exit(0),或者是被一个信号杀死,信号可能来自键盘,内核,他进程.

答案在父进程中调用wait的参数之中.父进程调用wait时传递一个整型变量的地址给函数,内核将子进程的退出状态保存在这个变量中. 如果子进程调用exit退出,那么内核会把exit返回值存放到这个变量中; 如果子进程是被杀死的,那么内核将信号序号存放在这个变量中.

这个整型变量由三部分组成 -- 高位的 8个bit是记录退出值 -- 低位的 1个bit用来知名发生错误并产生了内核映像(core dump) -- 低位的 7个bit是记录信号序号
```
#include <unistd.h>
#include <stdio.h>

#define DELAY 2
void child(int delay);
void parent(int childpid);


int main(int argc, char const *argv[])
{
	
	int childpid;
	printf("before: mypid is %d\n", getpid());

	childpid = fork();

	if(childpid == 0)
		child(DELAY);
	else if(childpid > 0)
		parent(childpid);


	return 0;
}


void child(int delay)
{
	printf("child %d will sleep for %d seconds\n", getpid(), delay);
	sleep(2);
	printf("child done. about to exit\n");
	exit(17);
}

void parent(int childpid)
{
	int waitret;
	int child_status;
	int high8,low7,bit7;

	waitret = wait(&child_status);
	printf("done waiting for %d. Wait return: %d\n", childpid, waitret);

	high8 = child_status >> 8;  // 左移8位相当于除2^8, 原始的实现是 (status & 0xff00) >> 8
	bit7 = child_status & 0x80; // 取status中低八位中的高位 1 bit
	low7 = child_status & 0x7F; // 取status中低八位中的低位 7 bit
	printf("status: exit = %d, sig = %d, core = %d\n", high8, low7, bit7);
}
```


也可以通过文件`sys/wait.h`中的宏`WEXITSTATUS`,`WCOREDUMP`,`WTERMSIG` 这三个宏位于文件`sys/wait.h`中 查看文件可以看到这个三个宏调用了另外带双下划线的宏,而这些宏位于`bits/waitstatus.h`文件中,在这个文件中 可以再看如下内容(截取一段)
```
union wait
  {
    int w_status;
    struct
     {
# if    __BYTE_ORDER == __LITTLE_ENDIAN
        unsigned int __w_termsig:7; /* Terminating signal.  */
        unsigned int __w_coredump:1; /* Set if dumped core.  */
        unsigned int __w_retcode:8; /* Return code if exited normally.  */
        unsigned int:16;
    }
}
```

由此可知status使用了16bit的整型值,分成三个部分,分别是高八位的return code,低八位中的一个core dump和7bit的term signal

## 4. Share:分享一篇有观点和思考的技术文章

https://www.cnblogs.com/graphics/archive/2010/06/21/1752421.html
文章中风险了统计一个整数中1的个数的各种算法，由浅入深

