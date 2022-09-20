---
template: reveal.html
reveal:
  showNotes: true
---

# CKB 内核简介

- [显示注释](?showNotes=true)
- [参与讨论](https://github.com/ckb-learn/ckb-learn.github.io/discussions/10)
- [在 GitHub 上编辑](https://github.com/ckb-learn/ckb-learn.github.io/edit/main/docs/kernel/onboarding/slides/ckb-kernel-introduction.zh.md)
- [返回课程列表](../../)

Note:
- [关闭注释](?showNotes=false)

---

## CKB 是什么？

- CKB 共识标准
- CKB 网络
- CKB 可执行程序
- CKB 链上代币名称
- CKB 链上代币单位

---

## CKB 内核 (Kernel)

全节点最基本的功能

- 通过 P2P 发现节点和同步链
- 维护共识

---

## CKB 的主要特性

CKB 是

- 基于 UTXO 模型的,
- 支持用户部署链上合约验证交易的,
- PoW 链

---

## UTXO 模型

- UTXO 的代表是 Bitcoin
- 推荐阅读 [精通比特币（第二版）](https://github.com/inoutcode/bitcoin_book_2nd)

![cover](https://github.com/inoutcode/bitcoin_book_2nd/raw/master/images/cover.png)
<!-- .element: class="r-stretch" -->

--

### CKB 交易

一笔交易告诉网络，一些 CKB 价值和数据的所有者已经授权将该价值和数据转移给另一个所有者。

--

### 交易的输入和输出

- 交易的输入是价值和数据的转出方
- 交易的输出是价值和数据的接收方
- 每一个输入都是对另外一个交易的某个输出的引用

--

### 交易的价值转移

- 每个输入都转出了一些 CKB 价值
- 这些 CKB 价值被转移到了输出中
- 输入价值总和高出输出总和的部分作为手续费被转让给矿工（出块者）

--

<!-- .slide: data-menu-title="交易的价值转移（图示）" -->

![https://excalidraw.com/#json=KeR5e08OsQ2Ya8taozY75,cX0-9MyN7B8uTIOK7YfkUA](https://user-images.githubusercontent.com/35768/189262977-fc108070-d628-4162-a925-43a4f0e724ae.png)

--

### TXO

- UTXO 中的 TXO 就是 Transaction Output 的简写
- Bitcoin 中也把 TXO 称为 Coin
- CKB 中相应的把 TXO 称为 *Cell*
    - 后方都会使用 Cell 来表示 TXO

--

### Cell 的特性

- Cell 不可更改
- Cell 是一次性的

--

### Cell 的特性（例）

- Alice 拥有一个价值为 200 CKB 的 Cell
- 她想要转 100 CKB 给 Bob。

--

<!-- .slide: data-menu-title="Cell 的特性（例，图示）" -->

<!-- https://excalidraw.com/#json=eWKEye-pxim97f_PcWEsy,EsBMaVHzECiW1E912g4Prg -->

![alice-100-ckb-to-bob](https://user-images.githubusercontent.com/35768/189264303-7698596f-c581-4b50-bd1e-88cc701a2998.png)
<!-- .element: class="r-stretch" -->

--

### Cell 的特性（例，错误做法）

Alice 不能把 200 CKB 的 Cell 余额改成 100 CKB

--

<!-- .slide: data-menu-title="Cell 的特性（例，错误做法图示）" -->

<!-- https://excalidraw.com/#json=A2O56hz-vxBVZNUMAgHct,eQ7Gwy5vZVJtDWbddBjJaA -->

![Alice Sends 100 CKB to Bob - Wrong](https://user-images.githubusercontent.com/35768/189358804-f68b9084-5f86-4b11-b0ae-11dc83be7070.png)
<!-- .element: class="r-stretch" -->

--

### Cell 的特性（例，正确做法）

- Alice 把 200 CKB 的 Cell 作为交易输入
- Alice 为 Bob 创建一个 100 CKB 价值的 Cell 作为交易输出
- Alice 为自己创建一个 99.9999 CKB 价值的 Cell 作为交易输出
- 差额 0.0001 CKB 作为交易的手续费

Note:
Alice 在交易输出中为自己创建的 Cell 被称为找零

--

<!-- .slide: data-menu-title="Cell 的特性（例，正确做法图示）" -->

<!-- https://excalidraw.com/#json=WPw1xcN1FGxatKLBYepb1,fni5-aKbzSNM_2xwArofow -->

![Alice Sends 100 CKB to Bob - Right](https://user-images.githubusercontent.com/35768/189360306-ee4b0024-e76e-42db-bce2-6187ddf0598a.png)
<!-- .element: class="r-stretch" -->

--

### Cell 的一次性

- 一个交易中的输入不能有多个指向相同的 Cell
- 如果两个交易的输入中引用了相同的 Cell，它们是互斥的
- 互斥的交易最多只能有一个上链

--

### Cell 的创建和销毁

因为 Cell 的一次性，很多文章中会使用术语创建和销毁。

- 一个交易销毁了它的输入所引用的所有 Cells
- 一个交易的所有输出就是它所创建的 Cells
- Cell 只能被销毁一次

--

### UTXO 和 Live Cell

- 被创建出来还没有被销毁的 TXO 就是 UTXO
  - U 表示 Unspent，还没有被花费的
- CKB 中一般也称为 Live Cell

--

### TXO 和 UTXO

- TX: Transaction
- TXO: Transaction Output / Cell
- UTXO: Unspent Transaction Output / Live Cell

--

### 交易依赖

- 如果交易 A 的某个输入引用的是交易 B 的某个输出，则 A 依赖 B。
- 在交易上链时，交易 B 必须先于交易 A 上链

--

### 交易顺序

- 交易顺序体现在区块中
- 每个区块中上链的交易是有序的
- 所有交易的链上排序如下
  - 先区块高度由低到高
  - 同区块交易按照区块中的交易顺序

---

## 合约

--

### Cell 也是数据容器

- Cell 可以写入任意数据
- Cell 的 CKB 价值同时也决定了可写入数据的大小
  - 1 CKB = 1 字节
  - Cell 的所有字段，包括用来表示有多少 CKB 价值，都需要占用空间

--

### CKB 价值即存储容量

<!-- https://excalidraw.com/#json=4V7AkzaAy1hh1jg9x8ZPn,EKMsuBYYHaFxq06esTM7bw -->

![ckb-value-as-capacity](https://user-images.githubusercontent.com/35768/189465832-e1d856ac-e473-4101-b724-32d397c2e70c.png)
<!-- .element: class="r-stretch" -->

--

### Cell 存放代码

- CKB 合约支持 ELF 格式的，RISC-V 指令集的二进制可执行文件
- 链上执行的二进制程序必须放在 Cell 中

--

### 如何使用代码

- 通过 Data Hash 引用
- 通过 Type Script Hash 引用

--

### Hash Type

<!-- https://excalidraw.com/#json=mxwG0wgm0hCAsTr8ShyCk,Y2a3KI-tp8q8swb1yFEyTw -->

![script-hash-type](https://user-images.githubusercontent.com/35768/189465995-7bd18ea6-2bef-44b9-b2c5-a200c4be39a5.png)
<!-- .element: class="r-stretch" -->

--

### Script

- `hash_type`: 使用 Data Hash 还是 Type Script Hash 匹配
- `code_hash`: Data Hash 或者 Type Script Hash 的目标值
- `args`: 调用合约时输入给可执行程序的 argv

--

### Script 是对 Cell 的间接引用

当 Script 需要执行时，必须将匹配的 Cell 作为 Cell Deps 组装进交易

--

### Cell Deps 要求

- 必须是 Live Cell
- 根据 `hash_type` 的取值，Data Hash 或者 Type Script Hash 必须和 `code_hash` 一模一样

--

### 执行时机

- Lock Script
- Type Script

Note:
- 只执行 Input Cells 上的所有 Lock Scripts。新创建的 Output Cells 上的  Lock Scripts 不执行。
- Type Scripts 出现在 Input 和 Output 都要执行。

---

## PoW

- Proof of Work
- 工作量证明

--

### 去中心化共识

- 全节点**独立**验证每笔交易
- 挖矿节点将交易**独立**地聚合到新的区块中，并通过 PoW 证明工作量
- 每个节点**独立**验证新的区块，并组装到区块链中
- 通过 PoW，每个节点**独立**选择具有最多累积工作量证明的链

--

### PoW 算法

- 输入是区块
- 执行 [Eaglesong][]
- 输出小于一个阈值才是合法的区块

[eaglesong]: https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0010-eaglesong/0010-eaglesong.md

--

### PoW 算法 (续)

- 区块中留有字段可以填充随机数据
- 最佳策略就是反复随机看结果是否满足

--

### 挖矿奖励

- 被网络选择到链中的区块的生产者可以获得奖励
- 具体奖励见 [CKB Cellbase](../ckb-cellbase/)

--

### PoW 如何抵抗攻击？

- 收益最大的行为是遵守共识
- 参与者越多，攻击成本越高

---

## 作业

- 使用 ckb-cli tx 子命令手动组装交易
- 在 testnet 部署 always success 合约，并使用 type id 作为 type script
- 创建一个 lock 和 type 都使用 type script hash 引用 always success 的 cell
- 升级一次 always success，可以不修改 data
- 使用升级后的 always success 组装交易解锁第三步中的 cell
