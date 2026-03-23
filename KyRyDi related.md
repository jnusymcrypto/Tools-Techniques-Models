
# KyRyDi

> From [2024-EC-A generic algorithm for efficient key recovery in differential attacks – and its associated tool](https://eprint.iacr.org/2024/288)
>
> Section 3 Modeling the key recovery problem
>
> 解释其所给出的 两个方法



差分密钥恢复攻击的原理见 []()

在此基础上, 对 SPN ciphers, 可以将差分密钥恢复视作 **有向图** 来构建模型, 其中 Sbox 作为 *node*, Sbox 之间的关系作为 *vertex*.

*e.g.* 对区分器前侧的扩展, $S_A$ 是第 $r$ 轮的 Sbox, $S_B$ 是第 $r+1$ 轮的 Sbox, 若 $S_B$ 依赖于 $S_A$ 的输出, 则有 $S_A\rightarrow S_B$.

一次猜测对应于一个 **子图**， 将问题转化为求

例子 (类 GIFT, 下述例子均为该例子)：

<font color=blue> 蓝色 </font> 为未知差分, 白色 (三角) 为固定差分.

<img width="576" height="470" alt="image" src="https://github.com/user-attachments/assets/f8acee08-8124-4867-8335-a6a1712aa5a6" />

<img width="416" height="415" alt="image" src="https://github.com/user-attachments/assets/75c91dc4-ac2a-4913-8611-581f5e9f1867" />

**定义: *solution***

假设密钥恢复阶段的 某个 Sbox 必须满足差分转移 $v_{in}\rightarrow v_{out}$. 一个 *solution* 是 一个 tuple $(x,x',S(x),S(x'))$, 其中 $x\oplus x'=v_{in}$, $S(x)\oplus S(x')=v_{out}$. 所以 *input solution* 是使 上述 *solution* 成立的 $(x,x')$, *output solution* 同理.

例子：图中蓝色对应的 solution 包括 $S_{0,1}, S_{0,3}$ 的 solution 和 $S_{2,0}$ 的 solution (左侧 2 bit 截断).

### Method 1. Pre-sieving

- principle

	通过存储 Sbox 所对应的 input solution, 其对应于明文对数量为 $N$ 在此处的空间大小, 对应到 output solution, 会有一部分输入不满足, 从而可以提前筛掉部分 pairs. 过滤后明文对数量变为 $N'$.

* example

	针对上图 $S_{0,0}$, 存储其无密钥加部分的取值, 有密钥加部分的差分, 记为 $((x_3,x'_3),(x_2,x'_2),x_1\oplus x'_1,x_0\oplus x'_0)$ 共 6 bits, 对应于 $N$ 中的 $2^6$, 但实际上能匹配上 output solution 的仅有 $36<2^6=64$ 个, 因此可以产生 $\frac{36}{64}$ 的筛选 (过滤) 效果.



*但该过滤如果仅针对单个 Sbox (或过滤掉的部分密钥一步内全猜测) 无意义.* 因为在猜对应位置的密钥之后依然可以进行筛选, 且这对该步骤的复杂度没有影响. 所以需要想办法使其有效, 即 **compensation**:

compensation: 先对所有输入已知的 Sbox 都进行过滤, 然后再将猜密钥的过程分段处理, 这样复杂度就会基于 $N'<N$, 使得起始的复杂度降低, 从而很可能后续复杂度都有所降低.

此外，该 pre-sieving 对 已知输入差分 的情况均有效，即使不在攻击的 首尾轮.

### Method 2. Precomputing partial solutions

* principle

  预计算一个 **子图解** 的所有可能取值，并将其存入 hash 表，用存储换取快速的查表时间复杂度.

* example

  计算 $S_{0,0}, S_{0,2} \rightarrow S_{1,0}$ 的总输入解数量.

  * 对上图的 $S_{1,0}$, 其对应的差分传播为 $(????)\rightarrow(1?00)$
  * 总数据量 (可能性) 有 $2^8$, 有 3 bits 差分可用于过滤, 所以有 $2^{8-3}=2^5$ 种可能的输出解
  * 又因为 Sbox 的 输入--输出 唯一性, 所以输入解数量也为 $2^5$, 形式为 $((x_3,x'_3),(x_2,x'_2),x_1\oplus x'_1,x_0\oplus x'_0)$.
  * 因为 $S_{1,0}$ 左侧 2 bits 来自 $S_{0,0}$ (值固定), 所以 $S_{0,0}$ 左侧 2 bits 不增加总输入解; 而 $S_{0,0}$ 右侧 2 bits 为固定差分, 值可取输出解空间为 $2^{4-2}=2^2$.
  *  因为 $S_{1,0}$ 右侧 2 bits 来自 $S_{0,2}$ (差分固定), 所以 $S_{0,2}$ 右侧 2 bits 输出解空间为 $2^{4-2}=2$; 而 $S_{0,0}$ 左侧 2 bits 为固定差分, 值可取输出解空间为 $2^{4-2}=2^2$; 所以 $S_{0,2}$ 输出解总空间为 $2^{2+2}=2^4$, 也为输入解总空间.
  * 所以 $S_{0,0}, S_{0,2} \rightarrow S_{1,0}$ 的总输入解空间为 $2^{5+2+4}=2^{11}$.

  而 $S_{0,0}, S_{0,2}$ 的总输入解空间为 $2^{6+6}=2^{12}$, 形式为 $((x_3,x'_3),(x_2,x'_2),x_1\oplus x'_1,x_0\oplus x'_0)$, 所以建表时会产生 $2^{11-12}=2^{-1}$ 的过滤.

  假设该步骤 (查表) 之前的对数为 $N''$, 且上述 compensation 为 $2^{0.84}$ (即 Method 1 的一部分过滤, 的倒数), 则查表的时间复杂度为 $N''\cdot 2^{0.84}\cdot 2^{-1}$, 需要存储 $2^{11}$ 大小的 hash 表, 每个表中存 $S_{0,0}, S_{0,2}$ 的输入 ($4\times 2\times 2=16$ bits) 与相应 3 个 Sbox 的密钥 (6 bits), 共 $22$ bits. 

  若不查表, 则相应的时间复杂度为 $N''\cdot 2^{0.84}$, 因为未算到 $S_{1,0}$ 所以过滤没法用.
