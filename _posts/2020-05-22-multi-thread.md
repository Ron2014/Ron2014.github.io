---
layout: post
title: 关于一道多线程题目的思考
date: 2020-05-22 19:15
description: 如果一条线程就是一个人,那是否能在代码中体验百味人生
toc: true
tags:
 - thread
---

# 初见

之前在[牛客网](https://www.nowcoder.com/practice/cd99fbc6154d4074b4da0e74224a1582?tpId=37&tqId=21272&tPage=1&rp=&ru=/ta/huawei&qru=/ta/huawei/question-ranking)上准备某公司的机考，在题库中有幸刷到这一题

{% highlight bash %}
问题描述：有4个线程和1个公共的字符数组。
线程1的功能就是向数组输出A，线程2的功能就是向字符输出B，线程3的功能就是向数组输出C，线程4的功能就是向数组输出D。
要求按顺序向数组赋值ABCDABCDABCD，ABCD的个数由线程函数1的参数指定。[注：C语言选手可使用WINDOWS SDK库函数]
接口说明：
void init();  //初始化函数
void Release(); //资源释放函数
unsigned int __stdcall ThreadFun1(PVOID pM); //线程函数1，传入一个int类型的指针[取值范围：1 – 250，测试用例保证]，用于初始化输出A次数，资源需要线程释放
unsigned int __stdcall ThreadFun2(PVOID pM);//线程函数2，无参数传入
unsigned int __stdcall ThreadFun3(PVOID pM);//线程函数3，无参数传入
Unsigned int __stdcall ThreadFun4(PVOID pM);//线程函数4，无参数传入
char  g_write[1032]; //线程1,2,3,4按顺序向该数组赋值。不用考虑数组是否越界，测试用例保证
{% endhighlight bash %}

如果想最快AC的话，当然可以

{% highlight python %}
n = input()
print("ABCD"*n)
{% endhighlight python %}

AC并不能代表什么。这是个多线程的题目，耐心来做才会有收获。至少，我发现它有以下考点：

- 逻辑实现：这个逻辑很简单，就是 g_write 的字符录入
- 不同操作系统的API：库文件、函数名
- 不同编程语言的API：C C++ Python 甚至是Lua的协程

参考 APUE 的例子，最初的版本是这样的

{% highlight c++ %}
#include <pthread.h>
#include <unistd.h>
#include "../test.h"

#define THREAD_COUNT 4

pthread_t aThread[THREAD_COUNT];
pthread_cond_t aCond[THREAD_COUNT];
pthread_mutex_t lock_mutex = PTHREAD_MUTEX_INITIALIZER;

char g_write[1032];
int g_len = 0;
int args[THREAD_COUNT*2];
bool isFinished;

void * ThreadFun(void *pM) {
    int *args = (int *)pM;
    int threadID = args[0];
    int n = args[1];
    
    while ((threadID==0 && n--) || (threadID!=0 && !isFinished)) {
        pthread_mutex_lock(&lock_mutex);
        pthread_cond_wait(&aCond[threadID], &lock_mutex);

        TEST_INFO2(thread, threadID, "awake to execute");
        g_write[g_len++] = 'A' + threadID;

        pthread_mutex_unlock(&lock_mutex);
        pthread_cond_signal(&aCond[(threadID+1)%THREAD_COUNT]);
    }
    if (threadID==0) isFinished = true;
    return (void *)0;
}

int main() {
    for (int i=0; i<THREAD_COUNT; i++) {
        aCond[i] = PTHREAD_COND_INITIALIZER;
    }

    int n;
    while(scanf("%d", &n) != EOF) {
        isFinished = false;
        g_len = 0;

        int err;
        for (int i=0; i<THREAD_COUNT; i++) {
            args[i*2] = i;
            if (i==0) args[i*2+1] = n;
            err = pthread_create(&aThread[i], NULL, ThreadFun, &args[i*2]);
            if (err) cout << "thread" << i << " is dead" << endl;
        }
        // sleep(10);
        // cout << "thread 0 send signal" << endl;
        pthread_cond_signal(&aCond[0]);
        for (int i=0; i<THREAD_COUNT; i++) {
            pthread_join(aThread[i], NULL);
        }

        g_write[g_len]= '\0';
        cout << g_write << endl;
    }
    return 0;
}
{% endhighlight c++ %}

因为 pthread_create pthread_join 是C的API，所以觉得这是个C版本。

大概思路就是用条件变量数组 aCond 来作为线程之间通信的桥梁。一开始让所有的线程停在，条件变量触发的这一句。

`pthread_cond_wait(&aCond[threadID], &lock_mutex);`

这也印证了互斥锁和条件变量配合的流程

1. 拿到互斥锁。如果拿不到，线程挂起。 `pthread_mutex_lock(&lock_mutex);`
2. 等待条件变量触发。如果条件变量未触发，线程挂起，并且释放对应的互斥锁。`pthread_cond_wait(&aCond[threadID], &lock_mutex);`
3. 若条件变量触发，线程恢复，并重新获取互斥锁。`pthread_cond_wait(&aCond[threadID], &lock_mutex);` 后面的部分。这就解释了，为什么后面可以直接释放互斥锁。`pthread_mutex_unlock(&lock_mutex);`

输入1后，打印结果是这样的

{% highlight bash %}
1
thread0 get the lock
thread1 get the lock
thread2 get the lock
thread3 get the lock
thread0 awake to execute
thread1 awake to execute
thread2 awake to execute
thread3 awake to execute
ABCD
{% endhighlight bash %}

注意，线程0的条件变量是由主线程来触发的。这就有个小问题：

`如果线程1不是第一个被初始化的，结果会怎样？`

答案是：会死锁。

{% highlight bash %}
1
thread3 get the lock
thread2 get the lock
thread1 get the lock
thread0 get the lock
{% endhighlight bash %}

从打印来看，每个线程都在等待自己的条件变量，而且线程0刚好又是最后等待条件变量的那个，理论上会被触发的啊。

如果解开注释，为 `pthread_cond_signal(&aCond[0]);` 调用添加一些信息会发现，主线程和子线程之间又存在竞争关系。

{% highlight bash %}
1
thread3 get the lock
thread2 get the lock
thread 0 send signal
thread1 get the lock
thread0 get the lock
{% endhighlight bash %}

创建子线程的for循环都结束了，但并不表示这些线程都处于激活（active）状态。如果真的是这样，那就让主线程sleep一会再发，看看会不会死锁。不出所料：线程0能收到条件变量的信号，不会死锁。

{% highlight bash %}
1
thread3 get the lock
thread2 get the lock
thread1 get the lock
thread0 get the lock
thread 0 send signal
thread0 awake to execute
thread1 awake to execute
thread2 awake to execute
thread3 awake to execute
ABCD
{% endhighlight bash %}

这种现象就好比，你要去公司A工作，可你还没有走完入职流程，你的好友就向公司A发了一个包裹给你。公司A查无此人，也没有人道主义式的失物招领处来暂存这个包裹，而是直接把这个包裹退还回去了。于是等你入职的那天，听朋友说有包裹，就陷入了忙等状态。

这个故事又补充了一点条件变量的特点：

1. 只有该线程进入到等待条件变量的状态后，再向其发送条件变量信号，它才能正确收到。

# 相识

上面简单的例子，经过一番特点总结之后，很容易发现它的不足之处。

1. 如果主线程的优先级非常高，它在for循环结束之后没有一个线程初始化（这时候线程0放在什么位置创建都不管用了），那它发送给线程0的信号不就又丢了，程序陷入死锁？
2. 为什么一定要主线程来通知线程0呢？线程0不能自己决定什么时候开始呢？

我们使用了条件变量，实际上就可以让线程自己判断条件是否满足，参考 APUE 的例子，句式类似于 

{% highlight c++ %}
    while(不满足条件) {
        pthread_cond_wait(&cond, &lock_mutex); // 继续等待
    }
{% endhighlight c++ %}

所以我们有了第二版。去掉复杂的谁该通知谁的设定，我们其实可以用一个条件变量来控制。

{% highlight c++ %}
#include <pthread.h>
#include "../test.h"

#define THREAD_COUNT 4

pthread_t aThread[THREAD_COUNT];
pthread_mutex_t lock_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

char g_write[1032];
int g_len = 0;
int args[THREAD_COUNT*2];
bool isFinished;

void * ThreadFun(void *pM) {
    int *args = (int *)pM;
    int threadID = args[0];
    int n = args[1];
    
    while ((threadID==0 && n--) || (threadID!=0 && !isFinished)) {
        pthread_mutex_lock(&lock_mutex);

        while(g_len%THREAD_COUNT!=threadID) {
            TEST_INFO2(thread, threadID, "wait");
            pthread_cond_wait(&cond, &lock_mutex);
        }

        TEST_INFO2(thread, threadID, "awake to execute");
        g_write[g_len++] = 'A' + threadID;

        pthread_mutex_unlock(&lock_mutex);
        pthread_cond_broadcast(&cond);
        
    }
    if (threadID==0) isFinished = true;
    return (void *)0;
}

int main() {
    int n;
    while(scanf("%d", &n) != EOF) {
        isFinished = false;
        g_len = 0;

        int err;
        for (int i=0; i<THREAD_COUNT; i++) {
            args[i*2] = i;
            if (i==0) args[i*2+1] = n;
            err = pthread_create(&aThread[i], NULL, ThreadFun, &args[i*2]);
            if (err) cout << "thread" << i << " is dead" << endl;
        }

        for (int i=0; i<THREAD_COUNT; i++) {
            pthread_join(aThread[i], NULL);
        }

        g_write[g_len]= '\0';
        cout << g_write << endl;
    }
    return 0;
}
{% endhighlight c++ %}

这个例子对于输入>=2的情况是不错的，但是输入1时将暴露它的缺陷：

因为线程初始化的时机是随机的，我们很难保证一个线程的第一个 while 循环一定在另一个线程之前。

线程0第一次执行循环体的时候。其他线程的状态可能为：

- `pthread_mutex_lock(&lock_mutex);`
- `pthread_cond_wait(&cond, &lock_mutex);`
- 甚至没有初始化

对于第三种情况，因为线程0执行一遍循环体就设置 isFinished 推出了。其他线程已初始化就看到 isFinished，就不会去跑自己的循环体。所以结果就会成这样

{% highlight bash %}
1
thread0 awake to execute
thread1 awake to execute
AB
1
thread0 awake to execute
thread1 awake to execute
thread2 awake to execute
ABC
{% endhighlight bash %}

有两种方法可以解决。

* 第一种比较老实。

其实你会发现，不是A最后一次输出就结束了。而是，A最后一次输出，代表最后一次ABCD的循环。在线程0推出主循环体后，需要老老实实计算 g_write 的长度是否满足循环次数，然后再设置 isFinished 标记。相当于设置标记被推迟了，可以保证不会出现线程一初始化就看到 isFinished 的情况。

同时，要考虑线程挂起的两个位置，能否及时读到 isFinished 标记。所以要再等待条件变量的地方加上标记的判断。**不然就会陷入死循环**。修改后是这样

{% highlight c++ %}
void * ThreadFun(void *pM) {
    int *args = (int *)pM;
    int threadID = args[0];
    int n = args[1];
    
    while ((threadID==0 && n--) || (threadID!=0 && !isFinished)) {
        pthread_mutex_lock(&lock_mutex);

        while(g_len%THREAD_COUNT!=threadID) {
            if (isFinished) {
                TEST_INFO2(thread, threadID, "suicide");
                pthread_mutex_unlock(&lock_mutex);
                return (void *)0;
            }
            TEST_INFO2(thread, threadID, "wait");
            pthread_cond_wait(&cond, &lock_mutex);
        }

        TEST_INFO2(thread, threadID, "awake to execute");
        g_write[g_len++] = 'A' + threadID;

        pthread_mutex_unlock(&lock_mutex);
        pthread_cond_broadcast(&cond);
        
    }

    if (threadID==0) {
        pthread_mutex_lock(&lock_mutex);
        while(g_len<args[1]*THREAD_COUNT) {
            TEST_INFO2(thread, threadID, "wait to finish");
            pthread_cond_wait(&cond, &lock_mutex);
        }
        TEST_INFO2(thread, threadID, "awake to finish");
        isFinished = true;
        pthread_mutex_unlock(&lock_mutex);
        pthread_cond_broadcast(&cond);
    }
    return (void *)0;
}
{% endhighlight c++ %}

* 第二种方法稍微有点取巧。

以为线程0每次都是循环的第一个，我们对其循环次数加1，这就有了最后一次循环的时候，就是 g_write 输入满足条件的时候。这样避免了 `g_len<args[1]*THREAD_COUNT` 之类繁琐的计算。

{% highlight c++ %}
void * ThreadFun(void *pM) {
    int *args = (int *)pM;
    int threadID = args[0];
    int n = args[1];
    if (threadID==0) n++;
    
    while ((threadID==0 && n--) || (threadID!=0 && !isFinished)) {
        pthread_mutex_lock(&lock_mutex);

        while(g_len%THREAD_COUNT!=threadID) {
            if (isFinished) {
                pthread_mutex_unlock(&lock_mutex);
                return (void *)0;
            }
            TEST_INFO2(thread, threadID, "wait");
            pthread_cond_wait(&cond, &lock_mutex);
        }

        if (threadID==0 && n==0) {
            isFinished = true;
            TEST_INFO2(thread, threadID, "set finished");

        } else {
            TEST_INFO2(thread, threadID, "awake to execute");
            g_write[g_len++] = 'A' + threadID;
        }

        pthread_mutex_unlock(&lock_mutex);
        pthread_cond_broadcast(&cond);
    }

    return (void *)0;
}
{% endhighlight c++ %}

同样的例子，用 Windows API 实现如下 

{% highlight c++ %}
#include <thread>
#include <mutex>
#include <condition_variable>
#include <iostream>

#include "../test.h"

void Init();        //初始化函数
void Release();     //资源释放函数

#define THREAD_COUNT 4

thread *aThread[THREAD_COUNT];

condition_variable cond;
mutex lock_mutex;

bool isFinished;

char g_write[1032];
int g_len = 0;

int args[THREAD_COUNT*2];

unsigned int Thread(void *pM) {
    // doing something
    int *args = (int *)pM;
    int threadID = args[0];
    int n = args[1];

    while ((threadID==0 && n--) || (threadID!=0 && !isFinished)) {
        unique_lock <mutex> lck(lock_mutex);
        while (g_len % THREAD_COUNT!=threadID) {
            if(isFinished) {
                TEST_INFO2(thread, threadID, "suicide");
                return 0;
            }
            TEST_INFO2(thread, threadID, "wait");
            cond.wait(lck);
        }

        TEST_INFO2(thread, threadID, "awake to execute");
        g_write[g_len++] = 'A' + threadID;
        cond.notify_all();
    }

    if (threadID==0) {
        unique_lock <mutex> lck(lock_mutex);
        while(g_len<args[1]*THREAD_COUNT) {
            TEST_INFO2(thread, threadID, "wait to finish");
            cond.wait(lck);
        }
        TEST_INFO2(thread, threadID, "awake to finish");
        isFinished = true;
        cond.notify_all();
    }

    return 0;
}

void Init(int n) {
    g_len = 0;
    isFinished = false;

    for (int i=0;i<THREAD_COUNT;i++) {
        args[i*2] = i;
        if (i==0) args[i*2+1] = n;
        aThread[i] = new thread(Thread, &args[i*2]);
    }
}

void Release() {
    for (int i=0;i<THREAD_COUNT;i++) {
        delete aThread[i];
    }
}

int main() {
    int n;
    while(cin>>n) {
        Init(n);

        for (int i=0;i<THREAD_COUNT;i++) {
            aThread[i]->join();
        }

        g_write[g_len]= '\0';
        cout << g_write << endl;

        Release();
    }
    return 0;
}
{% endhighlight c++ %}

# 相知

前面都只是热身，实际上这些 API 对刷题网站来讲都不支持。

{% highlight c++ %}
#include <pthread.h>
#include <thread>
#include <mutex>
#include <condition_variable>
{% endhighlight c++ %}

无奈之下，我们只能用强大的python。

{% highlight python %}import threading, sys

THREAD_COUNT = 4
isFinished = False
mutex = threading.Lock()

def test_print(*args):
    count = len(args)
    sys.stderr.write( ("%s "*count + '\n') % (args))

class MyThread(threading.Thread):
    def __init__(self, id, leader, args=None):
        super(MyThread, self).__init__()
        self.__id = id
        self.__leader = leader
        self.__args = args

    def run(self):
        global g_write, mutex, isFinished, THREAD_COUNT
        count = self.__args
        while True:
            if self.__leader:
                if count==0:
                    break
            else:
                if isFinished:
                    return

            if mutex.acquire(1):
                if len(g_write)%THREAD_COUNT != self.__id:
                    test_print("thread", self.__id, "wait")
                    mutex.release()
                    continue
                test_print("thread", self.__id, "work")
                g_write.append(chr(ord('A')+self.__id))
                if self.__leader: count-=1
                mutex.release()

        if self.__leader:
            while True:
                if mutex.acquire(1):
                    if len(g_write) < self.__args*THREAD_COUNT:
                        test_print("thread", self.__id, "wait finish")
                        mutex.release()
                        continue
                    test_print("thread", self.__id, "set finish")
                    isFinished = True
                    mutex.release()
                    break

while True:
    line = sys.stdin.readline().strip()
    if not line:
        break
    count = int(line.split()[0])
    g_write = []
    isFinished = False

    threads = []
    for i in xrange(THREAD_COUNT):
        threads.append(MyThread(i, i==0, count if i==0 else None))
    for i in xrange(THREAD_COUNT):
        threads[i].start()
    for i in xrange(THREAD_COUNT):
        threads[i].join()
    print("".join(g_write))
    
{% endhighlight python %}

这种做法比较传统，连条件变量都没用上。一旦条件不满足，我们就让逻辑跳回到 while 开头。

{% highlight python %}
    if len(g_write)%THREAD_COUNT != self.__id:
        test_print("thread", self.__id, "wait")
        mutex.release()
        continue
{% endhighlight python %}

还能学到的是，sys.stderr 没有缓冲区，只有用它来打印不会错行，而且尽量将参数都放在一行生成。

![img]({{ '/assets/images/multithread_time1.png' | relative_url }}){: .center-image }*conditions*

虽然代码AC了，但没有用到条件变量会有很多无效的 while-continue 空逻辑，如果线程逻辑很简单，线程调度的开销就会比较大（user time < system time）。让我们引入条件变量，如果是一个 notify_all() 的话，system time 的改变并不大；分别 notify() 能看到明显效果。
    
{% highlight python %}

import threading, sys

THREAD_COUNT = 4
isFinished = False
conds = []
for i in xrange(THREAD_COUNT):
    conds.append(threading.Condition())

def test_print(*args):
    count = len(args)
    sys.stderr.write( ("%s "*count + '\n') % (args))

class MyThread(threading.Thread):
    def __init__(self, id, leader, args=None):
        super(MyThread, self).__init__()
        self.__id = id
        self.__leader = leader
        self.__args = args

    def run(self):
        global g_write, isFinished, THREAD_COUNT, conds
        id = self.__id
        isLeader = self.__leader

        count = self.__args
        cond = conds[self.__id]

        while True:
            if isFinished:
                return

            if cond.acquire(True):
                while len(g_write)%THREAD_COUNT != id:
                    test_print("thread", id, "wait")
                    if isFinished:
                        cond.release()
                        return
                    cond.wait()

                if isLeader:
                    if count==0:
                        test_print("thread", id, "set finish")
                        isFinished = True
                        cond.release()
                        for i in xrange(THREAD_COUNT):
                            next_cond = conds[i]
                            if next_cond.acquire(True):
                                next_cond.notify()
                                next_cond.release()
                        continue

                test_print("thread", id, "work")
                g_write.append(chr(ord('A')+id))
                if isLeader: count-=1
                cond.release()

                next_cond = conds[(id+1)%THREAD_COUNT]
                if next_cond.acquire(True):
                    next_cond.notify()
                    next_cond.release()

while True:
    line = sys.stdin.readline().strip()
    if not line:
        break
    count = int(line.split()[0])
    g_write = []
    isFinished = False

    threads = []
    for i in xrange(THREAD_COUNT):
        threads.append(MyThread(i, i==0, count if i==0 else None))
    for i in xrange(THREAD_COUNT):
        threads[i].start()
    for i in xrange(THREAD_COUNT):
        threads[i].join()
    print("".join(g_write))
    
{% endhighlight python %}

![img]({{ '/assets/images/multithread_time2.png' | relative_url }}){: .center-image }*conditions*

# 相伴

题目做完了，但是还是意犹未尽。线程之间形成的多人运动的现象有点让我小兴奋，毕竟这个程序真的跑起来，在我脑海中其实是这番景象。

{% highlight bash %}
随着自然环境越来越恶化，每个村子需要自发组织安检工作
现检疫站制定的安检流程如下：
1. 安检人员都是该村的村民
2. 安检是轮班制，单位为天
3. 安检人员仅一人
4. 轮到当天值班的安检人员，必须上岗，不能有任何推脱，不能调班
5. 每天的工作结束后，安检人员需要在值班表上签名
6. 村民中会选出一人作为安检队长
7. 安检工作有个暂定的期限，期限结束之后，安检队长需要上交值班表
8. 只有安检队长知道安检工作的期限，其他村民都不知道
9. 安检工作的第一天从队长开始
10. 检疫站可以根据需要提供一定数量的对讲机，用于安检工作中的通信
11. 值班表上交后，检疫站有权利决定，是否开启新的安检周期，或者结束安检

现有一个村子住着 A B C D 四个村民，检疫站指派了 A 作队长，而你是检疫站的文员。
检疫站需要你设计一个程序，去模拟上述的过程，来验证收到的值班表是否正确。

注意事项：
1. 所有人通过对讲机来互相通知谁该接班
2. 所有人总是会随时醒来，他们只能通过值班表来分析出当天是否自己值班
2. 安检队长实际上要比所有人多上一天班来上交值班表，只是这天他不用再签名

输入：
期限

输出：
期限结束后，检疫站应该看到的值班表的内容（签名彼此相连，中间没有空格）
{% endhighlight bash %}

其中，概念的对应关系是这样

{% highlight bash %}
1. 检疫站 ----------------------------- 主线程或者进程
2. 村民 ------------------------------- 子线程
3. 当天的安检人员 ---------------------- 活动线程，当前抢占到资源的线程
4. 安检队长 ---------------------------- 还是子线程，只是多一些对其他线程的控制而已
5. 值班表 ------------------------------ g_write 字符数组缓冲区
6. 值班表的上交 ------------------------ print 
7. 对讲机 ------------------------------ 条件变量
8. 通信 -------------------------------- notify 或者 notify_all
{% endhighlight bash %}

在线程的概念里，他们彼此并不能直接交流，比如通过 global threads 数组搞什么。

{% highlight python %}
global threads
threads[0].notify()
{% endhighlight python %}

这个 threading.Thread 只是提供了一些基础的接口。线程之间的通信还是得通过 **事件-监听** 的方式执行，这种方式用的就是条件变量。

python 提供的条件变量和C++不同的是，其内部自己封装了一个互斥锁，所以互斥锁的创建就省了。

只用互斥锁，网上还看到一些其他的做法：

1. 初始化时，除线程0，其他的锁都锁住。
2. 线程逻辑：为了执行自己的逻辑，锁住自己的锁。
3. 线程逻辑：执行玩自己的逻辑后，并不是释放自己的锁，而是释放下一个人的锁。

{% highlight python %}
import threading
import sys

def out_A(num):
    for i in range(num):
        lock1.acquire()
        print('A', end='')
        lock2.release()

def out_B(num):
    for i in range(num):
        lock2.acquire()
        print('B', end='')
        lock3.release()

def out_C(num):
    for i in range(num):
        lock3.acquire()
        print('C', end='')
        lock4.release()

def out_D(num):
    for i in range(num):
        lock4.acquire()
        print('D', end='')
        lock1.release()

if __name__ == "__main__":
    for line in sys.stdin:
        NUM = int(line)
        lock1 = threading.Lock()
        lock2 = threading.Lock()
        lock3 = threading.Lock()
        lock4 = threading.Lock()

        lock2.acquire()
        lock3.acquire()
        lock4.acquire()

        t1 = threading.Thread(target=out_A, args=(NUM,))
        t2 = threading.Thread(target=out_B, args=(NUM,))
        t3 = threading.Thread(target=out_C, args=(NUM,))
        t4 = threading.Thread(target=out_D, args=(NUM,))

        t1.start()
        t2.start()
        t3.start()
        t4.start()
        
        t1.join()
        t2.join()
        t3.join()
        t4.join()
        
        print()
{% endhighlight python %}

值得一提的是，稍微增加一些变数，线程逻辑就会变复杂。比如我自己给自己出的附加题

***********************************************
附加题1：

前面的条件不变。
假设担任队长的A刚好又是村委会的代表，他每周五都要赶往市中心做报告。
检疫站特批他可以调班，即作报告当天，由轮班的下一任（即B）代替他值班，第二天的值班再交给A来完成。
整个过程中，也只有队长A知道当天星期几。
问，这样的程序该如何设计。

注意事项：
1. 如果上交值班表的当天队长需要去开会，则可能会让顶班的B多值班一天。这个情况是允许的。

输入：
期限 值班第一天为周几

输出：
期限结束后，检疫站应该看到的值班表的内容（签名彼此相连，中间没有空格）

***********************************************
附加题2：
前面的条件（包括附加题1）不变。
安检期间，某村民不幸感染需要隔离15天，在这段时间内将调整新的值班表，允许该村民缺席。
如果刚好该村民是队长，将选出之前接触（值班）最少的村民担任临时队长。
问，这样的程序该如何设计。

注意事项：
1. 临时队长也需要做报告，需要和轮班顺序中的下一位进行调班。
2. 隔离期结束后，该村民将当天立即值班。
3. 如果该村民是队长，将恢复原职。
4. 如果队长迟迟不回归，安检到期后临时队长也要上交值班表。

输入：
期限 值班第一天为周几 被感染的村民姓名 从哪一天开始隔离

输出：
期限结束后，检疫站应该看到的值班表的内容（签名彼此相连，中间没有空格）

为了实现这两个程序，着实折腾了我好几天。

然而，等我好不容易把逻辑都跑过之后，我发现自己的做法并不完美。

对于python来说，其实还有互斥锁、条件变量意外的做法。

这是两个附加题引入了 **调班** **病假** 等频繁调整 **线程优先级** 后不得不去探索的方法。

确实，其实每个线程的主要逻辑就是向 g_write 输入自己对应的字符而已，何必要把线程调度混在一起？

如果一些类本身提供了同步的、线程安全的机制，确实非常值得一试。

先列出目前对附加题的做法，我将在以后来改进它们。

* 附加题1

{% highlight python %}
import threading, sys

THREAD_COUNT = 4
WEEK_DAYS = 7
MEETTING_DAY = 5

isFinished = False
isMeetting = 0
conds = []
for i in xrange(THREAD_COUNT):
    conds.append(threading.Condition())

def test_print(*args):
    return
    count = len(args)
    sys.stderr.write( ("%s "*count + '\n') % (args))

class MyThread(threading.Thread):
    def __init__(self, id, leader, args=None):
        super(MyThread, self).__init__()
        self.__id = id
        self.__leader = leader
        self.__args = args
        self.next_id = None
    
    def exchange(self, next_id):
        self.next_id = next_id

    def run(self):
        global g_write, isFinished, isMeetting, conds, THREAD_COUNT, WEEK_DAYS, MEETTING_DAY, threads
        id = self.__id
        isLeader = self.__leader

        if isLeader:
            count = self.__args[0]
            weekday = self.__args[1]
        cond = conds[self.__id]

        while True:
            if isFinished:
                break

            if cond.acquire(True):
                while (len(g_write)%THREAD_COUNT != id and self.next_id is None) or \
                    (len(g_write)%THREAD_COUNT == id and self.next_id is not None):
                    test_print("thread", id, "wait")
                    if isFinished:
                        break
                    cond.wait()

                if isFinished:
                    cond.release()
                    continue

                next_id = (id+1)%THREAD_COUNT
                if isLeader:
                    while (weekday+len(g_write)) % WEEK_DAYS == MEETTING_DAY:
                        test_print("thread", id, "wait exchange")
                        next_id = 2
                        threads[1].exchange(0)
                        if conds[1].acquire(True):
                            conds[1].notify()
                            conds[1].release()
                        cond.wait()

                    threads[1].exchange(None)

                    if count==0:
                        test_print("thread", id, "set finish")
                        isFinished = True
                        cond.release()
                        continue
                    
                test_print("thread", id, "work")
                g_write.append(chr(ord('A')+id))
                if isLeader:
                    count-=1
                cond.release()

                if self.next_id is not None:
                    next_id = self.next_id
                    
                next_cond = conds[next_id]
                if next_cond.acquire(True):
                    next_cond.notify()
                    next_cond.release()

        if isLeader:
            for i in xrange(1, THREAD_COUNT):
                next_cond = conds[i]
                if next_cond.acquire(True):
                    next_cond.notify()
                    next_cond.release()
while True:
    line = sys.stdin.readline().strip()
    if not line:
        break
    lines = line.split()
    count = int(lines[0])
    if len(lines) > 1:
        weekday = int(lines[1])
    else:
        weekday = int(sys.stdin.readline().strip()) % WEEK_DAYS
    g_write = []
    isFinished = False
    isMeetting = 0

    threads = []
    for i in xrange(THREAD_COUNT):
        threads.append(MyThread(i, i==0, (count, weekday) if i==0 else None))
    for i in xrange(THREAD_COUNT):
        threads[i].start()
    for i in xrange(THREAD_COUNT):
        threads[i].join()
    print("".join(g_write))

{% endhighlight python %}

* 附加题2

{% highlight python %}
import threading, sys

THREAD_COUNT = 4
WEEK_DAYS = 7
MEETTING_DAY = 5
SEPARATE_DAYS = 15

isFinished = False
count = 0
threads = []
g_write = []

circle_count = 0
circle_sign = 0

conds = []
for i in xrange(THREAD_COUNT):
    conds.append(threading.Condition())
# cond_lock = threading.Condition()
cond_lock = threading.Lock()
m_lock = threading.Lock()

duty = []
separates = set([])

def has_meetting(start_weekday):
    return (start_weekday+len(g_write)) % WEEK_DAYS == MEETTING_DAY

def get_lowest_one():
    global g_write, separates
    stat = {}
    for n in g_write:
        ni = ord(n)-ord('A')
        stat[ni] = stat.get(ni, 0) + 1
    key_list = stat.keys()
    sorted(key_list, key=lambda x: x + stat.get(x, 0)*2)
    for n in key_list:
        if n not in separates:
            return n

def switch_duty(i, j):
    global duty
    ii = duty.index(i)
    ij = duty.index(j)
    duty[ii], duty[ij] = duty[ij], duty[ii]

def add_duty(i):
    global duty
    assert(i not in duty, "add_duty error")
    duty.insert(len(g_write) % len(duty), i)
    return True

def rem_duty(i):
    global duty
    assert(i in duty, "rem_duty error")
    duty.remove(duty.index(i))
    return True

def is_weak(weak_day):
    # test_print("is_weak", weak_day, "".join(g_write))
    if weak_day is None:
        return False
    return len(g_write)>=weak_day and len(g_write)<(weak_day+SEPARATE_DAYS)

def test_print(*args):
    return 
    sys.stderr.write("".join("%s "*len(args) % (args))+"\n")
    
class MyThread(threading.Thread):
    def __init__(self, id, leader, weak_day):
        super(MyThread, self).__init__()
        self.__id = id
        self.__weak_day = weak_day
        self.is_leader = leader
        self.left_separate = 0
    
    def exchange(self, next_id):
        switch_duty(self.__id, next_id)
    
    def getWeakDay(self):
        return self.__weak_day
    
    def setLeader(self, isLeader):
        self.is_leader = isLeader

    def run(self):
        global g_write, isFinished, conds, cond_lock, m_lock, circle_count, circle_sign, separates, threads, THREAD_COUNT, SEPARATE_DAYS
        id = self.__id
        weak_day = self.__weak_day
        origin_leader = self.is_leader

        cond = conds[id]
        lock = cond_lock
        meetting = False

        while True:
            if isFinished:
                break

            if cond.acquire(True):
                while id in separates and is_weak(weak_day):
                    test_print("thread", id, "is separate")
                    cond.wait()
                cond.release()

                test_print("thread", id, "is awake")
                if lock.acquire(True):

                    # test_print("thread", id, "enter ........")
                    cur_idx = len(g_write)%len(duty)
                    next_idx = (len(g_write)+1)%len(duty)
                    last_idx = (len(g_write)-1)%len(duty)

                    if id in separates:
                        test_print("thread", id, "leave hospital")

                        separates.discard(id)
                        add_duty(id)

                        cur_idx = len(g_write)%len(duty)
                        while duty[cur_idx] != id:
                            n = duty.pop()
                            duty.insert(0, n)
                        test_print("thread", id, "change duty to", ",".join(map(str, duty)))

                        if origin_leader:
                            for i, t in enumerate(threads):
                                if i!=id:
                                    t.setLeader(False)
                        else:
                            self.is_leader = False
                    else:
                        # make room for backflow
                        someone_is_leaving_hospital = False
                        for oid in separates:
                            if not is_weak(threads[oid].getWeakDay()):
                                someone_is_leaving_hospital = True
                                break
                        if someone_is_leaving_hospital:
                            lock.release()
                            continue

                    if duty[cur_idx] != id:
                        # test_print("thread", id, "wait")
                        lock.release()
                        continue
                    
                    # test_print("thread", id, "do something .......................")
                    next_idx = (len(g_write)+1)%len(duty)
                    next_id = duty[next_idx]

                    if is_weak(weak_day):
                        # sperate
                        test_print("thread", id, "goto hospital")
                        separates.add(id)
                        rem_duty(id)
                            
                        if origin_leader or self.is_leader:
                            new_leader_id = get_lowest_one()
                            threads[new_leader_id].setLeader(True)
                            test_print("thread", id, "set new leader", new_leader_id)

                        next_idx = len(g_write)%len(duty)
                        while duty[next_idx] != next_id:
                            n = duty.pop()
                            duty.insert(0, n)
                        test_print("thread", id, "change duty to", ",".join(map(str, duty)))

                        lock.release()
                        continue

                    if self.is_leader:
                        global count, weekday
                        if has_meetting(weekday):
                            test_print("thread", id, "has meetting, wait exchange with", next_id)
                            meetting = True
                            threads[next_id].exchange(id)
                            test_print("thread", id, "change duty to", ",".join(map(str, duty)))

                            lock.release()
                            continue

                        if meetting:
                            meetting = False
                            threads[duty[last_idx]].exchange(id)
                            test_print("thread", id, "change duty to", ",".join(map(str, duty)))

                        if circle_count>=count:
                            test_print("thread", id, "set finish")
                            isFinished = True
                            lock.release()
                            continue

                    g_write.append(chr(ord('A')+id))
                    test_print("thread", id, "work", "".join(g_write))

                    circle_sign += 1
                    if circle_sign >= len(duty):
                        test_print("thread", id, "get signed", circle_sign, "and add circle", circle_count+1)
                        circle_sign = circle_sign % len(duty) 
                        circle_count += 1

                    lock.release()

                    nsep = set(separates)
                    for sid in nsep:
                        if conds[sid].acquire(True):
                            test_print("thread", id, "notify", sid)
                            conds[sid].notify()
                            conds[sid].release()

while True:
    line = sys.stdin.readline().strip()
    if not line:
        break
    lines = line.split()
    count = int(lines[0])
    if len(lines) > 1:
        weekday, weak_name, weak_day = int(lines[1]), lines[2], int(lines[3])
    else:
        weekday = int(sys.stdin.readline().strip()) % WEEK_DAYS
        weak_name = sys.stdin.readline().strip()
        weak_day = int(sys.stdin.readline().strip())

    g_write = []
    isFinished = False
    weak_idx = ord(weak_name) - ord('A')
    
    duty = []
    for i in xrange(THREAD_COUNT):
        duty.append(i)

    test_print(",".join(map(str,duty)))
    separates = set([])
    circle_count = 0
    circle_sign = 0
    
    threads = []
    for i in xrange(THREAD_COUNT):
        threads.append(MyThread(i, i==0, weak_day if i==weak_idx else None))
    for i in xrange(THREAD_COUNT):
        threads[i].start()
    for i in xrange(THREAD_COUNT):
        threads[i].join()
    print("".join(g_write))
    
{% endhighlight python %}

![img]({{ '/assets/images/yeah.gif' | relative_url }}){: .center-image }*写完了附加题2的我*