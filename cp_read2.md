

## 4. 安全

这一节将讨论下面的图所展示的设置描述。特别的，系统是由下面这些实体构成的：成员管理基础架构，如从一个实体集合中区分出不同用户身份的职责（使用系统中任意形式的标识，如：信用卡，身份证），为这个用户注册开户，并生成必要的证书以便通过fabric成功的创建交易，部署或调用链码。

![figure-architecture](./images/sec-sec-arch.png)

 * Peers，它们被分为验证 peer 和非验证 peer。验证 peer（也被称为验证器）是为了规范并处理（有效性检查，执行并添加到区块链中）用户消息（交易）提交到网络上。非验证 peer（也被称为 peer）代表用户接受用户交易，并通过一些基本的有效性检查，然后把交易发送到它们附近的验证 peer。peer 维护一个最新的区块链副本，只是为了做验证，而不会执行交易(处理过程也被称为*交易验证*)。
 * 注册到我们的成员服务管理系统的终端用户是在获取被系统认定的*身份*的所有权之后，并将获取到的证书安装到客户端软件后，提交交易到系统。
 * 客户端软件，为了之后能完成注册到我们成员服务和提交交易到系统所需要安装在客户端的软件。
 * 在线钱包，用户信任的用来维护他们证书的实体，并独自根据用户的请求向网络提交交易。在线钱包配置在他们自己的客户端软件中。这个软件通常是轻量级的，它只需有对自己和自己的钱包的请求做授权。也有 peer 为一些用户扮演*在线钱包*的角色，在接下来的会话中，对在线钱包做了详细区分。

希望使用fabric的用户，通过提供之前所讨论的身份所有权，在成员管理系统中开立一个账户，新的链码被链码创建者（开发）以开发者的形式通过客户端软件部署交易的手段，公布到区块链网络中。这样的交易是第一次被 peer 或验证器接收到，并流传到整个验证器网络中，这个交易被区块链网络执行并找到自己的位置。用户同样可以通过调用交易调用一个已经部署了的链码

下一节提供了由商业目标所驱动的安全性需求的摘要。然后我们游览一下安全组件和它们的操作，以及如何设计来满足安全需求。

### 4.1 商业安全需求
这一节表述与fabric相关的商业安全需求。

**身份和角色管理相结合**

为了充分的支持实际业务的需求，有必要超越确保加密连续性来进行演进。一个可工作的B2B系统必须致力于证明/展示身份或其他属性来开展业务。商业交易和金融机构的消费交互需要明确的映射到账户的所有者。商务合同通常需要与特定的机构和/或拥有交易的其他特定性质的各方保证有从属关系。问责制和不可陷害性是身份管理作为此类系统关键组成部分的两个原因。


问责制意味着系统的用户，个人或公司，谁的胡作非为都可以追溯到并为自己的行为负责。在很多情况下，B2B系统需要它们的会员使用他们的身份（在某些形式）加入到系统中，以确保问责制的实行。问责制和不可陷害性都是B2B系统的核心安全需求，并且他们非常相关。B2B系统需要保证系统的诚实用户不会因为其他用户的交易而被指控。

此外，一个B2B系统需要具有可再生性和灵活性，以满足参与者角色和/或从属关系的改变。

**交易隐私.**

B2B系统对交易的隐私有着强烈的需求，如：允许系统的终端用户控制他与环境交互和共享的信息。例如：一家企业在交易型B2B系统上开展业务，要求它的交易得其他企业不可见，而他的行业合作伙伴无权分享机密信息。

在fabric中交易隐私是通过下面非授权用户的两个属性来实现的:

* 交易匿名，交易的所有者隐藏在一个被称为*匿名集*的组建中，在fabric中，它是用户的一个集合。

* 交易不可关联，同一用户的两个或多个交易不能被关联起来。


根据上下文，非授权用户可以是系统以外的任何人，或用户的子集。

交易隐私与B2B系统的两个或多个成员之间的保密协议的内容强烈相关。任何授权机制的匿名性和不可关联性需要在交易时考虑。

**通过身份管理协调交易隐私.**

就像文档之后描述的那样，这里所采用的方法用来协调身份管理与用户隐私，并使有竞争力的机构可以像下面一样在公共的区块链（用于内部和机构间交易）上快速的交易：

1. 为交易添加证书来实现“有权限的”区块链

2. 使用两层系统：

   1. 向登记的证颁发机构（CA）注册来获得(相对的) 静态登记证书 (ECerts)

   2. 通过交易CA获取能如实但伪匿名的代表登记用户的交易证书(TCerts).

3. 提供对系统中未授权会员隐藏交易内用的机制

**审计支持.** 

商业系统偶尔会受到审核。在这种情况下，将给予审计员检查某些交易，某组交易，系统中某些特定用户或系统自己的一些操作的手段。因此，任意与商业伙伴通过合同协议进行交易的系统都应该提供这样的能力。

### 4.2 使用成员管理的用户隐私
成员管理服务是由网络上管理用户身份和隐私的几个基础架构来组成的。这些服务验证用户的身份，在系统中注册用户，并为他/她提供所有作为可用、兼容的参数者来创建和/或调用交易所需要的证书。公开密钥体系（Public Key Infrastructure
，PKI）是一个基于不仅对公共网络上交换的数据的加密而且能确认对方身份的公共密钥加密的。PKI管理密钥和数字证书的生成，发布和废止。数字证书是用来建立用户证书，并对消息签名的。使用证书签名的消息保证信息不被篡改。典型的PKI有一个证书颁发机构（CA），一个登记机构（RA），一个证书数据库，一个证书的存储。RA是对用户进行身份验证，校验数据的合法性，提交凭据或其他证据来支持用户请求一个或多个人反映用户身份或其他属性的可信任机构。CA根据RA的建议为特定的用户发放根CA能直接或分级的认证的数字证书。另外，RA的面向用户的通信和尽职调查的责任可以看作CA的一部分。成员服务由下图展示的实体组成。整套PKI体系的引入加强了B2B系统的强度（如：超过比特币）。

![Figure 1](./images/sec-memserv-components.png)

*根证书颁发机构(根CA):* 它代表PKI体系中的信任锚。数字证书的验证遵循信任链。根CA是PKI层次结构中最上层的CA。

*登记机构(RA):* 它是一个可以确定想要加入到带权限区块链的用户的有效性和身份的可信实体。它是负责与用户外的带外通信来验证他/她的身份和作用。它是负责与用户进行频外通信来验证他/她的身份和角色。它创建登记时所需要的注册证书和根信任上的信息。

*注册证书颁发机构(ECA):* 负责给通过提供的注册凭证验证的用户颁发注册证书(ECerts)

*交易认证中心(TCA):* 负责给提供了有效注册证书的用户颁发交易证书(TCerts)

*TLS证书颁发机构(TLS-CA):* 负责签发允许用户访问其网络的TLS证书和凭证。它验证用户提供的包含该用户的特定信息的，用来签发TLS证书的，证书或证据。

*注册证书(ECerts)*
ECerts是长期证书。它们是颁发给所有角色的，如用户，非验证 peer，验证 peer。在给用户颁发的情况下，谁向区块链提交候选人申请谁就拥有TCerts（在下面讨论），ECerts有两种可能的结构和使用模式：

 * Model A: ECerts 包含所有者的身份/注册ID，并可以在交易中为TCert请求提供只用来对名义实体身份做验证。它们包含两个密钥对的公共部分：签名密钥对和加密/密钥协商密钥对。 ECerts是每个人都可以访问。

 * Model B: ECerts 包含所有者的身份/注册ID，并只为TCert请求提供名义实体的身份验证。它们包含一个签名密钥对的公共部分，即，签名验证公钥的公共部分。作为依赖方，ECerts最好只由TCA和审计人员访问。他们对交易是不可见的，因此（不像TCerts）签名密钥对不在这一层级发挥不可抵赖的作用。

*交易证书(TCerts)*
TCerts是每个交易的短期证书。它们是由TCA根据授权的用户请求颁发的。它们安全的给一个交易授权，并可以被配置为隐藏谁参与了交易或选择性地暴露这样身份注册ID这样的信息。他们包含签名密钥对的公共部分，并可以被配置为包含一个密钥协议的密钥对的公共部分。他们仅颁发给用户。它们唯一的关联到所有者，它们可以被配置为这个关联只有TCA知道知道（和授权审核员）。TCert可以配置成不携带用户的身份信息。它们使得用户不仅以匿名方式参与到系统中，而且阻止了交易之间的关联性。

然而，审计能力和问责制的要求TCA能够获取给定身份的TCert，或者获取指定TCert的所有者。有关TCerts如何在部署和调用交易中使用的详细信息参见4.3节，交易安全是在基础设施层面提供的。

TCerts可容纳的加密或密钥协议的公共密钥（以及数字签名的验证公钥）。
如果配备好TCert，那么就需要注册证书不能包含加密或密钥协议的公钥。

这样的密钥协议的公钥，Key_Agreement_TCertPub_Key，可以由交易认证机构（TCA）使用与生成Signature_Verification_TCertPub_Key同样的方法，使用TCertIndex + 1 而不是TCertIndex来作为索引个值来生成，其中TCertIndex是由TCA为了恢复而隐藏在TCert中的。

交易证书（TCert）的结构如下所述：
* TCertID – 交易证书ID（为了避免通过隐藏的注册ID发生的意外可关联性，最好由TCA随机生成）.
* Hidden Enrollment ID: AES_Encrypt<sub>K</sub>(enrollmentID), 其中密钥K = [HMAC(Pre-K, TCertID)]<sub>256位截断</sub>其中为每个K定义三个不同的密钥分配方案：(a)， (b) and (c)。
* Hidden Private Keys Extraction: AES_Encrypt<sub>TCertOwner_EncryptKey</sub>(TCertIndex || 已知的填充/校验检查向量) 其中||表示连接，其中各个批次具有被加到计数器的唯一（每个批次）的时间戳/随机偏移量（这个实现中初始化为1）来生成TCertIndex。该计数器可以通过每次增加2来适应TCA生成公钥，并由这两种类型的私钥的TCert所有者来恢复，如签名密钥对和密钥协商密钥对。
* Sign Verification Public Key – TCert签名验证的公共密钥。
* Key Agreement Public Key – TCert密钥协商的公钥。
* Validity period – 该交易证书可用于交易的外/外部签名的时间窗口。

这里至少有三种方式来配置考虑了隐藏注册ID域密钥的分配方案：

*(a)* Pre-K在注册期间发给用户，peer 和审计员，并对TCA和授权的审计员可用。它可能，例如由K<sub>chain</sub>派生（会在这个规范的后面描述）或为了链码的保密性使用独立的key(s)。

*(b)* Pre-K对验证器，TCA和授权的审计员可用。K是在验证器成功响应用户的查询交易（通过TLS）后可用给的。查询交易可以使用与调用交易相同的格式。对应下面的例1，如果查询用户又有部署交易的ACL中的一张TCert，那么就可以得到创建这个部署交易的用户的注册ID（enrollmentID）。对应下面的例2，如果查询所使用TCert的注册ID（enrollmentID）与部署交易中访问控制域的其中一个隶属关系/角色匹配，那么就可以得到创建这个部署交易的用户的注册ID（enrollmentID）。

*(c)*
Pre-K对TCA和授权的审计员可用。对于批量中的所有TCert，TCert特有的K可以和TCert一起分发给TCert的所有者（通过TLS）。这样就通过K的TCert所有者启用目标释放（TCert所有者的注册ID的可信通知）。这样的目标释放可以使用预定收件人的密钥协商公钥和/或PK<sub>chain</sub>其中SK<sub>chain</sub>就像规范的后面描述的那样对验证器可用。这样目标释放给其它合同的参与者也可以被纳入到这个交易或在频外完成。


如果TCert与上述的ECert模型A的结合使用，那么使用K不发送给TCert的所有者的方案（c）就足够了。
如果TCert与上述的ECert模型A的结合使用，那么TCert中的密钥协商的公钥域可能就不需要了。

交易认证中心(TCA)以批量的方式返回TCert，每个批量包含不是每个TCert都有的，但是和TCert的批量一起传递到客户端的KeyDF_Key(Key-Derivation-Function Key) （通用TLS）。KeyDF_Key允许TCert的拥有者派生出真正用于从AES_Encrypt<sub>TCertOwner_EncryptKey</sub>（TCertIndex || 已知的填充/校验检查向量）的TCertIndex恢复的TCertOwner_EncryptKey。

*TLS证书(TLS-Certs)*
TLS-Certs 是用于系统/组件到系统/组件间通讯所使用的证书。他们包含所有者的身份信息，使用是为了保证网络基本的安全。

成员管理的这个实现提供下面描述的基础功能：ECerts是没有到期/废止的；TCert的过期是由验证周期的时间窗口提供的。TCerts是没有废止的。ECA，TCA和TLS CA证书是自签名的，其中TLS CA提供信任锚点。


#### 4.2.1 用户/客户端注册过程

下面这个图高度概括了用户注册过程，它具有离线和在线阶段。

![Registration](./images/sec-registration-high-level.png)

*离线处理:* 在第一步中，每个用户/非验证 peer /验证 peer 有权在线下将较强的识别凭证（身份证明）到导入到注册机构（RA）。这需要在频外给RA提供为用户创建（存储）账号的证据凭证。第二步，RA返回对应的用户名/密码和信任锚点（这个实现中是TLS-CA Cert）给用户。如果用户访问了本地客户端，那么这是客户端可以以TLS-CA证书作为信任锚点来提供安全保障的一种方法。

*在线阶段:* 第三步，用户连接客户端来请求注册到系统中。用户发送它的用户名和密码给客户端。客户端代表用户发送请求给PKI框架。第四步，接受包，第五步，包含其中一些对应于由客户端私有/秘密密钥的若干证书。一旦客户端验证包中所有加密材料是正确/有效的，他就把证书存储在本地并通知用户。这时候用户注册就完成了。

![Figure 4](./images/sec-registration-detailed.png)

图4展示了注册的详细过程。PKI框架具有RA， ECA，
TCA和TLS-CA这些实体。第一步只收，RA调用“AddEntry”函数为它的数据库输入（用户名/密码）。这时候用户已正式注册到系统数据库中。客户端需要TLS-CA证书（当作信任锚点）来验证与服务器之间的TLS握手是正确的。第四步，客户端发送包含注册公钥和像用户名，密码这样的附加身份信息的注册请求到ECA（通过TLS备案层协议）。ECA验证这个用户是否真实存在于数据库。一旦它确认用户有权限提交他/她的注册公钥，那么ECA就会验证它。这个注册信息是一次性的。ECA会更新它的数据库来标识这条注册信息（用户名，密码）不能再被使用。ECA构造，签名并送回一张包含用户注册公钥的（步骤5）注册证书（ECert）。它同样会发送将来会用到（客户端需要向TCA证明他/她的ECert使用争取的ECA创建的）的ECA证书（ECA-Cert)）。（尽管ECA-Cert在最初的实现中是自签名的，TCA，TLS-CA和ECA是共址）第六步，客户端验证ECert中的公钥是最初由客户端提交的（即ECA没有作弊）。它同样验证ECert中的所有期望的信息存在且形式正确。

同样的，在第七步，客户端发送包含它的公钥和身份的注册信息到TLS-CA。TLS-CA验证该用户在数据库中真实存在。TLS-CA生成，签名包含用户TLS公钥的一张TLS-Cert（步骤8）。TLS-CA发送TLS-Cert和它的证书（TLS-CA Cert）。第九步类似于第六步，客户端验证TLS Cert中的公钥是最初由客户端提交的，TLS Cert中的信息是完整且形式正确。在第十步，客户端在本地存储中保存这两张证书的所有证书。这时候用户就注册完成了。


在这个版本的实现中验证器的注册过程和 peer 的是一样的。尽管，不同的实现可能使得验证器直接通过在线过程来注册。

![Figure 5](./images/sec-request-tcerts-deployment.png)
![Figure 6](./images/sec-request-tcerts-invocation.png)

*客户端:* 请求TCert批量需要包含（另外计数），ECert和使用ECert私钥签名的请求（其中ECert的私钥使用本地存储获取的）

*TCA为批量生成TCerts:* 生成密钥派生函数的密钥，KeyDF_Key, 当作HMAC(TCA_KDF_Key, EnrollPub_Key). 为每张TCert生成公钥(使用TCertPub_Key = EnrollPub_Key + ExpansionValue G, 其中384位的ExpansionValue = HMAC(Expansion_Key, TCertIndex) 和384位的Expansion_Key = HMAC(KeyDF_Key, “2”)). 生成每个AES_Encrypt<sub>TCertOwner_EncryptKey</sub>(TCertIndex || 已知的填充/校验检查向量)， 其中|| 表示连接，且TCertOwner_EncryptKey被当作[HMAC(KeyDF_Key,
“1”)]派生<sub>256位截断</sub>.

*客户端:* 为部署，调用和查询，根据TCert来生成TCert的私钥：KeyDF_Key和ECert的私钥需要从本地存储中获取。KeyDF_Key是用来派生被当作[HMAC(KeyDF_Key, “1”)]<sub>256位截断</sub>的TCertOwner_EncryptKey；TCertOwner_EncryptKey是用来对TCert中的 AES_Encrypt<sub>TCertOwner_EncryptKey</sub>(TCertIndex ||
已知的填充/校验检查向量)域解密的；TCertIndex是用来派生TCert的私钥的： TCertPriv_Key = (EnrollPriv_Key + ExpansionValue)模n，其中384位的ExpansionValue = HMAC(Expansion_Key, TCertIndex)，384位的Expansion_Key = HMAC(KeyDF_Key, “2”)。

#### 4.2.2 过期和废止证书
实际是支持交易证书过期的。一张交易证书能使用的时间窗是由‘validity period’标识的。实现过期支持的挑战在于系统的分布式特性。也就是说，所有验证实体必须共享相同的信息；即，与交易相关的有效期验证需要保证一致性。为了保证有效期的验证在所有的验证器间保持一致，有效期标识这一概念被引入。这个标识扮演着逻辑时钟，使得系统可以唯一识别有效期。在创世纪时，链的“当前有效期”由TCA初始化。至关重要的是，此有效期标识符给出随时间单调增加的值，这使得它规定了有效期间总次序。

对于指定类型的交易，系统交易有效周期标识是用来一起向区块链公布有效期满的。系统交易涉及已经在创世纪块被定义和作为基础设施的一部分的合同。有效周期标识是由TCA周期性的调用链码来更新的。注意，只有TCA允许更新有效期。TCA通过给定义了有效期区间的‘not-before’和‘not-after’这两个域设置合适的整数值来为每个交易证书设置有效期。

TCert过期:
在处理TCert时，验证器从状态表中读取与总账中的‘current validity period’相关的值来验证与交易相关的外部证书目前是否有效。状态表中的当前值需要落在TCert的‘not-before’和‘not-after’这两个子域所定义的区间中。如果满足，那么验证器就继续处理交易。如果当前值没有在这个区间中，那么TCert已经过期或还没生效，那么验证器就停止处理交易。

ECert过期:
注册证书与交易证书具有不同的有效期长度。

废止是由证书废止列表（CRLs）来支持的，CRLs鉴定废止的证书。CRLs的改变，增量的差异通过区块链来公布

### 4.3 基础设施层面提供的交易安全

fabric中的交易是通过提交用户-消息来引入到总账中的。就像之前章节讨论的那样，这些信息具有指定的结构，且允许用户部署新的链码，调用已经存在的链码，或查询已经存在的链码的状态。因此交易的方式被规范，公布和处理在整个系统提供的隐私和安全中起着重要的作用。

一方面我们的成员服务通过检查交易是由系统的有效用户创建的来提供验证交易的手段，为了把用户身份和交易撇清，但是在特定条件下又需要追踪特定个体的交易（执法，审计）。也就是说，成员服务提供结合用户隐私与问责制和不可抵赖性来提供交易认证机制。

另一方面，fabric的成员服务不能单独提供完整的用户活动隐私。首先fabric提供完整的隐私保护条款，隐私保护认证机制需要通过交易保密协同。很明显，如果认为链码的内容可能会泄漏创建者的信息，那么这就打破了链码创建者的隐私要求。第一小节讨论交易的保密性。

<!-- @Binh, @Frank: PLEASE REVIEW THIS PARAGRAPH -->
<!-- Edited by joshhus ... April 6, 2016 -->
为链码的调用强制访问控制是一个重要的安全要求。fabric暴露给应用程序（例如，链码创建者）这意味着当应用利用fabric的成员服务是，需要应用自己调用访问控制。4.4节详细阐述了这一点。

<!--Enforcing access control on the invocation of chaincodes is another requirement associated
to the security of chaincodes. Though for this one can leverage authentication mechanisms
of membership services, one would need to design invocation ACLs and perform their
validation in a way that non-authorized parties cannot link multiple invocations of
the same chaincode by the same user. Subection 5.2.2 elaborates on this.-->

重放攻击是链码安全的另一个重要方面，作为恶意用户可能复制一个之前的，已经加入到区块链中的交易，并向网络重放它来篡改它的操作。这是第4.3.3节的话题。

本节的其余部分介绍了基础设施中的安全机制是如何纳入到交易的生命周期中，并分别详细介绍每一个安全机制。

#### 4.3.1 交易安全的生命周期
交易在客户端创建。客户端可以是普通的客户端，或更专用的应用，即，通过区块链处理（服务器）或调用（客户端）具体链码的软件部分。这样的应用是建立在平台（客户端）上的，并在4.4节中详细介绍。

新链码的开发者可以通过这些fabric的基础设施来新部署交易：
* 希望交易符合保密/安全的版本和类型
* 希望访问部分链码的用户有适当的（读）访问权限
<!-- (read-access code/state/activity, invocation-access) -->
* 链码规范
* 代码元数据，包含的信息需要在链码执行时传递给它（即，配置参数），和
* 附加在交易结构上的并只在应用部署链码时使用的交易元数据

具有保密限制的链码的调用和查询交易都是用类似的方式创建。交易者提供需要执行的链码的标识，要调用的函数的名称及其参数。可选的，调用者可以传递在链码执行的时候所需要提供的代码调用元数据给交易创建函数。交易元数据是调用者的应用程序或调用者本身为了它自己的目的所使用的另外一个域。

最后，交易在客户端，通过它们的创建者的证书签名，并发送给验证器网络。
验证器接受私密交易，并通过下列阶段传递它们：
* *预验证*阶段，验证器根据根证书颁发机构来验证交易证书，验证交易（静态的）中包含交易证书签名，并验证交易是否为重放（参见，下面关于重放攻击的详细信息）。
* *共识*阶段， 验证器把这笔交易加入到交易的全序列表中（最终包含在总账中）
* *预执行*阶段， 验证交易/注册证书是否在当前的有效期中
  解密交易（如果交易是加密的），并验证交易明文的形式正确（即，符合调用访问控制，包含TCert形式正确）
  在当前处理块的事务中，也执行了简单的重放攻击检查。
* *执行*阶段， (解密的) 链码和相关的代码元数据被传递给容器，并执行。
* *提交* 阶段， (解密的)更新的链码的状态和交易本身被提交到总账中。


#### 4.3.2 交易保密性

在开发人员的要求下，交易机密性要求链码的原文，即代码，描述，是不能被未授权的实体（即，未被开发人员授权的用户或 peer）访问或推导（assuming a computational attacker）出来。对于后者，*部署*和*调用*交易的内容始终被隐藏对链码的保密需求是至关重要的。本着同样的精神，未授权方，不应该能联系链码（调用交易）与链码本身（部署交易）之间的调用关系或他们之间的调用。


任何候选的解决方案的附加要求是，满足并支持底层的成员服务的隐私和安全规定。此外，在fabric中他不应该阻止任何链码函数的调用访问控制，或在应用上实现强制的访问控制机制(参看4.4小结)。

下面提供了以用户的粒度来设置的交易机密性机制的规范。最后小结提供了一些如何在验证器的层次来扩展这个功能的方针。当前版本所支持的特性和他的安全条款可以在4.7节中找到。


目标是达到允许任意的子集实体被允许或限制访问链码的下面所展示的部分：
1. 链码函数头，即，包含在链码中函数的原型
<!--access control lists, -->
<!--and their respective list of (anonymous) identifiers of users who should be
   able to invoke each function-->
2. 链码[调用&] 状态，即， 当一个或多个函数被调用时，连续更新的特定链码的状态。
3. 所有上面所说的

注意，这样的设计为应用提供利用fabric的成员管理基础设施和公钥基础设施来建立自己的访问控制策略和执法机制的能力。

##### 4.3.2.1 针对用户的保密

为了支持细粒度的保密控制，即，为链码创建者定义的用户的子集，限制链码的明文读权限，一条绑定到单个长周期的加密密钥对的链（PK<sub>chain</sub>, SK<sub>chain</sub>）。

尽管这个密钥对的初始化是通过每条链的PKI来存储和维护的，在之后的版本中，这个限制将会去除。链（和相关的密钥对）可以由任意带有*特定*（管理）权限的用户通过区块链来触发（参看4.3.2.2小节）

**搭建**. 在注册阶段， 用户获取（像之前一样）一张注册证书，为用户u<sub>i</sub>标记为Cert<sub>u<sub>i</sub></sub>，其中每个验证器v<sub>j</sub>获取的注册证书标记为Cert<sub>v<sub>j</sub></sub>。注册会给用户或验证器发放下面这些证书：

1. 用户：

   a. 声明并授予自己签名密钥对(spk<sub>u</sub>, ssk<sub>u</sub>)

   b. 申明并授予他们加密密钥对(epk<sub>u</sub>, esk<sub>u</sub>)，

   c. 获取链PK<sub>chain</sub>的加密（公共）密钥

2. 验证器:

   a. 声明并授予他们签名密钥对(spk<sub>v</sub>, ssk<sub>v</sub>)

   b. 申明并授予他们加密密钥对 (epk<sub>v</sub>, esk<sub>v</sub>)，

   c. 获取链SK<sub>chain</sub>的解密（秘密）密钥

因此，注册证书包含两个密钥对的公共部分：
* 一个签名密钥对[为验证器标记为(spk<sub>v<sub>j</sub></sub>,ssk<sub>v<sub>j</sub></sub>)，为用户标记为(spk<sub>u<sub>i</sub></sub>, ssk<sub>u<sub>i</sub></sub>)] 和
* 一个加密密钥对[为验证器标记为(epk<sub>v<sub>j</sub></sub>,esk<sub>v<sub>j</sub></sub>)，为用户标记为(epk<sub>u<sub>i</sub></sub>, esk<sub>u<sub>i</sub></sub>)]

链，验证器和用户注册公钥是所有人都可以访问的。

除了注册证书，用户希望通过交易证书的方式匿名的参与到交易中。用户的简单交易证书u<sub>i</sub>被标记为TCert<sub>u<sub>i</sub></sub>。交易证书包含的签名密钥对的公共部分标记为(tpk<sub>u<sub>i</sub></sub>,tsk<sub>u<sub>i</sub></sub>)。

下面的章节概括性的描述了如何以用户粒度的方式提供访问控制。

**部署交易的结构.**
下图描绘了典型的启用了保密性的部署交易的结构。

![FirstRelease-deploy](./images/sec-usrconf-deploy.png)

注意，部署交易由几部分组成：
* *基本信息*部分: 包含交易管理员的详细信息，即这个交易对应于哪个链（链接的），交易的类型（设置''deplTrans''），实现的保密协议的版本号，创建者的身份（由注册证书的交易证书来表达），和主要为了防止重放攻击的Nonce。
* *代码信息*部分: 包含链码的源码，函数头信息。就像下图所展示的那样，有一个对称密钥(K<sub>C</sub>)用于链码的源代码，另一个对称密钥(K<sub>H</sub>)用于函数原型。链码的创建者会对明文代码做签名，使得信函不能脱离交易，也不能被其他东西替代。
* *链验证器*部分: 为了(i)解密链码的源码(K<sub>C</sub>),(ii)解密函数头，和(iii)当链码根据(K<sub>S</sub>)调用时加密状态。尤其是链码的创建者为他部署的链码生产加密密钥对(PK<sub>C</sub>, SK<sub>C</sub>)。它然后使用PK<sub>C</sub>加密所有与链码相关的密钥：
<center> [(''code'',K<sub>C</sub>) ,(''headr'',K<sub>H</sub>),(''code-state'',K<sub>S</sub>), Sig<sub>TCert<sub>u<sub>c</sub></sub></sub>(\*)]<sub>PK<sub>c</sub></sub>, </center>并把
where appropriate key material is passed to the
  In particular, the chain-code creator
  generates an encryption key-pair for the chain-code it deploys
  (PK<sub>C</sub>, SK<sub>C</sub>). It then uses PK<sub>C</sub>
  to encrypt all the keys associated to the chain-code:
  <center> [(''code'',K<sub>C</sub>) ,(''headr'',K<sub>H</sub>),(''code-state'',K<sub>S</sub>), Sig<sub>TCert<sub>u<sub>c</sub></sub></sub>(\*)]<sub>PK<sub>c</sub></sub>, </center>私钥SK<sub>C</sub>通过链指定的公钥：
  <center>[(''chaincode'',SK<sub>C</sub>), Sig<sub>TCert<sub>u<sub>c</sub></sub></sub>(*)]<sub>PK<sub>chain</sub></sub>.</center>
传递给验证器。
* *合同用户*部分: 合同用户的公共密钥，即具有部分链码读权限的用户，根据他们的访问权限加密密钥：

  1. SK<sub>c</sub>使得用户能读取与这段链码相关的任意信息（调用，状态，等）

  2. K<sub>C</sub>使用户只能读取合同代码

  3. K<sub>H</sub> 使用户只能读取头信息

  4. K<sub>S</sub>使用户只能读取与合同相关的状态

  最后给用户发放一个合同的公钥PK<sub>c</sub>，使得他们可以根据合同加密信息，从而验证器(or any in possession of SK<sub>c</sub>)可以读取它。每个合同用户的交易证书被添加到交易中，并跟随在用户信息之后。这可以使得用户可以很容易的搜索到有他们参与的交易。注意，为了信函可以在本地不保存任何状态的情况下也能通过分析总账来获取这笔交易，部署交易也会添加信息到链码创建者u<sub>c</sub>。

整个交易由链码的创建者的证书签名，即：由后者决定使用注册还是交易证书。
两个值得注意的要点：
* 交易中的信息是以加密的方式存储的，即，code-functions，
* code-hdrs在使用TCert加密整个交易之前会用想用的TCert签名，或使用不同的TCert或ECert（如果交易的部署需要带上用户的身份。一个绑定到底层交易的载体需要包含在签名信息中，即，交易的TCert的哈希是签名的，因此mix\&match攻击是不可能的。我们在4.4节中详细讨论这样的攻击，在这种情况下，攻击者不能从他看到的交易中分离出对应的密文，即，代码信息，并在另一个交易中使用它。很明显，这样会打乱整个系统的操作，链码首先有用户A创建，现在还属于恶意用户B（可能没有权限读取它）
* 为了给用户提供交叉验证的能力，会给他们访问正确密钥的权限，即给其他用户相同的密钥，使用密钥K对交易加密成密文，伴随着对K的承诺，而这一承诺值开放给所有在合同中有权访问K的用户，和链验证器。
  <!-- @adecaro: please REVIEW this! -->
  在这种情况下，谁有权访问该密钥，谁就可以验证密钥是否正确传递给它。为了避免混乱，这部分在上图中省略了。


**调用交易的结构.**
下图结构化描述了，交易调用链码会触发使用用户指定的参数来执行链码中的函数

![FirstRelease-deploy](./images/sec-usrconf-invoke.png)

调用交易和部署交易一样由一个*基本信息*， *代码信息*，*链验证器*和一个*合同用户*，并使用一张调用者的交易证书对所有进行签名。

- 基本信息 与部署交易中对应部分遵循相同的结构。唯一的不同是交易类型被设置为''InvocTx''，链码的标识符或名字是通过链指定的加密（公共）密钥来加密的。

- 代码信息 部署交易中的对应结构具有相同展现。在部署交易中作为代码有效载荷，现在由函数调用明细（调用函数的名字，对应的参数），由应用提供的代码元数据和交易创建者（调用者
  u）的证书，TCert<sub>u</sub>。在部署交易的情况下，代码有效载荷和是通过调用者u的交易证书TCert<sub>u</sub>签名的。在部署交易的情况下，代码元数据，交易数据是由应用提供来使得信函可以实现他自己的访问控制机制和角色（详见4.4节）。

- 最后，合同用户和链验证器部分提供密钥和有效荷载是使用调用者的密钥加密的，并分别链加密密钥。在收到此类交易，验证器解密 [code-name]<sub>PK<sub>chain</sub></sub>使用链指定的密钥SK<sub>chain</sub> ，并获取被调用的链码身份。给定的信封，验证器从本地的获取链码的解密密钥SK<sub>c</sub>，并使用他来解密链验证器的信息，使用对称密钥
  K<sub>I</sub>对调用交易的有效荷载加密。给定信函，验证器解密代码信息，并使用指定的参数和附加的代码元数据（参看4.4节的代码元数据详细信息）执行链码。当链码执行后，链码的状态可能就更新了。
  加密所使用的状态特定的密钥K<sub>s</sub>在链码部署的时候就定义了。尤其是，在当前版本中K<sub>s</sub> 和K<sub>iTx</sub>被设计成一样的（参看4.7节）。

**查询交易的结构.**
查询交易和调用交易具有同样的格式。唯一的区别是查询交易对链码的状态没有影响，且不需要在执行完成之后获取（解密的）并/或更新（加密的）状态。

##### 4.3.2.2 针对验证器的保密
这节阐述了如何处理当前链中的不同（或子集）集合的验证器下的一些交易的执行。本节中抑制IP限制，将在接下的几个星期中进行扩展。


#### 4.3.3 防重放攻击
在重放攻击中，攻击者“重放”他在网络上“窃听”或在区块链''看到''的消息
由于这样会导致整个验证实体重做计算密集型的动作（链码调用）和/或影响对应的链码的状态，同时它在攻击侧又只需要很少或没有资源，所以重放攻击在这里是一个比较大的问题。如果交易是一个支付交易，那么问题就更大了，重放可能会导致在不需要付款人的参与下，多于一次的支付。
当前系统使用以下方式来防止重放攻击：
* 在系统中记录交易的哈希。这个方法要求验证器为每个交易维护一个哈希日志，发布到网络上，并把每个新来的交易与本地存储的交易记录做对比。很明显这样的方法是不能扩展到大网络的，也很容易导致验证器花了比真正做交易还多的时间在检查交易是不是重放上。
* 利用每个用户身份维护的状态（Ethereum）.Ethereum保存一些状态，即，对每个身份/伪匿名维护他们自己的计数器（初始化为1）。每次用户使用他的身份/伪匿名发送交易是，他都把他的本地计数器加一，并把结果加入到交易中。交易随后使用用户的身份签名，并发送到网络上。当收到交易时，验证器检查计数器并与本地存储的做比较；如果值是一样的，那就增加这个身份在本地的计数器，并接受交易。否则把交易当作无效或重放的而拒绝掉。尽管这样的方法在有限个用户身份/伪匿名(即，不太多)下工作良好。它最终在用户每次交易都使用不同的标识（交易证书），用户的伪匿名与交易数量成正比时无法扩展。

其他资产管理系统，即比特币，虽然没有直接处理重放攻击，但它防止了重放。在管理（数字）资产的系统中，状态是基于每个资产来维护的，即，验证器只保存谁拥有什么的记录。因为交易的重放根据协议（因为只能由资产/硬币旧的所有者衍生出来）可以直接认为无效的，所以防重放攻击是这种方式的直接结果。尽管这合适资产管理系统，但是这并不表示在更一般的资产管理中需要比特币系统。

在fabric中，防重放攻击使用混合方法。
这就是，用户在交易中添加一个依赖于交易是匿名（通过交易证书签名）或不匿名（通过长期的注册证书签名）来生成的nonce。更具体的：
* 用户通过注册证书来提交的交易需要包含nonce。其中nonce是在之前使用同一证书的交易中的nonce函数（即计数器或哈希）。包含在每张注册证书的第一次交易中的nonce可以是系统预定义的（即，包含在创始块中）或由用户指定。在第一种情况中，创世区块需要包含nonceall，即，一个固定的数字和nonce被用户与身份IDA一起用来为他的第一笔注册证书签名的交易将会
<center>nonce<sub>round<sub>0</sub>IDA</sub> <- hash(IDA, nonce<sub>all</sub>),</center>
其中IDA出现在注册证书中。从该点之后的这个用户关于注册证书的连续交易需要包含下面这样的nonce
  <center>nonce<sub>round<sub>i</sub>IDA</sub> <- hash(nonce<sub>round<sub>{i-1}</sub>IDA</sub>),</center>
这表示第i次交易的nonce需要使用这样证书第{i-1}次交易的nonce的哈希。验证器持续处理他们收到的只要其满足上述条件的交易。一旦交易格式验证成功，验证器就使用nonce更新他们的数据库。

  **存储开销**:

  1. 在用户侧：只有最近使用的nonce

  2. 在验证器侧: O(n)， 其中n是用户的数量
* 用户使用交易证书提交的交易需要包含一个随机的nonce，这样就保证两个交易不会产生同样的哈希。如果交易证书没有过期的话，验证器就向本地数据库存储这笔交易的哈希。为了防止存储大量的哈希，交易证书的有效期被利用。特别是验证器为当前或未来有效周期来维护一个接受交易哈希的更新记录。

  **存储开销** (这里只影响验证器):  O(m)， 其中m近似于有效期内的交易和对应的有效标识的数量（见下方）

### 4.4 应用的访问控制功能

应用是运行在区块链客户端软件上的一个具有特定功能的软件。如餐桌预订。应用软件有一个开发版本，使后者可以生成和管理一些这个应用所服务的行业所需要的链码，而且，客户端版本可以允许应用的终端用户调用这些链码。应用可以选择是否对终端用户屏蔽区块链。

本节介绍应用中如何使用链码来实现自己的访问控制策略，并提供如何使用成员服务来达到相同的目的。

这个报告可以根据应用分为调用访问控制，和读取访问控制。


#### 4.4.1 调用访问控制
为了允许应用在应用层安全的实现自己的访问问控制，fabric需要提供特定的支持。在下面的章节中，我们详细的说明的fabric为了达到这个目的而给应用提供的工具，并为应用如何来使用它们使得后者能安全的执行访问控制提供方针。

**来自基础设施的支持.**
把链码的创建者标记为 *u<sub>c</sub>*，为了安全的实现应用层自己的调用访问控制，fabric必须需要提供特定的支持。
更具体的，fabric层提供下面的访问能力：

1. 客户端-应用可以请求fabric使用指定的客户端拥有的交易证书或注册证书来签名和验证任何消息； 这是由Certificate Handler interface来处理的。

2. 客户端-应用可以请求fabric一个*绑定*将身份验证数据绑定到底层的交易传输的应用程序；这是由Certificate Handler interface来处理的。

3. 为了支持交易格式，允许指定被传递给链码在部署和调用时间的应用的元数据；后者被标记为代码元数据。

**Certificate Handler**接口允许使用底层证书的密钥对来对任意消息进行签名和验证。证书可以是TCert或ECert。

```
// CertificateHandler exposes methods to deal with an ECert/TCert
type CertificateHandler 
```
**Transaction Handler**借口允许创建交易和访问可利用的底层*绑定*来链接应用数据到底层交易。绑定是在网络传输协议引入的概念（参见，https://tools.ietf.org/html/rfc5056）记作*通道绑定*，*允许应用在网络层两端的建立安全通道，与在高层的认证绑定和在低层是一样的。
这允许应用代理保护低层会话，这具有很多性能优势。*
交易绑定提供识别fabric层次交易的身份，这就是应用数据要加入到总账的容器。

```
// TransactionHandler represents a single transaction that can be uniquely determined or identified by the output of the GetBinding method.
// This transaction is linked to a single Certificate (TCert or ECert).
type TransactionHandler 
}
```
对于版本1，*绑定*由*hash*（TCert， Nonce）组成，其中TCert是给整个交易签名的交易证书，Nonce是交易所使用的nonce。

**Client**接口更通用，提供之前接口实例的手段。

```
type Client 

}
```

为了向链码调用控制提供应用级别的的访问控制列表，fabric的交易和链码指定的格式需要存储在应用特定元数据的额外的域。
这个域在图1中通过元数据展示出来。这个域的内容是由应用在交易创建的时候决定的。fabric成把它当作非结构化的字节流。

为了帮助链码执行，在链码调用的时候，验证器为链码提供额外信息，如元数据和绑定。

**应用调用访问控制.**
这一节描述应用如何使用fabric提供的手段在它的链码函数上实现它自己的访问控制。
这里考虑的情况包括：

1. **C**: 是只包含一个函数的链码，如，被成为*hello*

2. **u<sub>c</sub>**: 是**C**的部署;

3. **u<sub>i</sub>**: 是被授权调用**C**的用户。用户u<sub>c</sub>希望只有u<sub>i</sub>可以调用函数*hello*

*链码部署:* 在部署的时候，u<sub>c</sub>具有被部署交易元数据的完全控制权，可硬存储一个ACL的列表（每个函数一个），或一个应用所需要的角色的列表。存储在ACL中的格式取决于部署的交易，链码需要在执行时解析元数据。
为了定义每个列表/角色，u<sub>c</sub>可以使用u<sub>i</sub>的任意TCerts/Certs（或，如果可接受，其他分配了权限或角色的用户）。把它记作TCert<sub>u<sub>i</sub></sub>。
开发者和授权用户之间的TCerts和 Certs交换实在频外渠道进行的。

假设应用的u<sub>c</sub>需要调用 *hello*函数，某个消息*M*就被授权给授权的调用者（在我们的例子中是u<sub>i</sub>）。
可以区分为以下两种情况：

1. *M*是链码的其中一个函数参数;

2. *M*是调用信息本事，如函数名，函数参数。

*链码调用:*
为了调用C， u<sub>i</sub>的应用需要使用TCert/ECert对*M*签名，用来识别u<sub>i</sub>在相关的部署交易的元数据中的参与身份。即，TCert<sub>u<sub>i</sub></sub>。更具体的，u<sub>i</sub>的客户端应用做一下步骤：

1. Cert<sub>u<sub>i</sub></sub>， *cHandler*获取CertificateHandler

2. 获取新的TransactionHandler来执行交易， *txHandler*相对与他的下一个有效的TCert或他的ECert

3. 通过调用*txHandler.getBinding()*来得到*txHandler*的绑定

4. 通过调用*cHandler.Sign('*M* || txBinding')*来对*'*M* || txBinding'*签名， *sigma*是签名函数的输出。

5. 通过调用来发布一个新的执行交易，*txHandler.NewChaincodeExecute(...)*. 现在， *sigma*可以以一个传递给函数（情形1）参数或payload的元数据段的一部分(情形2)的身份包含在交易中。

*链码处理:*
验证器， 从u<sub>i</sub>处接受到的执行交易将提供以下信息：

1. 执行交易的*绑定*，他可以在验证端独立的执行；

2. 执行交易的*元数据*(交易中的代码元数据);

3. 部署交易的*元数据*(对应部署交易的代码元数据组建).

注意*sigma*是被调用函数参数的一部分，或者是存储在调用交易的代码元数据内部的（被客户端应用合理的格式化）。
应用ACL包含在代码元数据段中，在执行时同样被传递给链码。
函数*hello*负责检查*sigma*的确是通过TCert<sub>u<sub>i</sub></sub>在'*M* || *txBinding'*上的有效签名。

#### 4.4.2 读访问控制
这节描述fabric基础设施如何支持应用在用户层面执行它自己的读访问控制策略。和调用访问控制的情况一样，第一部分描述了可以被应用程序为实现此目的利用的基础设施的功能，接下来介绍应用使用这些工具的方法。

为了说明这个问题，我们使用和指点一样的例子，即：

1. **C**: 是只包含一个函数的链码，如，被成为*hello*

2. **u<sub>A</sub>**: 是**C**的部署者，也被成为应用;

3. **u<sub>r</sub>**: 是被授权调用**C**的用户。用户u<sub>A</sub>希望只有u<sub>r</sub>可以读取函数*hello*

**来自基础设施的支持.**
为了让**u<sub>A</sub>**在应用层安全的实现自己的读取访问控制我们的基础设施需要像下面描述的那样来支持代码的部署和调用交易格式。

![SecRelease-RACappDepl title="Deployment transaction format supporting application-level read access control."](./images/sec-usrconf-deploy-interm.png)

![SecRelease-RACappInv title="Invocation transaction format supporting application-level read access control."](./images/sec-usrconf-invoke-interm.png)

更具体的fabric层需要提供下面这些功能：

1. 为数据只能通过验证（基础设施）侧解密，提供最低限度的加密功能；这意味着基础设施在我们的未来版本中应该更倾向于使用非对称加密方案来加密交易。更具体的，在链中使用在上图中标记为 K<sub>chain</sub> 的非对称密钥对。具体参看<a href="./txconf.md">交易保密</a>小节

2. 客户端-引用可以请求基础设施，基于客户端侧使用特定的公共加密密钥或客户端的长期解密密钥来加密/解密信息。

3. 交易格式提供应用存储额外的交易元数据的能力，这些元数据可以在后者请求后传递给客户端应用。交易元数据相对于代码元数据，在执行时是没有加密或传递给链码的。因为验证器是不负责检查他们的有效性的，所以把它们当作字节列表。

**应用读访问控制.**
应用可以请求并获取访问用户**u<sub>r</sub>**的公共加密密钥，我们把它标记为**PK<sub>u<sub>r</sub></sub>**。可选的，**u<sub>r</sub>** 可能提供 **u<sub>A</sub>**的一张证书给应用，使应用可以利用，标记为TCert<sub>u<sub>r</sub></sub>。如：为了跟踪用户关于应用的链码的交易。TCert<sub>u<sub>r</sub></sub>和PK<sub>u<sub>r</sub></sub>实在频外渠道交换的。

部署时，应用 **u<sub>A</sub>**执行下面步骤：

1. 使用底层基础设施来加密**C**的信息，应用使用PK<sub>u<sub>r</sub></sub>来访问**u<sub>r</sub>**。标记C<sub>u<sub>r</sub></sub>为得到的密文。

2. (可选) C<sub>u<sub>r</sub></sub>可以和TCert<sub>u<sub>r</sub></sub>连接

3. 保密交易被构造为''Tx-metadata''来传递

在调用的时候，在 u<sub>r</sub>节点上的客户端-应用可以获取部署交易来得到**C**的内容。
这只需要得到相关联的部署交易的 **tx-metadata**域，并触发区块链基础设施客户端为C<sub>u<sub>r</sub></sub>提供的解密函数。注意，为u<sub>r</sub>正确加密**C**是应用的责任。
此外，使用**tx-metadata**域可以一般性的满足应用需求。即，调用者可以利用调用交易的同一域来传递信息给应用的开发者。

<font color="red">**Important Note:** </font>
要注意的是验证器在整个执行链码过程中**不提供**任何解密预测。
对payload解密由基础设施自己负责（以及它附近的代码元数据域）。并提供他们给部署/执行的容器。

### 4.5 在线钱包服务
这一节描述了钱包服务的安全设计，这是一个用户可以注册，移动他们的秘密材料到，办理交易的节点。
由于钱包服务是一个用户秘密材料所有权的服务，所以要杜绝没有安全授权机制的恶意钱包服务可以成功模拟客户。
因此，我们强调的是，设计一种**值得信赖**的，只有在代表用户的客户端同意的情况下，钱包服务才能执行交易。
这里有两种终端用户注册到在线钱包服务的情况：

1. 当用户注册到注册机构并获得他/她的 `<enrollID, enrollPWD>`，但是没有安装客户端来触发完整的注册过程。

2. 用户已经安装客户端并完成注册阶段

首先，用户与在线钱包服务交互，允许他们进行身份验证的钱包服务发布证书。即用户给定用户名和密码，其中用户名在成员服务中识别用户，标记为AccPub，密码是关联的秘密，标记为AccSec，这是由用户和服务分享的。

为了通过在线钱包服务注册，用户必须提供下面请求对象到钱包服务：


    AccountRequest
OBCSecCtx指向用户证书，它依赖于注册过程中的阶段。可以是用户的注册ID和密码，`<enrollID, enrollPWD>` 或他的注册证书和关联的密钥(ECert<sub>u</sub>, sk<sub>u</sub>),  其中 sk<sub>u</sub>是用户签名和解密密钥的简化标记。
OBCSecCtx需要给在线钱包服务必要的信息来注册用户和发布需要的TCerts。

对于后续的请求，用户u需要提供给钱包服务的请求与下面这个格式类似。

     TransactionRequest 

这里，TxDetails指向在线服务代表用户构造交易所需要的信息，如类型，和用户指定的交易的内容。

AccSecProof<sub>u</sub>是对应请求中剩下的域的使用共享密钥的HMAC。
Nonce-based方法与我们在fabric中一样可以防止重放攻击。

TLS连接可以用在每种服务器端认证的情况，在网络层对请求加密（保密，防止重放攻击，等）


### 4.6 网络安全(TLS)
TLS CA需要给（非验证）peer，验证器，和单独的客户端（或具有存储私钥的游览器）发放TLS证书的能力。最好，这些证书可以使用之前的类型来区分。
各个类型的CA（如TLS CA， ECA， TCA）的TLS证书有可以通过中间CA（如，一个根CA的下属CA）发放。这里没有特定流量分析的问题，任意给定的TLS连接都可以相互验证，除了请求TLS CA的TLS证书的时候。

在当前的实现中，唯一的信任锚点是TLS CA的自签名证书来适应与所有三个（共址）服务器（即TLS CA，TCA和ECA）进行通信的单个端口限制。因此，与TLS CA的TLS握手来与TLS CA建立连接，所得到的会话密钥会传递给共址的TCA和ECA。因此，TCA和ECA的自签名证书的有效性的信任继承自对TLS CA的信任。在不提高TLS CA在其他CA之上的实现中，信任锚点需要由TLS CA和其他CA都认证的根CA替代。


### 4.7 当前版本的限制
这一小节列出了当前版本的fabric的限制。
具体的关注点是客户端操作和交易保密性设计，如4.7.1和4.7.2所述。

 - 客户端注册和交易的创建是由受信任不会模拟用户的非验证 peer 来完全执行。参看4.7.1节得到更多j信息。
 - 链码只能被系统的成员实体访问是保密性的最低要求，即，注册到我们成员服务的验证器和用户，其它的都不能访问。后者包含可以访问到存储区域维护的总账，或者可以看到在验证器网络上公布的交易。第一个发布版本在4.7.2小节中详细介绍。
 - 代码为注册CA（ECA）和交易CA（TCA）使用自签名的证书
 - 防重放攻击机制还不可用
 - 调用访问控制可以在应用层强制执行：
   安全性的保证取决于应用对基础设施工具的正确使用。这说明如果应用忽略了fabric提供的交易绑定*绑定*安全交易的处理可能在存在风险。

#### 4.7.1 简化客户端

客户端的注册和交易的创建是由非验证 peer 以在线钱包的角色全部执行的。特别地，终端用户利用他们的注册证书向非验证 peer 开立账户，并且使用这些证书来进一步授权 peer 建立代表用户的交易。
需要注意的是，这样的设计不会为 peer 代表用户提交的交易提供安全**授权**，如恶意 peer 可以模拟用户。
网上钱包的涉及安全问题设计规范的详细信息，可以在4.5节找到。
目前用户可以注册和执行交易的 peer 数量是一。

#### 4.7.2 简化交易保密

**免责声明:** 当前版本的交易保密是最小的，这被用来作为中间步骤来达到允许在未来版本中的细粒度（调用）的访问控制的执行设计。

在当前的格式，交易的保密仅仅在链层面提供，即，保存在总账中的交易内容对链的所有成员，如，验证器和用户，都是可读的。于此同时，不是系统的成员的应用审计人员，可以给予被动的观察区块链数据的手段。同时保证给予他们只是为了与被审计应用程序相关的交易。状态通过一种加密，同时不破坏底层共识网络的正常运行的方式来满足这样的审计要求

更具体的，当前使用对称密钥加密来提供交易保密性。
在这种背景下，一个最主要的挑战是特定于区块链的设置，验证器需要在区块链的状态上打成共识，即，除了交易本身，还包括个人合同或链码的状态更新。
虽然对于非机密链码这是微不足道的，对于机密链码，需要设计状态的加密机制，使得所得的密文语义安全，然而，如果明文状态是相同的那么他们就相等。

为了克服这一难题，fabric利用了密钥的层级，使用相同的密钥进行加密来降低密文数。同时，由于部分这些密钥被用于IV的生成，这使得验证方执行相同的事务时产生完全相同的密文（这是必要的，以保持不可知到底层共识算法），并通过只披露给审计实体最相关的密钥来提供控制审计的可能性。

**方法描述:**
成员服务为总账生成对称密钥 (K<sub>chain</sub>)，这是在注册到区块链系统所有实体时发布的，如，客户端和验证实体已通过链的成员服务发放证书。
在注册阶段，用户获取（像之前一样）一张注册证书，为用户u<sub>i</sub>记作Cert<sub>u<sub>i</sub></sub>，每个验证器v<sub>j</sub>获取它的被记作Cert<sub>v<sub>j</sub></sub>的证书。

实体注册将得到提高，如下所示。除了注册证书，用户希望以匿名方式参与交易发放交易证书。
为了简化我们把用户 u<sub>i</sub> 的交易证书记作 TCert<sub>u<sub>i</sub></sub>。
交易证书包含签名密钥对的公共部分记作 (tpk<sub>u<sub>i</sub></sub>,tsk<sub>u<sub>i</sub></sub>)。

为了防止密码分析和执行保密，下面的密钥层级被用来生成和验证保密的交易：
为了提交保密交易（Tx）到总账，客户端首先选择一个nonce(N)，这是需要提交区块链的所有交易中是唯一的，并通过以K<sub>chain</sub>作为密钥，nonce作为输入的HMAC函数生成一个交易对称密钥（K<sub>Tx</sub>)）K<sub>Tx</sub>= HMAC(K<sub>chain</sub>, N)。
对于K<sub>Tx</sub>，客户端生成两个AES密钥：
K<sub>TxCID</sub>当作HMAC(K<sub>Tx</sub>, c<sub>1</sub>), K<sub>TxP</sub> 当作 HMAC(K<sub>Tx</sub>, c<sub>2</sub>)) 分别加密链码名称或标识CID和代码（或payload）P.
c<sub>1</sub>, c<sub>2</sub> 是公共常量。nonce，加密的链码ID（ECID）和加密的Payload（EP）被添加到交易Tx结构中，即最终签名和认证的。
下面的图显示了如何产生用于客户端的事务的加密密钥。这张图中的剪头表示HMAC的应用，源由密钥锁定和使用在箭头中的数量作为参数。部署/调用交易的密钥键分别用d/I表示。



为了验证客户端提交到区块链的保密交易Tx，验证实体首先通过和之前一样的K<sub>chain</sub>和Tx.Nonce再生成K<sub>TxCID</sub>和K<sub>TxP</sub>来解密ECID和EP。一旦链码和Payload被恢复就可以处理交易了。


当V验证一个机密事务，相应的链码可以访问和修改链码的状态。V保持链码的状态加密。为了做到这一点，V生成如上图所示的对称密钥
。让iTX是一个之前由保密交易dTx部署的保密的交易调用一个函数（注意iTx可以是dTx本身）在这种情况下，例如，dTx具有初始化链码状态的设置函数。然后V像下面一样生成两个对称密钥K<sub>IV</sub>和K<sub>state</sub>：

1. 计算K<sub>dTx</sub>如，对应部署交易的交易密钥和N<sub>state</sub> = HMAC(K<sub>dtx</sub> ,hash(N<sub>i</sub>))其中N<sub>i</sub>是在调用交易中出现的nonce， *hash*是哈希函数
2. 它设K<sub>state</sub> = HMAC(K<sub>dTx</sub>, c<sub>3</sub> || N<sub>state</sub>)，截断用来加密底层密码； c<sub>3</sub> 是一个常数
3. 它设K<sub>IV</sub> = HMAC(K<sub>dTx</sub>, c<sub>4</sub> || N<sub>state</sub>); c<sub>4</sub> 是一个常数

为了加密状态变量S，验证器首先生成IV像 HMAC(K<sub>IV</sub>, crt<sub>state</sub>)正确截断，其中 crt<sub>state</sub>是计数器值，并在每次同样链码调用时请求状态更新时增加。当链码执行终止是计数器丢弃。IV产生之后，V认证加密（即，GSM模式）S的值连接Nstate（实际上，N<sub>state</sub>只需要认证而不需要加密）。在所得的密文（CT）， N<sub>state</sub>和IV被追加。为了解密加密状态CT||  N<sub>state'</sub>，验证器首次生成对称密钥K<sub>dTX</sub>' ,K<sub>state</sub>'，然后解密CT。

IV的生成: 任何底层共识算法是不可知的，所有的验证各方需要一种方法以产生相同的确切密文。为了做到这一点，需要验证使用相同的IV。重用具有相同的对称密钥相同的IV完全打破了底层密码的安全性。因此，前面所描述的方法制备。特别是，V首先通过计算HMAC(K<sub>dTX</sub>, c<sub>4</sub> || N<sub>state</sub> )派生的IV生成密钥K<sub>IV</sub>，其中c<sub>4</sub>是一个常数，并为(dTx,
iTx)保存计数器crt<sub>state</sub>初始设置为0。然后，每次必须生成一个新的密文，验证器通过计算HMAC(K<sub>IV</sub>, crt<sub>state</sub>)作为输出生成新的IV，然后为crt<sub>state</sub>增加1。

上述密钥层次结构的另一个好处是控制了审计的能力。
例如，当发布K<sub>chain</sub>会提供对整个供应链的读取权限，当只为交易的(dTx，iTx)发布K<sub>state</sub>访问只授予由iTx更新的状态，等等


下图展示一个部署和调用交易在目前在代码中的形式。

![FirstRelease-deploy](./images/sec-firstrel-depl.png)

![FirstRelease-deploy](./images/sec-firstrel-inv.png)


可以注意到，部署和调用交易由两部分组成：

* *基本信息*部分: 包含交易管理细节，如，把这个交易链接到的（被链接到的），交易的类型（被设置为''deploymTx''或''invocTx''），保密策略实现的版本号，它的创建者标识（由TCert，Cert表达）和一个nonce（主要为了防止重放攻击）

* *代码信息*部分: 包含在链码的源代码的信息。本质上是链码标识符/名称和源代码的部署交易，而对调用链码是是被调用函数名称和它的参数。就像在两张图中展示的代码信息那样他们最终是使用链指定的对称密钥K<sub>chain</sub>加密的。

## 5. 拜占庭共识
``obcpbft``包是[PBFT](http://dl.acm.org/citation.cfm?id=571640 "PBFT")共识协议[1]的实现，其中提供了验证器之间的共识，虽然验证器的阈作为_Byzantine_，即，恶意的或不可预测的方式失败。在默认的配置中，PBFT容忍t<n/3的拜占庭验证器。

处理提供PBFT共识协议的参考实现，``obcpbft`` 插件还包含了新颖的_Sieve_共识协议的实现。基本上Sieve背后的思想为_non-deterministic_交易提供了fabric层次的保护，这是PBFT和相似的协议没有提供的，``obcpbft``可以很容易配置为使用经典的PBFT或Sieve。

在默认配置中，PBFT和Sieve设计运行在至少*3t +1 *验证器（副本），最多容忍*T*个出现故障（包括恶意或*拜占庭*）副本。

### 5.1 概览
`obcpbft`插件提供实现了`CPI`接口的模块，他可以配置运行PBFT还是Sieve共识协议。模块化来自于，在内部，`obcpbft`定义了`innerCPI` 接口（即， _inner consensus programming interface_），现在包含在 `pbft-core.go`中。

该`innerCPI`接口定义的所有PBFT内部共识（这里称为*core PBFT*并在`pbft-core.go`实现）和使用core PBFT的外部共识之间的相互作用。`obcpbft`包包含几个core PBFT消费者实现

  - `obc-classic.go`，  core PBFT周围的shim，实现了`innerCPI`接口并调用`CPI`接口;
  - `obc-batch.go`， `obc-classic`的变种，为PBFT添加批量能力；
  - `obc-sieve.go`， core PBFT消费者，实现Sieve共识协议和`innerCPI`接口， 调用`CPI interface`.

总之，除了调用发送消息给其他 peer(`innerCPI.broadcast` 和 `innerCPI.unicast`)，`innerCPI`接口定义了给消费者暴露的共识协议。
这使用了用来表示信息的原子投递的`innerCPI.execute`调用的一个经典的*总序（原子）广播* API[2]。经典的总序广播在*external validity* checks [2]中详细讨论(`innerCPI.verify`)和一个功能相似的对不可靠的领导失败的检查&Omega; [3] (`innerCPI.viewChange`).

除了`innerCPI`， core PBFT 定义了core PBFT的方法。core PBFT最重要的方法是`request`有效地调用总序广播原语[2]。在下文中，我们首先概述core PBFT的方法和``innerCPI``接口的明细。然后，我们简要地描述，这将在更多的细节Sieve共识协议。

### 5.2 Core PBFT函数
下面的函数使用非递归锁来控制并发，因此可以从多个并行线程调用。然而，函数一般运行到完成，可能调用从CPI传入的函数。必须小心，以防止活锁。

#### 5.2.1 newPbftCore

签名:

```
func newPbftCore(id uint64, config *viper.Viper, consumer innerCPI, ledger consensus.Ledger) *pbftCore
```

newPbftCore构造器使用指定的`id`来实例化一个新的PBFT箱子实例。`config`参数定义了PBFT网络的操作参数：副本数量*N*，检查点周期*K*，请求完成的超时时间，视图改变周期。

| configuration key            | type       | example value | description                                                    |
|------------------------------|------------|---------------|----------------------------------------------------------------|
| `general.N`                  | *integer*  | 4             | Number of replicas                                             |
| `general.K`                  | *integer*  | 10            | Checkpoint period                                              |
| `general.timeout.request`    | *duration* | 2s            | Max delay between request reception and execution              |
| `general.timeout.viewchange` | *duration* | 2s            | Max delay between view-change start and next request execution |

接口中传递的`consumer`和`ledger`参数是一旦它们全部排好序后用来查询应用状态和调用应用请求的。参阅下面这些接口的相应部分。

## 6. 应用编程接口
fabric的主要接口是REST API。 REST API允许应用注册用户，查询区块链，并发布交易。 CLI为了开发，同样提供有效API的子集。CLI允许开发人员能够快速测试链码或查询交易状态。

应用程序通过REST API与非验证的 peer 节点，这将需要某种形式的认证，以确保实体有适当的权限进行交互。该应用程序是负责实现合适的身份验证机制和 peer 节点随后将使用客户身份对发出消息签名。

![Reference architecture](images/refarch-api.png) <p>

fabric API 设计涵盖的类别如下，虽然当前版本的其中一些实现不完整。[REST API（＃62-REST的API）节将说明API当前支持。


*  身份 - 注册来获得或吊销一张证书
*  Address - 交易的源或目的
*  Transaction - 总账上的执行单元
*  Chaincode - 总账上运行的程序
*  Blockchain - 总账的内容
*  Network - 区块链 peer 网络的信息
*  Storage - 文件或文档的外部存储
*  Event Stream - 区块链上订阅/发布事件

## 6.1 REST Service
REST服务可以（通过配置）在验证和非验证 peer 被启用，但是建议在生产环境中只启用非验证 peer 的REST服务。

```
func StartOpenchainRESTServer(server *oc.ServerOpenchain, devops *oc.Devops)
```

这个函数读取`core.yaml``peer`处理的配置文件中的`rest.address`。`rest.address`键定义了 peer 的HTTP REST服务默认监听的地址和端口。

假定REST服务接收来已经认证的终端用户的应用请求。

## 6.2 REST API

您可以通过您所选择的任何工具与REST API的工作。例如，curl命令行实用程序或一个基于浏览器的客户端，如Firefox的REST客户端或Chrome Postman。同样，可以通过 [Swagger](http://swagger.io/) 直接触发REST请求。为了获得REST API Swagger描述，点击 [这里](https://github.com/hyperledger/fabric/blob/master/core/rest/rest_api.json)。目前可用的API总结于以下部分。


### 6.2.1 REST Endpoints

* [Block](#6211-block-api)
  * GET /chain/blocks/{block-id}
* [Blockchain](#6212-blockchain-api)
  * GET /chain
* [Chaincode](#6213-chaincode-api)
  * POST /chaincode
* [Network](#6214-network-api)
  * GET /network/peers
* [Registrar](#6215-registrar-api-member-services)
  * POST /registrar
  * GET /registrar/{enrollmentID}
  * DELETE /registrar/{enrollmentID}
  * GET /registrar/{enrollmentID}/ecert
  * GET /registrar/{enrollmentID}/tcert
* [Transactions](#6216-transactions-api)
  * GET /transactions/{UUID}

#### 6.2.1.1 块API

* **GET /chain/blocks/{block-id}**

使用块API来从区块链中检索各个块的内容。返回的块信息结构是在[3.2.1.1](#3211-block)节中定义

块检索请求:
```
GET host:port/chain/blocks/173
```

块检索响应:


#### 6.2.1.2 区块链API

* **GET /chain**

使用链API来检索区块链的当前状态。返回区块链信息消息被定义如下。

```
message BlockchainInfo {
    uint64 height = 1;
    bytes currentBlockHash = 2;
    bytes previousBlockHash = 3;
}
```

* `height` - 区块链中块的数量，包括创始区块

* `currentBlockHash` - 当前或最后区块的哈希

* `previousBlockHash` - 前一区块的哈希

区块链检索请求:
```
GET host:port/chain
```

区块链检索响应:


#### 6.2.1.3 链码API

* **POST /chaincode**

使用链码API来部署，调用和查询链码
部署请求需要客户端提供`path`参数，执行文件系统中链码的目录。部署请求的响应要么是包含成功的链码部署确认消息要么是包含失败的原因的错误。
它还含有所生成的链码的`name`域在消息中，这是在随后的调用和查询交易中使用的已部署链码的唯一标识。

要部署链码，需要提供ChaincodeSpec的payload，在[3.1.2.2](#3122-transaction-specification)节中定义。

部署请求:
```
POST host:port/chaincode

```

部署响应:


当启用安全时，修改所需的payload包括传递的登录用户注册ID的`secureContext`元素如下：

启用安全的部署请求:
```
POST host:port/chaincode


该调用请求要求客户端提供一个`name`参数，这是之前从部署交易响应得到的。调用请求的响应要么是包含成功执行的确认消息，要么是包含失败的原因的错误。

要调用链码，需要提供ChaincodeSpec的payload，在[3.1.2.2](#3122-transaction-specification)节中定义

调用请求:
```
POST host:port/chaincode


调用响应:


当启用安全时，修改所需的payload包括传递的登录用户注册ID的`secureContext`元素如下：


启用安全的调用请求:


查询请求需要在客户端提供一个`name`参数，这是之前在部署交易响应中得到了。查询请求的响应取决于链码的实现。响应要么是包含成功执行的确认消息，要么是包含失败的原因的错误。在成功执行的情况下，响应将包含链码请求的状态变量的值

要查询链码，需要提供ChaincodeSpec的payload，在[3.1.2.2](#3122-transaction-specification)节中定义。

查询请求:


查询响应:



当启用安全时，修改所需的payload包括传递的登录用户注册ID的`secureContext`元素如下：



启用安全的查询请求:


#### 6.2.1.4 网络API

使用网络API来获取组成区块链 fabric 的 peer 节点的网络信息

/network/peers 端点返回的目标 peer 节点的所有现有的网络连接的列表。该列表包括验证和非验证 peer。peer 的列表被返回类型`PeersMessage`是包含`PeerEndpoint`的数组，在第[3.1.1]（#311-discovery-messages发现的消息）定义。


```
message PeersMessage
```

网络请求:
```
GET host:port/network/peers
```

网络响应:


#### 6.2.1.5 注册API (成员服务)

* **POST /registrar**
* **GET /registrar/{enrollmentID}**
* **DELETE /registrar/{enrollmentID}**
* **GET /registrar/{enrollmentID}/ecert**
* **GET /registrar/{enrollmentID}/tcert**

使用注册API来管理的证书颁发机构（CA）的最终用户注册。这些API端点用于注册与CA用户，确定指定用户是否已注册，并从本地存储中删除任何目标用户的登录令牌，防止他们执行任何进一步的交易。注册API也用于从系统中检索用户注册和交易证书。

`/registrar`端点使用与CA注册用户所需的秘密payload定义如下。注册请求的响应可以是一个成功的注册的确认或包含失败的原因的错误。



* `enrollId` - 在证书颁发机构的注册ID
* `enrollSecret` - 在证书颁发机构的密码

注册请求:
```
POST host:port/registrar

```

注册响应:
```
{
    "OK": "Login successful for user 'lukas'."
}
```

`GET /registrar/{enrollmentID}`端点用于确认一个给定的用户是否与CA注册如果是，确认将被反悔。否则，将导致授权错误。

注册验证请求:
```
GET host:port/registrar/jim
```

注册验证返回:
```
{
    "OK": "User jim is already logged in."
}
```

注册验证请求:
```
GET host:port/registrar/alex
```

注册验证返回:
```
{
    "Error": "User alex must log in."
}
```

`DELETE /registrar/{enrollmentID}` 端点用于删除一个目标用户的登录令牌。如果登录令牌成功删除，确认将被反悔。否则，将导致授权错误。此端点不需要payload。

删除注册请求:
```
DELETE host:port/registrar/lukas
```

删除注册返回:
```
{
    "OK": "Deleted login token and directory for user lukas."
}
```

`GET /registrar/{enrollmentID}/ecert`
端点用于检索从本地存储给定用户的登记证书。如果目标用户已与CA注册，响应将包括注册证书的URL-encoded版本。如果目标用户尚未注册，将返回一个错误。如果客户希望使用检索后返回的注册证书，请记住，它必须是URL-decoded。

注册证书检索请求:
```
GET host:port/registrar/jim/ecert
```

注册证书检索响应:
```
{
    "OK": "-----BEGIN+CERTIFICATE-----%0AMIIBzTCCAVSgAwIBAgIBATAKBggqhkjOPQQDAzApMQswCQYDVQQGEwJVUzEMMAoG%0AA1UEChMDSUJNMQwwCgYDVQQDEwNPQkMwHhcNMTYwMTIxMDYzNjEwWhcNMTYwNDIw%0AMDYzNjEwWjApMQswCQYDVQQGEwJVUzEMMAoGA1UEChMDSUJNMQwwCgYDVQQDEwNP%0AQkMwdjAQBgcqhkjOPQIBBgUrgQQAIgNiAARSLgjGD0omuJKYrJF5ClyYb3sGEGTU%0AH1mombSAOJ6GAOKEULt4L919sbSSChs0AEvTX7UDf4KNaKTrKrqo4khCoboMg1VS%0AXVTTPrJ%2BOxSJTXFZCohVgbhWh6ZZX2tfb7%2BjUDBOMA4GA1UdDwEB%2FwQEAwIHgDAM%0ABgNVHRMBAf8EAjAAMA0GA1UdDgQGBAQBAgMEMA8GA1UdIwQIMAaABAECAwQwDgYG%0AUQMEBQYHAQH%2FBAE0MAoGCCqGSM49BAMDA2cAMGQCMGz2RR0NsJOhxbo0CeVts2C5%0A%2BsAkKQ7v1Llbg78A1pyC5uBmoBvSnv5Dd0w2yOmj7QIwY%2Bn5pkLiwisxWurkHfiD%0AxizmN6vWQ8uhTd3PTdJiEEckjHKiq9pwD%2FGMt%2BWjP7zF%0A-----END+CERTIFICATE-----%0A"
}
```

`/registrar/{enrollmentID}/tcert`端点检索已与证书机关登记给定用户的交易证书。如果用户已注册，确认消息将包含URL-encoded交易证书的列表被返回。否则，将会导致一个错误。交易证书的所需数量由可选的'count'查询参数指定。返回交易证书的默认数量为1;500是可以与单个请求中检索证书的最大数量。如果客户端希望使用取回后的交易证书，请记住，他们必须是URL-decoded。

交易证书检索请求:
```
GET host:port/registrar/jim/tcert
```

交易证书检索响应:
```
{
    "OK": [
        "-----BEGIN+CERTIFICATE-----%0AMIIBwDCCAWagAwIBAgIBATAKBggqhkjOPQQDAzApMQswCQYDVQQGEwJVUzEMMAoG%0AA1UEChMDSUJNMQwwCgYDVQQDEwN0Y2EwHhcNMTYwMzExMjEwMTI2WhcNMTYwNjA5%0AMjEwMTI2WjApMQswCQYDVQQGEwJVUzEMMAoGA1UEChMDSUJNMQwwCgYDVQQDEwNq%0AaW0wWTATBgcqhkjOPQIBBggqhkjOPQMBBwNCAAQfwJORRED9RAsmSl%2FEowq1STBb%0A%2FoFteymZ96RUr%2BsKmF9PNrrUNvFZFhvukxZZjqhEcGiQqFyRf%2FBnVN%2BbtRzMo38w%0AfTAOBgNVHQ8BAf8EBAMCB4AwDAYDVR0TAQH%2FBAIwADANBgNVHQ4EBgQEAQIDBDAP%0ABgNVHSMECDAGgAQBAgMEMD0GBioDBAUGBwEB%2FwQwSRWQFmErr0SmQO9AFP4GJYzQ%0APQMmcsCjKiJf%2Bw1df%2FLnXunCsCUlf%2FalIUaeSrT7MAoGCCqGSM49BAMDA0gAMEUC%0AIQC%2FnE71FBJd0hwNTLXWmlCJff4Yi0J%2BnDi%2BYnujp%2Fn9nQIgYWg0m0QFzddyJ0%2FF%0AKzIZEJlKgZTt8ZTlGg3BBrgl7qY%3D%0A-----END+CERTIFICATE-----%0A"
    ]
}
```

交易证书检索请求:
```
GET host:port/registrar/jim/tcert?count=5
```

交易证书检索响应:


#### 6.2.1.6 交易API

* **GET /transactions/{UUID}**

使用交易API来从区块链中检索匹配UUID的单个交易。返回的交易消息在[3.1.2.1](#3121-transaction-data-structure)小节定义

交易检索请求:
```
GET host:port/transactions/f5978e82-6d8c-47d1-adec-f18b794f570e
```

交易检索响应:


## 6.3 CLI

CLI包括可用的API的一个子集，使开发人员能够快速测试和调试链码或查询交易状态。CLI由Golang实现和可在多个操作系统上操作。当前可用的CLI命令归纳在下面的部分：

### 6.3.1 CLI命令

To see what CLI commands are currently available in the implementation, execute the following:

要查看当前可用的CLI命令，执行如下命令

    cd $GOPATH/src/github.com/hyperledger/fabric/peer
    ./peer

你可以获得和下面类似的响应：

```
    Usage:
      peer [command]

    Available Commands:
      peer        Run the peer.
      status      Status of the peer.
      stop        Stop the peer.
      login       Login user on CLI.
      vm          VM functionality on the fabric.
      chaincode   chaincode specific commands.
      help        Help about any command

    Flags:
      -h, --help[=false]: help


    Use "peer [command] --help" for more information about a command.
```

Some of the available command line arguments for the `peer` command are listed below:

* `-c` - 构造函数: 用来为部署触发初始化链码状态的函数

* `-l` - 语言: 指定链码的实现语言，目前只支持Golang

* `-n` - 名字: 部署交易返回的链码的标识。在后续的调用和查询交易中必须使用

* `-p` - 路径: 链码在本地文件系统中的标识。在部署交易时必须提供。

* `-u` - 用户名: 调用交易的登入的用户的注册ID

上述所有命令并非完全在当前版本中实现。如下所述全面支持的命令是有助于链码的开发和调试的。

所有 peer 节点的设置都被列在`core.yaml`这个`peer`处理的配置文件中，可能通过命令行的环境变量而被修改。如，设置`peer.id`或 `peer.ddressAutoDetect`，只需要传递`CORE_PEER_ID=vp1`和`CORE_PEER_ADDRESSAUTODETECT=true`给命令行。

#### 6.3.1.1 peer

`peer`CLI命令在开发和生产环境中都会执行 peer 处理。开发模式会在本地运行单个 peer 节点和本地的链码部署。这使得在链码开修改和调试代码，不需要启动一个完整的网络。在开发模式启动 peer 的一个例子：

```
./peer peer --peer-chaincodedev
```

在生产环境中启动peer进程，像下面一样修改上面的命令：

```
./peer peer
```

#### 6.3.1.2 登录

登录的CLI命令会登入一个已经在CA注册的用户。要通过CLI登录，发出以下命令，其中`username`是注册用户的注册ID。
```
./peer login <username>
```

下面的例子演示了用户`jim`登录过程。
```
./peer login jim
```

该命令会提示输入密码，密码必须为此用户使用证书颁发机构注册登记的密码相匹配。如果输入的密码不正确的密码匹配，将导致一个错误。



您也可以与`-p`参数来提供用户的密码。下面是一个例子。

```
./peer login jim -p 123456
```

#### 6.3.1.3 链码部署

`deploy`CLI命令为链码和接下来的部署包到验证 peer 创建 docker 镜像。如下面的例子。

```
./peer chaincode deploy -p github.com/hyperledger/fabric/example/chaincode/go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'
```

启用安全性时，命令必须修改来通过`-u`参数传递用户登录的注册ID。下面是一个例子

```
./peer chaincode deploy -u jim -p github.com/hyperledger/fabric/example/chaincode/go/chaincode_example02 -c '{"Function":"init", "Args": ["a","100", "b", "200"]}'
```

#### 6.3.1.4 链码调用

`invoke`CLI命令执行目标来代码中的指定函数。如下：

```
./peer chaincode invoke -n <name_value_returned_from_deploy_command> -c '{"Function": "invoke", "Args": ["a", "b", "10"]}'
```

启用安全性时，命令必须修改来通过`-u`参数传递用户登录的注册ID。下面是一个例子

```
./peer chaincode invoke -u jim -n <name_value_returned_from_deploy_command> -c '{"Function": "invoke", "Args": ["a", "b", "10"]}'
```

#### 6.3.1.5 链码查询

`query`CLI命令在目标链码上触发指定的查询。返回的响应取决于链码实现。下面是一个例子。

```
./peer chaincode query -l golang -n <name_value_returned_from_deploy_command> -c '{"Function": "query", "Args": ["a"]}'
```

启用安全性时，命令必须修改来通过`-u`参数传递用户登录的注册ID。下面是一个例子

```
./peer chaincode query -u jim -l golang -n <name_value_returned_from_deploy_command> -c '{"Function": "query", "Args": ["a"]}'
```


## 7. 应用模型

### 7.1 应用的组成
<table>
<col>
<col>
<tr>
<td width="50%"><img src="images/refarch-app.png"></td>
<td valign="top">
一个遵循MVC-B架构的应用– Model, View, Control, BlockChain.
<p><p>

<ul>
  <li>VIEW LOGIC – 与控制逻辑集成的移动或WEB 用户界面。</li>
  <li>CONTROL LOGIC – 协调用户界面、数据模型和交易与链码的API </li>
  <li>DATA MODEL – 应用数据模型– 管理包括文档和大文件这样的非链（off-chain）数据</li>
  <li>BLOCKCHAIN  LOGIC – 区块链逻辑是控制逻辑和数据模型在区块链领域的扩展，链码（chaincode）加强了控制逻辑，区块链上的交易加强了数据模型。</li>
</ul>
<p>

例如，使用 Node.js 的一个 Bluemix PaaS 的应用程序可能有一个 Web 前端用户界面或与 Cloudant 数据服务后端模型中的原生移动应用。控制逻辑可以被 1 或多个链码交互以处理对区块链交易。

</td>
</tr>
</table>

### 7.2 应用样例


## 8. 未来发展方向
### 8.1 企业集成
### 8.2 性能与可扩展性
### 8.3 附加的共识插件
### 8.4 附加的语言

