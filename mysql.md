# 1.  Linux内核namespace机制

Linux Namespaces机制提供一种资源隔离方案。PID,IPC,Network等系统资源不再是全局性的，而是属于某个特定的Namespace。每个namespace下的资源对于其他namespace下的资源都是透明，不可见的。因此在操作系统层面上看，就会出现多个相同pid的进程。系统中可以同时存在两个进程号为0,1,2的进程，由于属于不同的namespace，所以它们之间并不冲突。而在用户层面上只能看到属于用户自己namespace下的资源，例如使用ps命令只能列出自己namespace下的进程。这样每个namespace看上去就像一个单独的Linux系统。  
![](http://blog.chinaunix.net/attachment/201409/17/20788636_1410956873UMVp.jpg)

# 2 .  Linux内核中namespace结构体

在Linux内核中提供了多个namespace，其中包括fs \(mount\), uts, network, sysvipc,等。一个进程可以属于多个namesapce,既然namespace和进程相关，那么在task\_struct结构体中就会包含和namespace相关联的变量。在task\_struct结构中有一个指向namespace结构体的指针nsproxy。

struct task\_struct {

……..

/\* namespaces \*/

         struct nsproxy \*nsproxy;

…….

}

再看一下nsproxy是如何定义的，在include/linux/nsproxy.h文件中，这里一共定义了5个各自的命名空间结构体，在该结构体中定义了5个指向各个类型namespace的指针，由于多个进程可以使用同一个namespace，所以nsproxy可以共享使用，count字段是该结构的引用计数。

/\* 'count' is the number of tasks holding a reference.

 \* The count for each namespace, then, will be the number

 \* of nsproxies pointing to it, not the number of tasks.

 \* The nsproxy is shared by tasks which share all namespaces.

 \* As soon as a single namespace is cloned or unshared, the

 \* nsproxy is copied

\*/

struct nsproxy {

         atomic\_t count;

         struct uts\_namespace \*uts\_ns;

         struct ipc\_namespace \*ipc\_ns;

         struct mnt\_namespace \*mnt\_ns;

         struct pid\_namespace \*pid\_ns\_for\_children;

         struct net             \*net\_ns;

};

\(1\)     UTS命名空间包含了运行内核的名称、版本、底层体系结构类型等信息。UTS是UNIX Timesharing System的简称。

\(2\)    保存在struct ipc\_namespace中的所有与进程间通信（IPC）有关的信息。

\(3\)    已经装载的文件系统的视图，在struct mnt\_namespace中给出。

\(4\)    有关进程ID的信息，由struct pid\_namespace提供。

\(5\)     struct net\_ns包含所有网络相关的命名空间参数。

系统中有一个默认的nsproxy，init\_nsproxy，该结构在task初始化是也会被初始化。\#define INIT\_TASK\(tsk\)  \

{

         .nsproxy   = &init\_nsproxy,      

}

其中init\_nsproxy的定义为：

static struct kmem\_cache \*nsproxy\_cachep;



struct nsproxy init\_nsproxy = {

         .count                         = ATOMIC\_INIT\(1\),

         .uts\_ns                       = &init\_uts\_ns,

\#if defined\(CONFIG\_POSIX\_MQUEUE\) \|\| defined\(CONFIG\_SYSVIPC\)

         .ipc\_ns                        = &init\_ipc\_ns,

\#endif

         .mnt\_ns                      = NULL,

         .pid\_ns\_for\_children        = &init\_pid\_ns,

\#ifdef CONFIG\_NET

         .net\_ns                       = &init\_net,

\#endif

};

对于         .mnt\_ns  没有进行初始化，其余的namespace都进行了系统默认初始。  
![](http://blog.chinaunix.net/attachment/201409/17/20788636_1410956966R19R.jpg)



# 3.使用clone创建自己的Namespace

如果要创建自己的命名空间，可以使用系统调用clone\(\),它在用户空间的原型为

int clone\(int \(\*fn\)\(void \*\), void \*child\_stack, int flags, void \*arg\)

这里fn是函数指针，这个就是指向函数的指针，, child\_stack是为子进程分配系统堆栈空间,flags就是标志用来描述你需要从父进程继承那些资源，arg就是传给子进程的参数也就是fn指向的函数参数。下面是flags可以取的值。这里只关心和namespace相关的参数。

CLONE\_FS         子进程与父进程共享相同的文件系统，包括root、当前目录、umask

CLONE\_NEWNS    当clone需要自己的命名空间时设置这个标志，不能同时设置CLONE\_NEWS和CLONE\_FS。

Clone\(\)函数是在libc库中定义的一个封装函数，它负责建立新轻量级进程的堆栈并且调用对编程者隐藏了clone系统条用。实现clone\(\)系统调用的sys\_clone\(\)服务例程并没有fn和arg参数。封装函数把fn指针存放在子进程堆栈的每个位置处，该位置就是该封装函数本身返回地址存放的位置。Arg指针正好存放在子进程堆栈中的fn的下面。当封装函数结束时，CPU从堆栈中取出返回地址，然后执行fn\(arg\)函数。

/\* Prototype for the glibc wrapper function \*/

**\#include**

**int clone\(int \(\***fn**\)\(void \*\), void \***child\_stack**,**

**int**flags**, void \***arg**, ...**

**/\* pid\_t \***ptid**, struct user\_desc \***tls**, pid\_t \***ctid**\*/ \);**

       /\* Prototype for the raw system call \*/

**long clone\(unsigned long**flags**, void \***child\_stack**,**

**void \***ptid**, void \***ctid**,**

**struct pt\_regs \***regs**\);**

我们在Linux内核中看到的实现函数，是经过libc库进行封装过的，在Linux内核中的fork.c文件中，有下面的定义，最终调用的都是do\_fork\(\)函数。

\#ifdef \_\_ARCH\_WANT\_SYS\_CLONE

\#ifdef CONFIG\_CLONE\_BACKWARDS

SYSCALL\_DEFINE5\(clone, unsigned long, clone\_flags, unsigned long, newsp,

                    int \_\_user \*, parent\_tidptr,

                    int, tls\_val,

                    int \_\_user \*, child\_tidptr\)

\#elif defined\(CONFIG\_CLONE\_BACKWARDS2\)

SYSCALL\_DEFINE5\(clone, unsigned long, newsp, unsigned long, clone\_flags,

                    int \_\_user \*, parent\_tidptr,

                    int \_\_user \*, child\_tidptr,

                    int, tls\_val\)

\#elif defined\(CONFIG\_CLONE\_BACKWARDS3\)

SYSCALL\_DEFINE6\(clone, unsigned long, clone\_flags, unsigned long, newsp,

                   int, stack\_size,

                   int \_\_user \*, parent\_tidptr,

                   int \_\_user \*, child\_tidptr,

                   int, tls\_val\)

\#else

SYSCALL\_DEFINE5\(clone, unsigned long, clone\_flags, unsigned long, newsp,

                    int \_\_user \*, parent\_tidptr,

                    int \_\_user \*, child\_tidptr,

                    int, tls\_val\)

\#endif

{

         return do\_fork\(clone\_flags, newsp, 0, parent\_tidptr, child\_tidptr\);

}

\#endif

## 3.1  do\_fork函数

在clone\(\)函数中调用do\_fork函数进行真正的处理，在do\_fork函数中调用copy\_process进程处理。

long do\_fork\(unsigned long clone\_flags,

               unsigned long stack\_start,

               unsigned long stack\_size,

               int \_\_user \*parent\_tidptr,

               int \_\_user \*child\_tidptr\)

{

         struct task\_struct \*p;

         int trace = 0;

         long nr;



         /\*

          \* Determine whether and which event to report to ptracer.  When

          \* called from kernel\_thread or CLONE\_UNTRACED is explicitly

          \* requested, no event is reported; otherwise, report if the event

          \* for the type of forking is enabled.

          \*/

         if \(!\(clone\_flags & CLONE\_UNTRACED\)\) {

                   if \(clone\_flags & CLONE\_VFORK\)

                            trace = PTRACE\_EVENT\_VFORK;

                   else if \(\(clone\_flags & CSIGNAL\) != SIGCHLD\)

                            trace = PTRACE\_EVENT\_CLONE;

                   else

                            trace = PTRACE\_EVENT\_FORK;



                   if \(likely\(!ptrace\_event\_enabled\(current, trace\)\)\)

                            trace = 0;

         }



p = copy\_process\(clone\_flags, stack\_start, stack\_size,

                             child\_tidptr, NULL, trace\);

         /\*

          \* Do this prior waking up the new thread - the thread pointer

          \* might get invalid after that point, if the thread exits quickly.

          \*/

         if \(!IS\_ERR\(p\)\) {

                   struct completion vfork;

                   struct pid \*pid;



                   trace\_sched\_process\_fork\(current, p\);



                   pid = get\_task\_pid\(p, PIDTYPE\_PID\);

                   nr = pid\_vnr\(pid\);



                   if \(clone\_flags & CLONE\_PARENT\_SETTID\)

                            put\_user\(nr, parent\_tidptr\);



                   if \(clone\_flags & CLONE\_VFORK\) {

                            p-&gt;vfork\_done = &vfork;

                            init\_completion\(&vfork\);

                            get\_task\_struct\(p\);

                   }



                   wake\_up\_new\_task\(p\);



                   /\* forking complete and child started to run, tell ptracer \*/

                   if \(unlikely\(trace\)\)

                            ptrace\_event\_pid\(trace, pid\);



                   if \(clone\_flags & CLONE\_VFORK\) {

                            if \(!wait\_for\_vfork\_done\(p, &vfork\)\)

                                     ptrace\_event\_pid\(PTRACE\_EVENT\_VFORK\_DONE, pid\);

                   }



                   put\_pid\(pid\);

         } else {

                   nr = PTR\_ERR\(p\);

         }

         return nr;

}

## 3.2  copy\_process函数

在copy\_process函数中调用copy\_namespaces函数。

static struct task\_struct \*copy\_process\(unsigned long clone\_flags,

                                               unsigned long stack\_start,

                                               unsigned long stack\_size,

                                               int \_\_user \*child\_tidptr,

                                               struct pid \*pid,

                                               int trace\)

{

          int retval;

          struct task\_struct \*p;

/\*下面的代码是对clone\_flag标志进行检查，有部分表示是互斥的，例如CLONE\_NEWNS和CLONENEW\_FS\*/

          if \(\(clone\_flags & \(CLONE\_NEWNS\|CLONE\_FS\)\) == \(CLONE\_NEWNS\|CLONE\_FS\)\)

                   return ERR\_PTR\(-EINVAL\);



          if \(\(clone\_flags & \(CLONE\_NEWUSER\|CLONE\_FS\)\) == \(CLONE\_NEWUSER\|CLONE\_FS\)\)

                   return ERR\_PTR\(-EINVAL\);



          if \(\(clone\_flags & CLONE\_THREAD\) && !\(clone\_flags & CLONE\_SIGHAND\)\)

                   return ERR\_PTR\(-EINVAL\);



          if \(\(clone\_flags & CLONE\_SIGHAND\) && !\(clone\_flags & CLONE\_VM\)\)

                   return ERR\_PTR\(-EINVAL\);



          if \(\(clone\_flags & CLONE\_PARENT\) &&

                                      current-&gt;signal-&gt;flags & SIGNAL\_UNKILLABLE\)

                   return ERR\_PTR\(-EINVAL\);



……

retval = copy\_namespaces\(clone\_flags, p\);

          if \(retval\)

                   goto bad\_fork\_cleanup\_mm;

          retval = copy\_io\(clone\_flags, p\);

          if \(retval\)

                   goto bad\_fork\_cleanup\_namespaces;

          retval = copy\_thread\(clone\_flags, stack\_start, stack\_size, p\);

          if \(retval\)

                   goto bad\_fork\_cleanup\_io;

/\*do\_fork中调用copy\_process函数，该函数中pid参数为NULL，所以这里的if判断是成立的。为进程所在的namespace分配pid，在3.0的内核之前还有一个关键函数，就是namespace创建后和cgroup的关系，

if \(current-&gt;nsproxy != p-&gt;nsproxy\) {

retval = ns\_cgroup\_clone\(p, pid\);

if \(retval\)

goto bad\_fork\_free\_pid;

但在3.0内核以后给删掉了，具体请参考[remove the ns\_cgroup](http://lists.linuxfoundation.org/pipermail/containers/2011-January/026343.html)\*/

          if \(pid != &init\_struct\_pid\) {

                   retval = -ENOMEM;

                   pid = alloc\_pid\(p-&gt;nsproxy-&gt;pid\_ns\_for\_children\);

                   if \(!pid\)

                            goto bad\_fork\_cleanup\_io;

          }…..

}  
![](http://blog.chinaunix.net/attachment/201409/17/20788636_1410957283qry2.jpg)

## 3.3  copy\_namespaces函数

在kernel/nsproxy.c文件中定义了copy\_namespaces函数。

int copy\_namespaces\(unsigned long flags, struct task\_struct \*tsk\)

{

         struct nsproxy \*old\_ns = tsk-&gt;nsproxy;

         struct user\_namespace \*user\_ns = task\_cred\_xxx\(tsk, user\_ns\);

         struct nsproxy \*new\_ns;

 /\*首先检查flag，如果flag标志不是下面的五种之一，就会调用get\_nsproxy对old\_ns递减引用计数，然后直接返回0\*/

         if \(likely\(!\(flags & \(CLONE\_NEWNS \| CLONE\_NEWUTS \| CLONE\_NEWIPC \|

                                  CLONE\_NEWPID \| CLONE\_NEWNET\)\)\)\) {

                   get\_nsproxy\(old\_ns\);

                   return 0;

         }

  /\*当前进程是否有超级用户的权限\*/

         if \(!ns\_capable\(user\_ns, CAP\_SYS\_ADMIN\)\)

                   return -EPERM;



         /\*

          \* CLONE\_NEWIPC must detach from the undolist: after switching

          \* to a new ipc namespace, the semaphore arrays from the old

          \* namespace are unreachable.  In clone parlance, CLONE\_SYSVSEM

          \* means share undolist with parent, so we must forbid using

          \* it along with CLONE\_NEWIPC.

对CLONE\_NEWIPC进行特殊的判断，\*/

         if \(\(flags & \(CLONE\_NEWIPC \| CLONE\_SYSVSEM\)\) ==

                   \(CLONE\_NEWIPC \| CLONE\_SYSVSEM\)\)

                   return -EINVAL;

 /\*为进程创建新的namespace\*/

         new\_ns = create\_new\_namespaces\(flags, tsk, user\_ns, tsk-&gt;fs\);

         if \(IS\_ERR\(new\_ns\)\)

                   return  PTR\_ERR\(new\_ns\);



         tsk-&gt;nsproxy = new\_ns;

         return 0;

}

## 3.4  create\_new\_namespaces函数

create\_new\_namespaces创建新的namespace

static struct nsproxy \*create\_new\_namespaces\(unsigned long flags,

         struct task\_struct \*tsk, struct user\_namespace \*user\_ns,

         struct fs\_struct \*new\_fs\)

{

         struct nsproxy \*new\_nsp;

         int err;

    /\*为新的nsproxy分配内存空间，并对其引用计数设置为初始1\*/

         new\_nsp = create\_nsproxy\(\);

         if \(!new\_nsp\)

                   return ERR\_PTR\(-ENOMEM\);

  /\*如果Namespace中的各个标志位进行了设置，则会调用相应的namespace进行创建\*/

         new\_nsp-&gt;mnt\_ns = copy\_mnt\_ns\(flags, tsk-&gt;nsproxy-&gt;mnt\_ns, user\_ns, new\_fs\);

         if \(IS\_ERR\(new\_nsp-&gt;mnt\_ns\)\) {

                   err = PTR\_ERR\(new\_nsp-&gt;mnt\_ns\);

                   goto out\_ns;

         }



         new\_nsp-&gt;uts\_ns = copy\_utsname\(flags, user\_ns, tsk-&gt;nsproxy-&gt;uts\_ns\);

         if \(IS\_ERR\(new\_nsp-&gt;uts\_ns\)\) {

                   err = PTR\_ERR\(new\_nsp-&gt;uts\_ns\);

                   goto out\_uts;

         }



         new\_nsp-&gt;ipc\_ns = copy\_ipcs\(flags, user\_ns, tsk-&gt;nsproxy-&gt;ipc\_ns\);

         if \(IS\_ERR\(new\_nsp-&gt;ipc\_ns\)\) {

                   err = PTR\_ERR\(new\_nsp-&gt;ipc\_ns\);

                   goto out\_ipc;

         }



         new\_nsp-&gt;pid\_ns\_for\_children =

                   copy\_pid\_ns\(flags, user\_ns, tsk-&gt;nsproxy-&gt;pid\_ns\_for\_children\);

         if \(IS\_ERR\(new\_nsp-&gt;pid\_ns\_for\_children\)\) {

                   err = PTR\_ERR\(new\_nsp-&gt;pid\_ns\_for\_children\);

                   goto out\_pid;

         }



         new\_nsp-&gt;net\_ns = copy\_net\_ns\(flags, user\_ns, tsk-&gt;nsproxy-&gt;net\_ns\);

         if \(IS\_ERR\(new\_nsp-&gt;net\_ns\)\) {

                   err = PTR\_ERR\(new\_nsp-&gt;net\_ns\);

                   goto out\_net;

         }



         return new\_nsp;



out\_net:

         if \(new\_nsp-&gt;pid\_ns\_for\_children\)

                   put\_pid\_ns\(new\_nsp-&gt;pid\_ns\_for\_children\);

out\_pid:

         if \(new\_nsp-&gt;ipc\_ns\)

                   put\_ipc\_ns\(new\_nsp-&gt;ipc\_ns\);

out\_ipc:

         if \(new\_nsp-&gt;uts\_ns\)

                   put\_uts\_ns\(new\_nsp-&gt;uts\_ns\);

out\_uts:

         if \(new\_nsp-&gt;mnt\_ns\)

                   put\_mnt\_ns\(new\_nsp-&gt;mnt\_ns\);

out\_ns:

         kmem\_cache\_free\(nsproxy\_cachep, new\_nsp\);

         return ERR\_PTR\(err\);

}

### 3.4.1 create\_nsproxy函数

static inline struct nsproxy \*create\_nsproxy\(void\)

{

         struct nsproxy \*nsproxy;



         nsproxy = kmem\_cache\_alloc\(nsproxy\_cachep, GFP\_KERNEL\);

         if \(nsproxy\)

                   atomic\_set\(&nsproxy-&gt;count, 1\);

         return nsproxy;

}



例子1：namespace pid的例子

  


\#include

\#include

\#include

\#include

\#include

\#include

\#include



static int fork\_child\(void \*arg\)

{

         int a = \(int\)arg;

         int i;

         pid\_t pid;

         char \*cmd  = "ps -el;

         printf\("In the container, my pid is: %d\n", getpid\(\)\);

 /\*ps命令是解析procfs的内容得到结果的，而procfs根目录的进程pid目录是基于mount当时的pid namespace的，这个在procfs的get\_sb回调中体现的。因此只需要重新mount一下proc,mount -t proc proc /proc\*/

         mount\("proc", "/proc", "proc", 0, ""\);

         for \(i = 0; i

                   pid = fork\(\);

                   if \(pid &lt;0\)

                            return pid;

                   else if \(pid\)

                            printf\("pid of my child is %d\n", pid\);

                   else if \(pid == 0\) {

                            sleep\(30\);

                            exit\(0\);

                   }

         }

         execl\("/bin/bash", "/bin/bash","-c",cmd, NULL\);

         return 0;

}

int main\(int argc, char \*argv\[\]\)

{

         int cpid;

         void \*childstack, \*stack;

         int flags;

         int ret = 0;

         int stacksize = getpagesize\(\) \* 4;

         if \(argc != 2\) {

                   fprintf\(stderr, "Wrong usage.\n"\);

                   return -1;

         }

         stack = malloc\(stacksize\);

         if\(stack == NULL\)

         {

                   return -1;

         }

         printf\("Out of the container, my pid is: %d\n", getpid\(\)\);

         childstack = stack + stacksize;

         flags = CLONE\_NEWPID \| CLONE\_NEWNS;

         cpid = clone\(fork\_child, childstack, flags, \(void \*\)atoi\(argv\[1\]\)\);

         printf\("cpid: %d\n", cpid\);

         if \(cpid &lt;0\) {

                   perror\("clone"\);

                   ret = -1;

                  goto out;

         }

         fprintf\(stderr, "Parent sleeping 20 seconds\n"\);

         sleep\(20\);

         ret = 0;

out:

         free\(stack\);

         return ret;

}

}运行结果：

root@ubuntu:~/c\_program\# ./namespace 7

Out of the container, my pid is: 8684

cpid: 8685

Parent sleeping 20 seconds

In the container, my pid is: 1

pid of my child is 2

pid of my child is 3

pid of my child is 4

pid of my child is 5

pid of my child is 6

pid of my child is 7

pid of my child is 8

F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD

4 R     0     1     0  0  80   0 -  1085 -      pts/0    00:00:00 ps

1 S     0     2     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     3     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     4     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     5     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     6     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     7     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

1 S     0     8     1  0  80   0 -   458 hrtime pts/0    00:00:00 namespace

例子2：UTS的例子

\#define \_GNU\_SOURCE

\#include

\#include

\#include

\#include

\#include

\#include

\#include



\#define errExit\(msg\)    do { perror\(msg\); exit\(EXIT\_FAILURE\); \

} while \(0\)



         static int              /\* Start function for cloned child \*/

childFunc\(void \*arg\)

{

         struct utsname uts;

         /\* Change hostname in UTS namespace of child \*/

         if \(sethostname\(arg, strlen\(arg\)\) == -1\)

                   errExit\("sethostname"\);

         /\* Retrieve and display hostname \*/

         if \(uname\(&uts\) == -1\)

                   errExit\("uname"\);

         printf\("uts.nodename in child:  %s\n", uts.nodename\);

         /\* Keep the namespace open for a while, by sleeping.

          \*               This allows some experimentation--for example, another

          \*                             process might join the namespace. \*/

         sleep\(200\);

         return 0;           /\* Child terminates now \*/

}

\#define STACK\_SIZE \(1024 \* 1024\)    /\* Stack size for cloned child \*/



         int

main\(int argc, char \*argv\[\]\)

{

         char \*stack;                    /\* Start of stack buffer \*/

         char \*stackTop;                 /\* End of stack buffer \*/

         pid\_t pid;

         struct utsname uts;

         if \(argc &lt; 2\) {

                   fprintf\(stderr, "Usage: %s\n", argv\[0\]\);

                   exit\(EXIT\_SUCCESS\);

         }

         /\* Allocate stack for child \*/

         stack = malloc\(STACK\_SIZE\);

         if \(stack == NULL\)

                   errExit\("malloc"\);

         stackTop = stack + STACK\_SIZE;  /\* Assume stack grows downward \*/

         /\* Create child that has its own UTS namespace;

          \*               child commences execution in childFunc\(\) \*/

         pid = clone\(childFunc, stackTop, CLONE\_NEWUTS \| SIGCHLD, argv\[1\]\);

         if \(pid == -1\)

                   errExit\("clone"\);

         printf\("clone\(\) returned %ld\n", \(long\) pid\);

         /\* Parent falls through to here \*/

         sleep\(1\);           /\* Give child time to change its hostname \*/



         /\* Display hostname in parent's UTS namespace. This will be

          \*               different from hostname in child's UTS namespace. \*/



         if \(uname\(&uts\) == -1\)

                   errExit\("uname"\);

         printf\("uts.nodename in parent: %s\n", uts.nodename\);

         if \(waitpid\(pid, NULL, 0\) == -1\)    /\* Wait for child \*/

                   errExit\("waitpid"\);

         printf\("child has terminated\n"\);

         exit\(EXIT\_SUCCESS\);

}

root@ubuntu:~/c\_program\# ./namespace\_1 test

clone\(\) returned 4101

uts.nodename in child:  test

uts.nodename in parent: ubuntu

