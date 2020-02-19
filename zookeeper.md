ZooKeeper更像是分布式的辅助工具，使得应用开发人员可以更多关注应用本身的逻辑，而不是协同关注。类似于文件系统API,
使得开发人员可以实现通用的协作任务，包括选举主节点，管理组内成员关系，管理元数据等。
Zookeeper使用实例：
	Apache Hbase：用于选举一个集群内的主节点，以便跟踪可用的服务器，并保存集群的元数据
	Apache Kafka：基于发布-订阅魔性的消息系统。用于检测崩溃，实现主题的发现，并保持主题的生产和消费状态
	

主从：
	主要任务：选举主节点，跟踪有效的从节点，维护应用元数据
	必须解决的问题：主节点崩溃，从节点崩溃，通信故障
	主节点失效：
		需要备份主节点，主节点崩溃时进行故障转移，要能恢复主节点崩溃时的状态
	从节点失效：
		客户端向主节点提交任务，之后主节点将任务派发到有效的从节点中，从节点接收到派发的任务，执行完这些任务后会向主节点报告执行状态，要求neng
	通信故障：

Paxos算法
虚拟同步技术

	/
		server id /master
		/workers
			foo.com:2181 /workers/worker-1
		/tasks
			run cmd	/task/task-1-2
		/assign
			/assign/worker-1
				/assign/worker-1/task-1-1
./workers 节点作为父节点，其下每个znode子节点保存了系统中一个可用从节点信息
./tasks 节点作为父节点，其下每个znode子节点保存了所有已经创建并等待从节点执行的任务的信息
./assign 节点作为父节点，其下每个znode子节点保存了分配到某个从节点的一个任务的信息，当主节点为某个从节点分配了一个任务，就会在/assign下增加一个子节点

ZooKeeper的api暴露了以下方法：
	create /path data 创建了一个名为/path的znode节点，并包含数据 data
	delete/path 删除名为/path的znode
	exists /path检查是否存在名为/path的节点
	setData/path data 设置名为/path的znode的数据为data
	getData/path 返回名为/path节点的所有子节点列表
	getChildren/path 返回所有/path 节点的所有子节点的类表
znode持久节点和临时节点
	持久节点，需要delete删除
	临时节点，和会话有关
一个临时node在两种情况下将会被删除：
	1.当创建该znode的客户端的会话因超时或主动关闭而终止时
	2.当某个客户端主动删除该节点时
临时znode不允许有子节点


有序节点
	当创建有序节点时，一个序号会被追加到路径之后，在路径之后追加序号

znoe一共四种：
	持久的，临时的，持久有序的，临时有序

zookeeper为了高效率，放弃使用轮询，使用通知的机制，通知是一次性的，所有可能在处理通知事件时错过新的事件，所以在处理完之后通常会刷新一次看是不是有新的事件

使用版本来避免避免并发引起的问题，类似于cas

zookeeper工作模式
	独立模式
	仲裁模式
仲裁模式下，Zookeeper复制集群中的所有服务器的数据树，但如果让一个客户端等待每个服务器完成数据保存后再继续，延迟问题无法接受
所以设定法定人数，即为了使Zookeeper工作必须有效运行的服务器的最小数量
	通过使用多数方案，我们就可以容许f个服务器的崩溃，在这里，f为小于集合中服务器数量的一半
即五个，法定人数为3
3个服务器，法定人数为3

集群使用：
	配置文件中指明所有服务器的地址和端口，打开三个，就会自动连接

实现一个原语：通过zookeeper实现锁

	为了获得一个锁，每个进程尝试创建znode，名为/lock，如果进程p成功创建了znode，就表示他获得了锁并可以继续执行其临界区域的代码，不过一个潜在的问题是进程p可能崩溃，导致这个锁永远无法释放，这种情况下，没有任何其他进程可以再次获得这个锁，整个系统可能因死锁失灵，所以创建的节点应该为临时节点


一个使用主从模式例子的实现：
	主节点，从节点，客户端
	创建一个临时znode，名为/master
	create -e /master "master1.example.com:2223"
	第二个进程：
		创建主节点失败
		设置监视点
		stat /master true
	创建/workers，/tasks和/assign
	从节点：create -e /workers/worker1.example.com "worker1.example.com:2224"
	create -e /workers/worker1.example.com "worker1.example.com:2224"
	监视： ls /assign/worker1.example.com true

	客户端：
		create -s /tasks/task- "cmd"
		监视执行情况：ls /tasks/task-00000000 true
	主节点收到新任务通知
		ls /tasks
		ls /workers
		create /assign/worker1.example.com/task-000000 ""
	从节点接收到新任务通知
		ls /assign/worker1.example.com
	从节点完成任务之后新家一个状态：
	create /tasks/task-00000000/status "done"
	客户端收到通知：
		并检查执行结果
