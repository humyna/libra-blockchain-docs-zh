# Libra命令行工具

原文链接：[https://developers.libra.org/docs/reference/libra-cli](https://developers.libra.org/docs/reference/libra-cli)<br/>
译者：humyna<br/>
日期：2019.09.27<br/>
版权及转载声明：本文采用[知识共享署名-非商业性使用-禁止演绎 4.0 国际许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)进行许可。<br/>

本指南介绍如何使用Libra命令行接口（CLI）与Libra 区块链进行交互。 CLI作为一个交互式终端被调用。 它提供了创建帐户，铸币，转账和查询区块链的基本命令。 您可以使用CLI客户端通过指定节点的主机名与testnet上的验证器节点、本地 Libra 区块链或远程区块链进行交互。

## 调用

有三种方法可以调用Libra CLI客户端。

### 通过CLI客户端连接到Testnet

要通过CLI连接到testnet，可以使用脚本来调用CLI，无需指定参数。 要调用它，请切换到`libra`目录并运行:

```
./scripts/cli/start_cli_testnet.sh
```

### 运行本地Libra网络和生成一个CLI客户端

要启动本地Libra网络并生成连接到此本地网络的CLI客户端，请运行：

```
cargo run -p libra_swarm -- -s
```

`-s` 选项使CLI在本地Libra网络启动后运行。 请注意，这可能需要几分钟才能构建并启动。

### 运行CLI客户端连接到任意Libra网络

要调用CLI客户端并自行配置它，请运行：

```
cargo run -p client --bin client -- [OPTIONS] --host <host> --validator_set_file <validator_set_file>
```

#### 选项

运行CLI客户端命令连接到任何Libra网络的选项包括：


- `-m | --faucet_key_file_path` — 为faucet帐户生成密钥对的路径。 faucet帐户可用于铸币。 如果未通过，将为您生成一个新的密钥对，并放在临时目录中。 要手动生成密钥对，请使用generate_keypair： `cargo run -p generate_keypair -- -o <output_file_path>`.
- `-f | --faucet_server` — 操作faucet服务的主机。如果没有通过，将从host参数中派生。
- `-a | —-host` — CLI要连接的目标主机。
- `-p | -—port` — 客户端连接的目标Libra 区块链的公共端口，默认值为30307。
- `-n | --mnemonic_file` — 加载用户帐户地址/密钥生成的助记词的文件位置。 如果未通过，则使用libra_wallet在当前目录中生成新的助记词文件。
- `-s | --validator_set_file` — 加载受信任的验证器配置的文件位置。 它用于验证验证器查询响应中的验证器签名。 该文件至少应包含客户端信任的所有验证器的公钥 — 通常应该是网络上的所有验证器。 要连接到testnet，请使用 'libra/scripts/cli/trusted_peers.config.toml'。本地测试可以通过libra-config生成: `cargo run --bin libra-config` ，但首选方法是使用libra-swarm来运行本地网络。

## 命令

一旦使用前面提到的三个命令中的任何一个启动后，就可以使用以下CLI命令了：

```
major_command subcommand [options]
```

如果仅输入主命令，它将显示该命令的帮助信息。 主命令可以是以下任何一种：

---

#### `account | a` — 帐户相关操作，子命令包括：

`create | c` — 使用私钥/公钥对创建随机帐户。 帐户信息仅保存在内存中。 创建的帐户不会保存到链中。

```
Usage:
    create|c
```

`list | la` — 输出显示已创建或加载的所有帐户。

```
Usage:
    list|la
```

`recover | r <file_path>` — 通过 `account write` 命令恢复写入文件的所有帐户。

```
Usage:
    recover|r <file_path>
 Arguments:
     file_path - File path from which to load mnemonic recover seed.  Must have been written via `account write`
```

`write | w <file path>` — 将Libra钱包助记词恢复种子保存到磁盘。 这将允许通过 `account recover` 命令恢复账户。

```
Usage:
    write|w <file_path>
 Arguments:
     file_path - File path at which to save the mnemonic recovery seed to disk.
```

`<mint | m> | <mintb| mb>` — 铸币到帐户上。 后缀'b'用于块标记，如果指定了块（使用后缀'b'），CLI将查询链，直到交易完成/可用。 其他子“块”命令也是如此。

```
Usage:
    mint|mint|m|b <receiver_account_ref_id>|<receiver_account_address> <number_of_coins>
  Arguments:
      receiver_account_ref_id | receiver_account_address - The receiver account to mint the coins to.
            If the receiver account does not exist, it will be created first.
            Either receiver_account_address or receiver_account_ref_id (an internal index of
            the account in the CLI client) can be used to specify receiver account (as in
            other commands). If gas is being charged, the account that sent this mint transaction
            (currently preloaded genesis account) pays for the gas.
      number_of_coins - The number of coins to be minted to the receiver account.
```

---

#### `transfer | transferb | t | tb` — 将币从一个帐户转移到另一个帐户。后缀'b'基于块。

```
Usage:
    transfer|transferb|t|tb <sender_account_address>|<sender_account_ref_id> <receiver_account_address>|<receiver_account_ref_id> <number_of_coins> [gas_unit_price (default=0)] [max_gas_amount (default 10000)]
Arguments:
    sender_account_address | sender_account_ref_id - The account from which this transfer transaction
        is sent. The sender account pays for the gas.
    receive_account_address | receiver_account_ref_id - The account to which this transaction sends coins.
        If the receiver account does not exist, it will be created first. The sender will pay for
        gas required for both account creation and coin transfer.
    number_of_coins - The number of coins transferred to receiver account.
    gas_unit_price - The unit price to pay for gas.
    max_gas_amount - Max units of gas user is willing to pay for this transaction.
```

---

#### `query | q` — 从目标链查询数据。 所有查询操作都是基于块的。 子命令包括：

`balance | b` — 获取帐户的当前余额

```
Usage:
    balance | b <account_ref_id>|<account_address>
 Arguments:
     account_ref_id | account_address - The account to query balance for.
```

`sequence | s` — 获取帐户的当前序列号。

```
Usage:
    sequence | s <account_ref_id>|<account_address> [reset_sequence_number=true|false]
  Arguments:
      account_ref_id | account_address - The account to get current/latest sequence number.
      reset_sequence_number - If the sequence number known locally by the CLI differs from the
            value known on chain, this will reset the local sequence number to to on-chain
            value.  This is useful when a user encounters an invalid sequence number error.
```

`account_state | as` — 获取帐户的最新状态。

```
Usage:
    account_state | as <account_ref_id>|<account_address>
  Arguments:
      account_ref_id | account_address - The account to query latest state.
```

`txn_acc_seq | ts` — 按帐户和序列号获取已提交的交易。

```
Usage:
    txn_acc_seq | ts <account_ref_id>|<account_address> <sequence_number> <fetch_events=true|false>
  Arguments:
      account_ref_id | account_address - The account to query committed transaction.
      sequence_number - The sequence number of committed transaction.
      fetch_events - Set to true to fetch events emitted by this transaction.
```

`txn_range | tr` — 按范围获取已提交的交易。

```
Usage:
    txn_range | tr <start_version> <limit> <fetch_events=true|false>
  Arguments:
      start_version - The version to query the transaction from.
      limit - The maximum number of transactions to query.
      fetch_events - Set to true to fetch events emitted by each transaction.
```

`event | ev` — 通过帐户和路径获取事件。

```
Usage:
    event | ev <account_ref_id>|<account_address> <sent|received> <start_sequence_number> <ascending=true|false> <limit>
  Arguments:
      account_ref_id | account_address - The account to query events from.
      sent | received - Fetch sent or received events for this account.
            Note that this will later evolve into selecting any event path.
      start_sequence_number - The sequence number of events to query starting from.
      ascending - The direction of query from start_sequence_number.
      limit - The maximum number of events to query.
      ---
```

#### dev | d — Move交易脚本和模块相关的操作。

> 注意：这个命令只能用于本地网络，不能用于testnet。


子命令包括:
`compile | c` — 编译Move程序。

```
Usage:
  compile | c <sender_account_address>|<sender_account_ref_id> <file_path> <module|script> [output_file_path (compile into tmp file by default)]
Arguments:
  sender_account_address|sender_account_ref_id - Address of the sender account|Local index of the sender account.
  module|script - Distinguishes between move modules and move scripts.
  file_path - Path to the source Move program written in Intermediate Representation (IR).
  output_file_path - (Optional) Where the compiled module will be saved.
```

`publish | p` —发布一个Move模块到本地区块链上。

```
Usage:
  publish | p <sender_account_address>|<sender_account_ref_id> <compiled_module_path>
Arguments:
  sender_account_address|sender_account_ref_id - Address of the sender account|Local index of the sender account
  compiled_module_path - Path to the compiled module.
```

`execute | e` — 执行Move交易脚本。

```
Usage:
  execute|e <sender_account_address>|<sender_account_ref_id> <compiled_module_path> [script args]
Arguments:
  sender_account_address|sender_account_ref_id - Address of sender account|Local index of the sender account.
  compiled_module_path -  Path to the compiled transaction script.
  script args - Arguments expected by the transaction script.
```

---

**`quit | q!` — 退出CLI。 不需要子命令.**

---

**`help | h` — 显示输出帮助。 不需要子命令。**

### Testnet上帐户创建/铸币(Faucet)

CLI提供的帐户创建会生成本地密钥对，但区块链上不会创建任何内容。 要在区块链上创建帐户：


- 将币转到你要创建的地址。 如果接收方帐户不存在，将首先创建接收方帐户，然后转移币。 发送方支付创建帐户和转账的费用。
- 发送铸币交易到一个帐户。 如果该帐户不存在，将首先创建该帐户，并且稍后将铸币。 与其他交易不同，不存在的账户本身可以请求铸币交易。 对于测试网，没有明确限制帐户可以铸造的币的数量。 它旨在允许用户创建币以便在testnet上进行实验。
