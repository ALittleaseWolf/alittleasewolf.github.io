---
layout: post
title: 并发控制
---

{{ page.title }}
================

<p class="meta">27 May 2023 - Hangzhou</p>

[TOC]

# 生产者-消费者问题

```c
void Tproduce(){while(1) printf("(");}// 往队列里面放入任务
void Tconsume(){while(1) printf(")");}//从队列中取出资源执行
```
在printf前后添加代码，使得打印的括号序列满足
- 一定是某个合法括号序列的前缀
- 括号嵌套的深度不超过n
- ==**同步** #FC7930==
	- 等到有空位再打印左括号
	- 等到能配对再打印右括号

## 互斥锁实现
- 左括号：嵌套深度(队列)不足n时才能打印
- 右括号：嵌套深度(队列)>1时才能打印
  
 ```C
 void Tproduce(){
 	while(1){
 retry:
		mutex_lock(&lk);
		if(count == n){
			mutex_unlock(&lk);
			goto retry;
		}
		count ++;
		printf("(");
		mutex_unlock(&lk);
	 }
 }
 ```
 ```C
 void Tconsume(){
 	while(1){
 retry:
		mutex_lock(&lk);
		if(count == 0){
			mutex_unlock(&lk);
			goto retry;
		}
		count --;
		printf(")");
		mutex_unlock(&lk);
	 }
 }
 ```
 
 ## 条件变量：实现并行计算
 ```C
 struct job{
   void(*run) (void *arg);
   void *arg;
 }
 while(1){
   struct job * job;
   mutex_lock(&mutex);
   while(!(job=get_job())){
   	 wait(&cv,&mutex);
   }
   mutex_unlock(&mutex);
   job->run(job->arg);
 }
 ```
 
 ## 信号量
 > 信号量设计的重点
	 > 考虑手环每个单位的资源是什么，谁获取，谁创造
 
 ```C
 void produce(){
 	while(1){
		p(&empty);
		prinf("C");
		V(&fill);
	}
 }
 void consumer(){
 	while(1){
		p(&fill);
		printf(")");
		V(&empty);
	}
 }
 ```
 > 在一单位资源明确的问题上更好用
 
# 并发的方法

## 单线程+事件模型
尽可能少但是又足够的并发
- 一个线程，全局的事件队列，按序执行
- 耗时的API调用会立即返回
	- 条件满足时想队列里增加一个事件
	  
	  
## 异步事件模型
并发模型简单了很多

Async-Await： Even Better
promise object
async_func()

# 总结
## 并发编程的真实应用场景
- 高性能计算，注重任务分解：生产者消费者MPI/openmp
- 数据中心，注重系统调用：线程-协程 Gorountine
- 人机交互，注重易用性：事件流图promise
 
 
 # 应对并发bug的方法
 
 ## 基本思路：否定你自己
始终假设自己的代码是错的
- 做好测试
- 检测哪儿错了
- 再检测哪儿错了
 
 bug多的根本原因：编程语言的缺陷
 >软件是需求在计算机数字世界的投影
 
 ## 防御性编程
 > assert 断言
 
 操作系统内核
 ```
assert(IN_RANGE(heap));
assert(0<=pid && pid <= 1024);

CHECK_INT(waitlist->count, >=0)
CHECK_INT(pid,<MAX_PROCS)
CHECK_HEAP(ctx->rip)
```

# 并发bug
## deadlock死锁
出现线程互相等待的情况
- AA-Deadlock
  自旋锁关中断
	 ```C
	 void os_run(){
	   spin_lock(&list_lock);
	   spin_lock(&xxx);
	   spin_unlock(&xxx);
	 }

	 void on_interrupt(){
	   spin_lock(&list_lock);
	   spin_unlock(&list_lock);
	 }
	 ```

 - ABBA-Deadlock
   ```C
   void swap(int i,int j){
    spin_lock(&lock[i]);
	 spin_lock(&lock[j]);
	 arr[i]=NULL;
	 arr[j]=arr[i];
	 spin_unlock(&lock[j]);
	 spin_unlock(&lock[i]);
   }
   ```

   swap顺序本身看起来没什么问题
   但是swap(1,2);swap(2,3);swap(3,1)并发死锁
   
死锁产生的必要条件
- ==互斥 #CC5595== 一个资源每次只能被一个进程使用
- ==请求与保持 #CC5595== 一个进程请求资源阻塞的时候，不释放以获得的资源
- ==不剥夺 #CC5595== 进程以获得的资源不能被强行剥夺
- ==循环等待 #CC5595== 若干进程之间形成头尾相接的循环等待资源关系

### 避免死锁
AA-DeadLock
- AA型死锁容易检测
- if(holding(lk)) panic();

ABBA-DeadLock
- 任意时刻系统中的锁都是有限的
- 严格按照固定顺序获得所有锁（lock ordering;消除循环等待）
	- 遇事不决可视化
	- 进而证明T_1:A->B->C T_2:B->C是安全的

## 数据竞争
> 不同线程同时访问同一段内存，且至少一个是写

- 两个内存访问在"赛跑"，"跑赢"的操作先执行

Peterson算法告诉我们：
- 我们写不对无锁的并发程序
- 用==互斥锁 #CC5595==保护好共享数据，消灭一切数据竞争，且只持有一把锁

## 花式犯错
- 互斥锁(lock/unlock)-原子性
- 条件变量(wait/signal)-同步

忘记上锁--原子性违反(Atomicity Violation AV)
忘记同步--顺序违反(order Violation OV)

### 原子性违反(AV)
"ABA"
-我以为一段代码没啥事，但是被人强势违反
"TOCTTOU" time of check to time of use
![enter description here](/imgs/1680250839650.png)

## lockdep运行时死锁检测
lockdep规约

## ThreadSanitizer运行时数据竞争检查
为所有事件简历happens-before关系图
Program-order + release-acquire

## 动态分析工具: Sanitizers
```shell
  -fsanlitize=address,thread
```
AddressSanitizer(asan): 非法内存访问
ThreadSanitizer(tsan):数据竞争 
MemorySanitizer(msan):未初始化的读取
UBSanitizer(ubsan): 未定义的行为

## 防御型编程
### Buffer Overrun检查
计算机中的canary——牺牲一部分内存单元检查memory泄露

```c
#define MAGIC 0x55555
#define BOTTOM (STK_SZ/sizeof(u32)-1)
struct stack {char data[STK_SZ];};

void canary_init(struct stack *s){
  u32 *ptr = (u32 *)s;
  for(int i=0;i<CANARY_SZ;i++){
  	ptr[BOTTOM-i] = ptr[i] = MAGIC;
  }
}

void canary_check(struct stack *s){
  u32 *ptr = (u32*)s;
  for(int i=0;i<CANARY_SZ;i++){
     panic_on(ptr[BOTTOM-i]!=MAGIC, "underflow");
	  panic_on(ptr[i]!=MAGIC, "overflow");
  }
}
```

msvc debugmode guard/fence/canary
- 未初始化栈 0xcccccccc
- 未初始化堆 0xcdcdcdcd
- 对象头尾 0xfdfdfdfd
- 已回收内存 0xdddddddd
  
### 低配版lockdep
不必大费周章记录上锁顺序
- 统计当前spin count
	- 如果超过某个明显不正常的数值报告1s
	 ```C
	 int spin_cnt=0;
	 while(xchg(&lock,1)){
		 if(spin_cnt++>SPIN_LIMIT){
		   printf("too many spin);
		 }
	 }
	 ```
- 配合调试器和线程backtrace一秒诊断死锁

### 低配版Sanitizer(L1)
内存分配要求：已分配内存
`!$ S = [l_0,r_0) \cup [l_1,r_1) \cup ...$`
- kalloc(s)返回的l，r必须满足[l,r) \cap S = 空集
	- thread_local allocation + 并发free容易出错

// double malloc
// double free