---
permalink: 2023-12-07-lookups
---
# Twitter space: Lookups
## Twitter space 链接
<https://twitter.com/i/spaces/1gqxvQzwmbOJB>

## 快速笔记（不保证准确性）
### 郭老师
- 看起来简单，实际不简单
- 土耳其那边给大家讲了 landscape，前面都还算简单，到了 lasso 就有点绕了
- 但 lasso 可能不算 lookup，但可以拿来用 lookup
- lookup 算是目的，很多东西都可以做 lookup
- 多项式承诺也可以用来做 lookup
- 看起来容易聚合，实际也没有那么容易
- lookup 在大的电路里面有很多用法
- 总体上我觉得看起来不复杂，但协议细节还挺多的
- 应用场景要选哪种 lookup，还挺 tricky，很难有某个 lookup 可以一统江湖
- lookup 的性能比较也比较玄学
- lookup 的实现还比较少，大多在想法和实现层，实际用的时候还是用的比较老的东西
- lookup 最近没有新的论文

### 小熊
- sumcheck 和 GKR 属于 PIOP 这个类别
- cq 引用了 hab22，hab22 说自己叫 logup
- 最近看了在 decision tree 的应用，论文中的错误还挺多的，不过有三个贡献
	- 把 cq 改进了一点，加了 zk 属性，保持复杂度不变
	- matrix lookup
		- 我们平时 lookup，不关心表格的宽度
		- 和 vector 无关的 lookup
			- 我们 lookup 一行，会随机组合，通常不会很大，表格的宽度是线性的
			- 如果表格宽度是几十，我可以让复杂度和表格宽度无关
			- 应用场景在 zkml 中会用到
				- zkml 中一个 x 可能是 vector，有几十个特征
				- 这时候复杂度和 vector 长度线性有关的情况下，比较有用
	- 和 zkml 有关的 lookup
		- zkml 中不关心模型，只关心一个 valid 的输出和一个 proof
		- 这就可以使用 lookup argument
		- 比如：
			- prover commit 一个模型，对什么范围的输出，table 是模型的信息，是 commit 的
			- prover 需要提供 f(x) 的值，证明和 x 的关系
- 2+2 = 4 如何证明
	- 输入（2,2）和输出（4）都会放到表中，比如放到一行
	- 证明输入输出都在表的某一行中（lookup 到），就表示约束成立

### 0xhhh
- halo2 lookup
	- 有一个向量，希望去证明查询向量 f 是表格向量 t 中的
- plookup
	- 和 halo2 lookup 相比，不需要重排两个元素
- cq、caulk 相对这两个的优化在哪
	- 小熊
		- caulk 多了个预处理，使得查询的复杂度和表的大小无关
		- 这样可以搞很大的表
		- 自己主要看了 plookup 和 cq
- lasso
	- 对 t 用了 spark，拆成 sub table，变成低 degree 的多项式，可以做 commit
	- lasso 用到 GKR + spartan 的 spark，没有 get 精髓，想知道 GKR 的精髓在哪里

### 邹老师
- 最近没有看过 lookup
- ingonyama 写了个文章，做了个对比
- 很多 zkvm 项目用 logup，可以减少对 witness 承诺的开销
- lasso
	- lasso 实现和目标还有差距
	- lasso 想换方案，Justin Thaler 11 月份有个blog 谈这个事
		- 现在是基于曲线的承诺，可能要换，换成基于 binary shield 的，改 breakdown 和Ligero 的承诺

### Even
- 之前看过 plookup 和 flookup
- 之前看过 lasso
- 土耳其看了 caulk 和 caulk+
- 也在看 logup
- 感受
	- 看了 lasso 和 caulk 后，很多的协议，打开思路后都能用来做 lookup
	- 不过具体要看用到哪些场景下
	- 后面很多的 lookup 都可以用现有的协议去改做成 lookup
- 早期用 copy constraint
- caulk 用 kzg10 改改做 lookup
- lasso 用 spark 去做
- logup 和 cq 用 sumcheck
- 这些都用现有的东西，做一些现有的改动，然后去做 lookup
- 问题：dynamic lookup 是如何实现的
	- PSE zkevm 里面用过，lookup table 不一定是静态的数据，还可以是动态的

### 水滴💧-马
- 自己在研究一些 halo2 基础的协议
- 有个问题，看 halo2 的时候，有个 look any；lookup 的请求是和 table 对齐的，但 look any 是 witness 对 witness 的，不需要证明
	- 小熊：这是 fully zk
