# Libra区块链的状态机复制

原文链接：[https://developers.libra.org/docs/state-machine-replication-paper](https://developers.libra.org/docs/state-machine-replication-paper)<br />译者：humyna<br />日期：2019.07.22<br />版权及转载声明：本作品采用[知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)进行许可。

<a name="FuSUu"></a>
## 摘要
本报告提出了一个为Libra 区块链设计的健壮而高效的状态机复制系统—— LibraBFT共识算法。LibraBFT是一种基于HotStuff的最新的协议，该协议利用了数十年来在拜占庭容错 (BFT) 方面的科学进展，实现了互联网环境所需的强大的可扩展性和安全性。 LibraBFT进一步改善了HotStuff协议，引入了明确的活跃机制，并提供了具体的延迟分析。 为了推动与Libra区块链的整合，文档提供了从全功能模拟器中提取的规范。 这些规范包括状态复制接口和用于参与者之间的数据传输和状态同步的通信框架。最后，本报告提供了一个正式的安全证明，它包括检测BFT节点恶意行为的标准和一个简单的奖励惩罚机制。

<a name="pygrL"></a>
### 下载
[![](./pics/1-4-3-1-state-machine-replication.png)](https://developers.libra.org/docs/assets/papers/libra-consensus-state-machine-replication-in-the-libra-blockchain.pdf)
