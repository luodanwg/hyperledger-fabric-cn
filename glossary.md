
## Roles & Personas

#### _角色_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Chain Member</b></td>
<td>
实体不参与blockchain网络的验证过程，但有助于保持网络的完整性。与链事务处理者不同，链成员维护分类帐的本地副本。
</td>
</tr>
<tr>
<td width="20%"><b>链的事务处理者</b></td>
<td>
有权创建事务和查询网络数据的实体。
</td>
</tr>
<tr>
<td width="20%"><b>链的验证者</b></td>
<td>
拥有链网关系的实体。每个链验证器在确定事务是否有效时有投票权，因此链验证器可以询问发送到其链的所有事务。
</td>
</tr>
<tr>
<td width="20%"><b>链的审查者</b></td>
<td>
具有审查事务权限的实体。
</td>
</tr>
</table>

#### __参与者_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Solution User</b></td>
<td>
最终用户与链网络的细节无关，它们通常通过解决方案提供商提供的应用程序在链网络上发起交易。
<p><p>
<span style="text-decoration:underline">Roles:</span> None
</td>
</tr>
<tr>
<td width="20%"><b>解决方案提供商</b></td>
<td>
开发基于移动 和/或 基于浏览器的应用程序的组织，用于最终用户（解决方案）访问链网络。一些应用所有者也可以是网络所有者。
<p><p>
Roles: 链的交易执行者
</td>
</tr>
<tr>
<td width="20%"><b>网络所有者</b></td>
<td>
所有者设置和定义链网络的目的。他们是网络的利益相关者。
<p><p>
角色: 链的交易执行者, 链的验证者
</td>
</tr>
<tr>
<td width="20%"><b>网络所有者</b></td>
<td>
所有者是可以验证事务的网络的利益相关者。在网络首次启动后，其所有者（然后成为所有者）将邀请业务伙伴共同拥有网络（通过分配其验证节点）。添加到网络中的所有新所有者必须获得其现有所有者们的批准。
<p><p>
角色：链的交易执行着，链的验证者
</td>
</tr>
<tr>
<td width="20%"><b>网络成员</b></td>
<td>
成员是区块链网络的参与者，其不能验证交易，但是有权将用户添加到网络中。
<p><p>
Roles: Chain Transactor, Chain Member
角色：链的交易执行者，链的成员
</td>
</tr>
<tr>
<td width="20%"><b>网络使用者</b></td>
<td>
网络的最终用户也是解决方案用户。与网络所有者和成员不同，用户不具有节点。它们通过由成员或所有者节点提供的入口点与网络进行交易。
<p><p>
角色：链的交易执行者
</td>
</tr>
<tr>
<td width="20%"><b>网络审查者</b></td>
<td>
具有审查交易许可的个人或组织。
<p><p>
角色：链的审查者
</td>
</tr>
</table>

&nbsp;

## 商业网络

#### _网络的类型 (商业视点)_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>工业网络</b></td>
<td>
一个链式网络，为特定行业提供服务解决方案
</td>
</tr>
<tr>
<td width="20%"><b>区域产业网络</b></td>
<td>
一个链式网络，为特定行业和地区构建的应用程序提供服务。
</td>
</tr>
<tr>
<td width="20%"><b>应用网络</b></td>
<td>
一个只服务单一解决方案的链式网络。
</td>
</tr>
</table>

#### _链的类型 (概念视点)_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>主链</b></td>
<td>
业务网络;每个主链操作由同一组组织验证的一个或多个应用/解决方案。
</td>
</tr>
<tr>
<td width="20%"><b>保密链</b></td>
<td>
创建用于运行保密业务逻辑的专用链，只能由合同利益相关者访问。
</td>
</tr>
</table>


&nbsp;

## 网络管理

#### _成员管理_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>所有者注册</b></td>
<td>
向区块链网络注册和邀请新所有者的过程。给参与者添加或删除网络所有权时，需要获得现有网络所有者们的批准。
</td>
</tr>
<tr>
<td width="20%"><b>成员注册</b></td>
<td>
向区块链网络注册和邀请新网络成员的过程。
</td>
</tr>
<tr>
<td width="20%"><b>用户注册</b></td>
<td>
将新用户注册到区块链网络的过程。成员和所有者可以自己注册用户，只要他们遵循其网络的策略。
</td>
</tr>
</table>


&nbsp;

## 交易

#### _交易类型_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>部署事务</b></td>
<td>
将新链码部署到链的事务。
</td>
</tr>
<tr>
<td width="20%"><b>调用事务</b></td>
<td>
调用chaincode上的功能的事务。
</td>
</tr>
</table>


#### _交易保密性_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>公共交易</b></td>
<td>
A transaction with its payload in the open. Anyone with access to a chain network can interrogate the details of public transactions.
一个事务的是开放的。任何可以访问链式网络的人都可以查询公共交易的详细信息。
</td>
</tr>
<tr>
<td width="20%"><b>Confidential Transaction</b></td>
<td>
A transaction with its payload cryptographically hidden such that no one besides the stakeholders of a transaction can interrogate its content.
</td>
</tr>
<tr>
<td width="20%"><b>Confidential Chaincode Transaction</b></td>
<td>
A transaction with its payload encrypted such that only validators can decrypt them. Chaincode confidentiality is determined during deploy time. If a chaincode is deployed as a confidential chaincode, then the payload of all subsequent invocation transactions to that chaincode will be encrypted.
</td>
</tr>
</table>


#### _Inter-chain Transactions_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Inter-Network Transaction</b></td>
<td>
Transactions between two business networks (main chains).
</td>
</tr>
<tr>
<td width="20%"><b>Inter-Chain Transaction</b></td>
<td>
Transactions between confidential chains and main chains. Chaincodes in a confidential chain can trigger transactions on one or multiple main chain(s).
</td>
</tr>
</table>

&nbsp;

## Network Entities

#### _Systems_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Application Backend</b></td>
<td>
  Purpose: Backend application service that supports associated mobile and/or browser based applications.
  <p><p>
  Key Roles:<p>
  1)	Manages end users and registers them with the membership service
  <p>
  2)	Initiates transactions requests, and sends the requests to a node
  <p><p>
  Owned by: Solution Provider, Network Proprietor
</td>
</tr>
<tr>
<td width="20%"><b>Non Validating Node (Peer)</b></td>
<td>
  Purpose: Constructs transactions and forwards them to validating nodes. Peer nodes keep a copy of all transaction records so that solution providers can query them locally.
  <p><p>
  Key Roles:<p>
  1)	Manages and maintains user certificates issued by the membership service<p>
  2)	Constructs transactions and forwards them to validating nodes <p>
  3)	Maintains a local copy of the ledger, and allows application owners to query information locally.
  <p><p>
	Owned by: Solution Provider, Network Auditor
</td>
</tr>
<tr>
<td width="20%"><b>Validating Node (Peer)</b></td>
<td>
  Purpose: Creates and validates transactions, and maintains the state of chaincodes<p><p>
  Key Roles:<p>
  1)	Manages and maintains user certificates issued by membership service<p>
  2)	Creates transactions<p>
  3)	Executes and validates transactions with other validating nodes on the network<p>
  4)	Maintains a local copy of ledger<p>
  5)	Participates in consensus and updates ledger
  <p><p>
  Owned by: Network Proprietor, Solution Provider (if they belong to the same entity)
</td>
</tr>
<tr>
<td width="20%"><b>Membership Service</b></td>
<td>
  Purpose: Issues and manages the identity of end users and organizations<p><p>
  Key Roles:<p>
  1)	Issues enrollment certificate to each end user and organization<p>
  2)	Issues transaction certificates associated to each end user and organization<p>
  3)	Issues TLS certificates for secured communication between Hyperledger fabric entities<p>
  4)	Issues chain specific keys
  <p><p>
  Owned by: Third party service provider
</td>
</tr>
</table>


#### _Membership Service Components_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Registration Authority</b></td>
<td>
Assigns registration username & registration password pairs to network participants. This username/password pair will be used to acquire enrollment certificate from ECA.
</td>
</tr>
<tr>
<td width="20%"><b>Enrollment Certificate Authority (ECA)</b></td>
<td>
Issues enrollment certificates (ECert) to network participants that have already registered with a membership service. ECerts are long term certificates used to identify individual entities participating in one or more networks.
</td>
</tr>
<tr>
<td width="20%"><b>Transaction Certificate Authority (TCA)</b></td>
<td>
Issues transaction certificates (TCerts) to ECert owners. An infinite number of TCerts can be derived from each ECert. TCerts are used by network participants to send transactions. Depending on the level of security requirements, network participants may choose to use a new TCert for every transaction.
</td>
</tr>
<tr>
<td width="20%"><b>TLS-Certificate Authority (TLS-CA)</b></td>
<td>
Issues TLS certificates to systems that transmit messages in a chain network. TLS certificates are used to secure the communication channel between systems.
</td>
</tr>
</table>

&nbsp;

## Hyperledger Fabric Entities

#### _Chaincode_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Public Chaincode</b></td>
<td>
Chaincodes deployed by public transactions, these chaincodes can be invoked by any member of the network.
</td>
</tr>
<tr>
<td width="20%"><b>Confidential Chaincode</b></td>
<td>
Chaincodes deployed by confidential transactions, these chaincodes can only be invoked by validating members (Chain validators) of the network.
</td>
</tr>
<tr>
<td width="20%"><b>Access Controlled Chaincode</b></td>
<td>
Chaincodes deployed by confidential transactions that also embed the tokens of approved invokers. These invokers are also allowed to invoke confidential chaincodes even though they are not validators.
</td>
</tr>
</table>


#### _Ledger_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>Chaincode-State</b></td>
<td>
HPL provides state support; Chaincodes access internal state storage through state APIs. States are created and updated by transactions calling chaincode functions with state accessing logic.
</td>
</tr>
<tr>
<td width="20%"><b>Transaction List</b></td>
<td>
All processed transactions are kept in the ledger in their original form (with payload encrypted for confidential transactions), so that network participants can interrogate past transactions to which they have access permissions.
</td>
</tr>
<tr>
<td width="20%"><b>Ledger Hash</b></td>
<td>
A hash that captures the present snapshot of the ledger. It is a product of all validated transactions processed by the network since the genesis transaction.
</td>
</tr>
</table>


#### _Node_
---
<table border="0">
<col>
<col>
<tr>
<td width="20%"><b>DevOps Service</b></td>
<td>
The frontal module on a node that provides APIs for clients to interact with their node and chain network. This module is also responsible to construct transactions, and work with the membership service component to receive and store all types of certificates and encryption keys in its storage.
</td>
</tr>
<tr>
<td width="20%"><b>Node Service</b></td>
<td>
The main module on a node that is responsible to process transactions, deploy and execute chaincodes, maintain ledger data, and trigger the consensus process.
</td>
</tr>
<tr>
<td width="20%"><b>Consensus</b></td>
<td>
The default consensus algorithm of Hyperledger fabric is an implementation of PBFT.
</td>
</tr>
</table>
