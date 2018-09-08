随着Docker，Linux Containers等工具的出现，将Linux进程隔离到自己的小系统环境中变得非常简单。 这使得可以在单个真正的Linux机器上运行一系列应用程序，并确保其中没有两个可以相互干扰，而无需使用虚拟机。 这些工具对PaaS提供商来说是一个巨大的福音。 但究竟发生什么事情呢？  
这些工具依赖于Linux内核的许多功能和组件。 其中一些功能最近被引入，而其他功能还需要您修补内核本身。 但是，使用Linux命名空间的其中一个关键组件是Linux的一个功能，因为2.6.24版本是在2008年发布的。  
任何熟悉chroot的人都已经有了一个基本的想法，即Linux命名空间可以做什么，以及如何通用命名空间。 正如chroot允许进程看到任何任意目录作为系统的根（独立于其余进程），Linux命名空间也允许操作系统的其他方面也被独立修改。 这包括进程树，网络接口，挂载点，进程间通信资源等。

为什么使用namespaces来进行进程的隔离?  
在单用户计算机中，单个系统环境可能会很好。 但是，在要运行多个服务的服务器上，安全性和稳定性至关重要，即服务尽可能彼此隔离。 想象一下运行多个服务的服务器，其中一个服务器被入侵者泄露。 在这种情况下，入侵者可能能够利用该服务并对其他服务进行工作，甚至可能损害整个服务器。 命名空间隔离可以提供一个安全的环境来消除这种风险。  
Docker这样的命名空间工具还可以更好地控制进程对系统资源的使用，使得这些工具非常受PaaS提供者的使用。像Heroku和Google App Engine这样的服务使用这样的工具在相同的真实硬件上隔离和运行多个Web服务器应用程序。这些工具允许他们运行每个应用程序（可能已经由多个不同的用户部署），而不用担心其中一个使用太多的系统资源，或者干扰和或与同一台机器上的其他已部署的服务相冲突。通过这种过程隔离，甚至可以为每个孤立的环境完全不同的依赖软件（和版本）堆栈！  
如果您使用Docker这样的工具，您已经知道这些工具能够隔离小型“容器”中的进程。在Docker容器中运行的进程就像在虚拟机中运行它们一样，只有这些容器比虚拟机轻得多。虚拟机通常会模拟操作系统顶部的硬件层，然后运行另一个操作系统。这允许您在虚拟机中运行进程，与实际操作系统完全隔离。但虚拟机很重！另一方面，Docker容器使用真实操作系统的一些关键功能，包括命名空间，并确保类似的隔离级别，但不会模拟硬件并在同一台机器上运行另一个操作系统。这使得它们非常轻巧。

### Process Namespace {#process-namespace}

历史上，Linux内核维护了一个单一的进程树。该树包含对父子层次结构中当前运行的每个进程的引用。鉴于具有足够权限并满足某些条件的过程，可以通过向其附加示踪器或甚至可以将其杀死来检查另一过程。

随着Linux命名空间的引入，可以拥有多个“嵌套”进程树。每个进程树都可以有一个完全孤立的进程。这可以确保属于一个进程树的进程不能检查或杀死 - 实际上甚至不能知道其他兄弟或父进程树中的进程的存在。

每次启动Linux的计算机都会启动，它只需一个进程（进程标识符（PID））1。该进程是进程树的根目录，并通过执行适当的维护工作并启动它来启动系统的其余部分正确的守护进程/服务。所有其他进程从树中的这个进程下面开始。 PID命名空间允许使用自己的PID 1进程来分离一个新的树。执行此操作的过程将保留在原始树中的父命名空间中，但会使该子进程成为其自己的进程树的根。

通过PID命名空间隔离，子命名空间中的进程无法知道父进程的存在。但是，父命名空间中的进程具有子命名空间中进程的完整视图，就像它们是父命名空间中的任何其他进程一样。

可以创建嵌套的子命名空间集：一个进程在新的PID命名空间中启动子进程，该子进程在新的PID命名空间中产生另一个进程，依此类推。

随着PID命名空间的引入，单个进程现在可以有多个与之关联的PID，一个位于下面的每个命名空间。 在Linux源代码中，我们可以看到一个名为pid的结构，用于跟踪一个PID，现在可以通过使用名为upid的结构跟踪多个PID：

```
struct upid {
  int nr;                     // the PID value
  struct pid_namespace *ns;   // namespace where this PID is relevant
  //
...

};

struct pid {
  //
...

  int level;                  // number of upids
  struct upid numbers[
0
];     // array of upids
};

```

要创建一个新的PID命名空间，必须使用特殊标志CLONE\_NEWPID调用clone（）系统调用。 （C提供一个包装器来公开这个系统调用，许多其他流行的语言也是如此）。而且下面讨论的其他命名空间也可以使用unshare（）系统调用来创建，PID命名空间只能在新的 使用clone（）生成进程。 一旦使用此标志调用clone（），新进程将立即在新的PID命名空间中启动新的进程树。 这可以用简单的C程序来证明：

```
#define _GNU_SOURCE
#include 
<
sched.h
>
#include 
<
stdio.h
>
#include 
<
stdlib.h
>
#include 
<
sys/wait.h
>
#include 
<
unistd.h
>
static
char
 child_stack[
1048576
];

static
int
 child_fn() {
  
printf
(
"PID: %ld\n"
, (
long
)getpid());
  
return
0
;
}

int
 main() {
  pid_t child_pid = clone(child_fn, child_stack+
1048576
, CLONE_NEWPID | SIGCHLD, NULL);
  
printf
(
"clone() = %ld\n"
, (
long
)child_pid);

  waitpid(child_pid, NULL, 
0
);
  
return
0
;
}

```

  
使用root权限编译并运行此程序，您将注意到类似于此的输出：

```
clone
() = 5304

PID:

1

```

从child\_fn中打印的PID将为1。  
即使上述这个命名空间教程代码在某些语言中并不比“Hello，world”长得多，但幕后却发生了很多。 clone（）函数，正如您所期望的那样，通过克隆当前的进程并在child\_fn（）函数的开始处开始执行，创建了一个新进程。 但是，在执行此操作时，它将新进程与原始进程树分离，并为新进程创建了一个单独的进程树。

尝试用下面的代替static int child\_fn（）函数，从隔离进程的角度打印父PID：

```
static
int
 child_fn() {
  
printf
(
"Parent PID: %ld\n"
, (
long
)getppid());
  
return
0
;
}

```

这次运行程序产生以下输出：

```
clone
() = 
11449
Parent

PID: 
0

```

注意从隔离进程的角度来看，父PID是否为0，表明没有父级。 尝试再次运行相同的程序，但这次从clone（）函数调用中删除CLONE\_NEWPID标志：

```
pid_t child_pid = 
clone
（child_fn，child_stack + 
1048576
，SIGCHLD，
NULL
）;

```

这一次，你会注意到父PID不再是0：

```
clone（）
=
 11561
父PID：11560

```

但是，这只是我们教程中的第一步。 这些进程仍然可以无限制地访问其他常见或共享资源。 例如，网络接口：如果上面创建的子进程是在端口80上监听的，则会阻止系统上的其他进程能够监听。

Linux Network Namespace  
这是网络命名空间变得有用的地方。 网络命名空间允许这些进程中的每一个都可以看到完全不同的网络接口集。 即使每个网络命名空间的回环接口也是不同的。  
将进程隔离到自己的网络命名空间中涉及向clone（）函数调用引入另一个标志：CLONE\_NEWNET;

    #define _GNU_SOURCE
    #include 
    <
    sched.h
    >
    #include 
    <
    stdio.h
    >
    #include 
    <
    stdlib.h
    >
    #include 
    <
    sys/wait.h
    >
    #include 
    <
    unistd.h
    >
    static
    char
     child_stack[
    1048576
    ];

    static
    int
     child_fn() {

    printf
    (
    "New `net` Namespace:\n"
    );
      system(
    "ip link"
    );

    printf
    (
    "\n\n"
    );

    return
    0
    ;
    }

    int
     main() {

    printf
    (
    "Original `net` Namespace:\n"
    );
      system(
    "ip link"
    );

    printf
    (
    "\n\n"
    );

      pid_t child_pid = clone(child_fn, child_stack+
    1048576
    , CLONE_NEWPID |
    CLONE_NEWNET | SIGCHLD, NULL);
      waitpid(child_pid, NULL, 
    0
    );

    return
    0
    ;
    }


Output:

    Original `net` 
    Namespace
    :

    1
    : lo:

    <
    LOOPBACK,UP,LOWER_UP
    >
     mtu 
    65536
     qdisc noqueue state UNKNOWN
    mode 
    DEFAULT
    group
    default

        link/loopback

    00
    :
    00
    :
    00
    :
    00
    :
    00
    :
    00
     brd 
    00
    :
    00
    :
    00
    :
    00
    :
    00
    :
    00
    2
    : enp4s0:

    <
    BROADCAST,MULTICAST,UP,LOWER_UP
    >
     mtu 
    1500
     qdisc pfifo_fast
    state UP mode 
    DEFAULT
    group
    default
     qlen 
    1000

        link/ether

    00
    :
    24
    :
    8
    c:a1:ac:e7 brd ff:ff:ff:ff:ff:ff


    New
     `net` 
    Namespace
    :

    1
    : lo: 
    <
    LOOPBACK
    >

    mtu 
    65536
     qdisc noop state DOWN mode 
    DEFAULT
    group
    default


       link/loopback 
    00
    :
    00
    :
    00
    :
    00
    :
    00
    :
    00
     brd

    00
    :
    00
    :
    00
    :
    00
    :
    00
    :
    00


这里发生了什么？物理以太网设备enp4s0属于全局网络命名空间，如从此命名空间运行的“ip”工具所示。但是，物理接口在新的网络命名空间中不可用。此外，环回设备在原始网络命名空间中处于活动状态，但在子网络命名空间中为“down”。  
为了在子命名空间中提供可用的网络接口，有必要设置跨越多个命名空间的其他“虚拟”网络接口。一旦完成，就可以创建以太网网桥，甚至在命名空间之间路由数据包。最后，为了使整个工作起作用，必须在全局网络命名空间中运行“路由进程”，以从物理接口接收流量，并将其通过适当的虚拟接口路由到正确的子网络命名空间。也许你可以看到为什么像Docker这样的工具，为你做这一切都是如此受欢迎！

要手动执行此操作，您可以通过从父命名空间运行单个命令，在父命名空间和子命名空间之间创建一对虚拟以太网连接：

```
ip link add name veth0 
type
veth
peer
name
veth1
netns
<
pid
>

```

在这里，应该由父目录观察到的子命名空间中的进程的进程ID替换。 运行此命令将在这两个命名空间之间建立管状连接。 父命名空间保留veth0设备，并将veth1设备传递到子命名空间。 任何进入其中一个端点的东西都是通过另一端出来的，就像从两个真实节点之间的真实以太网连接所期望的一样。 因此，这个虚拟以太网连接的双方必须分配IP地址。

### Mount Namespace {#mount-namespace}

Linux还维护系统的所有安装点的数据结构。 它包括诸如安装了哪些磁盘分区，安装在哪里的信息，它们是否只读等等。 使用Linux命名空间，可以克隆此数据结构，以便不同命名空间下的进程可以更改安装点，而不会相互影响。  
创建单独的mount命名空间的作用类似于执行chroot（）。 chroot（）是好的，但它不提供完全的隔离，其效果仅限于根安装点。 创建单独的安装命名空间允许这些隔离进程中的每一个与原始安装点结构完全不同。 这允许您为每个独立进程拥有不同的根，以及特定于这些进程的其他安装点。 在本教程中谨慎使用时，可以避免暴露有关底层系统的任何信息。

```
The 
clone
() flag required to achieve this is CLONE_NEWNS:

clone
(child_fn,
child_stack+
1048576
, CLONE_NEWPID | CLONE_NEWNET | CLONE_NEWNS |
SIGCHLD, 
NULL
)

```

最初，子进程看到与其父进程完全相同的挂载点。但是，在新的安装命名空间下，子进程可以挂载或卸载其所需的任何端点，并且更改将不影响其父命名空间，也不影响整个系统中的任何其他安装命名空间。例如，如果父进程在根目录中安装了特定的磁盘分区，那么隔离的进程将会看到在开始时根安装完全相同的磁盘分区。但是隔离进程尝试将根分区更改为其他分区时，隔离安装命名空间的好处是显而易见的，因为该更改只会影响分离的安装命名空间。  
有趣的是，这实际上使得使用CLONE\_NEWNS标志直接生成目标子进程成为一个坏主意。一个更好的方法是使用CLONE\_NEWNS标志启动一个特殊的“init”进程，使“init”进程根据需要更改“/”，“/ proc”，“/ dev”或其他挂载点，然后启动目标进程。这在命名空间教程结尾附近有一些更详细的讨论。

### Other Namespaces {#other-namespaces}

还有其他命名空间可以将这些进程隔离开，即用户，IPC和UTS。 用户命名空间允许进程在命名空间中具有root权限，而不会授予对命名空间之外的进程的访问权限。 通过IPC命名空间隔离进程给它自己的进程间通信资源，例如System V IPC和POSIX消息。 UTS命名空间隔离系统的两个特定标识符：nodename和domainname。  
显示如何隔离UTS命名空间的快速示例如下所示：

```
#define _GNU_SOURCE
#include 
<
sched.h
>
#include 
<
stdio.h
>
#include 
<
stdlib.h
>
#include 
<
sys/utsname.h
>
#include 
<
sys/wait.h
>
#include 
<
unistd.h
>
static
char
 child_stack[
1048576
];


static
void
 print_nodename() {
  
struct
 utsname utsname;
  uname(
&
utsname);
  
printf
(
"%s\n"
, utsname.nodename);
}


static
int
 child_fn() {
  
printf
(
"New UTS namespace nodename: "
);
  print_nodename();
  
printf
(
"Changing nodename inside new UTS namespace\n"
);
  sethostname(
"GLaDOS"
, 
6
);

  
printf
(
"New UTS namespace nodename: "
);
  print_nodename();
  
return
0
;
}


int
 main() {
  
printf
(
"Original UTS namespace nodename: "
);
  print_nodename();
  pid_t child_pid = clone(child_fn, child_stack+
1048576
, CLONE_NEWUTS |SIGCHLD, NULL);
  sleep(
1
);
  
printf
(
"Original UTS namespace nodename: "
);
  print_nodename();
  waitpid(child_pid, NULL, 
0
);
  
return
0
;
}

```

This program yields the following output:

```
Original UTS 
namespace

nodename: XT

New
 UTS 
namespace

nodename: XT
Changing nodename inside

new
 UTS 
namespace
New
 UTS 
namespace

nodename: GLaDOS
Original
UTS 
namespace
 nodename: XT

```

在这里，child\_fn（）打印nodename，将其更改为其他内容，并再次打印。 当然，这种变化只发生在新的UTS命名空间内。  
有关所有命名空间提供和隔离的更多信息，请参见本教程

### Cross-Namespace Communication {#cross-namespace-communication}

通常有必要在父级和子级命名空间之间建立某种通信。这可能是在孤立的环境中进行配置工作，或者只是保留从外部窥视该环境的状态的能力。一种方法是在该环境中保持SSH守护程序的运行。每个网络命名空间内可以有一个单独的SSH守护进程。但是，运行多个SSH守护进程会使用诸如内存等宝贵资源。这是一个特殊的“init”进程再次证明是一个好主意。  
“init”进程可以在父命名空间和子命名空间之间建立通信通道。该通道可以基于UNIX套接字，甚至可以使用TCP。要创建跨越两个不同安装命名空间的UNIX套接字，您需要先创建子进程，然后创建UNIX套接字，然后将该子节点隔离为单独的安装命名空间。但是我们如何才能先创建流程，然后再隔离呢？ Linux提供了unshare（）。这个特殊的系统调用允许一个进程与原来的命名空间隔离，而不是使父进程首先隔离该小孩。例如，以下代码与网络命名空间部分中之前提到的代码具有完全相同的效果：

    #define _GNU_SOURCE
    #include 
    <
    sched.h
    >
    #include 
    <
    stdio.h
    >
    #include 
    <
    stdlib.h
    >
    #include 
    <
    sys/wait.h
    >
    #include 
    <
    unistd.h
    >
    static
    char
     child_stack[
    1048576
    ];


    static
    int
     child_fn() {

    //calling unshare() from inside the init process lets you create a new
    namespace
     after a 
    new
     process has been spawned
        unshare(CLONE_NEWNET);


    printf
    (
    "New `net` Namespace:\n"
    );
        system(
    "ip link"
    );

    printf
    (
    "\n\n"
    );

    return
    0
    ;
    }


    int
     main() {

    printf
    (
    "Original `net` Namespace:\n"
    );
      system(
    "ip link"
    );

    printf
    (
    "\n\n"
    );

      pid_t child_pid = clone(child_fn, child_stack+
    1048576
    , CLONE_NEWPID | SIGCHLD, NULL);

      waitpid(child_pid, NULL, 
    0
    );

    return
    0
    ;
    }


由于“init”过程是您设计的，所以您可以先执行所有必要的工作，然后在执行目标子进程之前将其与系统的其余部分隔离开来。

### Conclusion {#conclusion}

实现系统隔离的基本思想，这是Docker或Linux Containers等工具架构的组成部分。 在大多数情况下，最好简单地使用这些现有工具之一，这些工具已经是已知和测试的。 但在某些情况下，拥有自己的定制过程隔离机制可能是有意义的，在这种情况下，这个命名空间教程将会极大地帮助您。  
在本文中已经介绍了更多的内容，而且还有更多的方法可能会限制您的目标进程以增加安全性和隔离度。 但是，希望这对于有兴趣了解更多关于命名空间与Linux的隔离真正有效的人来说，这可以成为一个有用的起点。

