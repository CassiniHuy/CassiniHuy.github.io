---
title: Overview on Techniques in OT (Oblivious Transfer) and Its Extension
date: 2025-09-03 01:08:33
categories: [随笔]
tags: [安全多方计算, OT, IKNP]
mathjax: true
---


这里从一个较为通顺的视角去理解OT及其相关的拓展(extension)技术，包括OT, ROT, COT, IKNP OT extension，以及简单解释VOLE如何用以解决OT extension问题。

Ref:

Kumar, Nishant. "Techniques in OT extension." (2020).

Bellare, Mihir, and Silvio Micali. "Non-interactive oblivious transfer and applications." Conference on the Theory and Application of Cryptology. New York, NY: Springer New York, 1989.

Ishai, Yuval, et al. "Extending oblivious transfers efficiently." *Annual International Cryptology Conference*. Berlin, Heidelberg: Springer Berlin Heidelberg, 2003.

Peter Scholl. "[Vector Oblivious Linear Evaluation,PCGs and Correlated Randomness.](https://csrc.nist.gov/csrc/media/Presentations/2023/mpts2023-day3-talk-gadgets-vole-pcg/images-media/mpts2023-3a4-slides--scholl-vole-pcg-cr.pdf)" *NIST MPTC Workshop*. 28 September 2023.


<!--more-->

# Base OT (Oblivious Transfer)

### OT with Semi-Honest Adversary

OT是一个这样的两方安全计算问题：Sender有两个秘密消息，记为$x_0,x_1\in\{0,1\}^l$，这里$l$是消息的长度；Receiver有一个秘密比特$b\in\{0,1\}$。如何实现，Receiver在不泄露$b$而得到$x_b$的同时，Sender不泄露$x_{1-b}$？

分解来看，目标是Receiver得到他对应想要的那个消息$x_b$；同时保证Sender另一个消息$x_{1-b}$对于Receiver不可知，以及Receiver的选择$b$对于Sender不可知。

已经证明，OT很难归约到一个单向函数上；否则，相当于证明了$P\neq NP$。所以OT能用对称密码（其归约到一个假设存在的单向函数）建立起来的希望很小。并且，已经证明了OT等价于密钥交换，所以实际上相当于说OT只能用公钥密码实现，不可能基于对称密码。

下面是一种基于离散对数密码实现OT的思路：

1. Sender生成一个公钥$C$，然后发送给Receiver。
2. Receiver随机生成一对公私钥$sk, pk$，然后利用$C$得到$pk'=C\cdot pk^{-1}$；若$b=0$，则发送$(pk,pk')$，否则发送$(pk',pk)$。记Sender收到的公钥对为$(pk_0,pk_1)$。
    
    > 注意到利用$C$得到的$pk'=C\cdot pk^{-1}$在离散对数的群里也是一个公钥。
    > 
3. Sender用$pk_1,pk_2$去分别加密$x_0,x_1$，得到$ct_0=enc(x_0,pk_0),ct_1=enc(x_1,pk_1)$，发送给Receiver。注意到Sender不知道哪个公钥才是Receiver的$pk$。
4. Receiver通过$dec(ct_b,sk)$得到$x_b$。
    
    > 这里Receiver无法得知另一个消息$x_{1-b}$，因为他不知道$pk'$对应的私钥（这基于离散对数问题的困难性，如果Receiver能同时知道$pk$和$pk'$的私钥，即对应的离散对数，那他也能知道$C=pk\cdot pk'$对应的私钥）。
    > 

拆开来说安全性：一方面，公钥加密保证了已知公钥，无法解密密文，这用以保证Receiver无法知道Sender的另一个消息；另一方面，Sender分不清$pk_0$和$pk_1$哪个才是$pk$。

### OT with Malicious Adversary

注意到，上面是半诚实假设，即假设双方忠实执行协议。实际上，如果Receiver自己生成两个公私钥对，发送其中的两个公钥给Sender，他就可以利用对应的私钥解密所有Sender的消息。

OT可以在恶意模型上实现，这需要保证Receiver无法做到上面这一点。

例如，通过离散对数问题，在一个素数阶$q$的循环群$G$上进行，使用的生成元记为$g$：

1. Sender随机采样$C\in G$，发送给Receiver。
2. Receiver随机采样整数$k\in\mathbb{Z}_p$，得到$pk_b=g^k,pk_{1-b}=C\cdot g^{-k}$，发送$pk_0$给Sender。
    
    > 注意到，由于离散对数问题的困难性，从$g^k$中是得不到$k$的值的。
    > 
3. Sender得到$pk_0$后，计算$pk_1=C\cdot pk_0$；随机采样两个数$r_0,r_1\in \mathbb{Z}_p$，计算得到$ct_0=g^{r_0},ct_1=g^{r_1}$，然后用一个随机Oracle（记为$H$）去加密两个消息$x_0'=H(pk_0^{r_0})\oplus x_0, x_1'=H(pk_1^{r_1})\oplus x_1$，然后发送两组数$<ct_0,x_0'>,<ct_1,x_1'>$给Receiver。
    
    > 注意到，这里Sender得到了$pk_0,pk_1$，但不知道哪个是$g^k$，所以$b$对其不可知；这里$H$的作用是把循环群中元素拉到和消息一样的比特长度（用一个hash函数）；由于离散对数问题的困难性，Receiver收到这两组数之后，无法求解得到其中的$r_0,r_1$。
    > 
4. Receiver计算$x_b'\oplus H(ct_b^k)$从而得到$x_b$（注意到$ct_b^k=g^{r_bk}=pk_b^{r_b}$）。
    
    > 关于另一条被加密的消息$x_{1-b}$的安全性，其加密所用的随机Oracle种子是$pk_{1-b}^{r_{1-b}}=(pk_{1-b})^{r_{1-b}}=(C\cdot g^{-k})^{r_{1-b}}$，求解这个值是困难的（这基于Diffie-Hellman假设，即假如已知$g^a,g^b$，求解$(g^{a})^b=g^{ab}$是困难的）；同时注意到，除非$C$恰好等于$g^k$（概率极低），否则$C\cdot g^{-k}$是作为这个素数阶循环群的生成元的，那么$pk_{1-b}^{r_{1-b}}$的可能取值范围等同于这个群元素的数量。
    > 

> 在读论文查资料的多次中发现，在89年Bellare-Micali原文中使用的并不是一个素数阶的群，而只是$\mathbb{Z}^*_q$，所以原文中随机数如$k,r_0,r_1$的取值范围为$0$到$q-2$，因为生成元的阶为$q-1$而不是$q$。同时，使用这个群也没法保证$C\cdot g^{-k}$是生成元，从而存在$pk_{1-b}^{r_{1-b}}$落入一个小的子群而被攻击的可能。而素数阶的循环群可以解决这个问题（因为非单位元都是生成元），较新的大多数资料和论文也都是基于素数阶的循环群讨论这个协议。
> 

### 1-out-of-n OT

上面介绍的称之为1-out-of-2 OT，可以进一步地实现1-out-of-n OT，即Sender有$n$的消息，记为$x_0, x_2,\cdots,x_{n-1}$，然后Receiver从中选一个。在半诚实假设的情况下，这可以简单地用$n$次1-out-of-2 OT实现。比如在第$i$次Sender发送他的$(x_{i-1},x_i)$给Receiver，然后Receiver在其中选择他所需要的；但这无法防止一个恶意的Receiver窃取其他消息。

一个可能恶意的Receiver的情况下，也可以构建安全的1-out-of-n OT。实际上可以拓展一下上面所述的Bellare-Micali协议，只需要将其中的两对公钥相应拓展为$n$个。另一种方式是借用$n$次任意的黑盒1-out-of-2 OT实现，只需要每次发送下面的消息：

$$
(r_0, x_0)\\
(r_1, r_0\oplus x_1)\\
\cdots\\
(r_i, r_0\oplus r_1\oplus \cdots\ r_{i-1}\oplus x_i)\\
\cdots\\
(r_{n-1}, r_0\oplus r_1\oplus\cdots r_{n-2}\oplus x_{n-1})
$$

Receiver解开某个$x_i$必须要得知之前所有的随机数$r_0,\cdots,r_{i-1}$，这样他就无法得知前面的消息；同时，拿到这个$x_i$也就意味着拿不到$r_i$，也就得不到后面的消息。

### OT is Complete for MPC

OT应该怎么用呢？乍一眼看起来它所实现的并不是一个很通用的功能。实际上，OT已经被证明对于MPC(Multi-Party Computation)是完备的，也就是说OT可以用来算任何电路。最朴素的一个方法是：设这个电路的真值表有$n$个bit输入，$m$个比特输出，这个电路每一个输出比特的计算，都可以用一个1-out-of-$2^n$ OT来实现。

---

可以看到，1-out-of-2 OT是构成安全计算的一个基本单元，下面我们就关注这个问题。

# Random OT and Correlated OT

### ROT (Random OT)

所谓ROT即是Sender发送的两个消息是（伪）随机消息的情况（记为$r_0, r_1\in \{0,1\}^l$）。

如果我们需要$m$次发送$l$比特的OT（记为$OT^m_l$），我们可以通过$m$次发送$l$比特的ROT实现（记为$ROT^m_l$）；只需要每次OT基于一个ROT来实现即可。比如先进行一次ROT，Sender发送$r_0,r_1$，Receiver的选择是$b$；然后Sender发送两条加密消息$r_0\oplus x_0,r_1\oplus x_1$，Receiver使用对于的$r_b$解密即可获得对应的消息。

上面的例子需要ROT和后续消息发送基于一样的Receiver选择$b$，实际上可以基于不同的Receiver选择实现；这样可以将计算量大的公钥加密提前到双方各自的输入被决定前进行。实现的方式也很简单，假如ROT阶段的Receiver随机使用一个选择$a\in\{0,1\}$，在实际发送消息阶段，Receiver再发一个$c=a\oplus b$给Sender；如果$c$是1，说明Receiver选择变了，Sender调换一下消息的顺序，变为$(x_1,x_0)$，否则不变。注意这里并不会泄露$a$或$b$，Sender仅仅通过$c$看不出Receiver每次具体的选择是什么。

### COT (Correlated OT)

实际上上述的从OT转化到ROT，并没有降低问题的复杂度（但可以将公钥密码的计算提前）。这里我们接着将ROT转化为COT，后者可以将$ROT^m_l$转化为$COT^m_k$，其中$k$是安全参数，例如128；对于发送长消息来说，可以实现$k\ll l$的效果。

COT也是一种Sender发送消息的特殊类别，即两条消息是correlated，而不是互相独立的；一种常见的情况是，Sender不是发送两个独立的随机消息，而是$(r,r\oplus \Delta)$；其中$r$是随机比特串，$\Delta$是某个常量，他们都是$k$比特的消息。Sender的两条关联消息以及Receiver收到的那个消息，可以再经由一个随机Oracle把消息从$k$比特拉到$l$比特即可；这个Oracle是一个对correlated输入鲁棒的hash函数/伪随机生成器。

如果仔细观察上述的COT，似乎这里所用的correlated输入并无必要；使用ROT传输两条$k$比特的消息，然后用某种hash函数拓展到$l$比特也能实现上述功能。但是实现COT所依赖的条件更少，这意味着OT可以依赖“更少的”随机性来实现，即Sender不需要保证两条消息是互相独立的。例如，我们再观察一下上面详细讨论过的OT协议，其中发送的$(pk,C/pk)$，实际上就是一种COT；注意到$C$是公开的，只有$pk$以及两条消息的顺序是私有的，这依然能够实现OT。

除此之外，由于这一点，实际上构造COT会比构造ROT更易实现，即COT可以通过更高效的密码原语来做；例如VOLE, Vector Oblivious Linear Evaluation（后文会简略介绍）。

---

上文核心就是，OT可以转化为ROT，ROT可以转化为更易实现的COT，所以如果我们能安全地、低成本地构造COT，我们就能对应实现OT。

# OT Extension

上面说到，已经证明OT只能基于一个困难问题构建，而不能**仅**凭单向函数实现。但可以基于少量的这样的基于公钥的Base OT，然后利用单向函数将其拓展为大量的OT，称之为OT Extension。其中最经典的一个方法便是IKNP Extension (IKNP没有什么特别含义，就是论文作者首字母组合) ，能做到将$OT^m_l$通过$COT^k_m$实现。

### 一种“反向”COT

假设Sender拥有私有的$\Delta\in\{0,1\}^k$，Receiver拥有私有的$r\in\{0,1\}^k, b\in\{0,1\}$；考虑这样一个COT：Sender的消息对是$(r\oplus b\Delta, r\oplus \bar{b}\Delta)$。注意到这个COT的性质：对于Sender而言，这个消息对可能是$(r,r\oplus\Delta)$或者$(r\oplus\Delta,r)$；对于Receiver而言，无论$b$的取值，它获取到的一直是$r$，并且$r\oplus \Delta$对于他是未知的。

但是如何实现这样的COT呢？我们无法让Sender来决定这个消息对，因为需要保证$r$和$b$不泄露。这里我们考虑一种“反向”的COT：

1. Receiver先随机采样得到$r\in\{0,1\}^k, b\in\{0,1\}$，然后让Receiver作为发送方，逐比特地用OT发送消息$(r[i],r[i]\oplus b),0\le i\le k-1$给Sender。
2. Sender作为OT的接收方用$\Delta[i]$作为自己的选择；于是Sender收到的每个比特就是$r[i]\oplus b\Delta[i]$，$k$次OT接收到的比特拼起来，也就是$r\oplus b\Delta$，另一个消息可以通过$r\oplus \bar{b}\Delta=r\oplus b\Delta\oplus\Delta$计算得到。

这样就实现了具有上述性质的COT。Sender具有$(r\oplus b\Delta, r\oplus \bar{b}\Delta)$，且不知道消息的顺序（也就是不知道$b$和$r$）；Receiver得到Sender消息中的$r$，且不知道另一个消息（因为$\Delta$对其未知）。

### IKNP Extension：由$COT^k_m$到$ROT^m_l$

注意到上面的COT实际上是$k$次传递$1$比特消息的OT（即$COT^k_1$），只需要将传递消息的长度拓展到$m$来进行批量传递，也就是进行$COT^k_m$：

- 也就是Receiver有$m$对这样的$r$和$b$，这里重新定义为$r_0,\cdots,r_{m-1},b\in\{0,1\}^m$（注意到这里的$b$是$m$比特的了）；
- $k$次OT中，Receiver作为发送方的消息对是$(r_0[i]||\cdots||r_{m-1}[i],r_0[i]\oplus b[0]||\cdots||r_{n-1}[i]\oplus b[m-1])$，然后Sender作为OT接收方用$\Delta$的每一位作为选择，和上面一样。

经过$k$次OT，Sender会收到$k$个$m$长度比特串，只需要将每个比特串对应位置的比特取出来，拼接在一起，即会得到$m$个长度为$k$的比特串，即$r_j\oplus b_j\Delta,1\le j\le m-1$。

下面将这个过程写出来直观说明一下，为了简洁，这里用$q_{j,i}$表示Sender收到的每个比特$r_j[i]\oplus b_j\Delta[i]$，即第$i$次OT收到的比特串中的第$j$位，Sender收到的这$k$个比特串是:

$$
q_{0,0}||q_{1,0}||\cdots||q_{m-1,0}\\
q_{0,1}||q_{1,1}||\cdots||q_{m-1,1}\\
\cdots\\
q_{0,k-1}||q_{1,k-1}||\cdots||q_{m-1,k-1}
$$

看成一个比特矩阵，每一列取出来就是：

$$
r_0\oplus b_0\Delta =q_{0,0}||q_{0,1}||\cdots||q_{0,k-1}\\
r_1\oplus b_1\Delta =q_{1,0}||q_{1,1}||\cdots||q_{1,k-1}\\
\cdots\\
r_{m-1}\oplus b_{m-1}\Delta =q_{m-1,0}||q_{m-1,1}||\cdots||q_{m-1,k-1}
$$

和上面一样，再加上一个$\Delta$就会得到$r_j\oplus \bar{b_j}\Delta$。

这样，经过这种“反向”的$COT^k_m$，结果上，我们实现了$COT^m_k$：Sender得到$m$对$k$比特的消息$(r_j\oplus b_j\Delta,r_j\oplus \bar{b_j}\Delta)$，重新记为$(m^0_j,m^1_j)$，并且他们对于$\Delta$关联；然后Receiver拥有其中一个消息$r_j$，重新记为$m_j^{b_j}$。

再故技重施，用一个随机Oracle将这些$k$比特的消息，拉到$l$比特，就实现了$ROT^m_l$。

---

再捋一下IKNP如何拓展OT：基于少量（$k$次）Base OT实现大量（$m$次）OT的问题转化过程是$OT^m_l\to ROT^m_l\to COT^m_k\to COT^k_m$。实际上可以进一步优化$COT^k_m$的开销（也就是$OT^k_m$的开销）到$OT^k_k$，一个简单的想法是：我们不必使用$m$长度的Base OT来实现上述的每次COT，我们可以事先用$OT^k_k$传输随机长度为$k$的比特串，然后故技重施，用随机Oracle将其拉到$m$比特来实现$ROT^k_m$，用这个来实现$OT^k_m$（记得OT和ROT可以互相转化，当然，需要一次额外的发送一对$m$比特串的交互）。

### 大杀器：VOLE

IKNP曾一度是最优的OT Extension，直到最近几年一系列基于VOLE的OT Extension的出现。

VOLE (Vector Oblivious Linear Evaluation)实现的是Sender拥有$(w,m),w,m\in\mathbb{F}^n$，而Receiver有$\Delta\in\mathbb{F},k=m+\Delta w$。注意到，VOLE可以看作一种COT，例如当$\mathbb{F}=\mathbb{F}_2$的时候，它的运算变成了我们熟悉的模样（加和乘即是异或和与）；Sender发送的消息对即是$(m,m\oplus w)$，Receiver得到其中之一。

相比于IKNP，基于VOLE的OT能够带来非常显著的好处：IKNP的通信量是关于总的OT的数量线性增长的，而VOLE可以在一次性的VOLE Setup之后，借助PCG(Pseudorandom Correlation Generator)无需交互而拓展得到大量低成本OT（这种特性就是silent）。

这些技术的具体构造这里不再阐述，超出本文范围（最重要的是，我还没具体看这方面的内容）。

# EnD

如果能够坚持读完本文，（再问一问大模型），相信已经能够对OT及其相关技术（至少到IKNP为止）有一个虽不全面但是通畅的理解了。

