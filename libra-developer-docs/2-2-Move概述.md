# Move概述

原文链接：[https://developers.libra.org/docs/move-overview](https://developers.libra.org/docs/move-overview)<br />
译者：humyna<br />
日期：2019.08.02<br />
版权及转载声明：本文采用[知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)进行许可。

Move是一种新的编程语言，旨在为 Libra 区块链提供安全可编程的基础。 Libra区块链中的帐户是一个容器，它包含任意数量的Move资源和Move模块。 提交到 Libra 区块链的每一笔交易都是用 Move 语言编写的交易脚本来实现其逻辑。 交易脚本可以调用模块声明的过程来更新区块链的全局状态。
wechat-id:humyna
在这份指南的第一部分，我们将对 Move 语言的关键特性提供高层级的介绍：

1. [Move交易脚本启用可编程的交易](https://developers.libra.org/docs/move-overview#move-transaction-scripts-enable-programmable-transactions)
1. [Move模块允许可组合的智能合约](https://developers.libra.org/docs/move-overview#move-modules-allow-composable-smart-contracts)
1. [Move有一等资源](https://developers.libra.org/docs/move-overview#move-has-first-class-resources)

感兴趣的读者可以阅读 [Move技术白皮书](https://www.yuque.com/dkkomb/zn2x7l/od0b71) 来了解有关该语言更多的细节。

在这份指南的第二部分，我们将“深入底层机制”并向您展示如何在[Move中间表示层](https://developers.libra.org/docs/move-overview#move-intermediate-representation)中编写自己的Move程序。初始版本的测试网络testnet不支持自定义的Move程序，但你可以在本地试用这些功能。

## Move的关键特性
### Move交易脚本启用可编程的交易

- 每一笔Libra区块链上的交易都包含一个 **Move交易脚本** ，这个脚本用来编码验证器代表客户端执行的逻辑（例如，将 Libra币从 Alice的帐户移动到 Bob 的帐户）。
- 交易脚本通过调用一个或多个[Move模块](https://developers.libra.org/docs/move-overview#move-modules-allow-composable-smart-contracts)的过程与Libra 区块链的全局存储中发布的 [Move资源](https://developers.libra.org/docs/move-overview#move-has-first-class-resources) 进行交互。
- 交易脚本不被存储在区块链的全局状态中，其他的交易脚本也无法调用它。这是一次性使用的程序。
- 我们在[编写交易脚本](https://developers.libra.org/docs/move-overview#writing-transaction-scripts) 中提供了几个交易脚本示例。

### Move模块允许可组合的智能合约

Move模块（Modules）定义了更新 Libra 区块链全局状态的规则。 模块等价于其他区块链系统中的智能合约。 模块声明可以在用户帐户下发布的 [资源](https://developers.libra.org/docs/move-overview#move-has-first-class-resources) 类型。Libra 区块链中的每个帐户都是一个容器，可以容纳任意数量的资源和模块。

- 模块同时声明结构类型（包括资源，这是一种特殊的结构）和过程。
- Move模块的过程定义了创建，访问和销毁它声明的类型的规则。
- 模块可重复使用。 在一个模块中声明的结构类型可以使用在另一个模块中声明的结构类型；在一个模块中声明的过程可以调用在另一个模块中声明的公共过程； 模块可以调用其他Move模块中声明的过程；交易脚本可以调用一个已发布模块的任何公共过程。
- 最终，Libra用户将能够在他们自己的帐户下发布模块。

### Move有一等资源

- Move的关键特性是能够自定义资源类型。 资源类型被用来编码具有丰富可编程的安全的数字资产。
- 资源是语言中的普通类型值。 它们可以作为数据结构用来存储，作为参数传递给过程，作为过程返回值，等等。
- Move类型系统为资源提供特殊的安全性保证。 Move资源永远不会被复制，重用或丢弃。 资源类型只能由定义类型的模块创建或销毁。 这些保证由 [Move虚拟机](https://developers.libra.org/docs/reference/glossary#move-virtual-machine-mvm) 通过字节码验证器静态执行的。Move虚拟机将拒绝运行未通过字节码验证的代码。
- Libra货币由名为 `LibraCoin.T` 的资源类型实现。 `LibraCoin.T` 在语言中没有特殊的地位; 每个Move资源都享有相同的保护。

## Move: 底层机制
### Move中间表示层
wechat-id:humyna
本节介绍如何在Move中间表示层（IR）中编写 [交易脚本](https://developers.libra.org/docs/move-overview#writing-transaction-scripts)和[模块](https://developers.libra.org/docs/move-overview#writing-modules) 。提醒读者，IR是即将推出的Move源语言的早期（且不稳定的）先导版本 (了解详细信息请参阅 [未来开发体验](https://developers.libra.org/docs/move-overview#future-developer-experience) )。 Move IR是一个在Move字节码之上的轻量语法层，用于测试字节码验证器和虚拟机，它对开发人员不是特别友好。它足够高级可用于编写可读代码，但又足够底层可以直接编译成Move字节码。然后，尽管有些瑕疵，我们还是对Move语言感到兴奋，并希望开发人员能够尝试一下IR。wechat-id:humyna

我们将继续介绍评论激烈的Move IR的片段。 我们鼓励读者通过在本地编译，运行和修改示例来跟随这些示例。 README文件`libra/language/README.md` 和 `libra/language/ir_to_bytecode/README.md` 解释了如何按照示例操作。

### 编写交易脚本

正如我们在 [Move交易脚本启用可编程的交易](https://developers.libra.org/docs/move-overview#move-transaction-scripts-enable-programmable-transactions) 中所描述的, 用户编写交易脚本来请求更新Libra 区块链的全局存储。 几乎所有交易脚本中都会出现两个重要的构建块：资源类型 `LibraAccount.T` 和 `LibraCoin.T` 。 `LibraAccount` 是模块的名称, `T` 是该模块声明的资源的名称。这是Move中通用的命名规范; 模块声明的“main”类型通常命名为 `T`。wechat-id:humyna
wechat-id:humyna
当我们说某个用户 "在Libra区块链上有一个地址为 `0xff` 的帐号" 时, 我们的意思是地址 `0xff` 拥有 `LibraAccount.T` 资源的实例。每个非空地址都有一个 `LibraAccount.T` 资源。 此资源存储帐户数据，比如序列号，验证密钥和余额。 在Libra区块链上想要与帐户进行交互必须通过从资源 `LibraAccount.T` 读取数据或调用 `LibraAccount` 模块的过程.

账户余额是 `LibraCoin.T` 类型的资源. 正如我们在 [Move有一等资源](https://developers.libra.org/docs/move-overview#move-has-first-class-resources) 中所解释的那样，这是Libra 币的类型。与任何其他Move资源一样，这种类型是语言中的“一等公民”。 `LibraCoin.T` 类型的资源可以存储在程序变量中，在过程之间传递，等等。

我们鼓励感兴趣的读者查看 `libra/language/stdlib/modules/` 目录下 `LibraAccount` 和 `LibraCoin` 模块中这两个关键资源的Move IR定义。

现在让我们看看程序员如何在交易脚本中与这些模块和资源进行交互。

```rust
// Simple peer-peer payment example.

// Use LibraAccount module published on the blockchain at account address
// 0x0...0 (with 64 zeroes). 【0x0 is shorthand that the IR pads out to
// 256 bits (64 digits) by adding leading zeroes.】
import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee: address, amount: u64) {
  // The bytecode (and consequently, the IR) has typed locals.  The scope of
  // each local is the entire procedure. All local variable declarations must
  // be at the beginning of the procedure. Declaration and initialization of
  // variables are separate operations, but the bytecode verifier will prevent
  // any attempt to use an uninitialized variable.
  let coin: R#LibraCoin.T;
  // The R# part of the type above is one of two *kind annotation* R# and V#
  // (shorthand for "Resource" and "unrestricted Value"). These annotations
  // must match the kind of the type declaration (e.g., does the LibraCoin
  // module declare `resource T` or `struct T`?).

  // Acquire a LibraCoin.T resource with value `amount` from the sender's
  // account.  This will fail if the sender's balance is less than `amount`.
  coin = LibraAccount.withdraw_from_sender(move(amount));
  // Move the LibraCoin.T resource into the account of `payee`. If there is no
  // account at the address `payee`, this step will fail
  LibraAccount.deposit(move(payee), move(coin));

  // Every procedure must end in a `return`. The IR compiler is very literal:
  // it directly translates the source it is given. It will not do fancy
  // things like inserting missing `return`s.
  return;
}
```

这个交易脚本有一个不好的地方 — 如果 `收款人` 地址下没有帐户，它将失败。 如果账户不存在的话，我们将通过修改脚本来为 `收款人` 创建帐户来解决此问题。wechat-id:humyna

```rust
// A small variant of the peer-peer payment example that creates a fresh
// account if one does not already exist.

import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee: address, amount: u64) {
  let coin: R#LibraCoin.T;
  let account_exists: bool;

  // Acquire a LibraCoin.T resource with value `amount` from the sender's
  // account.  This will fail if the sender's balance is less than `amount`.
  coin = LibraAccount.withdraw_from_sender(move(amount));

  account_exists = LibraAccount.exists(copy(payee));

  if (!move(account_exists)) {
    // Creates a fresh account at the address `payee` by publishing a
    // LibraAccount.T resource under this address. If theres is already a
    // LibraAccount.T resource under the address, this will fail.
    create_account(copy(payee));
  }

  LibraAccount.deposit(move(payee), move(coin));
  return;
}
```

让我们看一个更复杂的例子。 在这个示例中，我们将使用一个交易脚本面向多个接收人。

```rust
// Multiple payee example. This is written in a slightly verbose way to
// emphasize the ability to split a `LibraCoin.T` resource. The more concise
// way would be to use multiple calls to `LibraAccount.withdraw_from_sender`.

import 0x0.LibraAccount;
import 0x0.LibraCoin;
main(payee1: address, amount1: u64, payee2: address, amount2: u64) {
  let coin1: R#LibraCoin.T;
  let coin2: R#LibraCoin.T;
  let total: u64;

  total = move(amount1) + copy(amount2);
  coin1 = LibraAccount.withdraw_from_sender(move(total));
  // This mutates `coin1`, which now has value `amount1`.
  // `coin2` has value `amount2`.
  coin2 = LibraCoin.withdraw(&mut coin1, move(amount2));

  // Perform the payments
  LibraAccount.deposit(move(payee1), move(coin1));
  LibraAccount.deposit(move(payee2), move(coin2));
  return;
}
```

这就结束了我们对交易脚本的“参观”。更多示例，包括测试网络testnet中支持的交易脚本，请参阅 `libra/language/stdlib/transaction_scripts`.

### 编写模块

现在我们将注意力转向编写我们自己的Move模块，而不是仅仅重用现有的 `LibraAccount` 和 `LibraCoin` 模块。 考虑这种场景： Bob将在未来的某个时间点在地址_a_创建一个帐户。 Alice希望向Bob“专项拨转”一些资金，这样一旦Bob的账户创建好了，他就可以把这些资金转到他新建的账户中。但如果Bob从未创建账户，她也希望能收回自己的资金。wechat-id:humyna

为了解决Alice的这个问题，我们将编写一个模块 `EarmarkedLibraCoin` ：

- 声明一个新的资源类型 `EarmarkedLibraCoin.T` 包含Libra 币和接收人地址。
- 允许Alice在她的帐户下创建一个类型并发布它（`create`过程）。
- 允许Bob声明资源（`claim_for_recipient`过程）。
- 允许任何拥有 `EarmarkedLibraCoin.T` 的人销毁它并获得其中的资金（`unwrap过程`）。

```rust
// A module for earmarking a coin for a specific recipient
module EarmarkedLibraCoin {
  import 0x0.LibraCoin;

  // A wrapper containing a Libra coin and the address of the recipient the
  // coin is earmarked for.
  resource T {
    coin: R#LibraCoin.T,
    recipient: address
  }

  // Create a new earmarked coin with the given `recipient`.
  // Publish the coin under the transaction sender's account address.
  public create(coin: R#LibraCoin.T, recipient: address) {
    let t: R#Self.T;

    // Construct or "pack" a new resource of type T. Only procedures of the
    // `EarmarkedLibraCoin` module can create an `EarmarkedLibraCoin.T`.
    t = T {
      coin: move(coin),
      recipient: move(recipient),
    };

    // Publish the earmarked coin under the transaction sender's account
    // address. Each account can contain at most one resource of a given type; 
    // this call will fail if the sender already has a resource of this type.
    move_to_sender<T>(move(t));
    return;
  }

  // Allow the transaction sender to claim a coin that was earmarked for her.
  public claim_for_recipient(earmarked_coin_address: address): R#Self.T {
    let t: R#Self.T;
    let t_ref: &R#Self.T;
    let sender: address;

    // Remove the earmarked coin resource published under `earmarked_coin_address`.
    // If there is no resource of type T published under the address, this will fail.
    t = move_from<T>(move(earmarked_coin_address));

    t_ref = &t;
    // This is a builtin that returns the address of the transaction sender.
    sender = get_txn_sender();
    // Ensure that the transaction sender is the recipient. If this assertion
    // fails, the transaction will fail and none of its effects (e.g.,
    // removing the earmarked coin) will be committed.  99 is an error code
    // that will be emitted in the transaction output if the assertion fails.
    assert(*(&move(t_ref).recipient) == move(sender), 99);

    return move(t);
  }

  // Allow the creator of the earmarked coin to reclaim it.
  public claim_for_creator(): R#Self.T {
    let t: R#Self.T;
    let sender: address;

    sender = get_txn_sender();
    // This will fail if no resource of type T under the sender's address.
    t = move_from<T>(move(sender));
    return move(t);
  }

  // Extract the Libra coin from its wrapper and return it to the caller.
  public unwrap(t: R#Self.T): R#LibraCoin.T {
    let coin: R#LibraCoin.T;
    let recipient: address;

    // This "unpacks" a resource type by destroying the outer resource, but
    // returning its contents. Only the module that declares a resource type
    // can unpack it.
    T { coin, recipient } = move(t);
    return move(coin);
  }

}
```

Alice可以通过创建一个交易脚本为Bob创建一个专用币，该脚本调用Bob的地址_a的_`create` 过程和她拥有的资源`LibraCoin.T` 。 一旦_a_创建，Bob就可以通过从_a_发送交易来获得币。 这将调用`claim_for_recipient`过程，并将结果传递给`unwrap`，然后将返回的`LibraCoin` 存储他希望存放的地方。 如果Bob花费太长时间在_a_下创建一个帐户，这时候Alice想要收回她的资金，她可以通过先调用`claim_for_creator过程`然后调用`unwrap过程`来实现。

细心的读者可能已经注意到，该模块中的代码与`LibraCoin.T`的内部结构无关。 它可以很容易地使用泛型编程来编写（例如，`resource T <AnyResource：R> {coin：AnyResource，...}`）。 我们目前正致力于为Move添加对这种参数多态的支持。

### 未来的开发者体验
wechat-id:humyna
在不久的将来，IR将稳定下来，编译和验证程序将变得更加用户友好。 此外，来自IR源的位置信息也可以被追踪并传递给验证程序，以使错误消息更易于调试。 但是，IR将继续作为测试Move字节码的工具。 它意味着底层字节码的语义透明表示。 为了进行有效的测试，IR编译器会生成一些错误的代码，这些代码将被字节码验证器拒绝或在编译器中运行时失败。 一个用户友好的源语言会做出不同的选择; 它应该拒绝编译将在管道后续步骤中失败的代码。wechat-id:humyna

将来，我们将拥有更高级的Move源语言。该源语言旨在安全地、容易地表达常见的Move语法和编程模式。 因为Move是一种新语言，而 Libra 区块链是一种新的编程环境，我们对应该支持的语法和模式的理解仍在不断发展。 Move源语言还处于开发的早期阶段，我们还没有发布它的时间表。
