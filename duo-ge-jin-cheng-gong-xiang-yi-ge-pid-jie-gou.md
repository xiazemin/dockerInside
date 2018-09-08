## 1.多个进程共享一个pid结构

  


      找了一遍代码，发现在fs/exec.c中有调用attach\_pid调用，这个调用的条件是在一个进程fork出一个线程，同时这个线程调用了exec类函数，可以想到线程执行exec类函数，导致了整个线程组的内存结构的变化，线程在执行exec类函数时，调用了函数de\_thread函数，这个函数的会杀死进程线程组中的其他的线程，包括主线程，同时把当前线程变成线程组中的主线程，同时把pid，attach到原来的主线程上。同时后面会在执行release\_task，这个函数是释放进程zombie状态下剩余的内存结构。

也就是说在attach函数和release\_task函数中间多个进程会共用一个pid结构。

  


引用一段源代码中的注释 2.6.35.13 fs/exec.c:880

  


 880                 /\* Become a process group leader with the old leader's pid. 

  


 881                  \* The old leader becomes a thread of the this thread group     . 

  


 882                  \* Note: The old leader also uses this pid until release\_task 

  


 883                  \*       is called.  Odd but simple and correct. 

  


 884                  \*/

  


  


  


下de\_thread函数做的一些事情：

  


 819         if \(signal\_group\_exit\(sig\)\) {   //对整个group发退出信号

  


 820                 /\* 

  


 821                  \* Another group action in progress, just 

  


 822                  \* return so that the signal is processed. 

  


 823                  \*/ 

  


 824                 spin\_unlock\_irq\(lock\); 

  


 825                 return -EAGAIN; 

  


 826         } 

  


 827 

  


 828         sig-&gt;group\_exit\_task = tsk;    //group\_exit\_tas变量还没太明白搞 

 829         sig-&gt;notify\_count = zap\_other\_threads\(tsk\);   //等待线程组中除了tsk线程的退出

 830         if \(!thread\_group\_leader\(tsk\)\) 

  


 831                 sig-

&gt;

notify\_count--; 

  


 832 

  


 833         while \(sig-

&gt;

notify\_count\) { 

  


 834                 \_\_set\_current\_state\(TASK\_UNINTERRUPTIBLE\); 

  


 835                 spin\_unlock\_irq\(lock\); 

  


 836                 schedule\(\); 

  


 837                 spin\_lock\_irq\(lock\); 

  


 838         } 

  


 841         /\* 

  


 842          \* At this point all other threads have exited, all we have to 

  


 843          \* do is to wait for the thread group leader to become inactive, 

  


 844          \* and to assume its PID: 

  


 845          \*/ 

  


 846         if \(!thread\_group\_leader\(tsk\)\) {        //如果当前不是线程组主线程，后面会把当前pid，attach到主线程上

  


 847                 struct task\_struct \*leader = tsk-

&gt;

group\_leader; 

  


 848 

  


 849                 sig-

&gt;

notify\_count = -1; /\* for exit\_notify\(\) \*/ 

  


 850                 for \(;;\) {                                          

  


 851                         write\_lock\_irq\(

&

tasklist\_lock\); 

  


 852                         if \(likely\(leader-

&gt;

exit\_state\)\)             //等待主线程的结束

  


 853                                 break; 

  


 854                         \_\_set\_current\_state\(TASK\_UNINTERRUPTIBLE\); 

  


 855                         write\_unlock\_irq\(

&

tasklist\_lock\); 

  


 856                         schedule\(\); 

  


 880                 /\* Become a process group leader with the old leader's pid. 

  


 881                  \* The old leader becomes a thread of the this thread group. 

  


 882                  \* Note: The old leader also uses this pid until release\_task 

  


 883                  \*       is called.  Odd but simple and correct. 

  


 884                  \*/ 

  


 885                 detach\_pid\(tsk, PIDTYPE\_PID\); 

  


 886                 tsk-

&gt;

pid = leader-

&gt;

pid;                            //获得主线程的pid结构

  


 887                 attach\_pid\(tsk, PIDTYPE\_PID,  task\_pid\(leader\)\);   //把当前线程的pid  attach到主线程的pid上，这时pid的tasks会有多个线程结构（task\_struct）

  


 888                 transfer\_pid\(leader, tsk, PIDTYPE\_PGID\); 

  


 889                 transfer\_pid\(leader, tsk, PIDTYPE\_SID\); 

  


 890 

  


 891                 list\_replace\_rcu\(

&

leader-

&gt;

tasks, 

&

tsk-

&gt;

tasks\); 

  


 892                 list\_replace\_init\(

&

leader-

&gt;

sibling, 

&

tsk-

&gt;

sibling\); 

  


 893 

  


 894                 tsk-

&gt;

group\_leader = tsk; 

  


 895                 leader-

&gt;

group\_leader = tsk; 

  


 896 

  


 897                 tsk-

&gt;

exit\_signal = SIGCHLD; 

  


 898 

  


 899                 BUG\_ON\(leader-

&gt;

exit\_state != EXIT\_ZOMBIE\); 

  


 900                 leader-

&gt;

exit\_state = EXIT\_DEAD; 

  


 901                 write\_unlock\_irq\(

&

tasklist\_lock\); 

  


 902 

  


 903                 release\_task\(leader\);                //这时释放掉主线程的内存结构。

  


  


  


 说明一下：每个task\_struct的thread\_group字段是内核中hlist中的一个节点，也就是说通过这个字段，通过container\_of函数来映射出task\_struct结构体，在fork函数中会初始化这个thread\_group，如果是fork的线程，那个会把这个task\_struct，放到父进程（主线程）的thread\_group中，也就是说每个线程的task\_group中所代表的线程组中都有当前进程的task\_struct,每个线程的主线程就是这些线程的父进程。 

  


代码：Kernel/fork.c

  


1258         if \(clone\_flags 

&

 CLONE\_THREAD\) {         //线程

  


1259                 current-

&gt;

signal-

&gt;

nr\_threads++; 

  


1260                 atomic\_inc\(¤t-

&gt;

signal-

&gt;

live\); 

  


1261                 atomic\_inc\(¤t-

&gt;

signal-

&gt;

sigcnt\); 

  


1262                 p-

&gt;

group\_leader = current-

&gt;

group\_leader;             

  


1263                 list\_add\_tail\_rcu\(

&

p-

&gt;

thread\_group, 

&

p-

&gt;

group\_leader-

&gt;

thread\_group\); //把当前的线程加入到父进程的线程组中 

  


1264         } 

  


  


## 2.寻找空闲的pid

  


    没有说清楚的就是如何从一个bitmap中寻找位为0的位置。上篇的分析我们知道，pid的分配的情况被记在了pid\_namespace中的pid\_map中，pid\_map被看做是一个bitmap，被用过的位置置为1，没有用过的位置位为0，寻找位为0的函数是find\_next\_zero\_bit。这个函数的思想就是把bitmap切成多个long（连续64位）来看，然后根据位移屏蔽到一些无关的为1的位（offset之前的位不看），然后取反，可知，如果这个取反之前如果有一位为0，那么取反之后的long的值肯定会大于0，那么如果剩下的long的值大于0，就可以判断在64位中有为0的位，那么在用汇编bsf指令找出这个为0的位置。所以函数分为两个步骤，第一步是确定一个范围内有没有0的位，第二步就是如果有0的位置，把它从中找出来。 

  


find\_next\_bit.c:

  


67 unsigned long find\_next\_zero\_bit\(const unsigned long \*addr, unsigned long size, 

  


 68                                  unsigned long offset\)      //addr是pid\_map的首地址，size是这个pid\_map的规模，offset是从哪个位移开始寻找位为0的位置。

  


 69 { 

  


 70         const unsigned long \*p = addr + BITOP\_WORD\(offset\); 

  


 71         unsigned long result = offset 

&

 ~\(BITS\_PER\_LONG-1\); 

  


 72         unsigned long tmp; 

  


 73 

  


 74         if \(offset 

&gt;

= size\) 

  


 75                 return size; 

  


 76         size -= result;  //2

  


 77         offset %= BITS\_PER\_LONG;   

  


 78         if \(offset\) { 

  


 79                 tmp = \*\(p++\); 

  


 80                 tmp \|= ~0UL 

&gt;

&gt;

 \(BITS\_PER\_LONG - offset\); 

  


 81                 if \(size 

&lt;

 BITS\_PER\_LONG\)    //如果size不足1个long型变量， 

  


 82                         goto found\_first;          

  


 83                 if \(~tmp\)                                    //tmp取反如果大于0，代表在这段空间中有0位

  


 84                         goto found\_middle;         

  


 85                 size -= BITS\_PER\_LONG;     

  


 86                 result += BITS\_PER\_LONG;   

  


 87         } 

  


 88         while \(size 

&

 ~\(BITS\_PER\_LONG-1\)\) {  //遍历下一个64位空间

  


 89                 if \(~\(tmp = \*\(p++\)\)\)       

  


 90                         goto found\_middle;         

  


 91                 result += BITS\_PER\_LONG;   

  


 92                 size -= BITS\_PER\_LONG;     

  


 93         } 

  


 94         if \(!size\) 

  


 95                 return result; 

  


 96         tmp = \*p; 

  


 97 

  


 98 found\_first: 

  


 99         tmp \|= ~0UL 

&lt;

&lt;

 size; 

  


100         if \(tmp == ~0UL\)        /\* Are any bits zero? \*/  

  


101                 return result + size;   /\* Nope. \*/ 

  


102 found\_middle: 

  


103         return result + ffz\(tmp\); 

  


104 } 

  


  


举例说明整个函数的思想：

![](http://blog.chinaunix.net/attachment/201301/17/27767798_1358390119yNoL.png)

  


      根据上图中假设上面有250位的内存地址空间，那么首地址就是addr，size是250，offset70，那么这个这个函数的目的就是寻找addr开始，最大位移为250的地址空间，从位移是70的位置开始寻找后面是否位是0的位置。 

  


 那么整个地址空间被切割成很多个64位来处理，因为可以把这64位转化为1个long型的变量，所以第一步就是取得包括位移为70的这个long型变量的首地址。如果这个offset这个位移求64位的掩码大于0，证明这个offset是在这个64位的中间位置（不是第一位），那么就到了tmp \|= ~0UL 

&gt;

&gt;

 \(BITS\_PER\_LONG –offset\)；这里tmp就是long变量的值，~0UL操作就是64位1，然后向右移动64-6=58位，那么~0UL

&gt;

&gt;

 \(BITS\_PER\_LONG –offset\)结果就是高位58个0，和低位的6个1，那么这个tmp再和前面那个结果做与的操作，那么可以知道tmp的6个低位肯定都是1，tmp中后面58位该是什么还是什么。这时再对tmp取反操作，那么低位都变成0了，高位0变为1，1变为0，这个如果tmp大于0的话就代表高位58位有为0的位，那么对应到寻找pid\_map中为0的位，那么就可以确定有0的位置了。 

  


 确认tmp中有1的位，那么就该寻找这个位究竟在什么位置了，通过fzz函数 

  


361 static inline unsigned long ffz\(unsigned long word\)

  


362 {

  


363         asm\("bsf %1,%0"

  


364                 : "=r" \(word\)

  


365                 : "r" \(~word\)\);

  


366         return word;

  


367 }

  


  


从网上摘了一段关于bsf指令的说明： 

  


bsfl汇编指令： 

  


Intel汇编指令：bsf

  


oprd1,oprd2; 

  


 顺向位扫描\(bitscan forward\) 

  


 从右向左（从位0--

&gt;

位15或位31）扫描字或双字操作数oprd2中第一个含"1"的位，并把扫描到的第一个含'1'的位的位号送操作数oprd1

