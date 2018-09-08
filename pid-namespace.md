## 1.pid Namespace涉及的基本数据结构

  


    linux通过命名空间管理进程pid，对于同一进程（同一个task\_struct）,在不同的命名空间中，看到的pid号不相同，每个pid命名空间有一套自己的pid管理方法，所以在不同的命名空间中调用getpid\(\)，看到的pid号是不同的。pid命名空间是一个父子关系的结构，系统初始只有一个pid命名空间，后面如果在fork进程的时候，加上新建pid命名空间的选项，那么这个新的命名空间的父命名空间就是初始的那个命名空间，在这个命名空间fork出的进程，在子命名空间和父命名空间都有一个pid号相对应到这个task\_struct上。

[![](http://blog.chinaunix.net/attachment/201301/10/27767798_1357826879Inx7.png)](http://blog.chinaunix.net/attachment/201301/10/27767798_1357826879Inx7.png)

  


      从上图中可以看出，假设namespace有3层，如果在Namespace2中fork进程，产生的进程task\_struct,如果pid是6，那么在根Namespace1中pid就是6，在Namespace2中pid就是4（自己的一套分配方式，递增方式，如果进程号被占用，就使用下一个空闲的id号，后面重点会说到id号的分配），在Namespace6中fork子进程，因为Namespace6来源于Namespace3，所以子命名空间fork的进程，这个命名空间的父命名空间都会看到这个进程，每个父命名空间根据自己id分配的情况，做一个task\_struct到内部id号的映射关系，然后在相应的命名空间中调用getpid会使用当前命名空间中的id号，而不是task\_struct中的pid。所以pid命名空间的作用就是，1个task\_struct,在不同的命名空间看到的pid是不一样的。

  


     关于pid namespace的管理，首先需要抽象出结构体pidNamespace：include/linux/pid\_namespace.h

1. struct pid\_namespace {  
2.           struct kref kref;                 //引用计数
3.           struct pidmap pidmap\[PIDMAP\_ENTRIES\]; //pid分配的bitmap，如果位为1，表示这个pid已经分配了
4.           int last\_pid;                //记录上次分配的pid，理论上，当前分配的pid=last\_pid+1
5.           struct task\_struct \*child\_reaper; //表示进程结束后，需要这个child\_reaper进程对这个进程进行托管
6.           struct kmem\_cache \*pid\_cachep;     //高速缓存，这个不太清楚，待这块分析源代码
7.           unsigned int level;                //记录这个pid namespace的深度
8.           struct pid\_namespace \*parent;      //记录父pid namespace
9.   \#ifdef CONFIG\_PROC\_FS
10.           struct vfsmount \*proc\_mnt;
11.   \#endif
12.   \#ifdef CONFIG\_BSD\_PROCESS\_ACCT
13.           struct bsd\_acct\_struct \*bacct;
14.   \#endif
15.   };
16. 
      这里比较重要的成员变量就是pidmap，它表示在这个pid命名空间的pid的分配情况，pidmap是个数组，每一位代表这个这个偏移量的pid是否分配出去，初始这个数组只有一个元素。

  


pidmap的结构：include/linux/pid\_namespace.h

1.   struct pidmap {
2.         atomic\_t nr\_free;//表示这个bitmap还有多少位为0，就是说对应的pid没有被分配出去
3.         void \*page;//表示一段连续的内存空间，每位的0或1表示对应pid是否被分配
4.   };

      默认情况下pid最大是32768，那么默认正好是1页能保存下的pid使用情况，linux默认一页的大小是4k=4\*1024\*8位=32768,如果pid的最大值超过32768那么pidmap数组就用上了，多个pidmap就是为了pid限制大于32768来设计的。

      child\_reaper的作用见[init进程对zombie](http://blog.chinaunix.net/uid-27767798-id-3466186.html)[进程的处理](http://blog.chinaunix.net/uid-27767798-id-3466186.html)。这个child\_reaper的作用就是当父进程先于子进程结束的时候，就把子进程的父进程更新为child\_reaper。

整体的pid管理结构图：

[![](http://blog.chinaunix.net/attachment/201301/10/27767798_13578276292qgB.png)](http://blog.chinaunix.net/attachment/201301/10/27767798_13578276292qgB.png)

  


      一个进程对应一个task\_struct，但是这个进程在多个namespace中都可以看见不同的pid，那么就需要一个表示pid的结构体。代码：include/linux/pid.h

1. struct pid
2.   {
3.           atomic\_t count;   //引用次数
4.           unsigned int level;//这个pid的深度
5.           /\* lists of tasks that use this pid \*/
6.           struct hlist\_head tasks\[PIDTYPE\_MAX\];//引用pid的task，看了很多的文章始终搞不清楚什么条件下，会分配同一个pid结构，看了fork中的一些逻辑，发现每次都是创建新的pid结构，这个有待研究
7.           struct rcu\_head rcu;
8.           struct upid numbers\[1\];//这个task\_struct在多个命名空间的显示。一个upid就是一个namespace的pid的表示。
9.   };
10. 
      这里最重要的成员变量就是numbers，它是个数组,表示一个task\_struct在每个namespace的id（这个id就是getpid\(\)所得到的值），number\[0\]表示最顶层的namespace，level=0,number\[1\]表示level=1，以此类推。

代码：include/linux/pid.h

1.  struct upid {
2.           /\* Try to keep pid\_chain in the same cacheline as nr for find\_vpid \*    /   
3.           int nr;                    //表示命名空间中的标识
4.           struct pid\_namespace \*ns;  //命名空间
5.          struct hlist\_node pid\_chain; //hash表中的端点
6.   };

     这里nr和ns成对出现，表示进程的在这个ns命名空间的pid为nr。管理这些pid结构，通常把他们防止在hash表中，pid\_chain是hash结构中的一个节点，所以pid\_chain就是hash表和数据之间的桥梁。这里linux内核中广泛的使用这种hash表，hash表中每个元素都是hlist\_node,那么取得每个元素所代表的value，就要通过指针和结构体，来倒推value的指针。实现机理通过函数container\_of 代码：include/linux/kernel.h

1.  /\*\*
2.   \* container\_of - cast a member of a structure out to the containing structu    re
3.   \* @ptr:        the pointer to the member.
4.   \* @type:       the type of the container struct this is embedded in.
5.   \* @member:     the name of the member within the struct.
6.   \*
7.   \*/
8.  \#define container\_of\(ptr, type, member\) \({                      \
9.          const typeof\( \(\(type \*\)0\)-
   &gt;
   member \) \*\_\_mptr = \(ptr\);    \
10.          \(type \*\)\( \(char \*\)\_\_mptr - offsetof\(type,member\) \);}\)

     这里ptr是结构体type中的成员变量member的指针，这个函数的实际含义是通过ptr指针根据结构体中member的具体偏移量来得到type结构体的首地址，然后在强转成type的指针。这里typeof是GCC内建函数，offsetof是获得结构体中member变量的指针的偏移量。这样member变量的内存地址减去member的偏移量就可以获得结构体的指针。

遗留的问题：不知道什么情况会多个进程会公用一个pid结构。

2.pid的分配

  


     fork进程的时候，需要为这个进程分配pid，应该根据这个namespace中pidmap的pid分配情况，分配适合的id，大体的过程就是根据当前namespace中的last\_pid+1,然后参照pidmap中这位是否为1，如果为1证明当前last\_pid+1已经被使用（导致这种情况是id被分配到了最大值，然后再重头选择id，之前的进程如果有还没结束，就会导致last\_pid+1,不可用），这时需要找到比last\_pid大的值，取离它最近的。如果找不到，则分配失败。

分配pid的函数：kernel/pid.c

  


1.  static int alloc\_pidmap\(struct pid\_namespace \*pid\_ns\)
2.  {
3.          int i, offset, max\_scan, pid, last = pid\_ns-
   &gt;
   last\_pid;  //取出last\_pid
4.          struct pidmap \*map;
5. 
6.          pid = last + 1;                                      //这里last+1,取得备选pid
7. //如果pid到了pidmax，那么重头开始寻找可用的pid，从RESERVED\_PIDS开始，保留RESERVED\_PIDS之前的pid号，默认300
8.          if \(pid 
   &gt;
   = pid\_max\)
9.                  pid = RESERVED\_PIDS;
10.          offset = pid 
    &
     BITS\_PER\_PAGE\_MASK;           //取得掩码，获得pidmap的掩码\(取余数\)。
11.          map = 
    &
    pid\_ns-
    &gt;
    pidmap\[pid/BITS\_PER\_PAGE\];    //根据pid获得pidmap
12.          max\_scan = \(pid\_max + BITS\_PER\_PAGE - 1\)/BITS\_PER\_PAGE - !offset;  //后面单独讲
13.          for \(i = 0; i 
    &lt;
    = max\_scan; ++i\) {
14.                  if \(unlikely\(!map-
    &gt;
    page\)\) {          //如果这个pidmap没有分配内存重新分配
15.                          void \*page = kzalloc\(PAGE\_SIZE, GFP\_KERNEL\);
16.                          /\*                          \* Free the page if someone raced with us
17.                           \* installing it:
18.                           \*/
19.                          spin\_lock\_irq\(
    &
    pidmap\_lock\);
20.                          if \(!map-
    &gt;
    page\) {
21.                                  map-
    &gt;
    page = page;
22.                                  page = NULL;
23.                          }
24.                          spin\_unlock\_irq\(
    &
    pidmap\_lock\);
25.                          kfree\(page\);
26.                          if \(unlikely\(!map-
    &gt;
    page\)\)
27.                                  break;
28.                  }
29.                     //如果nr\_free大于0表示map中还有空闲的pid的位
30.                  if \(likely\(atomic\_read\(
    &
    map-
    &gt;
    nr\_free\)\)\) {
31.                          do {
32. 33. //根据man-
    &gt;
    page基址，offset是偏移量，test\_and\_set\_bit把offset位的值置为1，可以知道如果offset位如果是1，那么还是1，返回原来被set之前的值1，表示这位表示的pid已经被使用，如果返回0，表示之前这位表示的pid未被使用，同时将这位置为了1（这个函数的实现是，内嵌汇编，bts操作）返回0，表示这位未被使用
34. 35.                                  if \(!test\_and\_set\_bit\(offset, map-
    &gt;
    page\)\) {
36.                                          atomic\_dec\(
    &
    map-
    &gt;
    nr\_free\);//空闲计数减一   
37.                                          pid\_ns-
    &gt;
    last\_pid = pid;   //重新设置last\_pid
38.                                          return pid;
39.                                  }
40.                                     //继续寻找offset之后，位为0的位置
41.                                  offset = find\_next\_offset\(map, offset\);
42.                                     //找到这个位置，根据map的序号和偏移量转换为pid
43.                                  pid = mk\_pid\(pid\_ns, map, offset\);
44.                          /\*
45.                           \* find\_next\_offset\(\) found a bit, the pid from it
46.                           \* is in-bounds, and if we fell back to the last
47.                           \* bitmap block and the final block was the same
48.                           \* as the starting point, pid is before last\_pid.
49.                           \*/
50. 51. //这里循环停止会有多种条件，如果偏移量找到了这个pid\_map的最后那么就停止查找了，因为已经到了这个map的最后一位了，那么应该从下一个pid\_map开始寻找，如果分配的pid大于允许分配最大pid的值，就该从第一个map开始寻找之前可能已经结束的进程，空闲出来的位置
52. 53.                          } while \(offset 
    &lt;
     BITS\_PER\_PAGE 
    &
    &
     pid 
    &lt;
     pid\_max 
    &
    &
54.                                          \(i != max\_scan \|\| pid 
    &lt;
     last \|\|
55.                                              !\(\(last+1\) 
    &
     BITS\_PER\_PAGE\_MASK\)\)\);
56.                  }
57. //如果当前的pid\_map没有到最后一个pid\_map,就继续寻找下一个pid\_map,这时offset=0，重头开始寻找
58.                 if \(map 
    &lt;
    &
    pid\_ns-
    &gt;
    pidmap\[\(pid\_max-1\)/BITS\_PER\_PAGE\]\) {
59.                          ++map;
60.                          offset = 0;
61.                  } else {
62. //如果当前的pid\_map到了最后一个pid\_map,那么重头第一个pid\_map开始寻找可用的pid，同时将offset设置成RESERVED\_PIDS，RESERVED\_PIDS之前的pid被保留了。
63.                          map = 
    &
    pid\_ns-
    &gt;
    pidmap\[0\];
64.                          offset = RESERVED\_PIDS;
65.                          if \(unlikely\(last == offset\)\)
66.                                  break;
67.                  }
68.                  pid = mk\_pid\(pid\_ns, map, offset\);
69.          }
70.          return -1;
71.  }
72. 
代码：

135 max\_scan = \(pid\_max + BITS\_PER\_PAGE - 1\)/BITS\_PER\_PAGE - !offset;

      这里max\_scan代表最多去寻找几个pid\_map,这里减去!offset的原因就是，如果offset为0，那么当前的pid\_map不需要重新递归寻找掩码之前的空闲位置，因为掩码为0，没有再前面的位置了，如果掩码不为0，那么需要再次递归当前的pid\_map,寻找掩码之前的位置的空闲位。

[![](http://blog.chinaunix.net/attachment/201301/10/27767798_135782902444yl.png)](http://blog.chinaunix.net/attachment/201301/10/27767798_135782902444yl.png)

  


     从上面的图看出来，如果last\_pid位于第一个pid\_map中的第三位，next就是第四位，那么max\_scan=4，如果pid\_map\[1\],pid\_map\[2\]都没有空闲位，那么需要重新查找pid\_map\[0\]中的空闲位，如果当前掩码是0，位于第一个pid\_map,那么不需要回来查找pid\_map\[0\]。

  


3.getpid函数的实现

  


     getpid函数是获得当前进程id，如果线程调用这个函数，得到的是这个线程的task\_group的pid，那么这个pid是当前namespace下的标识，并不是task\_struct中的pid值。这个函数的具体实现在kernel/timer.c

  


1.  SYSCALL\_DEFINE0\(getpid\)
 
2. {
 
3.      return task\_tgid\_vnr\(current\);
 
4. }

     系统调用直接到了这里，task\_tgid\_vnr的实现：include/linux/sched.h

  


1. static inline pid\_t task\_tgid\_vnr\(struct task\_struct\*tsk\)
 
2. {
 
3.       return pid\_vnr\(task\_tgid\(tsk\)\);
 
4. }

     这里task\_tgid\(tsk\)函数就是获得当前进程的task\_group（进程的task\_group就是它自己，线程的task\_group是它的父进程，调用pthread\_create的那个进程）的pid结构

  


1.  static inline struct pid\*task\_tgid\(struct task\_struct\*task\)
 
2. {
 
3.      return task-
   &gt;
   group\_leader-
   &gt;
   pids\[PIDTYPE\_PID\].pid;
 
4. }

     获得pid结构，就应该根据当前namespace获得pid结构中对应的进程标识了，代码：kernel/pid.c

  


1.  pid\_t pid\_vnr\(struct pid\*pid\)
 
2. {
 
3.       return pid\_nr\_ns\(pid,current-
   &gt;
   nsproxy-
   &gt;
   pid\_ns\);
 
4. }

     current-&gt;nsproxy-&gt;pid\_ns就是当前pid\_namespace

  


  


1.  pid\_t pid\_nr\_ns\(struct pid\*pid,struct pid\_namespace\*ns\)
 
2. {
 
3.      struct upid\*upid;
 
4.      pid\_t nr=0;
 
5. 6.      if\(pid
   &
   &
   ns-
   &gt;
   level
   &lt;
   =pid-
   &gt;
   level\){
 
7. //根据namespace的level深度获得upid结构，这里的upid-
   &gt;
   nr就是这个进程在这个namespace下的进程标识
8.           upid=
   &
   pid-
   &gt;
   numbers\[ns-
   &gt;
   level\];
 
9.           if\(upid-
   &gt;
   ns==ns\)
 
10.                nr=upid-
    &gt;
    nr;
 
11.     }
 
12.      return nr;
 
13. }

总结：

  


     pid命名空间可以把一个进程在不同的命名空间pid管理隔离开，使得每个命名空间都有自己的一套pid命名规则，在看以上的代码后，有疑问：什么情况下多个进程才会共用一个pid结构？希望大家给点建议 

     上面的问题，在[pid Namespace续](http://blog.chinaunix.net/uid-27767798-id-3476587.html)中解释了问题，多个进程共用一个pid结构的时机：父进程fork出子线程，然后子线程去调用exec，在这调用exec函数的过程中，首先子线程发信号使得父进程停止，子线程去attach父进程pid结构，最后再release

父进程，在段代码中，父进程和子线程会共用一个pid结构。

