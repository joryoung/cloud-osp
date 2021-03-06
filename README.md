https://blog.csdn.net/weixin_45273693/article/details/100108965
https://blog.csdn.net/qq_39472671/article/details/88704137

一致性Hash..
----
https://www.jianshu.com/p/e968c081f563


junit用法，before,beforeClass,after, afterClass的执行顺序
----
每次执行test方法的时候 before都会执行一次， beforeClass只执行一次，并且方法必须为 static void
beforeclass - （before - test - after ）N次   - afterclass

分布式锁
----
在分布式环境中，不同物理机之间进程共享同一资源（数据/文件/缓存）等，要保证同一时间写入
实现方式包含  Redis（SETNX） ZK和数据库
ZK实现 :
获取ZK所有节点信息，并对节点进行排序，由于临时节点是有序的，如果获取的节点信息是最小节点信息，则说明获取到锁
如果不是，则监控比自己小1的节点信息
如果获取锁不是当前锁，则等待，节点1执行完删除，则节点2发现自己是最小的，则获取锁成功，
如果获取失败则抛出异常，如果获取成功，则进行操作，并执行unlock进行锁释放

####缺点：
1.在创建锁和释放锁的时候，会不停的创建和删除节点，性能问题
2. 在网络抖动的时候，获取锁节点断开，zk自我机制会删除节点，客户端2获取锁信息，但实际上1还在继续进行操作，造成并发问题
Curator客户端优化了重试机制，多次重试后才会删除，这种现场不常发生

####Redis实现：
SET命令实现，在redis中设置一个资源 （如果不存在，则set成功，则说明锁获取成功）
解锁则删除当前键值

缺点：主从节点问题。主节点同步到从节点挂掉了，客户端1在获取锁成功，从节点升级为主节点，2同样set成功，因为从节点信息不存在

nginx的请求转发算法，如何配置根据权重转发
----
时间轮询（默认算法）：每个请求按时间顺序分配到不同后端服务器，如果某个后端服务器宕机，能自动剔除掉

权重轮询：nginx反向代理接收到客户端收到的请求后，可以给不同的后端服务器设置一个权重值（weight），用于调整不同的服务器上请求的分配率；权重数据越大，被分配到请求的几率越大；该权重值，主要是针对实际工作环境中不同的后端服务器配置进行配置的。比如说有些服务器的硬件配置高，比重就会比较大一点

Hash轮询：
每个请求按照发起客户端ip的hash结果进行匹配，这样的算法每一个固定的ip地址的客户端总会访问到同一个后端服务器，这也在一定程度上解决了集群部署环境下session共享的问题

fair：智能调整调度算法，动态的根据后端服务器的请求处理器的请求处理响应的时间来进行均衡分配，响应时间短，处理效率高的服务器分配到请求的概率高，响应时间长，处理效率低的服务器分配到的请求少；结合了前两者的优点的一种调度算法。但是需要注意的是nginx默认不支持fair算法，如果要使用这种算法，需要安装upstream_fair模块

url_hash：按照访问的url的hash结果分配请求，每个请求的url会指向后端固定的某个服务器，可以在nginx作为静态服务器的情况下提高缓存效率。同样要注意Nginx默认不支持这种调度算法，要使用的话需要安装nginx的hash软件包

红黑树（平衡二叉B树 - B-tree）
----
特点
根节点为黑色
节点可以为红色或者黑色
红色节点的叶子为黑色
排序
查找时间复杂度
平衡二叉树区别
前序遍历（根、左、右）
中序遍历（左、根、右）
后序遍历（左、右、根）
旋转（左旋转、右旋转、双旋转（先左后右、或者先右后左））
例如：C和D,  C的高度为3（左节点2，右节点1 2+1 = 3）， D的高度为1，则对C进行旋转
双旋转（ 左边绝对值超过1，并且在右节点超长    ，右边绝对值超过1，并且在左节点超长 ）


HashMap实现缓存的问题
----
HashMap是数组+链表结构，相同Hash值（hash冲突）的数组项会形成一个链表结构，在hash数组扩容的时候，会进行rehash操作，开始移动节点，但是顺序是相反的，因此在高并发状态下会形成CPU100%的问题，死锁状态
Java1.8对HashMap做过一次改进，在扩容头端增加元素改为从末端增加元素，解决循环链表指向问题，
对于链表长度超过8的时候，形成一颗红黑树，解决链表遍历的性能问题。
但是HashMap没有解决并发问题，在多线程并发访问的时候，扩容仍然会导致数据丢失的问题


ConcurrentHashMap
----
Java1.8以前采用分段锁的方式，
这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。有些方法需要跨段，比如size()和containsValue()，它们可能需要锁定整个表而而不仅仅是某个段，这需要按顺序锁定所有段，操作完毕后，又按顺序释放所有段的锁。这里“按顺序”是很重要的，否则极有可能出现死锁，在ConcurrentHashMap内部，段数组是final的，并且其成员变量实际上也是final的，但是，仅仅是将数组声明为final的并不保证数组成员也是final的，这需要实现上的保证。这可以确保不会出现死锁，因为获得锁的顺序是固定的

线程的状态
----
New - 新建一个线程对象，JVM为每个线程创建一个私有栈，包含执行方法，比如参数，局部变量等
Runable - 调用start方法后，线程进入就绪状态，等待调度程序调度， sleep结束，join方法结束，也都会进入就绪状态，
Block - 阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。知道线程进入就绪状态，才有机会转到运行状态
1. 等待阻塞：运行的线程执行wait()方法，该线程会释放占用的所有资源，JVM会把该线程放入“等待池”中。进入这个状态后，是不能自动唤醒的，必须依靠其他线程调用notify()或notifyAll()方法才能被唤醒
2. 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入“锁池”中。
3. 其他阻塞：运行的线程执行sleep()方法或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态
Waiting - 处于这种状态的线程不会被分配CPU执行时间，它们要等待被显式地唤醒，否则会处于无限期等待的状态
TIMED_WAITING - 该状态不同于WAITING，它可以在指定的时间后自行返回
TERMINATED - 表示该线程已经执行完毕

线程阻塞的方式sleep和wait及区别
----
①sleep()释放CPU执行权，但不释放同步锁；sleep时间结束自动进入就绪状态
②wait()释放CPU执行权，也释放同步锁，使得其他线程可以使用同步控制块或者方法，必须使用notify唤醒，否则处于无限期等待

java内存模型，垃圾回收机制，可达性算法
----
JVM运行时内存包含  
①数据区（  方法区，堆） 
②指令区： 程序计数器、虚拟机栈、本地方法区

在Java中，是通过可达性分析（Reachability Analysis）来判定对象是否存活的。该算法的基本思路就是通过一些被称为引用链（GC Roots）的对象作为起点，从这些节点开始向下搜索，搜索走过的路径被称为（Reference Chain)，当一个对象到GC Roots没有任何引用链相连时（即从GC Roots节点到该节点不可达），则证明该对象是不可用的。
在java语言中，可作为GC Root 的对象包括以下几种
　1 虚拟机栈中的引用对象
　2 方法区中类静态属性引用的对象
　3 方法区中常量引用的对象
　4 本地方法占中jin引用的对象


## 分布式系统从哪些方面考虑
- 应该符合 POSIX 的文件接口标准，使该系统易于使用，同时对于用户的遗留系统也无需改造；
- 对用户透明，能够像使用本地文件系统那样直接使用
- 持久化，保证数据不会丢失；
- 具有伸缩性，当数据压力逐渐增长时能顺利扩容；
- 具有可靠的安全机制，保证数据安全；
- 数据一致性，只要文件内容不发生变化，什么时候去读，得到的内容应该都是一样的。

## threadLocal
线程的一个本地副本


## 秒杀系统
https://www.cnblogs.com/jifeng/p/5264268.html
http://www.mayiwenku.com/p-1104910.html

## 流量控制策略
接口访问流量控制
APP流量控制
用户流量控制

## QPS TPS Throught
https://www.cnblogs.com/wangmo/p/8074879.html

## 限流的两种方法
在开发高并发系统时有三把利器用来保护系统：缓存、降级和限流
* 缓存：缓存的目的是提升系统访问速度和增大系统处理容量
* 降级：降级是当服务器压力剧增的情况下，根据当前业务情况及流量对一些服务和页面有策略的降级，以此释放服务器资源以保证核心任务的正常运行
* 限流：限流的目的是通过对并发访问/请求进行限速，或者对一个时间窗口内的请求进行限速来保护系统，一旦达到限制速率则可以拒绝服务、排队或等待、降级等处理
- 漏桶算法
	漏桶算法思路很简单，水（请求）先进入到漏桶里，漏桶以一定的速度出水，当水流入速度过大会直接溢出，可以看出漏桶算法能强行限制数据的传输速率。
	漏桶的出水速度是恒定的，那么意味着如果瞬时大流量的话，将有大部分请求被丢弃掉（也就是所谓的溢出）。
- 令牌桶算法
	对于很多应用场景来说，除了要求能够限制数据的平均传输速率外，还要求允许某种程度的突发传输。这时候漏桶算法可能就不合适了，令牌桶算法更为适合。如图2所示，令牌桶算法的原理是系统会以一个恒定的速度往桶里放入令牌，而如果请求需要被处理，则需要先从桶里获取一个令牌，当桶里没有令牌可取时，则拒绝服务。
	生成令牌的速度是恒定的，而请求去拿令牌是没有速度限制的。这意味，面对瞬时大流量，该算法可以在短时间内请求拿到大量令牌，而且拿令牌的过程并不是消耗很大的事情。



## 锁的升级（偏向锁-轻量级锁-重量级锁）
偏向锁：对象的代码一直被同一线程执行，不存在多个线程竞争，该线程在后续的执行中自动获取锁，降低获取锁带来的性能开销。偏向锁，指的就是偏向第一个加锁线程，该线程是不会主动释放偏向锁的，只有当其他线程尝试竞争偏向锁才会被释放。
偏向锁的撤销，需要在某个时间点上没有字节码正在执行时，先暂停拥有偏向锁的线程，然后判断锁对象是否处于被锁定状态。如果线程不处于活动状态，则将对象头设置成无锁状态，并撤销偏向锁；
如果线程处于活动状态，升级为轻量级锁的状态。
 
## 轻量级锁：轻量级锁是指当锁是偏向锁的时候，被第二个线程 B 所访问，此时偏向锁就会升级为轻量级锁，线程 B 会通过自旋的形式尝试获取锁，线程不会阻塞，从而提高性能。
当前只有一个等待线程，则该线程将通过自旋进行等待。但是当自旋超过一定的次数时，轻量级锁便会升级为重量级锁；当一个线程已持有锁，另一个线程在自旋，而此时又有第三个线程来访时，轻量级锁也会升级为重量级锁。
 
## 重量级锁：指当有一个线程获取锁之后，其余所有等待获取该锁的线程都会处于阻塞状态。
重量级锁通过对象内部的监视器（monitor）实现，而其中 monitor 的本质是依赖于底层操作系统的 Mutex Lock 实现，操作系统实现线程之间的切换需要从用户态切换到内核态，切换成本非常高

## Mysql 三种数据库引擎
InnoDB 、ISAM、MyISAM

#### ISAM
数据库被查询的次数要远大于更新的次数

#### InnoDB:
支持事务处理，支持外键、MVCC，聚集索引（表必须有索引）

选择 ： 频繁的更新、删除操作的数据库

#### MVCC （multiple-version-concurrency-control）多版本并发控制
#### 聚集索引：
以B+树的方式排列, 子节点存放数据，如果没有主键，以第一列为主键
只有一个聚集索引
普通索引指向聚集索引

## 如何确定线程数
1、针对IO密集型的，阻塞耗时w一般都是计算耗时几倍c，假设阻塞耗时=计算耗时的情况下，Nthreads=Ncpu*(1+1)=2Ncpu,所以这种情况下，建议考虑2倍的CPU核心数做为线程数
2、对于计算密集型的，阻塞耗时趋于0，即w/c趋于0，公式Nthreads = Ncpu。

1.获取CPU核数
CPU核数 = Runtime.getRuntime().availableProcessors()
2.分析下线程池处理的程序是CPU密集型，还是IO密集型
CPU密集型：核心线程数 = CPU核数 + 1
IO密集型：核心线程数 = CPU核数 * 2
* CPU密集型（CPU-bound）
CPU密集型也叫计算密集型，指的是系统的硬盘、内存性能相对CPU要好很多，此时，系统运作大部分的状况是CPU Loading 100%，CPU要读/写I/O(硬盘/内存)，I/O在很短的时间就可以完成，而CPU还有许多运算要处理，CPU Loading很高。
在多重程序系统中，大部份时间用来做计算、逻辑判断等CPU动作的程序称之CPU bound。例如一个计算圆周率至小数点一千位以下的程序，在执行的过程当中绝大部份时间用在三角函数和开根号的计算，便是属于CPU bound的程序。
CPU bound的程序一般而言CPU占用率相当高。这可能是因为任务本身不太需要访问I/O设备，也可能是因为程序是多线程实现因此屏蔽掉了等待I/O的时间。
* IO密集型（I/O bound）
IO密集型指的是系统的CPU性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是CPU在等I/O (硬盘/内存) 的读/写操作，此时CPU Loading并不高。
I/O bound的程序一般在达到性能极限时，CPU占用率仍然较低。这可能是因为任务本身需要大量I/O操作，而pipeline做得不是很好，没有充分利用处理器能力。

## redis的五种数据结构原理分析
* String  - 对应的redis命令  get set， 二进制安全
优点 ： 
String，redis对于KV的操作效率很高，可以直接用作计数器。例如，统计在线人数等等，另外string类型是二进制存储安全的，所以也可以使用它来存储图片，甚至是视频等

* list - 对应的redis命令  lput  rput  lget等
优点：
列表类型，可以用于实现消息队列，也可以使用它提供的range命令，做分页查询功能
* set - 对应redis命令 sadd smembers 等
优点：
集合，整数的有序列表可以直接使用set。可以用作某些去重功能，例如用户名不能重复等，另外，还可以对集合进行交集，并集操作，来查找某些元素的共同点

* hash
存放键值对，一般可以用来存某个对象的基本属性信息，例如，用户信息，商品信息等，另外，由于hash的大小在小于配置的大小的时候使用的是ziplist结构，比较节约内存，所以针对大量的数据存储可以考虑使用hash来分段存储来达到压缩数据量，节约内存的目的，例如，对于大批量的商品对应的图片地址名称

* zset(有序集合)
有序集合，可以使用范围查找，排行榜功能或者topN功能。

## Redis跳跃表原理
相当于数据结构中的平衡树，但是代码比平衡树简单，时间复杂度有为 O（logn），最大的时间复杂度为 O（n）
跳跃表层数越多，那么查找的时间越快
主要是以分值，当前level插入值，需要判断是否该node是否在上层存在，如果不存在，则所有上层都需要插入，分值以有序的方式排序
查找的时候，最leve最低的层级开始查找（最低level在最上层），也就是从顶层开始查找，到达空的时候，转到下一个层级 level（n-1）


## Redis的两种持久化方式RDB/AOF对比




## https的工作原理
首先看看组成HTTPS的协议：HTTP协议和SSL/TLS协议。HTTP协议就不用讲了，而SSL/TLS就是负责加密解密等安全处理的模块，所以HTTPS的核心在SSL/TLS上面。整个通信如下：
1、浏览器发起往服务器的443端口发起请求，请求携带了浏览器支持的加密算法和哈希算法。
2、服务器收到请求，选择浏览器支持的加密算法和哈希算法。
3、服务器下将数字证书返回给浏览器，这里的数字证书可以是向某个可靠机构申请的，也可以是自制的。
4、浏览器进入数字证书认证环节，这一部分是浏览器内置的TLS完成的：
4.1 首先浏览器会从内置的证书列表中索引，找到服务器下发证书对应的机构，如果没有找到，此时就会提示用户该证书是不是由权威机构颁发，是不可信任的。如果查到了对应的机构，则取出该机构颁发的公钥。
4.2 用机构的证书公钥解密得到证书的内容和证书签名，内容包括网站的网址、网站的公钥、证书的有效期等。浏览器会先验证证书签名的合法性（验证过程类似上面Bob和Susan的通信）。签名通过后，浏览器验证证书记录的网址是否和当前网址是一致的，不一致会提示用户。如果网址一致会检查证书有效期，证书过期了也会提示用户。这些都通过认证时，浏览器就可以安全使用证书中的网站公钥了。
4.3 浏览器生成一个随机数R，并使用网站公钥对R进行加密。
5、浏览器将加密的R传送给服务器。
6、服务器用自己的私钥解密得到R。
7、服务器以R为密钥使用了对称加密算法加密网页内容并传输给浏览器。
8、浏览器以R为密钥使用之前约定好的解密算法获取网页内容

 ## java事件机制
 java事件机制包括三个部分：事件、事件监听器、事件源
 1、事件。一般继承自java.util.EventObject类，封装了事件源对象及跟事件相关的信息
 2、事件监听器。实现java.util.EventListener接口,注册在事件源上,当事件源的属性或状态改变时,取得相应的监听器调用其内部的回调方法
 3、事件源。事件发生的地方，由于事件源的某项属性或状态发生了改变(比如BUTTON被单击、TEXTBOX的值发生改变等等)导致某项事件发生。换句话说就是生成了相应的事件对象。因为事件监听器要注册在事件源上,所以事件源类中应该要有盛装监听器的容器(List,Set等等)。
 
 
## 什么是索引为啥nosql没索引？nosql有索引滴
索引分为聚簇索引和非聚簇索引两种，聚簇索引是按照数据存放的物理位置为顺序的，而非聚簇索引就不一样了；聚簇索引能提高多行检索的速度，而非聚簇索引对于单行的检索很快。
聚簇索引：有主键时，根据主键创建聚簇索引；没有主键时，会用一个唯一且不为空的索引列做为主键，成为此表的聚簇索引；如果以上两个都不满足那innodb自己创建一个虚拟的聚集索引
非聚簇索引：非聚簇索引都是辅助索引，像复合索引、前缀索引、唯一索引

## B+树和B树区别？
B树的非叶子节点存储实际记录的指针，而B+树的叶子节点存储实际记录的指针
B+树的叶子节点通过指针连起来了, 适合扫描区间和顺序查找。

## JAVA的 classloader原理和内存模型


## 双亲委派模型


## 解决Hash冲突的三种方式
拉链法：
每个进入hashmap的hash值都会生成一个单向链表，一旦发现有重复的时候，这个hash值会进入链表（长度增加1），有序集合
会按照大小排列，并移动指针

开发地址法


Rehash
重新hash，知道不冲突为止


## 高性能I/O设计模式  Reactor 反应堆模式和Proactor 前摄器模式

## 五种IO模型
同步阻塞IO：
   在此种方式下，用户进程在发起一个IO操作以后，必须等待IO操作的完成，只有当真正完成了IO操作以后，用户进程才能运行。JAVA传统的IO模型属于此种方式！
   同步非阻塞IO:
在此种方式下，用户进程发起一个IO操作以后边可返回做其它事情，但是用户进程需要时不时的询问IO操作是否就绪，这就要求用户进程不停的去询问，从而引入不必要的CPU资源浪费。其中目前JAVA的NIO就属于同步非阻塞IO。
   异步阻塞IO：
   此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成IO操作以后会通知应用程序，这其实就是同步和异步最关键的区别，同步必须等待或者主动的去询问IO是否完成，那么为什么说是阻塞的呢？因为此时是通过select系统调用来完成的，而select函数本身的实现方式是阻塞的，而采用select函数有个好处就是它可以同时监听多个文件句柄，从而提高系统的并发性！
   异步非阻塞IO:
   在此种模式下，用户进程只需要发起一个IO操作然后立即返回，等IO操作真正的完成以后，应用程序会得到IO操作完成的通知，此时用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由内核完成了。目前Java中还没有支持此种IO模型



## TCP三次握手和4次挥手过程
#### 握手
1.客户端发送请求给服务端，想要创建连接
2.服务端收到请求连接报文，并回复给客户端
3.TCP客户进程收到确认后，还要向服务器给出确认，建立连接

#### 两次握手的弊端
lient发送了第一个连接的请求报文，但是由于网络不好，这个请求没有立即到达服务端，而是在某个网络节点中滞留了，直到某个时间才到达server，本来这已经是一个失效的报文，但是server端接收到这个请求报文后，还是会想client发出确认的报文，表示同意连接。假如不采用三次握手，那么只要server发出确认，新的建立就连接了，但其实这个请求是失效的请求，client是不会理睬server的确认信息，也不会向服务端发送确认的请求，但是server认为新的连接已经建立起来了，并一直等待client发来数据，这样，server的很多资源就没白白浪费掉了，采用三次握手就是为了防止这种情况的发生，server会因为收不到确认的报文，就知道client并没有建立连接。这就是三次握手的作用

#### 挥手
1. TCP发送一个FIN(结束)，用来关闭客户到服务端的连接
2. 服务端收到这个FIN，他发回一个ACK(确认)，服务端就进入了CLOSE-WAIT（关闭等待）状态。TCP服务器通知高层的应用进程，客户端向服务器的方向就释放了，这时候处于半关闭状态，即客户端已经没有数据要发送了，但是服务器若发送数据，客户端依然要接受。这个状态还要持续一段时间，也就是整个CLOSE-WAIT状态持续的时间
	客户端收到服务器的确认请求后，此时，客户端就进入FIN-WAIT-2（终止等待2）状态，等待服务器发送连接释放报文
3.服务端发送一个FIN(结束)到客户端，服务端关闭客户端的连接，服务器就进入了LAST-ACK（最后确认）状态，等待客户端的确认，客户端发送ACK(确认)报文确认，并将确认的序号+1，这样关闭完成。
4.客户端发送ACK(确认)报文确认，并将确认的序号+1，这样关闭完成。客户端收到服务器的连接释放报文后，必须发出确认，此时，客户端就进入了TIME-WAIT（时间等待）状态
#### 四次挥手的原因
关闭连接时，当收到对方的FIN报文通知时，它仅仅表示对方没有数据发送给你了；但未必你所有的数据都全部发送给对方了，所以你可以未必会马上会关闭SOCKET,也
即你可能还需要发送一些数据给对方之后，再发送FIN报文给对方来表示你同意现在可以关闭连接了，所以它这里的ACK报文和FIN报文多数情况下都是分开发送的。
可能有人会有疑问，tcp我握手的时候为何ACK(确认)和SYN(建立连接)是一起发送。挥手的时候为什么是分开的时候发送呢.
因为当Server端收到Client端的SYN连接请求报文后，可以直接发送SYN+ACK报文。其中ACK报文是用来应答的，SYN报文是用来同步的。但是关闭连接时，当Server端收到
FIN报文时，很可能并不会立即关闭 SOCKET，所以只能先回复一个ACK报文，告诉Client端，"你发的FIN报文我收到了"。只有等到我Server端所有的报文都发送完了，我才能
发送FIN报文，因此不能一起发送。故需要四步挥手

## Java 异常结构
Throwable - 
Error 
Exception
 - IOException
 - RuntimeException
 - SQLException
 - 用户自定义异常
 
 不检查异常：编译的时候不会检查 RuntimeException
 检查异常：IO SQL 自定义都会检查



## 基于Nginx+Redis+jvm堆缓存的多级缓存架构设计
https://www.cnblogs.com/z-3FENG/p/9603321.html

## Redis分布式集群倾斜问题
http://www.dataguru.cn/article-14564-1.html


## 服务熔断与降级（Hystrix）
https://blog.csdn.net/pengjunlee/article/details/86688858

## sentinel和hystrix的区别？我知道拼多多这二个框架都有使用，限流的一些参数怎么设置，依据是什么


## 订单表拆表的优化逻辑
https://blog.csdn.net/FansUnion/article/details/79621049
https://www.jianshu.com/p/ac61d9427d6f
拆表  订单的查询   


## sentinel和hystrix的区别？我知道拼多多这二个框架都有使用，限流的一些参数怎么设置，依据是什么表的

## 分布式数据库中间件—TDDL的使用介绍
 https://www.2cto.com/database/201806/752199.html
 
 ## 分布式事务
 https://blog.csdn.net/qq_27384769/article/details/79305402
 
## 大型电商系统的高并发的优化经验
 https://blog.csdn.net/weixin_34032827/article/details/92421607
 参考一些其他的
 
 DDD
 
 ##Redis的缓存淘汰策略LRU与LFU
 Redis只有在内存满的时候会执行缓存淘汰，可能会造成缓存雪崩的情况，需要考虑具体业态中周期性的缓存读取的情况
 LRU简单的讲就是以缓存创建的当前系统的时间戳为基准，时间越长的缓存优先被淘汰，缺点：忽略了在缓存中的使用频率
 LFU对使用频率进行了优化，如果使用频率比较低，那么衰减值增加，一段时间后，值时间创建最长+频率使用最低的被淘汰
 
 二级缓存机制及数据一致性问题
 堆内存（Ehcache） + 数据库缓存（Redis）


跨域访问：
Access-Control-Allow-Origin：*  允许所有域名的脚本访问该资源  
Access-Control-Allow-Credentials：true   是否允许后续请求携带认证信息（cookies）,该值只能是true,否则不返回

检查该方法A上是否有B注解A
AnnotationUtils.findAnnotation（A,B）
A: Method
B: @Nullable Annotation


Guava工具类
https://github.com/google/guava/wiki



 













<html xmlns:v="urn:schemas-microsoft-com:vml" xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word" xmlns:m="http://schemas.microsoft.com/office/2004/12/omml" xmlns="http://www.w3.org/TR/REC-html40"><head><meta http-equiv=Content-Type content="text/html; charset=utf-8"><meta name=Generator content="Microsoft Word 15 (filtered medium)"><style><!--
/* Font Definitions */
@font-face
	{font-family:Wingdings;
	panose-1:5 0 0 0 0 0 0 0 0 0;}
@font-face
	{font-family:SimSun;
	panose-1:2 1 6 0 3 1 1 1 1 1;}
@font-face
	{font-family:"Cambria Math";
	panose-1:2 4 5 3 5 4 6 3 2 4;}
@font-face
	{font-family:DengXian;
	panose-1:2 1 6 0 3 1 1 1 1 1;}
@font-face
	{font-family:Calibri;
	panose-1:2 15 5 2 2 2 4 3 2 4;}
@font-face
	{font-family:"\@DengXian";
	panose-1:2 1 6 0 3 1 1 1 1 1;}
@font-face
	{font-family:"\@SimSun";
	panose-1:2 1 6 0 3 1 1 1 1 1;}
/* Style Definitions */
p.MsoNormal, li.MsoNormal, div.MsoNormal
	{margin:0in;
	margin-bottom:.0001pt;
	font-size:11.0pt;
	font-family:"Calibri",sans-serif;}
a:link, span.MsoHyperlink
	{mso-style-priority:99;
	color:#6D6E71;
	text-decoration:underline;}
p.msipheader194d5149, li.msipheader194d5149, div.msipheader194d5149
	{mso-style-name:msipheader194d5149;
	mso-margin-top-alt:auto;
	margin-right:0in;
	mso-margin-bottom-alt:auto;
	margin-left:0in;
	font-size:11.0pt;
	font-family:"Calibri",sans-serif;}
span.EmailStyle24
	{mso-style-type:personal-reply;
	font-family:"Arial",sans-serif;
	color:windowtext;
	font-weight:normal;
	font-style:normal;}
.MsoChpDefault
	{mso-style-type:export-only;
	font-size:10.0pt;}
@page WordSection1
	{size:8.5in 11.0in;
	margin:1.0in 1.0in 1.0in 1.0in;}
div.WordSection1
	{page:WordSection1;}
/* List Definitions */
@list l0
	{mso-list-id:1492872565;
	mso-list-type:hybrid;
	mso-list-template-ids:1342062680 -1723419700 134807555 134807557 134807553 134807555 134807557 134807553 134807555 134807557;}
@list l0:level1
	{mso-level-start-at:0;
	mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;
	mso-fareast-font-family:DengXian;
	mso-bidi-font-family:Arial;}
@list l0:level2
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l0:level3
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l0:level4
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l0:level5
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l0:level6
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l0:level7
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l0:level8
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l0:level9
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l1
	{mso-list-id:1706826864;
	mso-list-type:hybrid;
	mso-list-template-ids:-4569534 727984062 134807555 134807557 134807553 134807555 134807557 134807553 134807555 134807557;}
@list l1:level1
	{mso-level-start-at:0;
	mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;
	mso-fareast-font-family:DengXian;
	mso-bidi-font-family:Arial;}
@list l1:level2
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l1:level3
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l1:level4
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l1:level5
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l1:level6
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l1:level7
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l1:level8
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l1:level9
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l2
	{mso-list-id:2082167144;
	mso-list-type:hybrid;
	mso-list-template-ids:-1100319764 -1989772540 134807555 134807557 134807553 134807555 134807557 134807553 134807555 134807557;}
@list l2:level1
	{mso-level-start-at:0;
	mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	margin-left:38.25pt;
	text-indent:-20.25pt;
	font-family:Wingdings;
	mso-fareast-font-family:DengXian;
	mso-bidi-font-family:Arial;}
@list l2:level2
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l2:level3
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l2:level4
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l2:level5
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l2:level6
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
@list l2:level7
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Symbol;}
@list l2:level8
	{mso-level-number-format:bullet;
	mso-level-text:o;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:"Courier New";}
@list l2:level9
	{mso-level-number-format:bullet;
	mso-level-text:;
	mso-level-tab-stop:none;
	mso-level-number-position:left;
	text-indent:-.25in;
	font-family:Wingdings;}
ol
	{margin-bottom:0in;}
ul
	{margin-bottom:0in;}
--></style><!--[if gte mso 9]><xml>
<o:shapedefaults v:ext="edit" spidmax="1026" />
</xml><![endif]--><!--[if gte mso 9]><xml>
<o:shapelayout v:ext="edit">
<o:idmap v:ext="edit" data="1" />
</o:shapelayout></xml><![endif]--></head><body lang=EN-GB link="#6D6E71" vlink="#2890C0"><div class=WordSection1><p class=msipheader194d5149 style='margin:0in;margin-bottom:.0001pt'><span style='font-size:9.0pt;font-family:"Arial",sans-serif;color:#A80000'>CONFIDENTIAL</span><o:p></o:p></p><p class=MsoNormal><span style='font-size:12.0pt;font-family:SimSun'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><a name="_GoBack"></a><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>Approved <o:p></o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><div><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>Thanks and regards<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>______________________________________________________________<o:p></o:p></span></p><p class=MsoNormal><b><span style='font-size:10.0pt;font-family:"Arial",sans-serif;color:#0070C0;letter-spacing:.25pt'>Liam Li<o:p></o:p></span></b></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black'>Senior Development Manager</span><span style='font-size:8.0pt;color:black'><o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>CIBFM-TradePostTrade-TJ-Prj<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>Standard Chartered Global Business Services Co., Ltd.<o:p></o:p></span></p><p class=MsoNormal><i><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>(formerly known as Scope International (China) Co., Ltd.)<o:p></o:p></span></i></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>Phone:&nbsp;&nbsp; 02258573165<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>Address: Standard Chartered Center,&nbsp; 2/F No.35 Xinhuanbei Road, <o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TEDA,&nbsp; Tianjin - 300457<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>E-mail:&nbsp;&nbsp; <a href="mailto:Liam.Li@sc.com">Liam.Li@sc.com</a><o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>______________________________________________________________<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:8.0pt;font-family:"Arial",sans-serif;color:black;letter-spacing:.25pt'>Please consider the environment before printing this email<o:p></o:p></span></p></div><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><div><div style='border:none;border-top:solid #E1E1E1 1.0pt;padding:3.0pt 0in 0in 0in'><p class=MsoNormal><b><span lang=EN-US>From:</span></b><span lang=EN-US> Yang3, Chen &lt;<a href="mailto:Chen.Yang3@sc.com">Chen.Yang3@sc.com</a>&gt; <br><b>Sent:</b> Monday, July 06, 2021 15:38 PM<br><b>To:</b> Li, Liam &lt;<a href="mailto:Liam.Li@sc.com">Liam.Li@sc.com</a>&gt;<br><b>Subject:</b> Please Approve My Timesheet For June 2021<o:p></o:p></span></p></div></div><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=msipheader194d5149 style='margin:0in;margin-bottom:.0001pt'><span style='font-size:9.0pt;font-family:"Arial",sans-serif;color:#A80000'>CONFIDENTIAL</span><o:p></o:p></p><p class=MsoNormal><o:p>&nbsp;</o:p></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>Hi&nbsp; Liam<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>Please approve my timesheet for June 2021&nbsp;</span><span lang=EN-US style='font-size:10.0pt;font-family:"Arial",sans-serif'>,</span><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>&nbsp;&nbsp;approval mail please cc to </span><span style='font-size:14.0pt;font-family:"Arial",sans-serif;color:#2352FF'>:&nbsp; &nbsp;&nbsp;&nbsp;<b><a href="mailto:cl.tj.timesheet@clpsglobal.com"><span style='color:#2352FF'>cl.tj.timesheet@clpsglobal.com</span></a></b></span><b><span style='font-size:10.0pt;font-family:"Arial",sans-serif'> &nbsp;&nbsp;&nbsp;involve attachment <o:p></o:p></span></b></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><table class=MsoNormalTable border=0 cellspacing=0 cellpadding=0 width=756 style='width:567.05pt;margin-left:.1pt;border-collapse:collapse'><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Name<o:p></o:p></span></b></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal><span lang=ZH-CN style='font-size:10.5pt;font-family:SimSun'>杨陈</span><span style='font-size:10.5pt'><o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;border:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>PWID<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal><span style='font-size:10.5pt'>&nbsp;<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:15.75pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Company<o:p></o:p></span></b></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>ChinaLink<o:p></o:p></span></b></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;border:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Month<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>24-Jun<o:p></o:p></span></b></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:14.25pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td></tr><tr style='height:14.25pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td></tr><tr style='height:15.75pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.75pt'></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Date<o:p></o:p></span></b></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Hrs<o:p></o:p></span></b></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Date<o:p></o:p></span></b></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Hrs<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Date<o:p></o:p></span></b></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border:solid windowtext 1.0pt;border-left:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><b><span style='color:black'>Hrs<o:p></o:p></span></b></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>1<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>11<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>21<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'>8<span style='color:black'><o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>2<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>12<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>SA<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>22<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>3<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>13<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>SU<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>23<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>4<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>14<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>PH<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>24<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>5<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>SA<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>15<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>25<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>6<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>SU<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>16<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>26<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'>SA<span style='color:black'><o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>7<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>17<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>27<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>S</span>U<span style='color:black'><o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>18<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>28<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'>8<span style='color:black'><o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>9<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>19<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'>SA<span style='color:black'><o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>29<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>10<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>20<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>S</span>U<span style='color:black'><o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>30<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>8<o:p></o:p></span></p></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>&nbsp;<o:p></o:p></span></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>&nbsp;<o:p></o:p></span></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>&nbsp;<o:p></o:p></span></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=center style='text-align:center'><span style='color:black'>&nbsp;<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;border:solid windowtext 1.0pt;border-top:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:16.5pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;border-top:none;border-left:solid windowtext 1.0pt;border-bottom:solid windowtext 1.0pt;border-right:none;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Total Working Day:<o:p></o:p></span></b></p></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal align=right style='text-align:right'><b><span style='color:black'>2</span>1<span style='color:black'><o:p></o:p></span></b></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>&nbsp;<o:p></o:p></span></b></p></td><td width=145 nowrap valign=bottom style='width:109.0pt;border-top:none;border-left:none;border-bottom:solid windowtext 1.0pt;border-right:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>&nbsp;<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:16.5pt'><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:15.6pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'><p class=MsoNormal><b><u><span style='color:black'>Total over time hour<o:p></o:p></span></u></b></p></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=60 nowrap valign=bottom style='width:45.35pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=104 nowrap style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=145 nowrap style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=113 nowrap style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td><td width=71 nowrap style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:15.6pt'></td></tr><tr style='height:30.0pt'><td width=323 nowrap colspan=5 valign=bottom style='width:242.05pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>SA = Satudary<o:p></o:p></span></p></td><td width=249 nowrap colspan=2 valign=bottom style='width:187.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>SU = Sunday<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>AL=Annual Leave<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'></td></tr><tr style='height:30.0pt'><td width=323 nowrap colspan=5 valign=bottom style='width:242.05pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>ML=Medical Leave<o:p></o:p></span></p></td><td width=249 nowrap colspan=2 valign=bottom style='width:187.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>PH=Public Holiday<o:p></o:p></span></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'><p class=MsoNormal style='text-align:justify'><span style='color:black'>NJ= Not Joined<o:p></o:p></span></p></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:30.0pt'></td></tr><tr style='height:14.25pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td></tr><tr style='height:14.25pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td></tr><tr style='height:16.5pt'><td width=572 nowrap colspan=7 valign=bottom style='width:429.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Requestor Signature: Yang Chen </span></b><b><span lang=ZH-CN style='font-family:SimSun;color:black'>杨陈</span><span style='color:black'><o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:16.5pt'><td width=323 nowrap colspan=5 valign=bottom style='width:242.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Date: 24-Jun-2021<o:p></o:p></span></b></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:14.25pt'><td width=141 nowrap colspan=2 valign=bottom style='width:106.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=181 nowrap colspan=3 valign=bottom style='width:136.05pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:14.25pt'></td></tr><tr style='height:16.5pt'><td width=572 nowrap colspan=7 valign=bottom style='width:429.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Line Manager Name:<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:16.5pt'><td width=572 nowrap colspan=7 valign=bottom style='width:429.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Signature<o:p></o:p></span></b></p></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr style='height:16.5pt'><td width=323 nowrap colspan=5 valign=bottom style='width:242.05pt;border:none;border-bottom:solid windowtext 1.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'><p class=MsoNormal style='text-align:justify'><b><span style='color:black'>Date: <o:p></o:p></span></b></p></td><td width=104 nowrap valign=bottom style='width:78.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=145 nowrap valign=bottom style='width:109.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=113 nowrap valign=bottom style='width:85.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td><td width=71 nowrap valign=bottom style='width:53.0pt;padding:0in 5.4pt 0in 5.4pt;height:16.5pt'></td></tr><tr><td width=71 style='width:53.25pt;padding:0in 0in 0in 0in'></td><td width=71 style='width:53.25pt;padding:0in 0in 0in 0in'></td><td width=60 style='width:45.0pt;padding:0in 0in 0in 0in'></td><td width=60 style='width:45.0pt;padding:0in 0in 0in 0in'></td><td width=60 style='width:45.0pt;padding:0in 0in 0in 0in'></td><td width=104 style='width:78.0pt;padding:0in 0in 0in 0in'></td><td width=145 style='width:108.75pt;padding:0in 0in 0in 0in'></td><td width=113 style='width:84.75pt;padding:0in 0in 0in 0in'></td><td width=71 style='width:53.25pt;padding:0in 0in 0in 0in'></td></tr></table><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>Thanks and regards<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'>Yang Chen<o:p></o:p></span></p><p class=MsoNormal><span style='font-size:10.0pt;font-family:"Arial",sans-serif'><o:p>&nbsp;</o:p></span></p></div></body></html>
