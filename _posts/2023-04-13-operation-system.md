---
layout: post
title: 操作系统
---

{{ page.title }}
================

<p class="meta">13 April 2023 - Hangzhou</p>

[TOC]
----------
## profiler和性能摘要
二八定律
木桶效应

**性能摘要** 需要对执行性能影响最小，往往不需要full_trace
中断实现的

## 进程的地址空间
// shell 和 libc
## c标准库的实现
### 为什么需要libc？
	- 裸奔编程：能用（而且绝对够用）但不好用 syscall 系统调用
- 任何程序都用得上的定义
	- Freestanding环境下也可以使用的定义
		- stddef.h - size_t
		- stdint.h -int23_t uint64_t
		- stdbool.h -bool true false
		- 指针 intptr_t uintptr_t
		- float.h
		- limits.h
		- stdarg.h  syscall就用到了
		- inttypes.h 

- 系统调用是操作系统紧凑的最小接口，但是并不是所有系统调用都像fork一样可以直接使用
 ```C
 	extern char **environ;
	char * argv[] = {"echo","hello","world", NULL,};
	if(execve(argv[0],argv,environ) < 0){
		perror("exec");
	}
	
	输出error exec
	//echo 没有找到合适的变量
 ```
  高级api
	-  execlp system 比较合适

### string.h字符串/数组操作
	```C
	void *memset(void *s, int c, size_t n){
		for(size_t i=0;i<n;i++){
			((char*)s)[i] = c;
		}
		return s;
	}
	```
	线程安全性？
	标准库只对标准库内部数据的线程负责 比如printf的buffer

###  排序和查找
 ```C
 void qsort(void *base, size_t nmemb, size_t size,
 	int (*compar)(const void *, const void *));
 ```
 ###  malloc free
	 - 内存区间，左闭右开，维护一个数据结构，有许多小的区间集合，不相交
	 - malloc操作，找到一个满足的区间，返回左边坐标
	 - free操作，刷新掉一块连续的内存
	- 实现高效的malloc free 关注于workload
			  
		  Minmallc: free list sharding in action (APLAS'19)
		  
	- 指导思想：O(n)大小的对象分配至少有W(n)的读写操作
		- 越小的对象创建/分配越频繁
		- 较为频繁地分配中等大小的对象
		- 低频率的大对象
		- 并行、并行、再并行
			- 所有分配都会在所有处理器上发生
			- 链表和区间树不一定是好想法
		- Fast / slow
			- fast path
				- 性能极好，并行度极高，覆盖大部分情况
				- 但是有小概率失败(fall back to slow path)
			- slow path
				- 不在乎那么块
				- 但把困难的事情做好
					- 计算机系统里有很多这样的例子 比如cache
 > 人类也是这样的系统
 > Fast and slow system
#### fast path
不需要上锁，如果有两个thread，根据page64KB分页，等待page分配完了上锁再接一个page或者一个大的

// slab segregated list
两种实现 全局大链表vs pre-page小链表
每个slab都需要上一个锁
O(1)
#### slow path
线段树，链表都可以

![enter description here](./images/1681458084386.png)

libc手册 RTFS

 