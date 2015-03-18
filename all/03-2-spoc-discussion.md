# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 个人思考题
---

（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的
```
(1)64 bitCPU支持的物理内存大小限制是2^64个字节;
(2)多级页表的结构是五级结构;
(3)多级页表的虚拟地址->物理地址的映射过程：从虚拟地址中取出一段来作为索引，在页目录表中查找下一级页表的起始地址，
然后再从虚拟地址中取出下一段来作为下一级页表的索引，以此类推，直至找到最后一级页表，并取页的物理起始地址，并与
虚拟地址中的页内偏移共同组成物理内存地址。
多级页表的好处是：如果页表项很大的话，可以不用存在一块连续的内存当中。
```
```
  + 采分点：说明64bit CPU架构的分页机制的大致特点和页表执行过程
  - 答案没有涉及如下3点；（0分）
  - 正确描述了64bit CPU支持的物理内存大小限制（1分）
  - 正确描述了64bit CPU下的多级页表的级数和多级页表的结构或反置页表的结构（2分）
  - 除上述两点外，进一步描述了在多级页表或反置页表下的虚拟地址-->物理地址的映射过程（3分）
 ```
- [x]  

>  

## 小组思考题
---

（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns。若缺页率是10%，为使有效访问时间达到0.5ms,求不在内存的页面的平均访问时间。请给出计算步骤。 

- [x]  

> 500=0.9*150 + 0.1*x   x=3650ns

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。
PDT(里面有PDE)  PT(里面有PTE)
PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
Write a program to compute the pde index and so on
```
#include <iostream>
#include <fstream>
#include <cstdio>
#include <vector>
using namespace std;

void parse(string vp, int* b) {
  int begin = 15;
  int end = 3;
  while(begin > 0) {
    if ('9'-vp[end] >= 0){
      int p = vp[end]-'0';
      b[begin--] = p % 2;
      b[begin--] = p % 4 / 2;
      b[begin--] = p % 8 / 4;
      b[begin--] = 0;
    } else {
      int p = vp[end] - 'a' + 10;
      b[begin--] = p % 2;
      b[begin--] = p % 4 / 2;
      b[begin--] = p % 8 / 4;
      b[begin--] = p / 8;
    }
    end--;
  }
  //for (int i=0;i<16;++i) cout << b[i];
  //cout << endl;
}
int main() {
  fstream fin("addr");
  string a[128][32];
  int num;
  while(fin >> num) {
    for (int i=0; i<32; ++i) {
      fin >> a[num][i];
    }
  }
  cout << "input vp:";
  string vp;
  cin >> vp;
  int b[16];
  parse(vp, b);
  vector<int> pde_index(b+1, b+6);
  int pde_index_num=0, two=1;
  for (int i=pde_index.size()-1; i>=0; --i) {
    pde_index_num += two * pde_index[i];
    two *= 2;
  }
  cout << pde_index_num << endl;
  cout << "pde index:0x";
  cout << pde_index_num/16;
  if (pde_index_num%16>=10) cout << char(97+(pde_index_num%16-10));
  else cout << pde_index_num%16;
  string tt = a[17][pde_index_num];
  cout << "  pde contents:";
  cout << "valid:" << (tt[0]>8) << ", pfn:";
  int d  = (tt[1]>='a')?(tt[1]-'a'+10):(tt[1]-'0');
  d += 16*((tt[0]>='a')?(tt[0]-'a'+10):(tt[0]-'0'));
  d = d % 128;
  printf("0x%x\n", d);

  cout << "pte index:";
  vector<int> pte_index(b+6, b+11);
  int pte_index_num=0; 
  two=1;
  for (int i=pte_index.size()-1; i>=0; --i) {
    pte_index_num += two * pte_index[i];
    two *= 2;
  }
  printf("0x%x  ", pte_index_num);
  tt = a[d][pte_index_num];
  cout << "pte contents:";
  cout << "valid:" << (tt[0]>8) << ", pfn:";
  d  = (tt[1]>='a')?(tt[1]-'a'+10):(tt[1]-'0');
  d += 16*((tt[0]>='a')?(tt[0]-'a'+10):(tt[0]-'0'));
  d = d % 128;
  printf("0x%x\n", d);
  cout << "physical address:";
  vector<int> bias(b+11,b+16);
  int bias_num=0;
  two=1;
  for (int i=bias.size()-1; i>=0; --i) {
    bias_num += two * bias[i];
    two *= 2;
  }
  int PA = (d << 5) + bias_num;
  printf("0x%x", PA);
  printf(", value:%s\n", a[d][bias_num].c_str());
  return 0;
}
```
```
Virtual Address 6c74
  -->pde index:0x1b   pde contents:(valid 1, pfn 0x20)
    --> pte index:0x3  pte contents:(valid 1, pfn 0x61)
      Translates to Physical Address 0xc34  -->  Value:06
Virtual Address 6b22
  -->pde index:0x1a   pde contents:(valid 1, pfn 0x52)
    --> pte index:0x19  pte contents:(valid 1, pfn 0x47)
      Translates to Physical Address 0x8e2  -->  Value:1a
Virtual Address 03df
  -->pde index:0x00   pde contents:(valid 1, pfn 0x5a)
    --> pte index:0x1e  pte contents:(valid 1, pfn 0x5)
      Translates to Physical Address 0xbf  -->  Value:0f
Virtual Address 69dc
  -->pde index:0x18   pde contents:(valid 1, pfn 0x7f)
    --> pte index:0xe  pte contents:(valid 1, pfn 0xf)
      Translates to Physical Address 0x1fc  -->  Value:7f
Virtual Address 317a
  -->pde index:0x0c   pde contents:(valid 1, pfn 0x18)
    --> pte index:0xb  pte contents:(valid 1, pfn 0x35)
      Translates to Physical Address 0x6ba  -->  Value:1e
Virtual Address 4546
  -->pde index:0x11   pde contents:(valid 1, pfn 0x21)
    --> pte index:0xa  pte contents:(valid 1, pfn 0x7f)
      Translates to Physical Address 0xfe6  -->  Value:0b
Virtual Address 2c03
  -->pde index:0x0b   pde contents:(valid 1, pfn 0x44)
    --> pte index:0x0  pte contents:(valid 1, pfn 0x57)
      Translates to Physical Address 0xae3  -->  Value:16
Virtual Address 7fd7
  -->pde index:0x1f   pde contents:(valid 1, pfn 0x12)
    --> pte index:0x1e  pte contents:(valid 1, pfn 0x7f)
      Translates to Physical Address 0xff7  -->  Value:09
Virtual Address 390e
  -->pde index:0x0c   pde contents:(valid 1, pfn 0x18)
    --> pte index:0x8  pte contents:(valid 1, pfn 0x7f)
      Translates to Physical Address 0xfee  -->  Value:0f
Virtual Address 748b
  -->pde index:0x1d   pde contents:(valid 1, pfn 0x0)
    --> pte index:0x0  pte contents:(valid 1, pfn 0x7f)
      Translates to Physical Address 0xfeb  -->  Value:13
```

比如答案可以如下表示：
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)
      
Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```



（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python, ruby, C, C++，LISP等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。


（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。


--- 

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。

--- 
