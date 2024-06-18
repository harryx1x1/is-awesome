# BitVM 介绍

这篇笔记基于 Robin Linus 在 ZKProof 会议上的分享, 视频链接: <https://www.youtube.com/live/VIg7BjX_lJw?si=djNaeeufQ6Pq0oIl>

由于我也在学习, 为方便更好的理解, 结合视频我加了一些我理解的内容, 这部分内容在笔记中我不做特别说明, 如果发现错误, 那肯定是我的错误, 感谢向我指出.

 ![[Pasted image 20240529113645.png]]

 这个分享的主题是 BitVM, 让 Bitcoin 拥有更加 smarter 的 contracts.

Motivation: 希望比特币可以被数十亿的人使用, 但目前还没有实现, BitVM 希望在这里有所帮助.

未来人们可能不会承担得起主网的交易费, 所以可能会增加区块大小以容纳更多的交易从而可以减少手续费, 但是增加区块大小会损害去中心化的程度, 因为节点的存储会增加, 验证区块的运算也会增加, 让一些配置低的机器难以运行节点, 所以区块大小也不会增加太多(比如不会增加到 GB 级别).

![[Pasted image 20240529114712.png]]

所以长期来看, 用户还是负担不了主网的费用, 这就让一些 sidechain, zk-rollup, zkCoins 等二层方案变得有用了. 另外在二层上, 可以快速和便宜的试验一些创新. 在这个理想的世界中, 我们希望所有的二层方案都用 Lightning 互相连接. 但是问题来了, 我们如何更好的构建这些 bridge 呢? 这涉及到我们如何用去信任或者信任最小化的方式把 BTC 跨到二层上, 而当前跨链的方式利用的是多签托管用户的资金, 显然这并不满足去信任的要求.

![[Pasted image 20240529115303.png]]

BitVM 实际上一个基于比特币的 hack, 技术栈如上图所述, 可以看到 BitVM 引入了带状态的脚本(Stateful scripts), 基于这些脚本构建了 BitVM, 基于 BitVM 构建了 Bridge. 这个分享也是围绕这个技术栈来讲的.

## Stateful scripts

BitVM 背后的核心思想是通过脚本引入状态. 有两种方式实现这个功能.

### 利用签名实现状态

![[Pasted image 20240529162206.png]]

比特币中没有状态的概念, 即没法在两个交易/脚本中保证用到的两个值是相等的. 举例来说, 对于  script1 和 script2, 它们都用到了 x = 42 这个值, 但在逻辑上无法约束 script1 和 script2 中用到的 x 值是相等的.

因为签名有对消息承诺的特性, 所以如果可以给 x = 42 进行签名, 则可以实现签名和消息的绑定, 这样持有这个签名就意味着"携带"了签名所承诺的消息, 就可以实现状态的功能. 比如 Alice 签名 x = 42, 生成签名 sig1, 则 Bob 或者任何人都可以在 script2 中使用 sig1, 这样就实现了状态功能.

如何用比特币脚本验证栈上任意值的签名?

目前主网没有操作码支持. 目前只支持对整个交易进行验签. 社区提议的 CSFS(Check Signature From Stack) 操作码可以做到对栈上的任何值验签, 但主网尚不支持.

还有其它的办法吗?

有. 解决方案是利用 Lamport 签名.

![[Pasted image 20240529164625.png]]

Lamport 签名概念上非常简单, 仅利用了 hash 函数, 可以利用比特币脚本将 Lamport 签名实现出来. 不过 Lamport 签名的缺点是秘钥和签名都占了太大的尺寸. 但 Lamport 签名依然非常有用, 可以用来对不同的数据结构进行签名, 比如 u8, u32 和 u160.

下面看看如何利用 Lamport 签名验证 1-bit 的消息. 详细原理可以参考我的[Bit commitment 笔记](https://harryx1x1.fun/2024-05-28/bit-commitment/).

![[Pasted image 20240529165434.png]]

利用上图中的比特币脚本:

- 如果提供数字 1 对应的签名(hash1 的原像)来执行脚本, 则签名会验证通过, 栈上会留下 1 这个值.
- 如果提供数字 0 对应的签名(hash0 的原像)来执行脚本, 则签名会验证通过, 栈上会留下 0 这个值.
- 如果提供其它值来执行脚本, 则脚本验证失败

因为这个脚本中包含 hash1 和 hash0 的各 20 字节, 再加上原像的 20 字节, 因此每个 bit 的 commitment 需要对应 60 个字节的数据, 这个尺寸比较大. 利用 Winternitz 签名可以把尺寸降低到 25 字节, 但是尺寸依然很大.

### 利用 Connector Outputs 实现状态

还有另外一种实现状态功能的方法, 叫做 Connector Outputs.

基本思想可以用下面的例子来阐述. Alice 提前签好了一笔交易, 承诺如果 Conditional Tx 可以将 Condition Tx 中的  `Connector` output 作为输入, 则 Bob 可以收到 Alice 的 10 BTC.

![[Pasted image 20240529170537.png]]

Connector Output 在上图的例子中指的是图中下部分蓝色线指示的 `Connector`, 这个 `Connector` 是上面两笔交易的连接器, 因为只有将 `Connector` 作为 input 输入到右侧的 Conditional Tx 中, Bob 才能收到 Alice 的 10 BTC.

![[Pasted image 20240529171304.png]]

在这里带有状态功能指的是, 如果有另外一笔交易 Alternative Tx 先将 `Connector` 作为 input 输入并成功发送了交易, 则 Bob 再也不能通过执行 Conditional Tx 来收到 Alice 的 10 BTC 了.

## BitVM 架构
![[Pasted image 20240529173218.png]]

BitVM 利用的机制是乐观的计算(Optimistic computation), 只有发生争议的时候才执行昂贵的操作, 正常的交易只是一笔普通的交易. BitVM 主要目的不会做具体的计算, 而是通过简洁的机制拒绝错误的交易, 保证状态的正确转换. 这一点和 ZKP(零知识证明)的机制很像, BitVM 也正在构建 SNARK verifier 用来验证通用的计算.

## Advanced Bitcoin Scripts

![[Pasted image 20240529174637.png]]

BitVM 团队也在构建一种高级脚本语言, 可以看做是比特币脚本语言的元语言(meta language), 可以相对容易的实现更复杂功能的比特币脚本.

比如可以展开循环和构建函数, 可以组合不同的 opcode. 利用这个语言实现了自己的 hash 函数. 也实现了 Lamport 签名和 Connector outputs. 还能实现更复杂的脚本, 比如复杂的 Taptree, 以及大型的 TX graph.

## BitVM Bridges

![[Pasted image 20240529174838.png]]

Bridge 的作用是可以把 BTC 带到别的系统中, 虽然目前 BitVM 的系统有些笨重, 但它仍然是有用的, 可以给大额资金做相对安全的跨链操作, 而对于普通用户的小额 swap 而言, 可以使用闪电网络(LN) 来完成. BitVM 的一个主要限制是 Operator 是固定的一组人, 不能随时替换, 而验证者可以是任何人. 这样就提供了很强的安全保证, 因为就算所有的 Operator 都作恶, 也不能拿走用户的钱.

Trusted Setup 阶段可以有最多 1000 个 Operator 对 TX graph 进行签名, 之后签名者需要删除丢弃自己签名所用的私钥, 只要保证有一个 Operator 丢弃了自己的私钥, trusted setup 就是安全的(safe).

总计有 100 个 Operator, 只需要有一个诚实的 Operator 正常行事, 就可以保证用户的 Peg out(出金)正常进行(live). 如果一方本身是流动性提供方, 那他可以参与 trusted setup ceremony, 那这个 bridge 对于他来说就是 trustless 的了.

![[Pasted image 20240529182234.png]]

上图显示了一个简化版本的 Bridge 的工作流程

- Peg in: Alice 发送 100 BTC 到多签地址, 在 sidechain 生成 100 Wrapped BTC 发送到 Bob 地址
- Peg out: 
	- Bob 在 sidechain burn 100 Wrapped BTC, 希望在比特币主网收到 100 BTC
	- Operator 观察到 Bob 的 burn 交易, Operator 把他自己的钱给 Bob
	- Bob 直接取走钱
	- Operator 希望取走 100 BTC, 对应的交易有多个输入:
		- 第一个来自于 Alice 的 Peg in 交易
		- 第二个来自于 Kick off 交易中的 delay output, 达成的效果是直到 Kick off 交易提交到链上的 6 个月后, Operator 才能用此脚本解锁拿到 100 BTC, 目的是留下足够的时间, 防止作恶的可能
		- 第三个来自于 Kick off 交易中的 Connector output. 这个 Connector output 也是被 Operator 签名过的, 这样如果 verifier 发现问题就可以通过 Slash 交易销毁 Connector output, 导致 Peg out 交易没法执行, Operator 无法取回 100 BTC, 达到惩罚的目的

![[Pasted image 20240529223856.png]]

目前 BitVM 使用的是基于配对的 SNARK, 具体用的是 fflonk. 链上 verifier 是用脚本来实现的, 尺寸非常大, 至少有几 GB. 然而一个区块中脚本最大只能为 4 MB, 没法一次性执行整个 verifier 程序, 验证某个计算是否正确. 所以 BitVM2 采用的方式是将一个大的计算拆分成多个小的计算, 把上一步得到的结果作为下一步的参数继续执行, 这样执行到最后可以获得正确的最终结果. 这样就需要 commit 多个中间结果, 比如 1000 个. 假设我们想计算 $f(x) = y$, 但是 $f$ 函数太大没法没法直接计算, 我们把计算过程拆成 1000 份, 然后 commit 中间结果:

- $f_1(x) = z_1$
- $f_2(z_1) = z_2$
- $f_3(z_2) = z_3$
- ...
- $f_{1000}(z_{2999}) = y$

每个 $f_i$ 都相对小且可以计算的. $f_i$ 最大可以为 4 MB, 1000 个相当于 4 GB 的脚本大小. 

如果发现问题, 只需要找到其中某一步执行错误就行了, 例如如果 $f_{42}(z_{41}) \neq  z_{42}$, 那整个验证就会失败.

因为执行一个 4 MB 的脚本就能验证某一步执行的错误, 这样就实现了可以在一个区块中验证几 GB 大小脚本代表的程序.

![[Pasted image 20240601122137.png]]
上图是一个更具体一些的 Bridge 如何工作的解释:

- Peg in: Alice 发送 100 BTC 到多签地址, 在 sidechain 生成 100 Wrapped BTC 发送到 Bob 地址
- Peg out: 
	- Bob 在 sidechain burn 100 Wrapped BTC, 希望在比特币主网收到 100 BTC
	- Operator 观察到 Bob 的 burn 交易, Operator 把自己的钱给 Bob
	- Bob 直接取走钱
- 剩下就看 Operator 这边的逻辑
- Operator 希望取走 100 BTC, 把钱取走的逻辑, 有两个分支可以做到:
	- Happy path 1: Kick off -> Take 2 ->  Operator 拿走 100 BTC
		- 如果没有任何人来 dispute, 则两周后可以取走 100 BTC
	- Happy path 2: Kick off -> Assert -> Take 2 -> Operator 拿走 100 BTC
		- Kick off: 
			- 如果有人怀疑 Operator 作恶, 则可以强制要求 Operator 将发送给 Bob 100 BTC 的 SNARK 证明的中间执行结果发到链上. 
			- 强制的方式是获取 Connector A 的所有权, 让 Operator 没法利用 Connector A 作为 input 成功执行 Take 1 交易进而取走 100 BTC
			- 如何获取 Connector A 的所有权? 可以直接购买. 由于 Connector A 是由 Operator 签名的, 这个 output 可以由任何人花费(利用了 ANYONECANYAP 操作码), 只要给支付 Operator 1 BTC 任何人都可以获得这个 output 的所有权. 这 1 BTC 为了支付 Operator 在下面挑战中(Assert)需要花费的成本
		- Assert: 现在 Operator 只能尝试让 Take 2 交易成功才能得到 100 BTC. 由于 Take 2 交易依赖 Assert 交易的 output, 而 Assert 需要 Operator 提交 SNARK 证明的中间执行结果作为输入和输出才行, 所以 Operator 一定会配合做这个动作
		- 如果没有人 Disprove, 则 Operator 过两周后可以取走 100 BTC
- Operator 被惩罚的逻辑
	- Unhappy path: Kick off -> Assert -> Disprove -> Connector C 被花费 -> Operator 无法花费 100 BTC
		- 任何人都可以执行 Disprove 交易
		- 如果 Disprove 交易成功, 则 Connector C 被花费(burn), Take 2 交易无法被执行, 导致 Operator 无法花费 100 BTC

上面的过程就是 Bridge 如何工作的核心原理.

### Bridge 的限制

![[Pasted image 20240530000751.png]]

- 复杂
	- 脚本实现起来很复杂
	- SNARK verifier 的脚本很大
- 平衡激励
	- Loser 始终应该支付 winner 手续费和 bounty
	- 然而具体奖惩的数额很难计算, 因为这些数值是在设置阶段就确定好的, 后续没法更新. 如果后续收费费变化太多, 可能 Disprove 过程需要更多的手续费
	- 如果奖惩机制比较有效, 那 Disprove 也不会发生, 因为作恶在经济上不值得. 这一点在 Optimistic 系统上有很好的证明
- Operator 必须垫付一笔 Peg out 的资金, 这笔资金会被锁定两周. 不过好的方面是, Operator 不需要 1:1 的抵押
- 对于每一笔 Peg in 交易, 1000 个 cosigner 必须预先签署大约 100 个 Peg out 交易，每个 Operator 持有其中的一个
- 这 1000 个 cosigner 可以对 Peg in 交易进行审查, 拒绝继续执行. 不过他们没法审查 Peg out 交易

## 总结

![[Pasted image 20240530001936.png]]

- BitVM 为比特币开启了更加智能的合约功能.
- 使用场景: 目前看来主要是用于 Layer 2 的 Bridge
- BitVM 是实用的, 只是有些臃肿
- 好的地方: 不需要软分叉就能实现 BitVM
	- 如果有几个操作码支持会更好: TXHASH, OP_MUL, OP_BLOCKCHAIN
- 希望年底可以上线主网