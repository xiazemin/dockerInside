在Linux系统中，可以同时存在多用户多进程，那么对他们的运行协调管理，通过进程调度和进度管理可以解决，但是，整体资源是有限的，怎么把有限的资源（进程号、通信资源、网络资源等等）合理分配给各个用户所在的进程？Linux中提出了namespace机制，这是一种轻量级的虚拟化形式。再次之前，Linux中很多资源是全局管理的，例如，系统中所有进程，都是通过PID来标识的，就像每个学生的学号一样，在整个学校范围内，肯定是唯一标识这个学生的。用户的ID管理，各个用户通过全局为UID来标识，每个学校的校长也只有有一个，它的UID为0，权利最大，可以对学校内全部老师和学生发起命令。每个学生可以看到其他学生的活动，但是无权把他们赶出学校，这是可以理解的。这种集中统一的管理方式，很适合大规模人群的管理。

    随着大数据、虚拟化的兴起，Linux为了提供更加精细的资源分配管理机制，给出了namespace机制解决方法。

[![](https://images0.cnblogs.com/blog/350213/201501/210731404692653.png "image")](https://images0.cnblogs.com/blog/350213/201501/210731388442123.png)

命名空间建立系统的不同视图， 对于每一个命名空间，从用户看起来，应该像一台单独的Linux计算机一样，有自己的init进程\(PID为0\)，其他进程的PID依次递增，A和B空间都有PID为0的init进程，子容器的进程映射到父容器的进程上，父容器可以知道每一个子容器的运行状态，而子容器与子容器之间是隔离的。

    Linux中有chroot的系统调用，该方法将进程限制到文件系统的某一部分，是一种简单的命名空间机制。

    在task\_struct结构体中，有struct nsproxy \*nsproxy 这个成员变量，

```
   1:  
/*
```

```
   2:  
 * A structure to contain pointers to all per-process
```

```
   3:  
 * namespaces - fs (mount), uts, network, sysvipc, etc.
```

```
   4:  
 *
```

```
   5:  
 * 'count' is the number of tasks holding a reference.
```

```
   6:  
 * The count for each namespace, then, will be the number
```

```
   7:  
 * of nsproxies pointing to it, not the number of tasks.
```

```
   8:  
 *
```

```
   9:  
 * The nsproxy is shared by tasks which share all namespaces.
```

```
  10:  
 * As soon as a single namespace is cloned or unshared, the
```

```
  11:  
 * nsproxy is copied.
```

```
  12:  
 */
```

```
  13:  
struct
 nsproxy {
```

```
  14:  
    atomic_t count;
```

```
  15:  
    spinlock_t nslock;
```

```
  16:  
struct
 uts_namespace *uts_ns;
```

```
  17:  
struct
 ipc_namespace *ipc_ns;
```

```
  18:  
struct
 mnt_namespace *mnt_ns;
```

```
  19:  
struct
 pid_namespace *pid_ns;
```

```
  20:  
};
```

uts\_ns：UTS为Unix Timesharing System的简称，包含内存名称、版本、底层体系结构等信息。

ipc\_ns：保存所有与进程间通讯（IPC）有关的信息。

mnt\_ns: 当前装载的文件系统

pid\_ns: 有关进程ID的信息

在高级版本上，还有net\_ns的网络信息，user\_ns的资源配额的信息等。

下面以uts命名空间为例子，介绍如何创建用户空间。

从上面的框架图可以看出，所谓的子空间，就是父进程fork一个子进程出来，然后子进程与父进程不共享某些资源，那么，就可以说，这个子进程在它自己的那个命名空间内。

要达到这种效果，就必须对fork的行为进行精确控制，内核提供的如下参数来设置：

[![](https://images0.cnblogs.com/blog/350213/201501/210731420316696.png "image")](https://images0.cnblogs.com/blog/350213/201501/210731415162839.png)

UTS命名空间没有层次结构，所有信息都汇集到如下结构：

[![](https://images0.cnblogs.com/blog/350213/201501/210731428444812.png "image")](https://images0.cnblogs.com/blog/350213/201501/210731424067510.png)，kref是引用计数器，用于跟踪内核中有多少地方使用uts\_namespace的实例。它提供的属性信息如下：

[![](https://images0.cnblogs.com/blog/350213/201501/210731437814497.png "image")](https://images0.cnblogs.com/blog/350213/201501/210731432816411.png)，从名字上，可以得知uts包含系统名称、版本号、机器名称等等。使用uname -a可以查看这些信息。

> 系统初始默认值保持在init/version.c 中的init\_uts\_ns全局变量中，在系统初始化task时，配置init\_task。
>
> 用户可以在fork时，传入CLONE\_NEWUTS标准，创建新的UTS命名空间。执行此操作，会生成先前uts\_namespace的一份副本，当前进程内部的nsproxy指向此副本，然后就可以修改了。父子进程对nx\_prosy的修改不会相互影响。

    由于最初的父命名空间需要掌握所有子命名空间的所有pid信息，所有，在各级层次的命名空间的fork中，pid的分配是需要统一协调控制，对于各级子命名空间中的task\_struct来说，同一个pid在不同命名空间看到的是不一样的。 

   同一个进程可以属于多个namespace，多个进程可以使用同一个namespace，

