

# KyRyDi

> From [2024-EC-A generic algorithm for efficient key recovery in differential attacks – and its associated tool](https://eprint.iacr.org/2024/288)
>
> Section 3 Modeling the key recovery problem
>
> 解释其所给出的 三个方法



差分密钥恢复攻击的原理见 []()

在此基础上, 对 SPN ciphers, 可以将差分密钥恢复视作 **有向图** 来构建模型, 其中 Sbox 作为 *node*, Sbox 之间的关系作为 *vertex*.

*e.g.* 对区分器前侧的扩展, $S_A$ 是第 $r$ 轮的 Sbox, $S_B$ 是第 $r+1$ 轮的 Sbox, 若 $S_B$ 依赖于 $S_A$ 的输出, 则有 $S_A\rightarrow S_B$.



例子：

<img width="576" height="470" alt="image" src="https://github.com/user-attachments/assets/f8acee08-8124-4867-8335-a6a1712aa5a6" />

<img width="416" height="415" alt="image" src="https://github.com/user-attachments/assets/75c91dc4-ac2a-4913-8611-581f5e9f1867" />

**定义: *solution***

假设密钥恢复阶段的 某个 Sbox 必须满足差分转移 $v_{in}\rightarrow v_{out}$. 一个 *solution* 是 一个 tuple $(x,x',S(x),S(x'))$, 其中 $x\oplus x'=v_{in}$, $S(x)\oplus S(x')=v_{out}$. 所以 *input solution* 是使 上述 *solution* 成立的 $(x,x')$, *output solution* 同理.



