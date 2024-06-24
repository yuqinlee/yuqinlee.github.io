# 区块链与安全Note


<!-- <h1>区块链与安全Note</h1> -->

>2020-11-04
>
<!-- >yuqin -->
>
> <https://www.bilibili.com/video/av37065233>

<!-- [toc] -->

## 一、BTC-密码学原理

### 1. Hash

#### 性质1 collision resistance

密文m

H(m)经过hash函数处理之后得到一个hash值，很难在修改m后再进行hash运算还是原来的值，尽管存在hash碰撞

<font color='red'>没有证明那个hash函数是不存在collision resistance的！！但是有函数是被验证是能够人为找出碰撞的，如MD5</font>

#### 性质2 hiding

hiding：hash函数的计算是单向且不可逆的，即给定x，可以算出其hash值$H(x)$ , $x->H(x)$但是没办法从$H(x)$反推出$x$

<font color='red'>hiding成立的前提是输入空间足够大(不然直接暴力遍历出原文),并且输入的取值比较均匀,各种取值差不多</font>

hiding与collision resistance结合起来,实现 **digital commitment(digital equivalent  of a sealed envelope)**

例如,呀进行股市的预测,需提前给出预测,但是又不能将结果直接公布,因为这个预测会对结果产生影响,所以,可以先将预测进行hash,得到一个hash值,由于性质1与性质2,在第二天结果知晓后,公布预测内容并计算hash值,对比之前公布的hash值,相同则预测正确

**实际操作中,为使输入的空间足够大,可以$H(x||nonce)$,即在输入内容后增加一个随机数(nonce)**

#### 性质3 puzzle friendly

puzzle friendly:hash值的计算事先是不可预测的,即如果想获得一个区间范围内的hash值,那只能一个一个试

例如,想要获得一个经过hash运算后，256位的hash值是形如"00……00xxxxx"前k位都是"0"的原文，那只能一个一个试，而没有方法实现知道。在挖矿中，$nonce$在$block\quad header$中,是可以自己设置的,挖矿过程就是找到适合的$nonce$使得满足以下式子:
$$
H(block\quad header)\leq target
$$
$H(block\quad header)$要落在指定的$target\quad space$中,只有经过大量的尝试,才能证明其中的"$proof\quad of\quad work$",在这个过程中,"挖矿"是很难的,但是去验证是很容易的,这个性质叫**"difficult to solve,but easy to verify"**

在比特币中使用的hash函数是$SHA-256(Secure Hash Algorithm)$,满足上述三个性质

### 2. 签名

$asymmetric\quad encryption\quad algorithm(非对称加密)$

在现实中开户需要到银行进行相关手续，但是比特币开户只需要在本地创建一对公钥与私钥即可$(public\quad key ,private\quad key)$,知道公钥就相当于知道银行账号,其他人可以往里转账,而私钥就好比是账户的口令,能将账户中的"钱"取出来。比特币是不加密的加密货币，这里公私钥是为了进行签名。

<font color='red'>会不会两人生成的公私钥对是相同的？答：可能性微乎其微</font>

这里生成公私钥要求有一个$a\quad good\quad source\quad of\quad randomness$，如果随机源选择不好的话就有可能两个人的公私钥相同了；在比特币中，除了生成公私钥对时需要一个好的随机源，在每一次加密的过程中都需要有好的随机源，不然可能泄露私钥

比特币中一般是先对message进行hash，然后再对hash值进行签名！！

*[区块链技术与应用——BTC密码学原理](https://www.cnblogs.com/wkfvawl/p/13138979.html)*

## 二、BTC-数据结构

### hash pointer、区块链

普通指针是存储一个结构体的地址值，而hash指针除了存储地址值外还存储hash值，这样一来可以找到该结构体的位置，二来可以知道该结构体是否被改变，一般使用$H()$表示

比特币中区块与普通的区块的一个区别就是使用hash指针代替了普通指针

>$Block\quad chain\quad is\quad a\quad linked\quad list\quad using\quad hash\quad pointers$

<img src="%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201104195623038.png" alt="image-20201104195623038" style="zoom: 67%;" />

后面一个hash值是将前面区块$block\quad header$整个取hash，包括其中的hash function，以此实现了 tamper-evident log，这样如果前一个区块中的内容发生改变，那么后面的hash值也会发生改变，依次类推，后面所有的区块内容都会发生改变，所以只要记住最后的hash值就可以检测出整个区块链中任何部位的修改，这也是和普通区块的不同。

本地不必存储所有的区块，在需要使用前面的区块时向其他节点要即可，验证真伪只需要将前面的hash值进行对比就可。

### Merkel tree

![image-20201104202141989](%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201104202141989.png)

两个data blocks的hash pointer放在父节点，这两个hash pointer组合再取hash值，构成整个父节点，两个父节点的的hash pointer再放在更上一层中，……

**binary tree与Merkel tree的区别**

- merkel tree使用hash指针代替了原有指针
- 只要记录root hash值，就能检测出对树中任何部位的修改
- 每个data block都是一个“交易”

<font color='red'>比特币中各个区块之间用hash 指针连接在一起，每个区块所包含的交易是组织成一个Merkel tree的形式</font>

每个区块分为$block\quad header$和$block\quad body$，在$block\quad header$中有根hash值，而没有交易的具体内容，$block\quad body$中是有交易列表的

**merkel tree的作用：提供Merkel proof**，示意图如下：

![image-20201104203540474](%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201104203540474.png)

1. 轻节点向全节点进行一个Merkel proof的请求，全节点向轻节点发送红色的H()

2. 轻节点在本地根据待证明tx可以计算出其上一个绿色H()，而红色的H()是向全节点请求能得到的，因此和红色H()结合能算出再上一层的绿色H()，……最后能算出一个根hash值

3. 此时与轻节点所保存的根hash值进行比较就可以验证是否待证明的tx交易在整个区块链中

以上证明称为$proof\quad of\quad membership$或$proof\quad of\quad inclusion$，其事件复杂度为$O(log(n))$,如果是证明$proof\quad of\quad no-membership$,其中一个方式是将整棵Merkel tree传回来，验证每一层的hash值都是正确的，则说明这树中只有这些叶节点，而没有待验证的此时复杂度为$O(n)$,如果没有对叶节点进行排序等操作，是没有更好的办法的，但是将tx按照交易的hash值进行排序，就能以$O(log(n))$的代价进行验证了，此时这棵树叫$sorted\quad merkel\quad tree$,**在比特币中是没有这种需求的！**

**没有环的数据结构中，hash pointer基本都可以代替普通指针，但是有环的是不行的**

## 三、BTC-协议

**前言**

**数字货币方案1**

假设央行发行数字货币，只采用非对称密码，央行保存私钥，并用私钥进行数字签名，这样发行出来的货币存在的问题：“double spending attack”，即可以将央行发行的数字货币进行复制，从而进行多次支付！这也是数字货币面临的主要挑战。

**数字货币方案2**

在之前的基础上为每个数字货币添加一个编号，央行对编号的所有者进行记录，同时在完成支付之后更改编号所有者。这样就能防范“double spending attack ”。

这套方案可行，但每笔交易都经过央行，过于繁琐，这是一种中心化的方案。

---

**简单区块链示意**

![image-20201106184509456](%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201106184509456.png)

上图中有两种hash指针，一种是区块链之间的连接的<font color='orange'>hash指针(橙色)</font>，另一种是<font color='green'>指向前面某个交易的(绿色)</font>

比特币系统中每个交易都包含输入和输出两个部分:

- 输入：①说明币的来源；②A的公钥是什么
- 输出：给出收款人的公钥的hash

    > <font color='red'>为什么要记录来源？</font>①证明这个钱不是凭空捏造的；②防止**double spending attack**
    > 例如上图中：B已经将钱转给C和D了，现在出现F，B要将钱再次转给F
    > ![image-20201106183904834](%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201106183904834.png)
    > 别的节点收到交易后，从B转向F的交易向前查询，当查到B转给C与D时就会出生问题，B在转给C与D的这次交易中就已将5个币花了出去，即说明B转给F的交易是不合法的！！

上述转账中，A转账给B，A需要的信息有：

- A的签名
- B的地址（比特币中收款的地址通过公钥推算出来，公钥取hash经过一些转换得到）

    > - 比特币系统是不会提供查询某个人的地址的服务的，想要知道别人的地址，需要通过其他渠道，例如在某支持比特币的购物网站上，收款人可以贴出自己的地址/公钥。
    >
    > - 上述转账中<font color='red'>B、乃至所有的节点还需要知道A的公钥</font>，因为B要知道支付人的信息，才能知道这笔交易的钱是从哪来的，同时A支付这一过程中A进行了签名，要验证签名就需要A的公钥（签名是私钥签，公钥验证）。A在交易的输入中就会说明A的公钥是什么。
    >
    >   问题：自己宣称公钥是否有漏洞？  
    > 如果存在B的同伙$B'$,这时候伪造一个A到B的转账交易，$B'$用自己的公钥在输入中说是A的公钥，用自己的私钥进行签名，当别的节点用假造的公钥去验证交易的合法性，就会盗走A账户的钱？
    >
    >   解决方式：
    >   在Create Coin过程的输出中包含着A的公钥的hash，在交易A向B转账时，A的公钥要和之前的输出过程的hash对得上才行

示意图中每个区块包含着一个交易，但实际过程中每个区块中包含着许多交易，这些交易组装成Merkel Tree，每个区块分为$Block\quad header$与$Block\quad body$两个部分。

$Block\quad header$包含着区块中的宏观信息:

- 使用的是比特币哪个版本的协议(version)

- 执行前一个区块的指针(hash of previous block header)

  > <font color='red'>前一个区块的hash只算的是$block\quad header$！！</font>
  >
  > 只有block header才有hash指针串联起来，每次取hash都是将块头进行取hash
  >
  > <img src="%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201106194111150.png" alt="image-20201106194111150" style="zoom: 40%;" />
  >
  > 因为Merkel root hash就能保证Block body中的交易列表是没有办法进行改变的。（因为要是改变了那header的hash值就会发生变化）

- 整个Merkel tree的根hash值(Merkel root hash)

- 难度目标阈值(target)

- 随机数(nonce)

  >$$
  >H(block\quad header)\leq target
  >$$
  >
  >$block\quad header$的hash值要小于target

### 关于full node（全节点）与light node（轻节点）

full node：也称fully validating node，保存全部信息

light node：只保存block header的信息，一般是无法独立验证交易的合法性的。轻节点知识利用区块链的信息进行一些查询等操作。

### 比特币中的共识协议（Consensus in BitCoin）

>账本的内容要取得分布式共识 distributed consensus
>
>FLP impossibility result：
>在异步式系统（网络传输时延没有上限）中，只有一个成员有问题，也不可能取得共识
>
>CAP Theorem：CAP三个性质中最多只能满足两个（见分布式系统）
>

共识协议要解决的问题：有些节点可能是有恶意的，这里假设系统中大多数节点是没有恶意的，有恶意的只是少数

> **假想：**投票机制，某个节点提出一个候选区块，根据收到的交易信息，选择那些交易是合法的，将这些交易打包到区块里，将这个候选区块发布给所有节点，每个节点收到这个区块后检查一下是不是每个交易都是合法的，都合法就投赞成票，有一个交易是非法的就投反对票，赞成过半就写入区块链中
>
> 存在的问题：
>
> 1. 提出候选区块的节点恶意加入非法区块到候选区块，并不断提交这种带非法交易的候选区块，造成一直投票却无法写入区块链
>
> 2. 无法保证每个节点都投票
>
> 3. 效率问题、网络还有延迟
>
> 4. <font color='red'>比特币系统中，投票权问题无法确定</font>，只要恶意产生足够多账户，超过半数，就可操作结果
>    （$sybil\quad  attack$）
>

在比特币中也是进行“投票”，但是是利用算力进行投票，每个节点都可以在本地组装出候选区块，将其认为合法的交易放到这个区块中，然后尝试各种nonce值，如果某个节点找到了nonce，使得$H(block\quad header)\leq target$,则该节点获得了记账权，即能往账本中写入下一个区块的权利。其他节点则检查是否符合要求。

### 最长合法链

下图中，区块1为C转账A，区块3为A转B，区块5为A再次转给自己，首先，这没有构成$double\quad spending\quad attack$,因为在链1<-2<-5中，A并没有使用两次，检测是否是”双花“时是不会检查其他链上的交易情况的。下图中5写到2的后面的情况显然是不被希望的。

<img src="%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201106205107454.png" alt="image-20201106205107454" style="zoom: 50%;" />

因此在比特币协议中，规定写在**<font color='red'>最长合法链</font>**之后的才是合法区块,上述例子称为”forking attack（分叉攻击）“，即：通过往区块链中间某个区块后插入区块实现回滚某个已经发生了的交易。

在正常情况下，如果两个节点同时找到了合适的nonce并发布出去，由于整个系统中节点众多，不同节点认同的这两个区块中的某一个，那么就会形成两个等长的链，按照最长合法链原则，这两个都是合法的，比特币系统中，接受某个区块就会往该区块后面继续添加区块，因此会存在临时的分支，但某条链最后会”胜出“。

<img src="%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201106210916913.png" alt="image-20201106210916913" style="zoom: 60%;" />

如上图，5，6同时发布，原先最长链分叉，这时形成两条合法链，如果在下一时刻，区块5后增加区块7，而6后并未增加新的区块，那么6所在的那条链就会被丢弃，称为”orphan block“。在”orphan block“中获得的block reward是不会被认可的

### block reward（出块奖励）

去中心化货币面临的两个问题：①谁来发行货币；②怎么保证交易的合法性

coinbase transaction是发行比特币的<font color='red'>**唯一方法**</font>。

> 前21万个区块里，每个区块发行50BTC，之后21万区块里，每个区块发行25BTC，下一21万区块中，每个区块发行12.5BTC
>
> 50BTC -> 25BTC -> 12.5BTC -> ……

**hash rate：**比特币系统中根据算力投票，不同区块获取符合nonce的概率是不同的，这个称为hash rate

针对“sybil attack”，由于投票权是以算力决定的，即使创建账户十分多，也无法改变其算力大小。

*[区块链技术与应用——BTC的共识协议](https://www.cnblogs.com/wkfvawl/p/13150376.html)*

## BTH-实现

比特币采用的是**基于交易的账本模式，transaction-based ledger**。

### UTXO：Unspend Transaction Output

即：还没有被花出去的交易的输出，区块链上有很多交易，有些交易的输出可能已经被花掉了，有些还没有被花掉，没有被花掉的交易的输出组成的集合就是UTXO。UTXO为比特币系统中的一个数据结构。

>例如：A转给B、C各5BTC，这时B将这5个比特币花掉了，C没有花掉，那么就只有A与C的输出在UTXO中
>
>如果这时B再转给D，D也没有花掉，那么B转给D的输出就会保存在UTXO
>
><img src="%E5%8C%BA%E5%9D%97%E9%93%BE%E4%B8%8E%E5%AE%89%E5%85%A8Note.assets/image-20201108125335047.png" alt="image-20201108125335047" style="zoom: 33%;" />

UTXO中的每个元素要给出产生这个输出的交易的hash值，以及它在这个交易里是第几个输出，就可以定位到UTXO中的输出

><font color='red'>为什么要维护UTXO这个数据结构？</font>
>
>花掉的币只有在UTXO这个集合中才是合法的，如果不在这个集合中，那么花的币要么不存在，要么以前已经被花过了。全节点要在内存中维护UTXO这个数据结构，以便快速检测“double spending”

**total inputs=total outputs**：一笔交易可以有多个输入（不一定来自同一个地址，所以一个交易也可能有多个签名），也可以有多个输出，要求交易的总输入等于总输出。

但有时total inputs 可能稍微大于outputs，因为还要给打包区块的节点提供费用，即$transaction\quad fee（交易费）$。一般交易费是比较小的，甚至有的交易没有交易费。

>为什么有交易费？  
>光有出块奖励机制还不够，有些“自私”的节点可能只去打包自己的交易，而不去管别人的交易，因为打包别人的交易还要验证交易的合法性，占用带宽等，，显然只打包自己的会方便很多，因此交易会提供费用给打包的人。

目前挖矿主要还是为了获得出块奖励，规定约10min出一个区块，出21万个区块就减半，因此出块奖励大约4年就减半。在很长时间之后出块奖励会变得非常小，那么交易费就会成主要奖励来源了

$$
T_{减半周期}=\frac {210000*10min}{60min*24h*365d}=3.995433
$$

与比特币这种$transaction-based\quad ledger$相对应的还有一种$account-based(基于账户的)\quad ledger$,**以太坊**就是用的此模式，这种模式下，系统显示记录每个账户上有多少个币。两种方式各有优缺点，前者显然隐私的保护会更好，但是代价就是要说明币的来源。

>创建时间：2020-11-04
>
>修改时间：2020-11-08

