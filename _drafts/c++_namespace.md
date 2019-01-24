---
layout: post
title: c++  namespace  C++
date: 2018-06-22
tag: C++
---


### 使用命名空间的三种方式：
```
#include <iostream>
//命名空间提供了对不同变量作用域的引用；只是限定了变量的作用域；
//iostream 提供了一个叫命名空间的动词，标准的命名空间是std

#if 0
//方式二：
using std::cout; //声明命名空间的一个变量
using std::endl;
#endif

//方式三：常用的
using namespace std;

int main(void){

#if 0
	//第一种方式
	std::cout << "hello bruce" << std::endl;
#endif

	 cout << "hello bruce" << endl;

}
```

### 自定义命名空间：
```
//自定义命名空间
namespace A{
	unsigned int a = 10;
}

namespace B{
	unsigned int a = 20;
}

int main(int argc, char const *argv[])
{
	using namespace B;
	using namespace A;

	cout << A::a << endl;
	cout << B::a << endl;

	return 0;
}
```
