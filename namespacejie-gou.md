##### 一、Linux Namespace

        Linux Namespace是Linux提供的一种OS-level virtualization的方法。目前在Linux系统上实现OS-level virtualization的系统有Linux VServer、OpenVZ、LXC Linux Container、Virtuozzo等，其中Virtuozzo是OpenVZ的商业版本。以上种种本质来说都是使用了Linux Namespace来进行隔离。

        那么究竟什么是Linux Namespace？Linux很早就实现了一个系统调用chroot，该系统调用能够为进程提供一个限制的文件系统。虽然文件系统的隔离要比单纯的chroot复杂的多，但是至少chroot提供了一种简单的隔离模式：chroot内部的文件系统无法访问外部的内容。Linux Namespace在此基础上，提供了对UTS、IPC、mount、PID、network的隔离机制，例如对不同的PID namespace中的进程无法看到彼此，而且每个PID namespace中的进程PID都是单独制定的。这一点对OS-level Virtualization非常有用，这是因为：对于不同的Linux运行环境中，都有一个init进程，其PID=0，由于不同的PID namespace中都可以指定自己的0号进程，所以可以通过该技术来进行PID环境的隔离。

        OS-level Virtualization相比其他的虚拟化技术更加轻量级。

        Linux在使用Namespace的时候，需要显式的在配置中指定将那些Namespace的支持编译到内核中。

##### 二、数据结构

        我们可以在struct task\_struct中找到对应的namespace结构。

&lt;sched.h&gt;

```
1
struct
 task_struct { 

2
... 

3
struct
 nsproxy *
nsproxy; 

4
... 

5
 };
```

        nsproxy就是每个进程自己的namespace，结构如下：

&lt;nsproxy.h&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
1
struct
 nsproxy { 

2
    atomic_t count; 

3
struct
 uts_namespace *
uts_ns; 

4
struct
 ipc_namespace *
ipc_ns; 

5
struct
 mnt_namespace *
mnt_ns; 

6
struct
 pid_namespace *
pid_ns; 

7
struct
 net          *
net_ns; 

8
}; 

9
extern
struct
 nsproxy init_nsproxy;
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)



        具体的参考后文，可以看到有一个init\_nsproxy，这个全局初始化使用的nsproxy，也是init进程使用的nsproxy。

##### 三、UTS namespace

        我们在这里只介绍两种namespace：UTS namespace和PID namespace，是因为这两种namespace比较有代表性。UTS namespace很简单，也没有树形或者很复杂的结构。struct uts\_namespace结构如下：

&lt;utsname.h&gt;

```
1
struct
 uts_namespace { 

2
struct
 kref kref; 

3
struct
 new_utsname name; 

4
struct
 user_namespace *
user_ns; 

5
}; 

6
extern
struct
 uts_namespace init_uts_ns;
```



        可以看到uts\_namespace中只关心utsname域（name）和user\_namespace域（user\_ns）。老版本的Linux user\_namespace是在nsproxy中的，新版本放在了uts\_namespace中。这部分的结构如下：

[![](https://images0.cnblogs.com/blog/594483/201312/25202459-e8b46359d9404ca4b741e48d86b8f29e.png "85677792\[4\]")](https://images0.cnblogs.com/blog/594483/201312/25202458-595e88bdef7247efa0c7d34ccf4fcb4c.png)

        需要说明的是，这张图是比较老的Linux Kernel，其中user\_namespace还在nsproxy结构中。对于uts\_namespace，由于不需要维护像pid\_namespace那样复杂的树状结构和复杂的搜索需求，所以对于每个uts\_namespace只需要维护一个实例就可以了。

##### 四、进程的若干个ID的意义

* PID：Process ID，进程ID，即进程的唯一标识
* TGID：处于某个线程组中的所有进程都有统一的线程组ID（Thread Group IP，TGID）。线程可以用clone加CLONE\_THREAD来创建。线程组中的主进程成为group leader，可以通过线程组中任何线程的的task\_struct-
  &gt;
  group\_leader成员获得。
* 独立进程可以合并成进程组（使用setpgrp系统调用）。进程组成员的task\_struct-
  &gt;
  pgrp属性值都是相同的（PGID），即进程组组长的PID。用管道连接的进程在一个进程组中。
* 几个进程组可以合并成一个会话。会话中所有进程都有同样的SID（Session ID，会话ID），保存在task\_struct-
  &gt;
  session中。SID可以通过setsid系统调用设置。

##### 五、PID namespace

        task\_struct中有几个成员是与PID有关的，如下：

&lt;sched.h&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
1
struct
 task_struct{ 

2
... 

3
    pid_t pid; 

4
    pid_t tgid; 

5
... 

6
struct
 pid_link pids[PIDTYPE_MAX]; 

7
... 

8
 }
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)



        对于所有的进程来说，都有两种ID（包含PID、TGID、PGRP、SID）：一个是全局的ID，保存在task\_struct-&gt;pid中；另一个是局部的ID，即属于某个特定的命名空间的ID。对于task\_struct-&gt;pids数组，数组编号的定义如下：

&lt;pid.h&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
1
enum
 pid_type 

2
{ 

3
    PIDTYPE_PID, 

4
    PIDTYPE_PGID, 

5
    PIDTYPE_SID, 

6
    PIDTYPE_MAX 

7
 };
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)



        可以看到这里只定义了PID、PGID和SID，至于TGID（线程组ID）无非是线程组组长的ID，所以就不需要用特殊的结构来组织了。

        下面给出struct pid\_link的结构：

&lt;pid.h&gt;

```
1
struct
 pid_link 

2
{ 

3
struct
 hlist_node node; 

4
struct
 pid *
pid; 

5
 };
```



        pid\_link包含两个成员，node用来组织struct task\_struct和struct pid的关系，后面会降到，pid则指向对应的struct pid结构。可以看到，给出指定的task\_struct，可以通过task\_struct-&gt;pids\[pid\_type\]-&gt;pid来找到对应的pid结构。下面给出struct pid结构定义：

&lt;pid.h&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
1
struct
 pid 

2
{ 

3
    atomic_t count; 

4
     unsigned 
int
 level; 

5
/*
 lists of tasks that use this pid 
*/
6
struct
 hlist_head tasks[PIDTYPE_MAX]; 

7
struct
 rcu_head rcu; 

8
struct
 upid numbers[
1
]; 

9
 };
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)



        其中count表示该pid结构被引用的次数，level表示该pid结构对应多少层的namespace，tasks则反向指向与该pid相连的task\_struct，即上文所述pid\_link-&gt;node就是组织在pid-&gt;tasks中的，rcu是将所有struct pid组织起来的辅助结构，numbers成员中存储的是struct upid结构，该结构是pid与pid\_namespace相关联的结构。这里需要注意，虽然numbers成员数组大小只有1，其实这种定义只是说明了：struct pid中的numbers事实上是一个数组，其长度至少为1（即只有一层namespace的情况）；对于pid属于多层namespace的情况，该数组是可以动态扩张的。struct pid定义如下：

&lt;pid.h&gt;

```
1
struct
 upid { 

2
/*
 Try to keep pid_chain in the same cacheline as nr for find_vpid 
*/
3
int
 nr; 

4
struct
 pid_namespace *
ns; 

5
struct
 hlist_node pid_chain; 

6
 };
```



        nr表示ID的数值，我们使用ls命令得到的进程的各种ID就是保存在这里，ns存储的是指向namespace的结构。此外，所有的upid都保存在一个散列表中，通过upid-&gt;pid\_chain组织。散列表的表头定义在：

&lt;kernel/pid.c&gt;

```
1
static
struct
 hlist_head *pidhash;
```



        对于upid指向的pid\_namespace，定义如下：

&lt;pid\_namespace.h&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
 1
struct
 pid_namespace { 

 2
struct
 kref kref; 

 3
struct
 pidmap pidmap[PIDMAP_ENTRIES]; 

 4
int
 last_pid; 

 5
struct
 task_struct *
child_reaper; 

 6
struct
 kmem_cache *
pid_cachep; 

 7
     unsigned 
int
 level; 

 8
struct
 pid_namespace *
parent; 

 9
#ifdef CONFIG_PROC_FS 

10
struct
 vfsmount *
proc_mnt; 

11
#endif
12
#ifdef CONFIG_BSD_PROCESS_ACCT 

13
struct
 bsd_acct_struct *
bacct; 

14
#endif
15
 };
```

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

        其中，kref保存实例被引用的次数；pidmap是一个位图，保存该namespace中pid的分配情况；last\_pid保存上一个分配的pid；对于每个namespace来说，都有一个进程来扮演Linux中init进程的角色，对所有的僵死进程执行wait4操作，child\_reaper指向的就是这个进程（一般来说都是当前namespace中的0号进程）；pid\_cachep是一个kmem\_cache对象，用来加速pid的分配，具体可以参考“Linux的物理内存管理”；level表示该namespace在整个命名空间的层次；parent表示该namespace的父namespace；proc\_mnt表示在/proc文件系统的显示结构；bacct保存BSD进程加速的结构。后两者我们在这里不涉及。

        整个数据结构的组织如下图：

[![](https://images0.cnblogs.com/blog/594483/201312/25202459-131274d832354e2f9a04a7568bbb4d0b.png "3710358\[4\]")](https://images0.cnblogs.com/blog/594483/201312/25202459-808b310157a642878ce3ae1ab8216403.png)

        可以看到图分为四个部分：

* 左下角的部分是PID namespace自己的组织结构，即是一个树形的结构。struct pid\_namespace自成体系，除了child\_reaper之外并没有更多和外部的联系。
* 右下角是所有的struct upid组成的散列表，通过pidhash为表头的散列表组织，便于对upid的查找。
* 右上角是使用pid的struct task\_struct，不同的进程可能对应到同一个struct pid，例如同一个进程组的task\_struct-
  &gt;
  pid\[PIDTYPE\_PGID\]，即同一进程组进程的进程组ID是相同的，指向同一个pid实例。
* 中间处于最核心地位的是struct pid，可以看到struct pid可以通过其task成员数组来找到所有指向该pid的pid\_link结构，从而找到对应的task\_struct；此外struct pid还可以通过其numbers成员数组来获得该struct pid所属的不同层次的pid\_namespace，二者的连接通过struct upid建立。

        总结来说，struct task\_struct和其对应的struct pid通过struct pid\_link连接，每个struct pid实例通过struct upid来和对应不同层次的struct pid\_namespace连接。真正的PID值保存在struct upid结构中，这也是为什么需要将struct upid通过pidhash组织起来的原因。

##### 六、getpid系统调用路径

        getpid调用到系统调用sys\_getpid，定义在timer.c中的SYSCALL\_DEFINE0\(getpid\)，之后的调用路径为：



SYSCALL\_DEFINE0\(getpid\)-&gt;task\_tgid\_vnr\(current\)-&gt;pid\_vnr\(\)-&gt;pid\_nr\_ns\(\)



        最终获得当前进程所属命名空间看到的线程组leader的局部pid。可以通过上述路径的代码来得到一个简单的pid\_namespace使用实例。由于代码很简单，在这里就不详细说明了。

##### 七、Kernel中pid\_namespace相关的函数

* void attach\_pid\(struct task\_struct \*task, enum pid\_type type, struct pid \*pid\)；

  将新申请的struct pid实例连接到指定的task。

* static inline struct pid \*task\_pid\(struct task\_struct \*task\)；
 
  获得与task\_struct相关联的PID的struct pid实例。
* static inline struct pid \*task\_pgrp\(struct task\_struct \*task\)；
 
  获得与task\_struct相关联的PGRP的struct pid实例。
* pid\_t pid\_nr\_ns\(struct pid \*pid, struct pid\_namespace \*ns\)；
 
  获得struct pid之后，从其numbers数组中的upid得到指定ns中的pid数值。
* pid\_t pid\_vnr\(struct pid \*pid\);
 
  获得struct pid实例在当前进程所属pid\_namespace中的pid数值。
* static inline pid\_t pid\_nr\(struct pid \*pid\)；
 
  获得struct pid全局的pid数值。
* static inline pid\_t task\_pid\_nr\_ns\(struct task\_struct \*tsk, struct pid\_namespace \*ns\);  
  pid\_t task\_tgid\_nr\_ns\(struct task\_struct \*tsk, struct pid\_namespace \*ns\);

  static inline pid\_t task\_pgrp\_nr\_ns\(struct task\_struct \*tsk, struct pid\_namespace \*ns\);

  static inline pid\_t task\_session\_nr\_ns\(struct task\_struct \*tsk, struct pid\_namespace \*ns\);  
  分别表示获得进程在对应namespace中的PID/TGID/PGRP/SID的值。

* struct pid \*find\_pid\_ns\(int nr, struct pid\_namespace \*ns\);  
  通过PID数值和指定namespace找到对应的struct pid结构。该函数的实现以来于pidhash组织的散列表。

* find\_task\_by\_pid\_type\_ns：通过pid数值、type、namespace找到对应的struct task\_struct。  
  find\_task\_by\_pid\_ns：通过pid数值和进程namespace找到对应的struct task\_struct。  
  find\_task\_by\_vpid：根据pid数值在当前进程所处namespace（即局部的数字pid）来找到对应的进程。  
  find\_task\_by\_pid：根据pid数值在全局的namespace来找到对应的进程

* struct pid \*alloc\_pid\(struct pid\_namespace \*ns\);  
  在指定namespace中分配一个struct pid结构。

        我们来详细分析一下alloc\_pid代码：

&lt;pid.c&gt;

[![](https://common.cnblogs.com/images/copycode.gif "复制代码")](javascript:void%280%29;)

```
 1
struct
 pid *alloc_pid(
struct
 pid_namespace *
ns) 

 2
{ 

 3
struct
 pid *
pid; 

 4
enum
 pid_type type; 

 5
int
 i, nr; 

 6
struct
 pid_namespace *
tmp; 

 7
struct
 upid *
upid; 

 8
//
 在ns-
>
pid_cachep指定的cache中分配struct pid 
 9
     pid = kmem_cache_alloc(ns-
>
pid_cachep, GFP_KERNEL); 

10
if
 (!
pid) 

11
goto
out
; 

12
//
 对指定namespace中每一层 

13
//
 通过alloc_pidmap函数分配在该层namespace对应的pid数值 

14
//
 保存在upid的numbers数组中 
15
     tmp =
 ns; 

16
for
 (i = ns-
>
level; i 
>
= 
0
; i--
) { 

17
         nr =
 alloc_pidmap(tmp); 

18
if
 (nr 
<
0
) 

19
goto
 out_free; 

20
         pid-
>
numbers[i].nr =
 nr; 

21
         pid-
>
numbers[i].ns =
 tmp; 

22
         tmp = tmp-
>
parent; 

23
    } 

24
//
 get_pid_ns增加对应ns的引用数 
25
    get_pid_ns(ns); 

26
//
 设置struct pid的层次 
27
     pid-
>
level = ns-
>
level; 

28
//
 设置struct pid的引用数 
29
     atomic_set(
&
pid-
>
count, 
1
); 

30
//
 初始化pid-
>
tasks数组 
31
for
 (type = 
0
; type 
<
 PIDTYPE_MAX; ++
type) 

32
         INIT_HLIST_HEAD(
&
pid-
>
tasks[type]); 

33
//
 将struct pid中numbers数组保存的upid添加到pidhash中 
34
     upid = pid-
>
numbers + ns-
>
level; 

35
     spin_lock_irq(
&
pidmap_lock); 

36
for
 ( ; upid 
>
= pid-
>
numbers; --
upid) 

37
         hlist_add_head_rcu(
&
upid-
>
pid_chain, 

38
&
pid_hash[pid_hashfn(upid-
>
nr, upid-
>
ns)]); 

39
     spin_unlock_irq(
&
pidmap_lock); 

40
//
 成功返回 
41
out
: 

42
return
 pid; 

43
//
 错误处理 
44
out_free: 

45
while
 (++i 
<
= ns-
>
level) 

46
         free_pidmap(pid-
>
numbers +
 i); 

47
     kmem_cache_free(ns-
>
pid_cachep, pid); 

48
     pid =
 NULL; 

49
goto
out
; 

50
 }
```



