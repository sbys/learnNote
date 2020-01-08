# learnNote
工具
jps 类似于ps，查看java进程
  参数：
    -q：只输出进程ID
    -m:输出传入main方法的参数
    -l:输出完全的报名，应用主类名，jar包的完全路径
    -v:输出jvm的参数
    -V：输出通过flag文件传递到jvm中的参数
    jps [args] [host]查看远程服务
 
jstat 根据pid查看jvm的gc情况
   jstat [args] [pid] [间隔时间/ms] [查询次数]
   参数：
    -class:类加载统计
      Loaded：加载的class数量
      Bytes：所占用空间大小
      unloaded：未加载数量
      bytes：未加载占用空间
      time：时间
     -compiler：编译统计
      compiled:编译数量
      failed：失败数量
      invalid：不可用数量
      time：时间
      failedtype：失败类型
      failedMethod：失败的方法
     -gc：垃圾回收统计
      SOC:第一个存活区大小
      S1C:第二个存活区大小
      S0U:第一个幸存区的使用大小
      S1U:第二个幸存区的使用大小
      EC:伊甸园区的大小
      EU:伊甸园区的使用大小
      OC:老年代大小
      OU:老年代使用大小
      MC:方法区大小
      MU:方法区使用大小
      CCSC:压缩类空间大小
      CCSU:压缩类空间使用大小
      YGC:年轻代垃圾回收
      YGCT:年轻代垃圾回收消耗时间
      FGC:老年代垃圾回收次数
      FGCT:老年代垃圾回收消耗时间
      GCT:垃圾回收消耗总时间
     -gccapacity：堆内存统计
     -gcnew:新生代垃圾回收统计
     -gcnewcapacity:新生代内存统计
     -gcold:老年代垃圾回收统计
     -gcoldcapacity:老年代内存统计
     -gcmetacapacity:元数据空间统计
     -gcutil:总结垃圾回收统计
     -printcompilation：jvm编译方法统计
jmap：jmap查看当前java堆和元数据区的详细信息 jmap [options] [pid]
  -heap:
  jmap -dump:format=b,file=文件名 PID 生成dump文件
jhat：分析dump文件
  jhat  -J-Xmx512M a.log，开启一个7000端口的本地服务
mat：推荐进行dump分析的工具
jstack:查看运行时线程栈信息 jstack [args] pid
jconsole:图形界面化查看cpu，内存，类加载，线程运行情况
jvisualvm：可以进行堆dump和线程dump
  
