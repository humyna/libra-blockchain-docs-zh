# Move语言

原文链接：[https://developers.libra.org/docs/crates/move-language](https://developers.libra.org/docs/crates/move-language)<br/>
译者：humyna<br/>
日期：2019.09.26<br/>
版权及转载声明：本文采用[知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)进行许可。<br/>

Move是一种新的编程语言，为Libra区块链提供一个安全可编程的基础。

## 概述

Move 语言目录由五个部分组成:


- [虚拟机](https://github.com/humyna/libra-blockchain-docs-zh/blob/master/libra-developer-docs/4-2-11-%E8%99%9A%E6%8B%9F%E6%9C%BA.md) (VM) —  包含字节码格式、字节码解释器和执行交易块的基础设施。该目录还包含生成创世块的基础设施。


- [字节码验证器](https://github.com/humyna/libra-blockchain-docs-zh/blob/master/libra-developer-docs/4-2-2-%E5%AD%97%E8%8A%82%E7%A0%81%E9%AA%8C%E8%AF%81%E5%99%A8.md) —  包含一个用于拒绝无效Move字节码的静态分析工具。在执行之前，虚拟机会对遇到的任何新Move代码运行字节码验器。编译器在其输出上运行字节码验证器，并向程序员提示错误信息。


- [编译器](https://developers.libra.org/docs/crates/compiler/) — 包含Move IR编译器，它将人类可读的程序文本编译成Move字节码。_警告：IR编译器是一个测试工具。__它会生成被Move字节码验证器拒绝的无效字节码。IR语法工作仍在进行中，或将经历重大的变化。_


- [标准库](https://developers.libra.org/docs/crates/stdlib/) —  包含核心系统模块(如 `LibraAccount` 和 `LibraCoin` )的Move IR代码。


- [功能测试](https://developers.libra.org/docs/crates/functional_tests/)  — 用于虚拟机，字节码验证器和编译器的测试。 这些测试是用Move IR 编写的，由测试框架运行，该测试框架解析运行一个测试(从注释中编码的特殊指令)的预期结果。

## Move语言如何融入Libra Core?

Libra Core组件通过VM与语言组件交互。 具体来说， [准入控制](https://github.com/humyna/libra-blockchain-docs-zh/blob/master/libra-developer-docs/4-2-1-%E5%87%86%E5%85%A5%E6%8E%A7%E5%88%B6.md) 组件使用有限的 VM 虚拟机功能的只读  [子集](https://developers.libra.org/docs/vm_validator/) 在无效的交易被允许进入内存池及共识之前丢弃他们。 [执行](https://developers.libra.org/docs/execution/) 模块使用VM执行一个交易块。

### 探索 Move IR

- 你可以在 [测试](https://developers.libra.org/docs/crates/functional_tests/tests/testsuite) 目录中找到很多小的Move IR例子. 使用Move IR进行试验的最简单方法是在此目录中创建一个新测试，并按照运行测试的提示进行操作。
- 找到更多的实例可以在 [标准库](https://developers.libra.org/docs/crates/stdlib/modules) 中。 最值得注意的是 [LibraAccount.mvir](https://developers.libra.org/docs/crates/stdlib/modules/libra_account.mvir)（它实现了Libra区块链上的账户） 和 [LibraCoin.mvir](https://developers.libra.org/docs/crates/stdlib/modules/libra_coin.mvir)（实现了 Libra coin）。
- Libra testnet支持四个交易脚本也在标准库目录中。 它们是 [点对点交易](https://developers.libra.org/docs/crates/stdlib/transaction_scripts/peer_to_peer_transfer.mvir)， [创建账户](https://developers.libra.org/docs/crates/stdlib/transaction_scripts/create_account.mvir)， [Libra铸币](https://developers.libra.org/docs/crates/stdlib/transaction_scripts/mint.mvir) (仅适用于具有适当权限的帐户) 以及[密钥轮换](https://developers.libra.org/docs/crates/language/stdlib/transaction_scripts/rotate_authentication_key.mvir)。
- Move IR语法最完整的文档是 [语法](https://developers.libra.org/docs/crates/compiler/ir_to_bytecode/src/parser.rs)。您还可以查看 [Move IR分析器](https://developers.libra.org/docs/crates/compiler/ir_to_bytecode/syntax/src/syntax.lalrpop)。
- 参阅 [IR 编译器说明文件](https://developers.libra.org/docs/crates/compiler/README.md) 了解编写Move IR代码的更多详细信息。

### 代码结构

```
├── README.md          # This README
├── benchmarks         # Benchmarks for the Move language VM and surrounding code
├── bytecode_verifier  # The bytecode verifier
├── e2e_tests          # Infrastructure and tests for the end-to-end flow
├── functional_tests   # Testing framework for the Move language
├── compiler           # The IR to Move bytecode compiler
├── stdlib             # Core Move modules and transaction scripts
├── test.sh            # Script for running all the language tests
└── vm
    ├── cost_synthesis # Cost synthesis for bytecode instructions
    ├── src            # Bytecode language definitions, serializer, and deserializer
    ├── tests          # VM tests
    ├── vm_genesis     # The genesis state creation, and blockchain genesis writeset
    └── vm_runtime     # The bytecode interpreter
```

