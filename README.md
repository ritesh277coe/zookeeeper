# zookeeeper
Zookeeper cheat sheet, source code explanation


ZooKeeper APIs to solve the problems of distributed synchronization, cluster configuration management, group membership.
**********************************************************************************************************************

$ wget http://www.gtlib.gatech.edu/pub/apache/zookeeper/stable/zookeeper-3.4.6.tar.gz<br/>
$ ls -alh zookeeper-3.4.6.tar.gz<br/>
-rw-rw-r-- 1 saurav saurav 17M Feb 20  2014 zookeeper-3.4.6.tar.gz

Installing<br/>
$ tar -C /usr/share -zxf zookeeper-3.4.6.tar.gz<br/>
$ cd /usr/share/zookeeper-3.4.6/<br/>
$ ls <br/>
bin      CHANGES.txt      contrib      docs      ivy.xml  LICENSE.txt      README_packaging.txt      recipes  zookeeper-3.4.6.jar      zookeeper-3.4.6.jar.md5 <br/>
build.xml      conf      dist-maven      ivysettings.xml  lib      NOTICE.txt      README.txt      src       zookeeper-3.4.6.jar.asc  zookeeper-3.4.6.jar.sha1	<br/>
<br/>
$ export ZK_HOME=/usr/share/zookeeper-3.4.6<br/>

Configuration:<br/>
$ cat conf/zoo.cfg<br/>
tickTime=2000 (in ms)<br/>
dataDir=/var/lib/zookeeper (This is the location to store the in-memory state of ZooKeeper; it includes database snapshots and the transaction log of updates to the database. Extracting the ZooKeeper archive won't create this directory, so if this directory doesn't exist in the system, you will need to create it and set writable permission to it.)<br/>
clientPort=2181 (client connects at this port)<br/>
<br/>
Need JAVA:<br/>
$java --version<br/>
export JAVA_HOME=/usr/ instead of /usr/bin<br/>
what is jps?<br/>
<br/>
Starting the ZooKeeper server in standalone mode:<br/>
$./bin/zkServer.sh<br/>
./bin/zkServer.sh<br/>
JMX enabled by default<br/>
Using config: /home/tom/work/zookeeper_kafka/zookeeper-3.4.6/bin/../conf/zoo.cfg<br/>
Usage: ./bin/zkServer.sh {start|start-foreground|stop|restart|status|upgrade|print-cmd}<br/>

So to start, we can either:<br/>
$./bin/zkServer.sh start (runs the server in backgroud)<br/>
OR<br/>
$./bin/zkServer.sh start-foreground (runs in foreground)<br/>
<br/>
$ps –ef | grep zookeeper <br/>
5511 (pid of zookeeper)<br/>
<br/>
$ which jps  //jps is like ps..just showing all java programs<br/>
jps is /usr/bin/jps<br/>
$ jps<br/>
5511 QuorumPeerMain<br/>
5565 Jps<br/>
<br/>
To check status of running zookeeper: $./bin/zkServer.sh status<br/>
To stop: $./bin/zkServer.sh stop<br/>
**********************************************************************************************************************
**********************************************************************************************************************
**********************************************************************************************************************<br/>
Connecting to ZooKeeper with a Java-based shell:<br/>
${ZK_HOME}/bin/zkCli.sh –server zk_server_name/ip:port<br/>
<br/>
${ZK_HOME}/bin/zkCli.sh –server localhost:2181<br/>
Connecting to localhost:2181<br/>
...............<br/>
...............<br/>
Welcome to ZooKeeper!<br/>
JLine support is enabled<br/>
...............<br/>
WATCHER::<br/>
WatchedEvent state:SyncConnected type:None path:null<br/>
[zk: localhost:2181(CONNECTED) 0]<br/>
<br/>
To see all the commands: help<br/>
ZooKeeper -server host:port cmd args<br/>
  connect host:port
  get path [watch]
  ls path [watch]
  set path data [version]
  rmr path
  delquota [-n|-b] path
  quit 
  printwatches on|off
  create [-s] [-e] path data acl
  stat path [watch]
  close 
  ls2 path [watch]
  history 
  listquota path
  setAcl path acl
  getAcl path
  sync path
  redo cmdno
  addauth scheme auth
  delete path [version]
  setquota -n|-b val path
  
**********************************************************************************************************************
**********************************************************************************************************************
**********************************************************************************************************************
Connecting to ZooKeeper with a C-based shell
${ZK_HOME}/src/c
$ ./configure -prefix="/home/tom/work/zookeeper_kafka/cbin"
$ make
$ make install

It will create folders: bin, include and lib. Inside bin, clis_st (single thread) cli_mt (multi thread)

$./bin/cli_mt localhost:2181
Watcher SESSION_EVENT state = CONNECTED_STATE
Got a new session id: 0x148b540cc4d0004

$help


single node zookeeper = standalone not recommended for production as this becomes single point of failure.
multinode zookeeper with replicated mode on = zookeeper ensemble
Multiple zookeeper instances running on multiple machines using the same config file is called quoram. Zookeeper has leader follower model

**********************************************************************************************************************
**********************************************************************************************************************
**********************************************************************************************************************

Setting up a multinode ZooKeeper cluster
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181					//clients connect to zookeeper server at this port
initLimit=5						//initLimit: This parameter is the timeout, specified in number of ticks, for a follower to initially connect to a leader
syncLimit=2						//This is the timeout, specified in number of ticks, for a follower to sync with a leader
server.1=zoo1:2888:3888			//2888 is port where zookeeper follower will connect to leader
server.2=zoo2:2888:3888			//3888 is port used by zookeeper servers to select leader.
server.3=zoo3:2888:3888

Just do: ${ZK_HOME}/bin/zkServer.sh start
on all the machines

Then to check status: ${ZK_HOME}/bin/zkServer.sh status
output can be like:
[zoo1] # ${ZK_HOME}/bin/zkServer.sh status
JMX enabled by default
Using config: /usr/share/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower
[zoo2] # ${ZK_HOME}/bin/zkServer.sh status
JMX enabled by default
Using config: /usr/share/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: leader
[zoo3] # ${ZK_HOME}/bin/zkServer.sh status
JMX enabled by default
Using config: /usr/share/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: follower


To connect client:
$zk_Cli.sh zoo1:2181,zoo2:2181,zoo3:2181
Connecting to zoo1:2181, zoo2:2181, zoo3:2181
… … … …
Welcome to ZooKeeper!
… … … …
[zk: zoo1:2181,zoo2:2181,zoo3:2181 (CONNECTED) 0]

**********************************************************************************************************************
Running multiple node modes for ZooKeeper::
Config file1:
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/var/lib/zoo1
clientPort=2181
server.1=localhost:2666:3666
server.2=localhost:2667:3667
server.3=localhost:2668:3668

Config file1:
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/var/lib/zoo2
clientPort=2182
server.1=localhost:2666:3666
server.2=localhost:2667:3667
server.3=localhost:2668:3668

Config file1:
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/var/lib/zoo3
clientPort=2183
server.1=localhost:2666:3666
server.2=localhost:2667:3667
server.3=localhost:2668:3668

We also need to fix the server ID parameter correctly in the myid file for each instance. This can be done using the following three commands:

$ echo 1 > /var/lib/zookeeper/zoo1/myid
$ echo 2 > /var/lib/zookeeper/zoo2/myid
$ echo 3 > /var/lib/zookeeper/zoo3/myid
Now, we are all set to start the ZooKeeper instances. Let's start the instances as follows:

$ ${ZK_HOME}/bin/zkServer.sh start ${ZK_HOME}/conf/zoo1.cfg
$ ${ZK_HOME}/bin/zkServer.sh start ${ZK_HOME}/conf/zoo2.cfg
$ ${ZK_HOME}/bin/zkServer.sh start ${ZK_HOME}/conf/zoo3.cfg
Once all the instances start, we can use the zkCli.sh script to connect to the multinode ZooKeeper cluster, like we did earlier:

$ ${ZK_HOME}/bin/zkCli.sh –server  localhost:2181, localhost:2182, localhost:2183
**********************************************************************************************************************

Nodes:
Persistent: Persist unless someone deletes it. Can be deleted by any authorized client.
			create /[root] "root_data"
			get /[root]
			
Ephemeral: Lifespan of the client that created it, But visible to all clients depending upon ACL. Can be deleted by any authorized client. Ephemeral can not have children.
			[zk: localhost(CONNECTED) 1] create -e /[PacktPub] "ApacheZooKeeper"
			Created /[PacktPub]
			
			[zk: localhost(CONNECTED) 2] create -e /[PacktPub]/EphemeralChild "ChildOfEphemeralZnode"
			Ephemerals cannot have children: /[PacktPub]/EphemeralChild
sequence: A sequential znode is assigned a sequence number by ZooKeeper as a part of its name during its creation. The value of a monotonously increasing counter
		(maintained by the parent znode) is appended to the name of the znode.
		Since both persistent and ephemeral znodes can be sequential znodes, we have a total of four modes of znodes:
		• persistent
		• ephemeral
		• persistent_sequential
		• ephemeral_sequential

		[zk: localhost(CONNECTED) 1] create -s /[PacktPub] "PersistentSequentialZnode"
		Created /[PacktPub]0000000001
		[zk: localhost(CONNECTED) 3] create -s -e /[PacktPub] "EphemeralSequentialZnode"
		Created /[PacktPub]0000000008
		
WATCHES:
		Clients can register with the ZooKeeper service for any changes associated with a znode. This registration is known as setting a watch on a znode in ZooKeeper terminology.
		A watch is a one-time operation, which means that it triggers only one notification. To continue receiving notifications over time, the client must reregister the watch upon 
		receiving each event notification.
		
		Although ZooKeeper guarantees that all registered watches get dispatched to the client, even if the client disconnects from one server and reconnects to another server
		within the ZooKeeper service, there is one possible scenario worth mentioning where a watch might be missed by a client. This specific scenario is when a client has set a
		watch for the existence of a znode that has not yet been created. In this case, a watch event will be missed if the znode is created, and deleted while the client is in the
		disconnected state.
		
The ZooKeeper operations:
	Operation 	Description
	create 		Creates a znode in a specified path of the ZooKeeper namespace
	delete 		Deletes a znode from a specified path of the ZooKeeper namespace
	exists 		Checks if a znode exists in the path
	getChildren Gets a list of children of a znode
	getData 	Gets the data associated with a znode
	setData 	Sets/writes data into the data field of a znode
	getACL 		Gets the ACL of a znode
	setACL 		Sets the ACL in a znode
	sync 		Synchronizes a client's view of a znode with ZooKeeper
	
	
	1. Create a znode called root with ThisIsTheRootNode as its data:
	[zk: localhost(CONNECTED) 0] create /root "ThisIsTheRootNode"
	Created /root
	2. Get the content of the just created znode root:
	[zk: localhost(CONNECTED) 1] get /root
	"ThisIsTheRootNode"

	Understanding the Inner Workings of Apache ZooKeeper

	3. Create a child znode child-1 for root with ThisIsChild-1 as its data:
	[zk: localhost(CONNECTED) 2] create /root/child-1 	"ThisIsChild-1"
	Created /root/child-1
	4. Create a child znode child-2 for root with ThisIsChild-2 as its data:
	[zk: localhost(CONNECTED) 3] create /root/child-2 	"ThisIsChild-2"
	Created /root/child-2
	5. List the children of root:
	[zk: localhost(CONNECTED) 4] ls /root
	[child-2, child-1]
	6. Get the access control listing for root:
	[zk: localhost(CONNECTED) 5] getAcl /root
	'world,'anyone
	: cdrwa
	7. Deleting the root is not allowed as root has 2 child znodes:
	[zk: localhost(CONNECTED) 6] delete /root
	Node not empty: /root
	8. Delete child-1:
	[zk: localhost(CONNECTED) 7] delete /root/child-1
	9. Delete child-2:
	[zk: localhost(CONNECTED) 8] delete /root/child-2
	10. List the content of root:
	[zk: localhost(CONNECTED) 9] ls2 /root
	[]

ZooKeeper also supports batch updates to znodes with an operation called multi. This batches together multiple primitive operations into a single unit. A multi operation is also atomic in nature, which means
that either all the updates succeed or the whole bunch of updates fails in its entirety.

• Read requests: These are processed locally in the ZooKeeper server to which the client is currently connected
• Write requests: These are forwarded to the leader and go through majority consensus before a response is generated

Zookeeper usage: 
        Group membership
	Leader Election
	Dynamic configuration
	Status Monitoring
	Queuing
	Barriers
	Critical sections
