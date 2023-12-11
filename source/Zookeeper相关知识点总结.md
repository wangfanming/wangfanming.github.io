---
title: Zookeeper相关知识点总结 # 标题
date: 2023-12-12 00:44:21 # 时间
categories: # 分类
- 大数据
tags: # 标签
- 分布式
---

# Zookeeper

#### Zookeeper概述
&emsp;&emsp;Zookeeper是一个**分布式协调服务**的开源框架。主要用来解决分布式集群中应用系统
的一致性问题，例如怎样避免同时操作同一数据造成脏读的问题。

&emsp;&emsp;Zookeeper本质上是一个分布式的小文件系统。提供基于类似文件系统的**目录树**方式的数据
存储，并且可以对树中的节点进行有效管理。从而用来维护和监控**储存的数据的状态的变化**。通过监控
这些数据状态的变化，从而可以达到基于数据的集群管理。提供诸如：统一命名服务、分布式配置管理、分布式消息队列、
分布式锁、分布式协调等功能。

**Zookeeper集群角色**
+ Leader ： Zookeeper集群工作的核心，是集群的事务(写操作)请求的唯一调度和处理者，保证集群事务处理的顺序性。
    同时也是集群内部各个服务的调度者。

+ Follower：处理客户端的**非事务**请求，转发事务请求给Leader，参与集群Leader选举投票。

+ Observer：充当观察者角色，观察Zookeeper集群的最新状态变化并将这些状态同步过来，其对于非事务请求可以进行独立
处理，对于事务请求，则会转发给Leader，不会参与任何形式的投票，通常用于在不影响集群事务处理能力的前提下
提升集群的非事务处理能力。

**Zookeeper特性**
+ 全局一致性：每个server保存一份相同的数据副本，client无论连接到哪个server，展示的数据都是一致的。
+ 可靠性：如果消息被其中一台服务器接收，那么将被所有的服务器接收。
+ 顺序性：包括全局有序和偏序两种。
+ 数据更新原子性：一次数据更新要么成功，要么失败，不存在中间状态。
+ 实时性：Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效信息。

>**全局有序**是指如果在一台服务器上消息 a 在消息 b 前发布，则在所有 Server 上消息 a
都将在消息 b 前被发布。

>**偏序**是指如果一个消息 b 在消息 a 后被同一个发送者发布， a 必
 将排在 b 前面。

---

#### Zookeeper shell

&emsp;&emsp;运行`zkCli.sh -server ip` 进入命令行工具。

&emsp;&emsp;输入`help`,输出如下提示：

![](https://github.com/wangfanming/blog/blob/467eedb713de54c813589293a487504a59304a36/image/zookeeper_shell_help.bmp)

带有`[watch]`的表示
1、shell基本操作

**创建节点**：

`create [-s] [-e] path data ac`

其中，`-s`或`-e`分别指定节点特性，顺序或临时节点，若不指定，则表示创建为持久节点，`acl`用来进行权限控制。

**读取节点**：

`ls path [watch]`

`get path [watch]`

`ls2 path [watch]`

&emsp;&emsp;与读相关的命令有`ls`命令和`get`命令，`ls`命令用来列出Zookeeper指定节点下的所有子节点，只能查看
指定节点下的**第一级**的所有子节点；`get`命令可以获取Zookeeper指定节点的数据内容和属性信息。

**更新节点**

`set path [watch]`

`data`就是要更新的内容，version表示数据版本。

**删除节点**

`delete path [watch]`

若删除的节点存在子节点，那么无法删除节点，必须先删除子节点，再删除父节点。
也可以使用`rmr path`,递归强制删除。

**对节点添加限制(弱限制)**

`setquota -|-b val path` 对节点增加限制。
- `n`:表示子节点的最大个数。
- `b`:表示数据值的最大长度。
- `val`:子节点最大格式或数据值的最大长度。
- `path`:节点路径。

**其他命令**

`history`:列出命令历史
`redo`:改命令可以重新执行指定命令Bain好的历史命令，通常结合`history`一起使用。

---

#### Zookeeper数据模型

&emsp;&emsp;Zookeeper采用**树形层次结构**，每个节点被称为`Znode`。

1、`Znode`节点的特性：

+ Znode兼具文件和目录两种特点。既像文件一样维护着数据、元信息、ACL(访问权限列表)、时间戳等
    数据结构，又像目录一样可以作为路径标识的一部分，并可以具有子`Znode`。
+ Znode具有原子性操作。
+ Znode存储数据大小有限制。通常以`KB`为单位。
+ Znode通过路径引用。**路径必须是绝对的**。

每个Znode节点都是由3部分组成：
+ `stat`:此为状态信息，描述Znode的版本、权限等信息。
+ `data`:与该Znode关联的数据。
+ `children`:该Znode下的子节点。

2、节点类型

&emsp;&emsp;Znode有两种，分别是**临时节点**和**永久节点**。节点类型在创建时即被确定，且不能改变。

**临时节点**：该节点的生命周期依赖于创建它们的会话。会话一旦结束，临时节点就会被自动删除。临时节点不允许有子节点。
**永久节点**：该节点的生命周期不依赖会话，并且只有在客户端显式删除时，它们才会被删除。

Znode还有一个序列化的特性，如果创建的时候指定的话，该Znode的名字后面会自动追加一个不断增加的序列号。
用来记录每个子节点创建的先后顺序。格式：`%10d`(10位数字，不够用0补充)

这样就存在了四种类型的`Znode`节点：
+ `PERSISTENT`:永久节点
+ `EPHEMERAL`:临时节点
+ `PERSISTENT_SEQUENTIAL`:永久、序列化节点
+ `EPHEMERAL_SEQUENTIAL`:临时、序列化节点

3、节点属性

&emsp;&emsp;每个Znode都包含了一系列的属性。
- `dataVersion`:数据版本号。每次对数据的`set`操作都会使其自加1，可以有效避免数据更新时的先后顺序问题。
- `cversion`:子节点的版本号。当`Znode`的子节点有变化时，其值都会加1。
- `aclVersion`:ACL的版本号。
- `cZid`:创建Znode的事务的ID。
- `mZid`:Znode被修改的事务ID。
- `ctime`:节点创建时的时间戳。
- `mtime`:节点最新一次更新发生的时间。
- `ephemeralOwner`:如果该节点为临时节点，其值表示与该节点绑定的session id，如果不是则为0。

在client和server通信之前，首先需要建立连接，该连接称为session。连接建立后如果发生超时、授权失败，或显式的关闭连接，
连接便处于CLOSED状态，此时session结束。

4、Zookeeper的Watch机制

&emsp;&emsp;Watch说的是Zookeeper的监听机制。**一个Watch事件是一个一次性的触发器**，当被设置了Watch
的数据发生了改变的时候，则服务器将这一改变打送给设置了Watch的客户端，以便通知它们。

&emsp;&emsp;改变有很多种方式，如：节点创建、节点删除、节点改变、子节点改变等。

**Watch机制的特点**

+ 一次性触发。
+ Watcher event异步发送。即watcher的通知事件从server到client是异步的。
+ 数据监视。`getdata()`、`exists()`设置了数据监视。`getchildren()`设置了子节点监视。
+ 注册watcher。`getData()`,`exists()`,`getchildren()`
+ 触发watcher。`create()`,`delete()`,`setData()`

5、Watch的事件类型

Zookeeper的watch实际上要处理两类事件。
+ 连接状态事件(type=None,path=null)

`None`:在客户端与Zookeeper集群中的服务器断开连接的时候，客户端会收到这个事件。


+ 节点事件

`NodeCreated`：Znode创建事件

`NodeDeleted`：Znode删除事件

`NodeDataChanged`：Znode数据内容更新事件。其实本质上该事件只关注dataVersion版本号，但是只要
调用了更新接口，`dataVersion`就会有变更。

`NodeChildrenChanged`：Znode子节点改变事件，只关注**子节点的个数**变更，子节点内容变更是不会通知的。

---
#### Zookeeper客户端API

&emsp;&emsp;`org.apache.zookeeper.Zookeeper` 是客户端入口主类，负责建立与 server
的会话， 它供以下几类主要方法：
+ `create`:在本地目录树中创建一个节点。
+ `delete`:删除一个节点。
+ `exists`:测试本地是否存在目标节点。
+ `get/set data`:从目标节点上读取/写 数据。
+ `getChildren`:检索一个子节点上的列表。

基本使用与模板代码：
1、引入pom依赖：
```shell
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.4.9</version>
</dependency>
```

2、模板代码：

```shell
public static void main(String[] args) throws Exception {
// 初始化 ZooKeeper 实例(zk 地址、会话超时时间，与系统默认一致、 watcher)
ZooKeeper zk = new ZooKeeper("node-21:2181,node-22:2181", 30000, new Watcher() {
@Override
public void process(WatchedEvent event) {
System.out.println("已经触发了" + event.getType() + "事件！ ");
}
});
// 创建一个目录节点
zk.create("/testRootPath", "testRootData".getBytes(), Ids.OPEN_ACL_UNSAFE,
CreateMode.PERSISTENT);
// 创建一个子目录节点
zk.create("/testRootPath/testChildPathOne", "testChildDataOne".getBytes(),
Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
System.out.println(new String(zk.getData("/testRootPath",false,null)));
// 取出子目录节点列表
System.out.println(zk.getChildren("/testRootPath",true));
// 修改子目录节点数据
zk.setData("/testRootPath/testChildPathOne","modifyChildDataOne".getBytes(),-1);
System.out.println("目录节点状态： ["+zk.exists("/testRootPath",true)+"]");
// 创建另外一个子目录节点
zk.create("/testRootPath/testChildPathTwo", "testChildDataTwo".getBytes(),
Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
System.out.println(new String(zk.getData("/testRootPath/testChildPathTwo",true,null)));
// 删除子目录节点
zk.delete("/testRootPath/testChildPathTwo",-1);
zk.delete("/testRootPath/testChildPathOne",-1);
// 删除父目录节点
zk.delete("/testRootPath",-1);
zk.close();
}
```
---
#### Zookeeper典型应用

1、数据发布与订阅

>&emsp;&emsp;发布与订阅模型，即所谓的配置中心。发布者将数据发布到ZK节点上，供订阅者动态获取数据，
    实现配置信息的集中式管理和动态更新。

   &emsp;&emsp;**适合数据量很小的场景**，这样数据更新可能会比较快。

2、命名服务

>&emsp;&emsp;在分布式系统中，通过使用命名服务，客户端应用能够根据指定名字来获取资源或服务的地址，
提供者等信息。被命名的实体通常可以是集群中的机器，提供服务的地址，远程对象等等，这些都可以通称为名字(Name)。
通过ZK提供的创建节点的API，能够很容易地创建一个全局唯一的path，这个path就可以作为一个名称。

阿里巴巴集团开源的分布式服务框架 Dubbo 中使用 ZooKeeper 来作为其命
名服务，维护全局的服务地址列表。

3、分布式锁

>&emsp;&emsp;分布式锁，这个主要得益于Zookeeper保证了数据的强一致性。锁服务可以分为两类，一个是保持独占，另一个是控制时序。

- 保持独占：就是所有试图来获取这个锁的客户端，最终只有一个可以成功获得这把锁。通常的做法是一个`Znode`看作是一把锁，通过`create znode`的方式实现。

- 控制时序：就是所有试图来获取这个锁的客户端，最终都是会被安排执行，只是有个全局时序。做法和上面基本类似，只是这里 /distribute_lock 已经
预先存在，客户端在它下面创建临时有序节点（这个可以通过节点的属性控制：CreateMode.EPHEMERAL_SEQUENTIAL 来指定）。Zk 的父节点（/distribute_lock）
维持一份 sequence,保证子节点创建的时序性，从而也形成了每个客户端的全局时序。
















