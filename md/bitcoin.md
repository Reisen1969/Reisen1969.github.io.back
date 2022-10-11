+++
author = "rayrain"
title = "bitcoin介绍"
date = "2022-10-07"
description = "learn me a bitcoin"
toc= true
math= true
tags = [
    "bitcoin","blockchain"
]

+++

本文是对https://learnmeabitcoin.com/ 的翻译

![img](https://learnmeabitcoin.com/common/images/learnmeabitcoin.png)



## 比特币如何工作?

比特币是一个诞生于2009年的电子支付系统.它允许你给世界上任何一个人转账, 你不需要征得任何人的许可就能拥有/创建账户.

它是作为现代金融体系的一种解决方案而创建的，在现代金融体系中, 有少数几家大银行控制谁获得账户以及处理哪些交易。这意味着对货币的控制是集中的，我们必须相信银行会负责任地行事。

> 银行被我们信任,才能进行存钱和电子转账的业务,但是银行却在信贷泡沫的浪潮中放贷, 几乎没有储备金(reserve).                ---- [中本聪](https://satoshi.nakamotoinstitute.org/posts/p2pfoundation/1/#selection-45.1-45.539)

中心化的银行和2007年的金融危机激发了比特币的发展.比特币是一个支付系统, 它可以在没有中央控制的情况下运行.它是由中本聪匿名设计的,与2009年1月发布.

任何人都可以运行该程序或使用该系统.

下面是它工作原理的简单解释.

## 比特币是什么?

比特币只是一个计算机程序,你可以[下载](https://bitcoin.org/en/download)运行它.

![img](https://learnmeabitcoin.com/common/images/home/1_1_program.png)

当您运行该程序时，它将连接到其他正在运行该程序的计算机，它们将开始共享一个文件。这个文件称为区块链，此文件实际上是一个巨大的交易列表(list of transactions)

> 该文大量使用了 **transaction** 一词, 我将它翻译为 "交易"

![img](https://learnmeabitcoin.com/common/images/home/1_2_network.png)

当一个新的交易进入网络时，它会在计算机之间进行转发，直到每个人都拥有该交易的副本。大约每隔10分钟，网络上的一台随机计算机(节点)将把它们收到的最新交易添加到区块链上，并与网络上的其他所有人共享更新。

![img](https://learnmeabitcoin.com/common/images/home/1_3_network_transactions.png)

因此, 比特币程序创建了一个大型计算机网络，这些计算机彼此通信，共享一个文件，并用新的交易更新它。

## 处理双重消费

在比特币出现之前, 通过计算机进行电子交易是可能的 , 然而, 可以在计算机网络中插入冲突的交易。例如，您可以创建两个使用相同数字货币的独立交易，并将这两个交易同时发送到网络。
这就是所谓的“双重消费”。

![img](https://learnmeabitcoin.com/common/images/home/2_1_why_double_spend.png)

因此，设计一个去中心化的电子支付系统，会遇到一个问题，即确定哪些交易是“先”进行的，当拥有一个由所有独立运行的计算机组成的网络时，这是一件很难做到的事情。有些计算机将首先接收绿色交易，而有些计算机将首先接收红色交易。

谁来决定哪一个“先”出现，并且应该是写入文件的唯一一个?
比特币通过迫使节点在写入文件之前将它们收到的所有交易保存在内存中来解决这个问题。然后，每隔10分钟，网络上的一个随机节点将把它们的内存中的交易添加到文件中。



![img](https://learnmeabitcoin.com/common/images/home/2_2_why_mining.png)

然后，这个更新后的文件与网络共享，节点将接受更新后文件中的交易为“正确的”，从它们的内存中删除任何冲突的交易。因此，不会将双重花费交易写到文件中，所有节点都可以按照彼此的协议更新它们的文件。

![img](https://learnmeabitcoin.com/common/images/home/2_3_why_solved.png)

向文件中添加交易的过程称为"挖矿", 它基本上是网络范围内的竞争,不能由网络上的单个节点控制.



## 矿工如何工作?

首先，每个节点将它们收到的最新交易存储在它们的内存池中，内存池只是它们计算机上的临时内存。然后，任何节点都可以尝试从它们的内存池中挖掘交易到文件(区块链)。

为此，节点将从其内存池中将交易收集到一个称为块的容器中，然后使用算力尝试将该交易块添加到区块链。

![img](https://learnmeabitcoin.com/common/images/home/3_1_mining.png)

那么，这种算力从何而来呢?要将这个块添加到区块链，您必须将您的交易块提供给一个称为哈希函数的东西。哈希函数基本上是一个小型计算机程序，它可以接收任意数量的数据，打乱它，然后吐出一个完全随机(但唯一)的数字。

![img](https://learnmeabitcoin.com/common/images/home/3_2_hash_function.png)

要将块成功添加到区块链，这个数字(块哈希)必须低于目标值，这是网络上所有人都同意的阈值。

![img](https://learnmeabitcoin.com/common/images/home/3_3_mining_block_hash.png)

如果得到的块哈希不低于目标，可以对块内的数据进行小的调整，并将其再次通过散列函数。这将产生一个完全不同的数字，很可能会低于目标。如果没有，需要调整块并重试。

![img](https://learnmeabitcoin.com/common/images/home/3_4_mining_nonce.png)

综上所述，矿工使用算力尽可能快地执行哈希计算，并尝试成为网络上第一个获得低于目标的块哈希的计算机。如果您成功了，您可以将您的交易块添加到区块链上，并与网络的其他部分共享它。

>尽管任何人都可以尝试挖掘区块，但在家用电脑上这样做已不再具有竞争力。现在有专门的硬件被设计成尽可能快(和高效)地执行哈希计算，这意味着现在大部分的挖矿工作都是由那些有专门硬件和廉价电力的人来完成的。



## 比特币的来源

为了激励人们使用算力，尝试在区块链上添加新的交易块，每个新块提供固定数量的比特币。因此，如果你能够成功挖出一个区块，你就能够“发送”给自己这些新的比特币作为你努力的奖励。

![img](https://learnmeabitcoin.com/common/images/home/3_5_mining_block_reward.png)

这种比特币被称为区块奖励，这也是为什么这个过程被称为“挖矿”.

## 为什么这个文件被叫做"区块链"?

正如我们所看到的，交易不会单独添加到文件中——它们被收集在一起并以块的形式添加。每个新块都建立在现有块的基础上，因此文件由一系列块组成.

![img](https://learnmeabitcoin.com/common/images/home/4_1_blockchain.png)

此外，网络上的每个节点将始终采用它们接收到的最长区块链作为区块链的“官方”版本。这意味着矿工总是试图在已知最长区块链的“顶端”上构建，因为不属于最长链的任何区块都不会被其他节点视为有效。

![img](https://learnmeabitcoin.com/common/images/home/4_2_blockchain_longest.png)

因此，如果有人想重写交易的历史，他们将需要重建一个更长的区块链，以创建一个新的最长链，以供其他节点采用。然而，要实现这一目标，单个矿机的计算机算力需要超过网络中其他矿机的算力之和。

![img](https://learnmeabitcoin.com/common/images/home/4_3_blockchain_hashpower.png)

因此，网络的共同努力使得任何个人都很难“跑赢”整个网络并重写区块链。



## 交易如何进行的?

您可以认为区块链是一个又一个的保险箱，我们称之为输出(outputs)。这些输出只是容纳不同数量比特币的容器。

![img](https://learnmeabitcoin.com/common/images/home/5_1_outputs.png)

当你进行比特币交易时，你选择一些输出并解锁它们，然后创建新的输出并对其添加新锁。

![img](https://learnmeabitcoin.com/common/images/home/5_2_transaction.png)

因此，当你“发送”某人比特币时，你实际上是将一定数量的比特币放入一个新的保险箱，并在保险箱上上锁，只有你“发送”比特币的目标才能解锁。
例如，如果我想给你发送一些比特币，我会从区块链中选择一些我可以解锁的输出，并从中创建一个只有你可以解锁的新输出。此外，如果我不想将我解锁的所有比特币发送给你，我会创建一个额外的输出作为我的“改变”，并将其锁定在自己身上。

![img](https://learnmeabitcoin.com/common/images/home/5_3_transaction_change.png)

继续向前，如果你想把你的比特币发送给别人，你将重复这个过程，选择现有的输出(你可以解锁)，并从它们创建新的输出。因此，比特币交易形成了一个类似图表的结构，比特币的移动通过一系列交易连接起来。

![img](https://learnmeabitcoin.com/common/images/home/5_4_transaction_graph.png)

最后，当一个交易被挖掘到区块链上时，在该交易中用完(消耗)的输出不能在另一个交易中使用，而新创建的输出将可在未来的交易中移动。

>观察下图, 原先的一个输出(10),被分割成了2 5 2 1几个新输出, 新的输出可以在未来被使用.

![img](https://learnmeabitcoin.com/common/images/home/5_5_transaction_blockchain_outputs.png)

## 如何拥有自己的比特币?

为了能够“接收”比特币，你需要有自己的一组密钥。这组密钥就像你的账号和密码，只不过在比特币中它们被称为你的公钥和私钥。
例如， 时，我将把您的公钥放在输出(保险箱)上的锁中。当你想将比特币发送给其他人时，你可以使用你的私钥来解锁这个输出。

![img](https://learnmeabitcoin.com/common/images/home/6_1_keys.png)

那么在哪里可以获得公钥和私钥呢?在密码学的帮助下，你可以自己生成它们。
简而言之，您的私钥只是一个大随机数，而您的公钥是由这个私钥计算出来的数字。但聪明的部分是;您可以将您的公钥提供给其他人，但他们无法从中计算出私钥。



![img](https://learnmeabitcoin.com/common/images/home/6_2_keys_generate.png)

现在，当你想要解锁分配给你的公钥的比特币时，你使用你的私钥来创建所谓的数字签名。这个数字签名证明你是公钥的所有者(因此可以解锁比特币)，而不必透露你的私钥。这个数字签名也只对创建它的交易有效，所以它不能用来解锁锁定在同一公钥上的其他比特币。



![img](https://learnmeabitcoin.com/common/images/home/6_3_keys_digital_signature.png)

该系统被称为“公开密钥密码术”，自19781年以来一直可用。比特币利用这个系统允许任何人创建密钥来安全地发送和接收比特币，而不需要中央机构颁发帐户和密码。



## 综合

要开始使用比特币，您需要生成自己的私钥和公钥。您的私钥只是一个非常大的随机数，而您的公钥是从它计算出来的。这些密钥可以在您的计算机上轻松生成，甚至可以在计算器这样简单的东西上生成。大多数人使用比特币钱包来帮助生成和管理密钥。
要接收比特币，你需要把你的公钥给想给你发送比特币的人。这个人会创建一个交易，解锁他们拥有的比特币，并创建一个新的比特币“保险箱”，把你的公钥放在锁里。
然后，这笔交易被发送到比特币网络上的任何节点，在那里，它从一台计算机中继到另一台计算机，直到网络上的每个节点都拥有该交易的副本。从这里开始，每个节点都有机会尝试和挖掘它们在区块链上收到的最新事务。
这个挖掘过程涉及到一个节点从它的内存池中收集事务到一个块中，并重复地将该块数据通过哈希函数(每次都有轻微的调整)，以尝试获得低于目标值的块哈希。
第一个找到低于目标的块散列的矿工将把该块添加到其区块链中，并将该块广播到网络上的其他节点。每个节点还将把这个块添加到它们的区块链(从它们的内存池中删除任何冲突的事务)，并重新启动挖掘进程，以尝试在链中的这个新块之上进行构建。
最后，挖掘该区块的矿工将在该区块中放置自己的特殊交易，这允许他们收集一定数量的不存在的比特币。这个区块奖励作为节点继续构建区块链的激励，同时在整个比特币网络中分发新币。

![img](https://learnmeabitcoin.com/common/images/home/7_1_bitcoin_system.png)

## 结尾

比特币是一种与世界各地的其他计算机共享安全文件的计算机程序。这个安全文件由交易组成，这些交易使用密码学来允许人们发送和接收数字保险箱。因此，这创建了一个任何人都可以使用的电子支付系统，并且运行时没有一个中央控制点。
自2009年1月发布以来，比特币网络一直在不间断地运行。2019年，比特币网络处理了超过1.12亿笔交易，总共转移了15,577,763,114,629.34美元(15.58万亿美元)2。
比特币程序本身也在积极开发中，自发布以来，有超过600人对代码做出了贡献。这是因为该软件是“开源”的，这意味着任何人都可以查看代码并对其进行改进。

[比特币白皮书][https://bitcoin.org/bitcoin.pdf ]

[bitcoin core 源码][https://github.com/bitcoin/bitcoin/ ]

## 获取更多

1. **[Beginners Guide](https://learnmeabitcoin.com/beginners)** - Sometimes you just need a complete walkthrough of the basics. This is the shortest and simplest guide I could write; I wrote it in 2015 as I was learning how Bitcoin works for the first time.
2. **[Technical Guide](https://learnmeabitcoin.com/technical)** - A more complete and in-depth guide to how Bitcoin works. Good for programmers.
3. **[Blockchain Explorer](https://learnmeabitcoin.com/explorer)** - You can get a feel for how bitcoin works by just browsing the data and seeing how it all connects together. It’s like opening the bonnet of a car and looking inside.
4. **[Videos (YouTube)](https://www.youtube.com/channel/UCj9MFr-7a02d_qe4xVnZ1sA/videos)** - These are *deep explanations* of the mechanics of bitcoin from the perspective of a programmer. These video lessons will get you going if you want to *code stuff* with bitcoin.
5. **[Code (GitHub)](https://github.com/in3rsha/learnmeabitcoin-code)** - Example code snippets for common bitcoin stuff.



 打赏本文的原作者`Greg`

[`3Beer3irc1vgs76ENA4coqsEQpGZeM5CTd`](https://learnmeabitcoin.com/explorer/address/3Beer3irc1vgs76ENA4coqsEQpGZeM5CTd)

![img](https://learnmeabitcoin.com/common/templates/beerqrcurrent.png)