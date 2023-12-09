# Lookup Arguments Overview

笔记整理自 2023.12.9 郭宇老师在 dapp learning 社区的分享
## 1. Lookup gate
- Lookup Arguments 现在变得特别重要，能解决很多棘手的问题
- 通常我们会证明 circuit，一般是加法或者乘法，每个门有两个输入和一个输出
- P 和 V 交互证明计算是对的，比如：
	- a + b = c
	- a * b = c
	- ![[Pasted image 20231209200944.png]]
- 有些运算需要比较高 degree 的门
- 新的门 lookup gate 可以解决这个问题
	- 可以把输入输出都放在表中
- 这样可以把电路中的某个门替换为 lookup gate
- 如果计算量不是很大，就可以把所有的运算放在 lookup table 中，这样计算可以转换为查表
	- ![[Pasted image 20231209201413.png]]
- 如何用这张表去 prove 这个门是正确的？
	- 把表合并成单个列的 table
	- 证明 f 属于 t
- Lookup argument：证明 f 属于 t
	- ![[Pasted image 20231209201441.png]]
- 使用场景
	- range check
	- bits operation
	- storage/memory

## 2. Plookup
- 证明 f 属于 t
- prover cost: O(NlogN)

## 3. Halo2-lookup
- 和 plookup 证明方法区别不大，不过更容易了解
- prover cost: O(NlogN)

到目前为止都还波澜不惊。

不过 2022 年发现了转机，Caulk 出现了。

## 4. Caulk
- 对前面的 lookup 做了很大的改进，可以优化 prover 的 cost（在 m << N 的情况下）
- prover cost: O(m^2 + mlogN)
- ![[Pasted image 20231209203226.png]]
- Caulk 基本的 idea是什么？为什么能做到 sub-linear 的 prover cost
	- cached quotients
		- kzg10: $f(x) - f(a) = (x-a) q(x)$
		- 构造一个通过 f(a) 和 f(b) 的 g(x)
	- extract subtable
		- ![[Pasted image 20231209203904.png]]
		- t(x) represents T
		- $t_I(x)$ represents S 属于 T
		- 则 F 属于 T
- Caulk 把证明过程拆成四个部分
	- ![[Pasted image 20231209205118.png]]

## 5. Caulk+
- 基本思想是替换掉 EQ4
	- $Z_I(X) Z_J(X)  = Z_V(X)$
- prover cost: O(m^2)，降低了
- 但是需要 commit J degree 的多项式
- ![[Pasted image 20231209205812.png]]

## 6. Flookup [GK22]
- 继续改进，prover cost from $O(m^2)$ 到 $O(mlog^2m)$
- 这样就有点实际的用处了
- Basic idea:
	- $t_I(X)$ 
	- $t_I(X)$  用来做 subtable
	- ![[Pasted image 20231209210310.png]]
- 缺点是它不是加法同态的，所以不能把表的列 merge 成更小的表。这样只能做简单的场景，比如 range check 和 membership check

## 7. Baloo [ZGKMR22]
- Prover cost: $O(mlog^2m)$, 保持不变
- 好处是支持加法同态
- cost：需要引入更多的 $G_2$ points
- ![[Pasted image 20231209210718.png]]

## 8. logup [Hab22]
- 最重要的贡献：引入 logarithmic derivatives（先取对数，再求导）
	- 好处是可以把连乘的关系转换成连加的关系
	- 这样既支持加法同态，性能也比较好
	- ![[Pasted image 20231209211254.png]]
- 问题：
	- 没有解决 caulk 要解决的问题，prover cost 和 t 表格的大小有关
	- 后面 CQ 可以解决这个问题

## 9. CQ
- Prover cost:  $O(mlogm)$
- 好处
	- 在 proof 中没有 $G_2$ 点，这样在合约中比较好验证
	- 解决了 logup 的问题，通过 pre-computation，让 prover cost 和 t 表格的大小无关
- ![[Pasted image 20231209211622.png]]
## 10. Lasso/Jolt
- 号称是 lookup singularity
- 但我很难把它归类到 lookup table
- idea 是把大表拆成小表
- 用了和 Baloo 很像的思想
- 不过对 M 和 t 有一定的要求
- ![[Pasted image 20231209211940.png]]