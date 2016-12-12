# 协议规范

## 1. 介绍
这份文档规范了适用于工业界的区块链的概念，架构和协议。

### 1.1 什么是 fabric?
fabric 是在系统中数字事件，交易调用，不同参与者共享的总账。总账只能通过共识的参与者来更新，而且一旦被记录，信息永远不能被修改。每一个记录的事件都可以根据参与者的协议进行加密验证。

交易是安全的，私有的并且可信的。每个参与者通过向网络membership服务证明自己的身份来访问系统。交易是通过发放给各个的参与者，不可连接的，提供在网络上完全匿名的证书来生成的。交易内容通过复杂的密钥加密来保证只有参与者才能看到，确保业务交易私密性。

总账可以按照规定规则来审计全部或部分总账分录。在与参与者合作中，审计员可以通过基于时间的证书来获得总账的查看，连接交易来提供实际的资产操作。

fabric 是区块链技术的一种实现，比特币是可以在fabric上构建的一种简单应用。它通过模块化的架构来允许组件的“插入-运行”来实现这份协议规范。它具有强大的容器技术来支持任何主流的语言来开发智能合约。利用熟悉的和被证明的技术是fabric的座右铭。

### 1.2 为什么是 fabric?

早期的区块链技术提供一个目的集合，但是通常对具体的工业应用支持的不是很好。为了满足现代市场的需求，fabric 是基于工业关注点针对特定行业的多种多样的需求来设计的，并引入了这个领域内的开拓者的经验，如扩展性。fabric 为权限网络，隐私，和多个区块链网络的私密信息提供一种新的方法。

### 1.3 术语
以下术语在此规范的有限范围内定义，以帮助读者清楚准确的了解这里所描述的概念。

**交易(Transaction)** 是区块链上执行功能的一个请求。功能是使用**链码(Chaincode)**来实现的。

**交易者(Transactor)** 是向客户端应用这样发出交易的实体。

**总账(Ledger)** 是一系列包含交易和当前**世界状态(World State)**的加密的链接块。

**世界状态(World State)** 是包含交易执行结果的变量集合。

**链码(Chaincode)** 是作为交易的一部分保存在总账上的应用级的代码（如[智能合约](https://en.wikipedia.org/wiki/Smart_contract)）。链码运行的交易可能会改变世界状态。

**验证Peer(Validating Peer)** 是网络中负责达成共识，验证交易并维护总账的一个计算节点。

**非验证Peer(Non-validating Peer)** 是网络上作为代理把交易员连接到附近验证节点的计算节点。非验证Peer只验证交易但不执行它们。它还承载事件流服务和REST服务。

**带有权限的总账(Permissioned Ledger)** 是一个由每个实体或节点都是网络成员所组成的区块链网络。匿名节点是不允许连接的。

**隐私(Privacy)** 是链上的交易者需要隐瞒自己在网络上身份。虽然网络的成员可以查看交易，但是交易在没有得到特殊的权限前不能连接到交易者。

**保密(Confidentiality)** 是交易的内容不能被非利益相关者访问到的功能。

**可审计性(Auditability)** 作为商业用途的区块链需要遵守法规，很容易让监管机构审计交易记录。所以区块链是必须的。


## 2. Fabric

fabric是由下面这个小节所描述的核心组件所组成的。

### 2.1 架构
这个架构参考关注在三个类别中：会员(Membership)，区块链(Blockchan)和链码(chaincode)。这些类别是逻辑结构，而不是物理上的把不同的组件分割到独立的进程，地址空间，（虚拟）机器中。

![Reference architecture](images/refarch.png)

### 2.1.1 成员服务
成员服务为网络提供身份管理，隐私，保密和可审计性的服务。在一个不带权限的区块链中，参与者是不需要被授权的，且所有的节点都可以同样的提交交易并把它们汇集到可接受的块中，既：它们没有角色的区分。成员服务通过公钥基础设施(Public Key Infrastructure
(PKI))和去中心化的/共识技术使得不带权限的区块链变成带权限的区块链。在后者中，通过实体注册来获得长时间的，可能根据实体类型生成的身份凭证（登记证书enrollment certificates）。在用户使用过程中，这样的证书允许交易证书颁发机构（Transaction Certificate Authority
(TCA)）颁发匿名证书。这样的证书，如交易证书，被用来对提交交易授权。交易证书存储在区块链中，并对审计集群授权，否则交易是不可链接的。

### 2.1.2 区块链服务
区块链服务通过 HTTP/2 上的点对点（peer-to-peer）协议来管理分布式总账。为了提供最高效的哈希算法来维护世界状态的复制，数据结构进行了高度的优化。每个部署中可以插入和配置不同的共识算法（PBFT, Raft, PoW, PoS）。

### 2.1.3 链码服务
链码服务提供一个安全的，轻量的沙箱在验证节点上执行链码。环境是一个“锁定的”且安全的包含签过名的安全操作系统镜像和链码语言，Go，Java 和 Node.js 的运行时和 SDK 层。可以根据需要来启用其他语言。

### 2.1.4 事件
验证 peers 和链码可以向在网络上监听并采取行动的应用发送事件。这是一些预定义好的事件集合，链码可以生成客户化的事件。事件会被一个或多个事件适配器消费。之后适配器可能会把事件投递到其他设备，如 Web hooks 或 Kafka。

### 2.1.5 应用编程接口(API)
fabric的主要接口是 REST API，并通过 Swagger 2.0 来改变。API 允许注册用户，区块链查询和发布交易。链码与执行交易的堆间的交互和交易的结果查询会由 API 集合来规范。

### 2.1.6 命令行界面(CLI)
CLI包含REST API的一个子集使得开发者能更快的测试链码或查询交易状态。CLI 是通过 Go 语言来实现，并可在多种操作系统上操作。

### 2.2 拓扑
fabric 的一个部署是由成员服务，多个验证 peers、非验证 peers 和一个或多个应用所组成一个链。也可以有多个链，各个链具有不同的操作参数和安全要求。

### 2.2.1 单验证Peer
功能上讲，一个非验证 peer 是验证 peer 的子集；非验证 peer 上的功能都可以在验证 peer 上启用，所以在最简单的网络上只有一个验证peer组成。这个配置通常使用在开发环境：单个验证 peer 在编辑-编译-调试周期中被启动。

![Single Validating Peer](images/top-single-peer.png)

单个验证 peer 不需要共识，默认情况下使用`noops`插件来处理接收到的交易。这使得在开发中，开发人员能立即收到返回。

### 2.2.2 多验证 Peer
生产或测试网络需要有多个验证和非验证 peers 组成。非验证 peer 可以为验证 peer 分担像 API 请求处理或事件处理这样的压力。

![Multiple Validating Peers](images/top-multi-peer.png)

网状网络（每个验证peer需要和其它验证peer都相连）中的验证 peer 来传播信息。一个非验证 peer 连接到附近的，允许它连接的验证 peer。当应用可能直接连接到验证 peer 时，非验证 peer 是可选的。

### 2.2.3 多链
验证和非验证 peer 的各个网络组成一个链。可以根据不同的需求创建不同的链，就像根据不同的目的创建不同的 Web 站点。


## 3. 协议
fabric的点对点（peer-to-peer）通信是建立在允许双向的基于流的消息[gRPC](http://www.grpc.io/docs/)上的。它使用[Protocol Buffers](https://developers.google.com/protocol-buffers)来序列化peer之间传输的数据结构。Protocol buffers 是语言无关，平台无关并具有可扩展机制来序列化结构化的数据的技术。数据结构，消息和服务是使用 [proto3 language](https://developers.google.com/protocol-buffers/docs/proto3)注释来描述的。

### 3.1 消息
消息在节点之间通过`Message`proto 结构封装来传递的，可以分为 4 种类型：发现（Discovery）, 交易（Transaction）, 同步(Synchronization)和共识(Consensus)。每种类型在`payload`中定义了多种子类型。


`payload`是由不同的消息类型所包含的不同的像`Transaction`或`Response`这样的对象的不透明的字节数组。例如：`type`为`CHAIN_TRANSACTION`那么`payload`就是一个`Transaction`对象。

### 3.1.1 发现消息
在启动时，如果`CORE_PEER_DISCOVERY_ROOTNODE`被指定，那么 peer 就会运行发现协议。`CORE_PEER_DISCOVERY_ROOTNODE`是网络（任意peer）中扮演用来发现所有 peer 的起点角色的另一个 peer 的 IP 地址。协议序列以`payload`是一个包含：



这样的端点的`HelloMessage`对象的`DISC_HELLO`消息开始的。


**域的定义:**

- `PeerID` 是在启动时或配置文件中定义的 peer 的任意名字
- `PeerEndpoint` 描述了端点和它是验证还是非验证 peer
- `pkiID` 是 peer 的加密ID
- `address` 以`ip:port`这样的格式表示的 peer 的主机名或IP和端口
- `blockNumber` 是 peer 的区块链的当前的高度

如果收到的`DISC_HELLO` 消息的块的高度比当前 peer 的块的高度高，那么它马上初始化同步协议来追上当前的网络。

`DISC_HELLO`之后，peer 会周期性的发送`DISC_GET_PEERS`来发现任意想要加入网络的 peer。收到`DISC_GET_PEERS`后，peer 会发送`payload`
包含`PeerEndpoint`的数组的`DISC_PEERS`作为响应。这是不会使用其它的发现消息类型。

### 3.1.2 交易消息
有三种不同的交易类型：部署（Deploy），调用（Invoke）和查询（Query）。部署交易向链上安装指定的链码，调用和查询交易会调用部署号的链码。另一种需要考虑的类型是创建（Create）交易，其中部署好的链码是可以在链上实例化并寻址的。这种类型在写这份文档时还没有被实现。


### 3.1.2.1 交易的数据结构

`CHAIN_TRANSACTION`和`CHAIN_QUERY`类型的消息会在`payload`带有`Transaction`对象：



**域的定义:**
- `type` - 交易的类型, 为1时表示:
  - `UNDEFINED` - 为未来的使用所保留.
  - `CHAINCODE_DEPLOY` - 代表部署新的链码.
	- `CHAINCODE_INVOKE` - 代表一个链码函数被执行并修改了世界状态
	- `CHAINCODE_QUERY` - 代表一个链码函数被执行并可能只读取了世界状态
	- `CHAINCODE_TERMINATE` - 标记的链码不可用，所以链码中的函数将不能被调用
- `chaincodeID` - 链码源码，路径，构造函数和参数哈希所得到的ID
- `payloadHash` - `TransactionPayload.payload`所定义的哈希字节.
- `metadata` - 应用可能使用的，由自己定义的任意交易相关的元数据
- `uuid` - 交易的唯一ID
- `timestamp` - peer 收到交易时的时间戳
- `confidentialityLevel` - 数据保密的级别。当前有两个级别。未来可能会有多个级别。
- `nonce` - 为安全而使用
- `cert` - 交易者的证书
- `signature` - 交易者的签名
- `TransactionPayload.payload` - 交易的payload所定义的字节。由于payload可以很大，所以交易消息只包含payload的哈希

交易安全的详细信息可以在第四节找到

### 3.1.2.2 交易规范
一个交易通常会关联链码定义及其执行环境（像语言和安全上下文）的链码规范。现在，有一个使用Go语言来编写链码的实现。将来可能会添加新的语言。



**域的定义:**
- `chaincodeID` - 链码源码的路径和名字
- `ctorMsg` - 调用的函数名及参数
- `timeout` - 执行交易所需的时间（以毫秒表示）
- `confidentialityLevel` - 这个交易的保密级别
- `secureContext` - 交易者的安全上下文
- `metadata` - 应用想要传递下去的任何数据

当 peer 收到`chaincodeSpec`后以合适的交易消息包装它并广播到网络

### 3.1.2.3 部署交易
部署交易的类型是`CHAINCODE_DEPLOY`，且它的payload包含`ChaincodeDeploymentSpec`对象。


**域的定义:**
- `chaincodeSpec` - 参看上面的3.1.2.2节.
- `effectiveDate` - 链码准备好可被调用的时间
- `codePackage` - 链码源码的gzip

当验证 peer 部署链码时，它通常会校验`codePackage`的哈希来保证交易被部署到网络后没有被篡改。

### 3.1.2.4 调用交易

调用交易的类型是`CHAINCODE_DEPLOY`，且它的payload包含`ChaincodeInvocationSpec`对象。



### 3.1.2.5 查询交易
查询交易除了消息类型是`CHAINCODE_QUERY`其它和调用交易一样

### 3.1.3 同步消息
同步协议以3.1.1节描述的，当 peer 知道它自己的区块落后于其它 peer 或和它们不一样后所发起的。peer 广播`SYNC_GET_BLOCKS`，`SYNC_STATE_GET_SNAPSHOT`或`SYNC_STATE_GET_DELTAS`并分别接收`SYNC_BLOCKS`, `SYNC_STATE_SNAPSHOT`或 `SYNC_STATE_DELTAS`。

安装的共识插件（如：pbft）决定同步协议是如何被应用的。每个消息是针对具体的状态来设计的：

**SYNC_GET_BLOCKS** 是一个`SyncBlockRange`对象，包含一个连续区块的范围的`payload`的请求。

接收peer使用包含 `SyncBlocks`对象的`payload`的`SYNC_BLOCKS`信息来响应



`start`和`end`标识包含的区块的开始和结束，返回区块的顺序由`start`和`end`的值定义。如：当`start`=3，`end`=5时区块的顺序将会是3，4，5。当`start`=5，`end`=3时区块的顺序将会是5，4，3。


**SYNC_STATE_GET_SNAPSHOT** 请求当前世界状态的快照。 `payload`是一个`SyncStateSnapshotRequest`对象



`correlationId`是请求 peer 用来追踪响应消息的。接受 peer 回复`payload`为`SyncStateSnapshot`实例的`SYNC_STATE_SNAPSHOT`信息



这条消息包含快照或以0开始的快照流序列中的一块。终止消息是len(delta) == 0的块

**SYNC_STATE_GET_DELTAS** 请求连续区块的状态变化。默认情况下总账维护500笔交易变化。 delta(j)是block(i)和block(j)之间的状态转变，其中i=j-1。 `payload`包含`SyncStateDeltasRequest`实例


接收 peer 使用包含 `SyncStateDeltas`实例的`payload`的`SYNC_STATE_DELTAS`信息来响应


delta可能以顺序（从i到j）或倒序（从j到i）来表示状态转变

### 3.1.4 共识消息
共识处理交易，一个`CONSENSUS`消息是由共识框架接收到`CHAIN_TRANSACTION`消息时在内部初始化的。框架把`CHAIN_TRANSACTION`转换为 `CONSENSUS`然后以相同的`payload`广播到验证 peer。共识插件接收这条消息并根据内部算法来处理。插件可能创建自定义的子类型来管理共识有穷状态机。3.4节会介绍详细信息。


### 3.2 总账

总账由两个主要的部分组成，一个是区块链，一个是世界状态。区块链是在总账中的一系列连接好的用来记录交易的区块。世界状态是一个用来存储交易执行状态的键-值(key-value)数据库


### 3.2.1 区块链

#### 3.2.1.1 区块

区块链是由一个区块链表定义的，每个区块包含它在链中前一个区块的哈希。区块包含的另外两个重要信息是它包含区块执行所有交易后的交易列表和世界状态的哈希



**域的定义:**
* `version` - 用来追踪协议变化的版本号
* `timestamp` - 由区块提议者填充的时间戳
* `transactionsHash` - 区块中交易的merkle root hash
* `stateHash` - 世界状态的merkle root hash
* `previousBlockHash` - 前一个区块的hash
* `consensusMetadata` - 共识可能会引入的一些可选的元数据
* `nonHashData` - `NonHashData`消息会在计算区块的哈希前设置为nil，但是在数据库中存储为区块的一部分
* `BlockTransactions.transactions` - 交易消息的数组，由于交易的大小，它们不会被直接包含在区块中

#### 3.2.1.2 区块哈希

* `previousBlockHash`哈希是通过下面算法计算的
  1. 使用protocol buffer库把区块消息序列化为字节码

  2. 使用[FIPS 202](http://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.202.pdf)描述的SHA3 SHAKE256算法来对序列化后的区块消息计算大小为512位的哈希值

* `transactionHash`是交易merkle树的根。定义merkle tree实现是一个代办

* `stateHash`在3.2.2.1节中定义.

#### 3.2.1.3 非散列数据(NonHashData)

NonHashData消息是用来存储不需要所有 peer 都具有相同值的块元数据。他们是建议值。



* `localLedgerCommitTimestamp` - 标识区块提交到本地总账的时间戳

* `TransactionResult` - 交易结果的数组

* `TransactionResult.uuid` - 交易的ID

* `TransactionResult.result` - 交易的返回值

* `TransactionResult.errorCode` - 可以用来记录关联交易的错误信息的代码

* `TransactionResult.error` - 用来记录关联交易的错误信息的字符串


#### 3.2.1.4 交易执行

一个交易定义了它们部署或执行的链码。区块中的所有交易都可以在记录到总账中的区块之前运行。当链码执行时，他们可能会改变世界状态。之后世界状态的哈希会被记录在区块中。


### 3.2.2 世界状态

peer 的*世界状态*涉及到所有被部署的链码的*状态*集合。进一步说，链码的状态由键值对集合来表示。所以，逻辑上说，peer 的世界状态也是键值对的集合，其中键由元组`{chaincodeID, ckey}`组成。这里我们使用术语`key`来标识世界状态的键，如：元组`{chaincodeID, ckey}` ，而且我们使用`cKey`来标识链码中的唯一键。

为了下面描述的目的，假定`chaincodeID`是有效的utf8字符串，且`ckey`和`value`是一个或多个任意的字节的序列

#### 3.2.2.1 世界状态的哈希
当网络活动时，很多像交易提交和同步 peer 这样的场合可能需要计算 peer 观察到的世界状态的加密-哈希。例如，共识协议可能需要保证网络中*最小*数量的 peer 观察到同样的世界状态。

因为计算世界状态的加密-哈希是一个非常昂贵的操作，组织世界状态来使得当它改变时能高效的计算加密-哈希是非常可取的。将来，可以根据不同的负载条件来设计不同的组织形式。

由于fabric是被期望在不同的负载条件下都能正常工作，所以需要一个可拔插的机制来支持世界状态的组织。

#### 3.2.2.1.1 Bucket-tree

*Bucket-tree* 是世界状态的组织方式的实现。为了下面描述的目的，世界状态的键被表示成两个组件(`chaincodeID` and `ckey`) 的通过nil字节的级联，如：`key` = `chaincodeID`+`nil`+`cKey`。

这个方法的模型是一个*merkle-tree*在*hash table*桶的顶部来计算*世界状态*的加密-哈希

这个方法的核心是世界状态的*key-values*被假定存储在由预先决定的桶的数量(`numBuckets`)所组成的哈希表中。一个哈希函数(`hashFunction`) 被用来确定包含给定键的桶数量。注意`hashFunction`不代表SHA3这样的加密-哈希方法，而是决定给定的键的桶的数量的正规的编程语言散列函数。

为了对 merkle-tree建模，有序桶扮演了树上的叶子节点-编号最低的桶是树中的最左边的叶子节点。为了构造树的最后第二层，叶子节点的预定义数量 (`maxGroupingAtEachLevel`)，从左边开始把每个这样的分组组合在一起，一个节点被当作组中所有叶子节点的共同父节点来插入到最后第二层中。注意最后的父节点的数量可能会少于`maxGroupingAtEachLevel`这个构造方式继续使用在更高的层级上直到树的根节点被构造。



下面这个表展示的在`{numBuckets=10009 and maxGroupingAtEachLevel=10}`的配置下会得到的树在不同层级上的节点数。



为了计算世界状态的加密-哈希，需要计算每个桶的加密-哈希，并假设它们是merkle-tree的叶子节点的加密-哈希。为了计算桶的加密-哈希，存储在桶中的键值对首先被序列化为字节码并在其上应用加密-哈希函数。为了序列化桶的键值对，所有具有公共chaincodeID前缀的键值对分别序列化并以chaincodeID的升序的方式追加在一起。为了序列化一个chaincodeID的键值对，会涉及到下面的信息：

   1. chaincodeID的长度(chaincodeID的字节数)
   - chaincodeID的utf8字节码
   - chaincodeID的键值对数量
   - 对于每个键值对(以ckey排序)
      - ckey的长度
      - ckey的字节码
      - 值的长度
      - 值的字节码

对于上面列表的所有数值类型项（如：chaincodeID的长度），使用protobuf的变体编码方式。上面这种编码方式的目的是为了桶中的键值对的字节表示方式不会被任意其他键值对的组合所产生，并减少了序列化字节码的总体大小。

例如：考虑具有`chaincodeID1_key1:value1, chaincodeID1_key2:value2, 和 chaincodeID2_key1:value1`这样名字的键值对的桶。序列化后的桶看上去会像：`12 + chaincodeID1 + 2 + 4 + key1 + 6 + value1 + 4 + key2 + 6 + value2 + 12 + chaincodeID2 + 1 + 4 + key1 + 6 + value1`


如果桶中没有键值对，那么加密-哈希为`nil`。

中间节点和根节点的加密-哈希与标准merkle-tree的计算方法一样，即：应用加密-哈希函数到所有子节点的加密-哈希从左到右级联后得到的字节码。进一步说，如果一个子节点的加密-哈希为`nil`，那么这个子节点的加密-哈希在级联子节点的加密-哈希是就被省略。如果它只有一个子节点，那么它的加密-哈希就是子节点的加密-哈希。最后，根节点的加密-哈希就是世界状态的加密-哈希。

上面这种方法在状态中少数键值对改变时计算加密-哈希是有性能优势的。主要的优势包括：
  - 那些没有变化的桶的计算会被跳过
  - merkle-tree的宽度和深度可以通过配置`numBuckets`和`maxGroupingAtEachLevel`参数来控制。树的不同深度和宽度对性能和不同的资源都会产生不同的影响。

在一个具体的部署中，所有的 peer 都期望使用相同的`numBuckets, maxGroupingAtEachLevel, 和 hashFunction`的配置。进一步说，如果任何一个配置在之后的阶段被改变，那么这些改变需要应用到所有的 peer 中，来保证 peer 节点之间的加密-哈希的比较是有意义的。即使，这可能会导致基于实现的已有数据的迁移。例如：一种实现希望存储树中所有节点最后计算的加密-哈希，那么它就需要被重新计算。


### 3.3 链码（Chaincode）
链码是在交易（参看3.1.2节）被部署时分发到网络上，并被所有验证 peer 通过隔离的沙箱来管理的应用级代码。尽管任意的虚拟技术都可以支持沙箱，现在是通过Docker容器来运行链码的。这节中描述的协议可以启用不同虚拟实现的插入与运行。


### 3.3.1 虚拟机实例化
一个实现VM接口的虚拟机



fabric在处理链码上的部署交易或其他交易时，如果这个链码的VM未启动（崩溃或之前的不活动导致的关闭）时实例化VM。每个链码镜像通过`build`函数构建，通过`start`函数启动，并使用`stop`函数停止。

一旦链码容器被启动，它使用gRPC来连接到启动这个链码的验证 peer，并为链码上的调用和查询交易建立通道。

### 3.3.2 链码协议
验证 peer 和它的链码之间是通过gRPC流来通信的。链码容器上有shim层来处理链码与验证 peer 之间的protobuf消息协议。



**域的定义:**
- `Type` 是消息的类型
- `payload` 是消息的payload. 每个payload取决于`Type`.
- `uuid` 消息唯一的ID

消息的类型在下面的小节中描述

链码实现被验证 peer 在处理部署，调用或查询交易时调用的`Chaincode`接口



`Init`, `Invoke` 和 `Query`函数使用`function` and `args`参数来支持多种交易。`Init`是构造函数，它只在部署交易时被执行。`Query`函数是不允许修改链码的状态的；它只能读取和计算并以byte数组的形式返回。

### 3.3.2.1 链码部署
当部署时（链码容器已经启动），shim层发送一次性的具有包含`ChaincodeID`的`payload`的`REGISTER`消息给验证 peer。然后 peer 以`REGISTERED`或`ERROR`来响应成功或失败。当收到`ERROR`后shim关闭连接并退出。

注册之后，验证 peer 发送具有包含`ChaincodeInput`对象的`INIT`消息。shim使用从`ChaincodeInput`获得的参数来调用`Init`函数，通过像设置持久化状态这样操作来初始化链码。

shim根据`Init`函数的返回值，响应`RESPONSE`或`ERROR`消息。如果没有错误，那么链码初始化完成，并准备好接收调用和查询交易。

### 3.3.2.2 链码调用
当处理调用交易时，验证 peer 发送`TRANSACTION`消息给链码容器的shim，由它来调用链码的`Invoke`函数，并传递从`ChaincodeInput`得到的参数。shim响应`RESPONSE`或`ERROR`消息来表示函数完成。如果接收到`ERROR`函数，`payload`包含链码所产生的错误信息。

### 3.3.2.3 链码查询
与调用交易一样，验证 peer 发送`QUERY`消息给链码容器的shim，由它来调用链码的`Query`函数，并传递从`ChaincodeInput`得到的参数。`Query`函数可能会返回状态值或错误，它会把它通过`RESPONSE`或`ERROR`消息来传递给验证 peer。

### 3.3.2.4 链码状态
每个链码可能都定义了它自己的持久化状态变量。例如，一个链码可能创建电视，汽车或股票这样的资产来保存资产属性。当`Invoke`函数处理时，链码可能会更新状态变量，例如改变资产所有者。链码会根据下面这些消息类型类操作状态变量：

#### PUT_STATE
链码发送一个`payload`包含`PutStateInfo`对象的`PU_STATE`消息来保存键值对。


#### GET_STATE
链码发送一个由`payload`指定要获取值的键的`GET_STATE`消息。

#### DEL_STATE
链码发送一个由`payload`指定要删除值的键的`DEL_STATE`消息。

#### RANGE_QUERY_STATE
链码发送一个`payload`包含`RANGE_QUERY_STATE`对象的`RANGE_QUERY_STATE`来获取一个范围内的值。



`startKey`和`endKey`假设是通过字典排序的. 验证 peer 响应一个`payload`是`RangeQueryStateResponse`对象的`RESPONSE`消息



如果相应中`hasMore=true`，这表示有在请求的返回中还有另外的键。链码可以通过发送包含与响应中ID相同的ID的`RangeQueryStateNext`消息来获取下一集合。



当链码结束读取范围，它会发送带有ID的`RangeQueryStateClose`消息来期望它关闭。



#### INVOKE_CHAINCODE
链码可以通过发送`payload`包含 `ChaincodeSpec`对象的`INVOKE_CHAINCODE`消息给验证 peer 来在相同的交易上下文中调用另一个链码

#### QUERY_CHAINCODE
链码可以通过发送`payload`包含 `ChaincodeSpec`对象的`QUERY_CHAINCODE`消息给验证 peer 来在相同的交易上下文中查询另一个链码


### 3.4 插拔式共识框架

共识框架定义了每个共识插件都需要实现的接口：

  - `consensus.Consenter`: 允许共识插件从网络上接收消息的接口
  - `consensus.CPI`:  共识编程接口_Consensus Programming Interface_ (`CPI`) 是共识插件用来与栈交互的，这个接口可以分为两部分：
	  - `consensus.Communicator`: 用来发送（广播或单播）消息到其他的验证 peer
	  - `consensus.LedgerStack`: 这个接口使得执行框架像总账一样方便

就像下面描述的细节一样，`consensus.LedgerStack`封装了其他接口，`consensus.Executor`接口是共识框架的核心部分。换句话说，`consensus.Executor`接口允许一个（批量）交易启动，执行，根据需要回滚，预览和提交。每一个共识插件都需要满足以所有验证 peer 上全序的方式把批量（块）交易（通过`consensus.Executor.CommitTxBatch`）被提交到总账中（参看下面的`consensus.Executor`接口获得详细细节）。

当前，共识框架由`consensus`, `controller`和`helper`这三个包组成。使用`controller`和`helper`包的主要原因是防止Go语言的“循环引入”和当插件更新时的最小化代码变化。

- `controller` 包规范了验证 peer 所使用的共识插件
- `helper` 是围绕公式插件的垫片，它是用来与剩下的栈交互的，如为其他 peer 维护消息。

这里有2个共识插件提供：`pbft`和`noops`：

-  `obcpbft`包包含实现 *PBFT* [1] 和 *Sieve* 共识协议的共识插件。参看第5节的详细介绍
-  `noops` 是一个为开发和测试提供的''假的''共识插件. 它处理所有共识消息但不提供共识功能，它也是一个好的学习如何开发一个共识插件的简单例子。

### 3.4.1 `Consenter` 接口


`Consenter`接口是插件对（外部的）客户端请求的入口，当处理共识时，共识消息在内部（如从共识模块）产生。NewConsenter`创建`Consenter`插件。`RecvMsg`以到达共识的顺序来处理进来的交易。

阅读下面的`helper.HandleMessage`来理解 peer 是如何和这个接口来交互的。

### 3.4.2 `CPI`接口

定义:

`CPI` 允许插件和栈交互。它是由`helper.Helper`对象实现的。回想一下这个对象是：

  1. 在`helper.NewConsensusHandler`被调用时初始化的
  2. 当它们的插件构造了`consensus.Consenter`对象，那么它对插件的作者是可访问的


### 3.4.3 `Inquirer`接口


这个接口是`consensus.CPI`接口的一部分。它是用来获取网络中验证 peer 的（`GetNetworkHandles`）句柄，以及那些验证 peer 的明细(`GetNetworkInfo`)：

注意peers由`pb.PeerID`对象确定。这是一个protobuf消息，当前定义为（注意这个定义很可能会被修改）：



### 3.4.4 `Communicator`接口

这个接口是`consensus.CPI`接口的一部分。它是用来与网络上其它 peer 通信的（`helper.Broadcast`, `helper.Unicast`）：

### 3.4.5 `SecurityUtils`接口



这个接口是`consensus.CPI`接口的一部分。它用来处理消息签名(`Sign`)的加密操作和验证签名(`Verify`)


### 3.4.6 `LedgerStack` 接口


`CPI`接口的主要成员，`LedgerStack` 组与fabric的其它部分与共识相互作用，如执行交易，查询和更新总账。这个接口支持对本地区块链和状体的查询，更新本地区块链和状态，查询共识网络上其它节点的区块链和状态。它是由`Executor`, `Ledger`和`RemoteLedgers`这三个接口组成的。下面会描述它们。

### 3.4.7 `Executor` 接口


executor接口是`LedgerStack`接口最常使用的部分，且是共识网络工作的必要部分。接口允许交易启动，执行，根据需要回滚，预览和提交。这个接口由下面这些方法组成。

#### 3.4.7.1 开始批量交易


这个调用接受任意的，故意含糊的`id`，来使得共识插件可以保证与这个具体的批量相关的交易才会被执行。例如：在pbft实现中，这个`id`是被执行交易的编码过的哈希。

#### 3.4.7.2 执行交易

这个调用根据总账当前的状态接受一组交易，并返回带有对应着交易组的错误信息组的当前状态的哈希。注意一个交易所产生的错误不影响批量交易的安全提交。当遇到失败所采用的策略取决与共识插件的实现。这个接口调用多次是安全的。

#### 3.4.7.3 提交与回滚交易


这个调用中止了批量执行。这会废弃掉对当前状态的操作，并把总账状态回归到之前的状态。批量是从`BeginBatchTx`开始的，如果需要开始一个新的就需要在执行任意交易之前重新创建一个。



这个调用是共识插件对非确定性交易执行的测试时最有用的方法。区块返回的哈希表部分会保证，当`CommitTxBatch`被立即调用时的区块是同一个。这个保证会被任意新的交易的执行所打破。


这个调用提交区块到区块链中。区块必须以全序提交到区块链中，``CommitTxBatch``结束批量交易，在执行或提交任意的交易之前必须先调用`BeginTxBatch`。


### 3.4.8 `Ledger` 接口



``Ledger`` 接口是为了允许共识插件询问或可能改变区块链当前状态。它是由下面描述的三个接口组成的

#### 3.4.8.1 `ReadOnlyLedger` 接口

定义：


`ReadOnlyLedger` 接口是为了查询总账的本地备份，而不会修改它。它是由下面这些函数组成的。


这个函数返回区块链总账的长度。一般来说，这个函数永远不会失败，在这种不太可能发生情况下，错误被传递给调用者，由它确定是否需要恢复。具有最大区块值的区块的值为`GetBlockchainSize()-1`

注意在区块链总账的本地副本是腐坏或不完整的情况下，这个调用会返回链中最大的区块值+1。这允许节点在旧的块是腐坏或丢失的情况下能继续操作当前状态/块。


这个调用返回区块链中块的数值`id`。一般来说这个调用是不会失败的，除非请求的区块超出当前区块链的长度，或者底层的区块链被腐坏了。`GetBlock`的失败可能可以通过状态转换机制来取回它。


这个调用返回总账的当前状态的哈希。一般来说，这个函数永远不会失败，在这种不太可能发生情况下，错误被传递给调用者，由它确定是否需要恢复。


#### 3.4.8.2 `UtilLedger` 接口

定义：


`UtilLedger` 接口定义了一些由本地总账提供的有用的功能。使用mock接口来重载这些功能在测试时非常有用。这个接口由两个函数构成。


尽管`*pb.Block`定义了`GetHash`方法，为了mock测试，重载这个方法会非常有用。因此，建议`GetHash`方法不直接调用，而是通过`UtilLedger.HashBlock`接口来调用这个方法。一般来说，这个函数永远不会失败，但是错误还是会传递给调用者，让它决定是否使用适当的恢复。



这个方法是用来校验区块链中的大的区域。它会从高的块`start`到低的块`finish`，返回第一个块的`PreviousBlockHash`与块的前一个块的哈希不相符的块编号以及错误信息。注意，它一般会标识最后一个好的块的编号，而不是第一个坏的块的编号。


#### 3.4.8.3 `WritableLedger` 接口



`WritableLedger`  接口允许调用者更新区块链。注意这_NOT_ _不是_共识插件的通常用法。当前的状态需要通过`Executor`接口执行交易来修改，新的区块在交易提交时生成。相反的，这个接口主要是用来状态改变和腐化恢复。特别的，这个接口下的函数_永远_不能直接暴露给共识消息，这样会导致打破区块链所承诺的不可修改这一概念。这个结构包含下面这些函数。

  -
  	```
	PutBlock
	```
     这个函数根据给定的区块编号把底层区块插入到区块链中。注意这是一个不安全的接口，所以它不会有错误返回或返回。插入一个比当前区块高度更高的区块是被允许的，同样，重写一个已经提交的区块也是被允许的。记住，由于哈希技术使得创建一个链上的更早的块是不可行的，所以这并不影响链的可审计性和不可变性。任何尝试重写区块链的历史的操作都能很容易的被侦测到。这个函数一般只用于状态转移API。

  -
  	```
	ApplyStateDelta
	```

    这个函数接收状态变化，并把它应用到当前的状态。变化量的应用会使得状态向前或向后转变，这取决于状态变化量的构造，与`Executor`方法一样，`ApplyStateDelta`接受一个同样会被传递给`CommitStateDelta` or `RollbackStateDelta`不透明的接口`id`

  -
 	```
	CommitStateDelta
	```

    这个方法提交在`ApplyStateDelta`中应用的状态变化。这通常是在调用者调用`ApplyStateDelta`后通过校验由`GetCurrentStateHash()`获得的状态哈希之后调用的。这个函数接受与传递给`ApplyStateDelta`一样的`id`。

  -
  	```
	RollbackStateDelta
	```

    这个函数撤销在`ApplyStateDelta`中应用的状态变化量。这通常是在调用者调用`ApplyStateDelta`后与由`GetCurrentStateHash()`获得的状态哈希校验失败后调用的。这个函数接受与传递给`ApplyStateDelta`一样的`id`。


  -
  	```
   	EmptyState
   	```

    这个函数将会删除整个当前状态，得到原始的空状态。这通常是通过变化量加载整个新的状态时调用的。这一般只对状态转移API有用。

### 3.4.9 `RemoteLedgers` 接口



`RemoteLedgers` 接口的存在主要是为了启用状态转移，和向其它副本询问区块链的状态。和`WritableLedger`接口一样，这不是给正常的操作使用，而是为追赶，错误恢复等操作而设计的。这个接口中的所有函数调用这都有责任来处理超时。这个接口包含下面这些函数：

  -
  	```
  	GetRemoteBlocks
  	```

    这个函数尝试从由`peerID`指定的 peer 中取出由`start`和`finish`标识的范围中的`*pb.SyncBlocks`流。一般情况下，由于区块链必须是从结束到开始这样的顺序来验证的，所以`start`是比`finish`更高的块编号。由于慢速的结构，其它请求的返回可能出现在这个通道中，所以调用者必须验证返回的是期望的块。第二次以同样的`peerID`来调用这个方法会导致第一次的通道关闭。


  -
  	```
   	GetRemoteStateSnapshot
   	```

    这个函数尝试从由`peerID`指定的 peer 中取出`*pb.SyncStateSnapshot`流。为了应用结果，首先需要通过`WritableLedger`的`EmptyState`调用来清空存在在状态，然后顺序应用包含在流中的变化量。

  -
  	```
   	GetRemoteStateDeltas
   	```

    这个函数尝试从由`peerID`指定的 peer 中取出由`start`和`finish`标识的范围中的`*pb.SyncStateDeltas`流。由于慢速的结构，其它请求的返回可能出现在这个通道中，所以调用者必须验证返回的是期望的块变化量。第二次以同样的`peerID`来调用这个方法会导致第一次的通道关闭。


### 3.4.10 `controller`包

#### 3.4.10.1 controller.NewConsenter

签名:

```
func NewConsenter
```
这个函数读取为`peer`过程指定的`core.yaml`配置文件中的`peer.validator.consensus`的值。键`peer.validator.consensus`的有效值指定运行`noops`还是`pbft`共识插件。（注意，它最终被改变为`noops`或`custom`。在`custom`情况下，验证 peer 将会运行由`consensus/config.yaml`中定义的共识插件）

插件的作者需要编辑函数体，来保证路由到它们包中正确的构造函数。例如，对于`pbft` 我们指向`pbft.GetPlugin`构造器。

这个函数是当设置返回信息处理器的`consenter`域时，被`helper.NewConsensusHandler`调用的。输入参数`cpi`是由`helper.NewHelper`构造器输出的，并实现了`consensus.CPI`接口

### 3.4.11 `helper`包

#### 3.4.11.1 高层次概述

验证 peer 通过`helper.NewConsesusHandler`函数(一个处理器工厂)，为每个连接的 peer 建立消息处理器(`helper.ConsensusHandler`)。每个进来的消息都会检查它的类型(`helper.HandleMessage`)；如果这是为了共识必须到达的消息，它会传递到 peer 的共识对象(`consensus.Consenter`)。其它的信息会传递到栈中的下一个信息处理器。

#### 3.4.11.2 helper.ConsensusHandler


共识中的上下文，我们只关注域`coordinator`和`consenter`。`coordinator`就像名字隐含的那样，它被用来在 peer 的信息处理器之间做协调。例如，当 peer 希望`Broadcast`时，对象被访问。共识需要到达的共识者会接收到消息并处理它们。

注意，`fabric/peer/peer.go`定义了`peer.MessageHandler` (接口)，和`peer.MessageHandlerCoordinator`（接口）类型。

#### 3.4.11.3 helper.NewConsensusHandler

签名:

```
func NewConsensusHandler
```

创建一个`helper.ConsensusHandler`对象。为每个`coordinator`设置同样的消息处理器。同时把`consenter`设置为`controller.NewConsenter(NewHelper(coord))`


### 3.4.11.4 helper.Helper


包含验证peer的`coordinator`的引用。对象是否为peer实现了`consensus.CPI`接口。

#### 3.4.11.5 helper.NewHelper

签名:

```
func NewHelper
```

返回`coordinator`被设置为输入参数`mhc`（`helper.ConsensusHandler`消息处理器的`coordinator`域）的`helper.Helper`对象。这个对象实现了`consensus.CPI`接口，从而允许插件与栈进行交互。


#### 3.4.11.6 helper.HandleMessage

回忆一下，`helper.NewConsensusHandler`返回的`helper.ConsesusHandler`对象实现了 `peer.MessageHandler` 接口：

```
type MessageHandler 
```

在共识的上下文中，我们只关心`HandleMessage`方法。签名：

```
func HandleMessage
```

这个函数检查进来的`Message`的`Type`。有四种情况：

  1. 等于`pb.Message_CONSENSUS`：传递给处理器的`consenter.RecvMsg`函数。
  2. 等于`pb.Message_CHAIN_TRANSACTION` (如：一个外部部署的请求): 一个响应请求首先被发送给用户，然后把消息传递给`consenter.RecvMsg`函数
  3. 等于`pb.Message_CHAIN_QUERY` (如：查询): 传递给`helper.doChainQuery`方法来在本地执行
  4. 其它: 传递给栈中下一个处理器的`HandleMessage`方法


### 3.5 事件
事件框架提供了生产和消费预定义或自定义的事件的能力。它有3个基础组件：
  - 事件流
  - 事件适配器
  - 事件结构

#### 3.5.1 事件流
事件流是用来发送和接收事件的gRPC通道。每个消费者会与事件框架建立事件流，并快速传递它感兴趣的事件。事件生成者通过事件流只发送合适的事件给连接到生产者的消费者。

事件流初始化缓冲和超时参数。缓冲保存着几个等待投递的事件，超时参数在缓冲满时有三个选项：

- 如果超时小于0，丢弃新到来的事件
- 如果超时等于0，阻塞事件知道缓冲再次可用
- 如果超时大于0，等待指定的超时时间，如果缓冲还是满的话就丢弃事件


#### 3.5.1.1 事件生产者
事件生产者暴露函数`Send(e *pb.Event)`来发送事件，其中`Event`可以是预定义的`Block`或`Generic`事件。将来会定义更多的事件来包括其它的fabric元素。


`eventType`和`payload`是由事件生产者任意定义的。例如，JSON数据可能被用在`payload`中。链码或插件发出`Generic`事件来与消费者通讯。

#### 3.5.1.2 事件消费者
事件消费者允许外部应用监听事件。每个事件消费者通过时间流注册事件适配器。消费者框架可以看成是事件流与适配器之间的桥梁。一种典型的事件消费者使用方式：

#### 3.5.2 事件适配器
事件适配器封装了三种流交互的切面：
  - 返回所有感兴趣的事件列表的接口
  - 当事件消费者框架接受到事件后调用的接口
  - 当事件总线终止时，事件消费者框架会调用的接口

引用的实现提供了Golang指定语言绑定

把gRPC当成事件总线协议来使用，允许事件消费者框架对于不同的语言的绑定可移植而不影响事件生成者框架。

#### 3.5.3 事件框架

这节详细描述了事件系统的消息结构。为了简单起见，消息直接使用Golang描述。

事件消费者和生产者之间通信的核心消息是事件。

每一个上面的定义必须是`Register`, `Block`或`Generic`中的一种。

就像之前提到过的一样，消费者通过与生产者建立连接来创建事件总线，并发送`Register`事件。`Register`事件实质上是一组声明消费者感兴趣的事件的`Interest`消息。


事件可以通过protobuf结构直接发送，也可以通过指定适当的`responseType`来发送JSON结构。

当前，生产者框架可以生成`Block`和`Generic`事件。`Block`是用来封装区块链中区块属性的消息。
