# 1. Algorithm: 一道leetcode的算法题
- 66 .   Plus One
```
void plusOne(vector<int>& digits) 
{
	int size = digits.size();
	for (int i = size - 1; i >= 0; --i)
	{
		if (digits[i] == 9)
		{
			digits[i] = 0;
		}
		else
		{
			digits[i]++;
			//return digits;
			return;
		}
	}
	digits[0] = 1;
	digits.push_back(0);
	//return digits;
}
```

之后发现还有运行时间更快的一种(没理解到底快在哪里,知道的话可以告诉我一下吗),如下
```
vector<int> plusOne(vector<int>& digits)
{
		int place = digits.size() - 1;

		digits.at(place) = digits.at(place) + 1;

		while (digits.at(place) == 10)
		{
			digits.at(place) = 0;
			place--;

			if (place == -1)
			{
				digits.insert(digits.begin(), 1);
				break;
			}
			else
			{
				digits.at(place) = digits.at(place) + 1;
			}
		}

		return digits;
}
```


# 2. Review:阅读并点评一篇英文技术文章
https://towardsdatascience.com/batch-normalization-in-neural-networks-1ac91516821c
这篇文章中讲了Normalization在Neural Networks中如何工作的

# 3. Tip: 学习至少一个技术技巧
### 尽量以const,enum,inline替换#define宏定义

### Best Practice:
### Tip 1. 对于单纯常量,最好以`const` 对象或者`enum`替换`#define`

如: 用`const double AspectRatio = 1.653;`替换`#define ASPECT_RATIO 1.653`

原因:
1. `#define`是被预处理器处理的,而`cosnt`定义常量量是被编译器处理的,加入了编译器的符号表sysbol table,如果在运用此常量但获得一个编译错误信息,可能会带来困惑,错误信息只会提到1.653而不会提到ASPECT_RATIO.
2. 使用`const`常量可能比使用`definne`宏更小的代码量,因为预处理器只是盲目地把宏名称进行替换,可能导致目标代码出现多份1.653, 而改用`cosnt`常量不会.

用常量替换#defines 有两种特殊情况

1.定义常量指针
    `const char* const authorName = "Mayer"`或者`const std::string authorName("Mayer")`, 推荐使用后者.
    
2.`class`专属常量

```
class GamePlayer
{
public:
    static const int NUM = 5;
    int scores[NUM];
};
```

`const int GamePlayer::NUM;` `NUM`定义需要放在实现文件中.

`#define`无法创建一个class的专属常量,因为define不重视作用于scope.

如果编译器不支持上述语法(在NUM声明时初始化),而数组scores有需要使用,这时候可以使用`the enum hack`的做法

```
class GamePlayer
{
private:
    enum { NUM = 5 };
    int scores[NUM];
};
```
### Tip 2. 对于形似函数的宏,用inline替换

```
#define MIN(a, b)  f((a) > (b) ? (b) : (a))

template <typename T>
inline void min(const T& a, const T& b)
{
    f(a > b ? b : a);
}
```


# 4. Share:分享一篇有观点和思考的技术文章
看了专栏Java核心技术36讲中的第二讲,理解了Exception与Error的设计区别,