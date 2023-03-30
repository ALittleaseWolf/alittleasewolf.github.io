---
layout: post
title: 并发控制
---

{{ page.title }}
================

<p class="meta">27 May 2023 - Hangzhou</p>

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
 