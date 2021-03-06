# lec5 SPOC思考题


NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)
```
最优匹配：
	优点：可避免大的空闲分区被拆分；可减小外部碎片的大小；相对简单
	缺点：外部碎片；分配大块时较慢
最差匹配：
	优点：中等大小的分配效果最好；避免出现太多小碎片
	缺点：容易破坏大的空间分区；释放分区较慢
最先匹配：
	优点：简单；在高地址空间有大的空闲分区
	缺点：产生碎片；分配大块分区时较慢
buddy system：
	优点：能够分配大小合理的分区；释放时合并能够产生较少的碎片
	缺点：需要维护空闲块二维数组
提出的连续内存分配算法：
综合上面三种分配方法，分配中等大小的分区时，采用最差匹配（避免出现小碎片）；
分配较小分区时，采用最优匹配（避免浪费大的空间，产生的小碎片也较小）；
分配较大分区时，采用最先匹配（简单）；
这样可一定程度上弱化三种方法各自的缺点。
```
```
  + 采分点：说明四种算法的优点和缺点
  - 答案没有涉及如下3点；（0分）
  - 正确描述了二种分配算法的优势和劣势（1分）
  - 正确描述了四种分配算法的优势和劣势（2分）
  - 除上述两点外，进一步描述了一种更有效的分配算法（3分）
 ```
- [x]  

>  

## 小组思考题

请参考ucore lab2代码，采用`struct pmm_manager` 根据你的`学号 mod 4`的结果值，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在应用程序层面(可以 用python,ruby,C++，C，LISP等高语言)来实现，给出你的设思路，并给出测试用例。 (spoc)
```
#include <iostream>

using namespace std;

#define Max 10000

int a[10000]={0};

void alloc(int begin, int end, int val) {
	a[begin]   = val;
	a[begin+1] = end - begin - 2;
	for (int i=begin+2; i<end; ++i) 
		a[i] = val;
}
void init() {
	alloc(0, 1000, 0);
	alloc(1000, 2000, 1);
	alloc(2000, 4000, 0);
	alloc(4000, 8000, 1);
	alloc(8000, 8500, 0);
	alloc(8500, 10000, 1);
}
int find(int need) {
	int pos = 0;
	while(pos < Max) {
		if (a[pos] == 1)
			pos += (a[pos+1] + 2);
		else if (a[pos] == 0) {
			if (a[pos+1] >= need) 
				return pos;
			else 
				pos += (a[pos+1] + 2);
		}
	}
	return -1;
}
int main(){
	init();
	int need;
	cin >> need;
	cout << find(need) << endl;
	return 0;
}
```
```
测试用例：
(1)input:4000  output:-1
(2)input:998   output:0
(3)input:999   output:2000
```
--- 

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次
MMU的工作机理？

- [x]  

>  http://en.wikipedia.org/wiki/Memory_management_unit

L1和L2高速缓存有什么区别？

- [x]  

>  http://superuser.com/questions/196143/where-exactly-l1-l2-and-l3-caches-located-in-computer
>  Where exactly L1, L2 and L3 Caches located in computer?

>  http://en.wikipedia.org/wiki/CPU_cache
>  CPU cache

### 5.2 地址空间和地址生成
编译、链接和加载的过程了解？

- [x]  

>  

动态链接如何使用？

- [x]  

>  


### 5.3 连续内存分配
什么是内碎片、外碎片？

- [x]  

>  

为什么最先匹配会越用越慢？

- [x]  

>  

为什么最差匹配会的外碎片少？

- [x]  

>  

在几种算法中分区释放后的合并处理如何做？

- [x]  

>  

### 5.4 碎片整理
一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？

- [x]  

>  

### 5.5 伙伴系统
伙伴系统的空闲块如何组织？

- [x]  

>  

伙伴系统的内存分配流程？

- [x]  

>  

伙伴系统的内存回收流程？

- [x]  

>  

struct list_entry是如何把数据元素组织成链表的？

- [x]  

>  



