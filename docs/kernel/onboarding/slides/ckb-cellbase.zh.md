---
template: reveal.html
reveal:
  showNotes: true
---

# CKB Cellbase

- [显示注释](?showNotes=true)
- [参与讨论](https://github.com/ckb-learn/ckb-learn.github.io/discussions/12)
- [在 GitHub 上编辑](https://github.com/ckb-learn/ckb-learn.github.io/edit/main/docs/kernel/onboarding/slides/ckb-cellbase.zh.md)
- [返回课程列表](../../)

Note:
- [关闭注释](?showNotes=false)

---

## 什么是 Cellbase

- Cellbase 名字来自 Bitcoin 的 Coinbase
- 每个区块的第一个交易
- 交易的唯一 Input 实际不指向任何交易的 Output
- 用于发放出块奖励和手续费

---

## Cellbase 结构

- 一个 Input
- 零个或者一个 Output

Note:
- 如果 Output Capacity 不足以覆盖 Cell 的空间占用就会出现特殊的无 Outputs 的 Cellbase

--

### Cellbase Input

- `tx_hash`: 全 0
- `index`: the max 32-bit integer
- `since`: the block number

Note:
- `since` 要求填 block number 是保证 tx hash 唯一性

--

### Cellbase Output

- 区块 N 的 Cellbase Output 是发放给 N - 11 的区块生产者
- 自己的奖励会在 N + 11 发放
- 用来领奖励的 Lock Script 先登记在 Witness 里

--

### Cellbase 结构图示

<!-- https://excalidraw.com/#json=McmrNbvcosefSMuS0LI_U,VX4DKKvcY_jH2uyFHt76rA -->

![ckb-cellbase](https://user-images.githubusercontent.com/35768/189477309-b983892f-ee10-48aa-97e0-c502f615088d.png)
<!-- .element: class="r-stretch" -->

Note:
- Block N 的 Cellbase
- L(n) 表示 Block N 中留的 Lock Script
- L(n-11) 表示 Block N-11 中留的 Lock Script

---

## Cellbase Output 组成

- 一类增发
- 二类增发
- 手续费
	- Commitment 手续费
	- Proposal 手续费

--

### 一类增发

- 每个 Epoch 的总额固定
- 每 8760 个 Epochs (约4年）奖励减半
- 根据 Epoch 长度平分到 Block 中

Note:
- 除不尽的部分平分给前面的 Blocks，比如 5 分成 3 个 blocks 就是 2, 2, 1

--

### 二类增发

- 每个 Epoch 的总额固定，不会减半
- 只有一部分是发给矿工

--

#### 二类增发分成

- CKB 会统计所有增发的 CKB
- T 表示当前已增发出的 CKB 问题，包括创世块中增发出来的
- A 表示锁定在 DAO 中的数量
- B 表示不在 DAO 中但是被用于 Cells 空间占用的数量
- C 则是不在 DAO 中也没有被用于 Cells 空间占用的数量

--

#### 矿工二类增发

- 先将每个 Epoch 的二类增发平分到 Blocks
- 矿工应得的二类增长是 Block 二类增发的 B / T

--

### 手续费

单个交易手续费:

Inputs 总额 - Outputs 总额

--

#### 两步确认

<!-- https://excalidraw.com/#json=S_v9pES-axk8VzaGfzYnt,mjCVQz6B45juudwz2vIV-A -->

![tx-2-step-commitment](https://user-images.githubusercontent.com/35768/189477890-bf7e6fed-0058-44ca-8bb0-f7b545c3fc88.png)
<!-- .element: class="r-stretch" -->

--

#### 提交窗口

Block N 确认区中的交易的短 ID 必须在 Block N - 10 到 N - 2 中的提交区或者叔块提交区中出现

--

#### 确认窗口

换句话说，Block N 中提交区的交易，有可能在 Block N + 2 到 N + 10 中被确认。

--

#### 手续费分成

- 每个区块结算确认区中的交易的手续费
- 60% 归当前区块生产者
- 每个交易单独确定在提交窗口中最先提交该交易的区块，该区块生产者获得 40% 手续费


---

## Cellbase 计算实例

![](https://user-images.githubusercontent.com/35768/189478271-8d930113-3dd7-47ba-a7dc-49f357df2c6d.png)
<!-- .element: class="r-stretch" -->

[Open in Explorer](https://explorer.nervos.org/transaction/0x23808638ec7f21162320d9bc009ce753ee24edb7114cc6d88db9c3263b12d03b)

--

### 奖励对象

```
8,046,803 - 11 = 8,046,792
Epoch 6,151 727/855
```

--

### 一类增发

```
1,917,808.21917808 / 855 = 2,243.05054874
```

--

### 二类增发

```
613698.63013698 / 855 = 717.77617559
```

--

### 二类增发矿工比例

查看父块 8,046,791 Header 的 dao 字段

```
> rpc get_header --hash 0xfd5072475ce88c0496fba24d0bb2be1a006988ccdfa84239f210b73d0f769048
dao: 0xc4399f480feb3d442dd6c50697f6260031a1d725b4152f0400b17ad8b3262407
```

--

### DAO 解码

```
> struct.unpack('<4Q', bytes.fromhex('c4399f480feb3d442dd6c50697f6260031a1d725b4152f0400b17ad8b3262407'))
(4917344819033881028, 10967177629128237, 301483563530297649, 514578811300000000)

# (C, AR, S, U) 比例： U / C
> 717.77617559 * (514578811300000000 / 4917344819033881028)
75.1121641469775
```

--

### Commit 手续费

确认区交易手续费总和的 60% https://explorer.nervos.org/block/8046792

```
> (0.00100000 + 0.01112663 + 0.01112663 + 0.01112663 + 0.00500000 + 0.00500000 + 0.00001455 + 0.00010000 + 0.00003662) * 0.6
0.026718635999999997
```

--

### Proposal 手续费

遍历 8046794 ~ 8046802 中确认并且 8046792 中提交的交易：

```
- 0xb15ac271ebdea5165e20 0.00001455
- 0xd26e05cd7f9944a0f8e8 0.00300000
- 0x23ec976070db8bd95b22
- 0x2b7bcefe8f8f24ba48f3
- 0xd419eb3e4023f2c499fb 0.00010000
- 0xe03d78ac946f7fff5958 0.00010000

> (0.00001455 + 0.00300000 + 0.00010000 + 0.00010000) * 0.4
0.00128582
```

---

## 作业

自己在 Explorer 找一个块手算 Cellbase 奖励和手续费
