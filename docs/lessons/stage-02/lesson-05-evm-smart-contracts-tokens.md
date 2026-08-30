# 课次 5：EVM、智能合约与 token

资料核对日期：2026-08-30

建议用时：75 分钟

> 本课只使用区块浏览器的只读页面。不要点击 Connect Wallet、Write Contract、Approve 或签名按钮；不要复制自己的地址。合约“已验证源码”只表示公开源码与部署字节码的对应关系得到工具核对，不等于安全审计。

## 本课要解决的问题

链上程序能做什么，不能保证什么？

学完后，你应该能做到：

1. 用状态转换解释 EVM 的作用；
2. 区分合约代码、存储、函数、事件与交易输入；
3. 区分只读调用与改变状态的交易；
4. 解释 ERC-20 为什么是接口标准而不是质量认证；
5. 说明 ETH 余额与 token 余额记录位置不同；
6. 识别授权、管理员、升级、预言机和组合性风险。

## 75 分钟安排

| 时间 | 内容 | 产出 |
| --- | --- | --- |
| 0～15 分钟 | EVM、代码与状态 | 状态转换图 |
| 15～28 分钟 | 部署、调用、ABI 与事件 | 五个术语解释 |
| 28～43 分钟 | ERC-20 接口 | 调用关系图 |
| 43～58 分钟 | 合约能力边界与风险 | 风险检查表 |
| 58～68 分钟 | 浏览器只读练习 | 一份公开合约观察 |
| 68～75 分钟 | 200 字输出与检查题 | 初稿 |

## 一、EVM 是确定性执行环境

Ethereum 虚拟机（Ethereum Virtual Machine, EVM）在每个执行节点上按照相同规则运行合约字节码。给定相同旧状态和有效交易，诚实实现应得到相同新状态。

```mermaid
flowchart LR
    S["旧状态 S"] --> E["EVM 执行交易 T"]
    T["签名交易<br/>to/value/data/gas"] --> E
    E -->|成功| S2["新状态 S'"]
    E -->|回退| R["状态修改撤销<br/>已消耗资源仍可能计费"]
```

EVM 不判断一个项目是否值得信任，也不理解现实世界合同的公平性。它只按协议和字节码执行。

## 二、合约的五个基本部件

### 1. 代码

开发者把高级语言编译成 EVM 字节码并部署到链上。链上执行的是字节码，不是网页界面展示的营销说明。

### 2. 存储

合约可保存长期状态，例如 token 余额、授权额度、管理员地址或应用参数。存储写入通常比简单计算消耗更多 Gas。

### 3. 函数与调用数据

交易的 `data` 可以编码要调用的函数及参数。应用通过合约 ABI（Application Binary Interface）知道如何编码调用、解码返回值和事件。

### 4. 事件日志

合约执行可以发出 event，方便浏览器和应用索引。例如 ERC-20 转账通常发出 `Transfer` 事件。事件适合观察，但它不是可以被后续合约直接读取的持久存储，也不能单独代替全部状态验证。

### 5. 余额

合约账户可以持有原生 ETH；合约还可以在自己的存储中记录各地址的 token 余额。这两类余额不要混为一谈。

## 三、部署、读取与写入

### 部署

部署合约是一笔创建交易。它携带初始化代码，执行后在新地址留下运行时代码和初始状态，需要占用计算与存储资源。

### 只读调用

读取余额、查看 `owner()` 或查询参数，可由本地节点模拟执行，不写入区块。常见的 `eth_call` 不产生链上状态变化，也不要求为链上执行支付 Gas。

“不付链上 Gas”不等于完全没有成本：RPC 服务商和节点仍消耗资源，第三方接口也可能收费或记录请求。

### 写入

转移 token、修改参数或执行兑换会改变状态，需要一笔被纳入区块的交易，并按实际执行消耗 Gas。传统流程由 EOA 签名；账户抽象也可由智能账户和代付机制组织。

### 失败与回退

若执行遇到 `revert`、超出 Gas 或其他异常，通常会回退该调用产生的状态修改，但验证者已经执行并消耗了区块资源，所以发送者仍可能支付已使用 Gas。失败交易不等于“什么都没发生”：nonce 往往已消耗，费用已经支付，日志和状态是否保留要按回退边界判断。

## 四、ERC-20 到底标准化什么

ERC-20 定义一组可复用接口和事件，例如：

- `totalSupply()`：读取 token 供应；
- `balanceOf(owner)`：读取某地址余额；
- `transfer(to, value)`：转移调用者的 token；
- `approve(spender, value)`：设置可代花额度；
- `allowance(owner, spender)`：读取授权额度；
- `transferFrom(from, to, value)`：在授权范围内代为转移；
- `Transfer` 与 `Approval` 事件。

```mermaid
flowchart LR
    U["token 持有人"] -->|approve 额度| C["ERC-20 合约"]
    D["被授权合约/地址"] -->|transferFrom| C
    C --> B["更新 balances"]
    C --> A["更新 allowances"]
    C --> L["发出事件日志"]
```

接口统一让钱包和应用更容易集成，但标准本身不保证：

- 供应上限固定；
- 发行方不能增发或冻结；
- 管理员权限安全；
- 合约经过审计；
- token 有真实资产支持；
- 市场有流动性或合理价格；
- 名称和图标不会仿冒。

## 五、授权为什么需要单独理解

`approve` 不等于立刻转账，而是允许指定 `spender` 在额度内调用 `transferFrom`。风险来自授权对象和额度：

- 恶意或被攻破的 spender 可能使用剩余额度；
- “无限授权”把未来余额也暴露在权限范围内；
- 前端显示的名称不能证明合约地址正确；
- 撤销授权本身通常也是状态写入交易。

本阶段只学习概念，不连接钱包查看或撤销真实授权。

## 六、“代码运行”不等于“系统安全”

把风险分层会更清晰：

| 风险层 | 要问的问题 |
| --- | --- |
| 代码 | 是否有重入、权限、精度、边界条件等漏洞？ |
| 升级 | 是否使用代理？谁能升级实现？是否有时间锁？ |
| 管理 | 谁能暂停、冻结、增发、修改参数？密钥是否多签？ |
| 预言机 | 链外价格或事件从哪里来？能否操纵或停止？ |
| 经济 | 抵押、清算、激励在压力下是否仍成立？ |
| 组合性 | 依赖的其他合约、桥或 token 失败会怎样？ |
| 界面 | 网站、DNS、前端和 RPC 是否可能被替换？ |

“不可篡改”也要具体分析：直接部署的字节码通常不变，但代理模式可以把用户调用转发到可升级实现；管理员还可能通过状态参数改变系统行为。

## 七、区块浏览器只读练习

选择一个发行方官网能直接链接到官方合约地址的知名 ERC-20。不要只按浏览器搜索结果选择同名 token。

下面以 Ethereum 主网上的 USDC 为例。整个过程只观察公开数据，不连接钱包，不使用自己的地址，也不执行任何交易。

### 第一步：从发行方官网取得地址

打开 [Circle 官方 USDC 合约地址页面](https://developers.circle.com/stablecoins/usdc-contract-addresses)，找到 Mainnet 表格中的 Ethereum。资料核对时，Circle 公布的地址为：

```text
0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
```

先从发行方官网取得地址，再将完整地址交给区块浏览器。不要直接搜索“USDC”后选择名称或图标相同的结果，因为任何人都能创建同名、同 symbol 的 token。

### 第二步：在 Etherscan 打开合约

打开 [Etherscan 上的 Ethereum USDC 页面](https://etherscan.io/token/0xA0b86991c6218b36c1d19d4a2e9eb0ce3606eb48)，核对：

- 网络是 Ethereum Mainnet；
- 页面地址与 Circle 公布的地址逐字符相同；
- token 名称和 symbol 是 USD Coin / USDC；
- decimals 是 6。

不要只核对地址开头和结尾。大小写差异可能只是校验和格式不同，但 40 个十六进制字符的内容必须一致。

### 第三步：判断源码验证和代理结构

进入 `Contract` → `Code`，查找 `Contract Source Code Verified`、`Exact Match` 或绿色验证标记。

USDC 页面还会显示 `Proxy` 和 `Implementation`：用户访问的是代理合约地址，代理把调用转给实现合约中的业务逻辑。观察代理合约时，应同时意识到管理员可能有升级实现的能力。

“源码已验证”只表示公开源码与链上字节码通过浏览器的匹配检查，不代表：

- 代码没有漏洞；
- 合约已经通过安全审计；
- 管理员不能升级、暂停、冻结或增发；
- token 一定有足额资产支持。

### 第四步：查询 Read 函数

进入 `Contract` → `Read as Proxy`；其他合约也可能显示为 `Read Contract`。展开函数并点击 `Query` 即可查询。只读查询不会修改链上状态，不需要连接钱包。

重点识别：

- `name()`：token 名称；
- `symbol()`：token symbol；
- `decimals()`：金额显示时使用的小数位数；
- `totalSupply()`：当前供应量；
- `balanceOf(address)`：指定地址的 token 余额；
- `allowance(owner, spender)`：`owner` 给 `spender` 的剩余代花额度；
- `paused()`：如果合约提供该函数，用于查询相关功能是否处于暂停状态。

合约通常以整数保存金额。USDC 的 `decimals` 是 6，所以查询结果按下面方式换算：

```text
显示金额 = 原始整数 ÷ 10^6
```

例如原始值 `1000000` 表示 `1 USDC`。`totalSupply()` 会随发行和赎回变化，记录结果时必须同时记录查询日期和时区。

如果要练习 `balanceOf(address)`，可以从 USDC 页面最近一笔公开转账中选择发送方或接收方地址。不要输入自己的地址，也不要使用任何私钥、助记词或钱包备份。

### 第五步：识别 Write 函数，但不执行

进入 `Contract` → `Write as Proxy`；其他合约也可能显示为 `Write Contract`。只观察函数名称和参数，不点击 `Connect to Web3`，不提交调用。

常见 Write 函数包括：

- `transfer`：转移调用者持有的 token；
- `approve`：设置另一个地址或合约的代花额度；
- `transferFrom`：在已有授权范围内代为转移；
- `increaseAllowance` / `decreaseAllowance`：调整授权额度；
- `mint` / `burn`：增发或销毁，通常受权限或业务规则控制；
- `pause` / `unpause`：暂停或恢复某些功能，通常只有特定角色可以调用。

Write 函数会尝试改变链上状态，通常需要连接钱包、签名、提交交易并支付 Gas。页面展示某个函数，不表示任何人都有权限成功调用它。

### 第六步：观察代理和管理员信息

查找 `Proxy`、`Implementation`、`Admin`、`Owner` 和 `Contract Creator` 等信息，注意它们并不等价：

- `Proxy` 是用户长期访问的入口合约；
- `Implementation` 保存代理当前委托执行的业务逻辑；
- `Admin` 可能拥有代理升级权限；
- `Owner` 是业务合约定义的所有者角色，未必是代理管理员；
- `Contract Creator` 是最初部署合约的地址，不代表它现在仍拥有管理权限。

浏览器标签只是调查线索。要判断某个角色究竟能做什么，还需要结合已验证源码、角色查询结果和历史交易。

### 第七步：查看一条 Transfer 事件

回到 token 页面的 `Transfers` 列表，选择任意一笔公开转账并点击 `Txn Hash`，然后在交易详情中查看 `Logs`。浏览器通常会将下面的事件解码为 `from`、`to` 和 `value`：

```solidity
Transfer(address indexed from, address indexed to, uint256 value)
```

- `from`：token 从哪个地址扣除；
- `to`：token 增加到哪个地址；
- `value`：未换算 decimals 的原始整数金额。

`Transfer` 是一笔交易执行时产生的事件日志，不是另一笔独立交易。一笔复杂交易可以产生多条 `Transfer`。如果 `from` 是零地址，通常表示铸造；如果 `to` 是零地址，通常表示销毁，但最终仍要结合合约实现判断。

### 第八步：区分链上事实和第三方标签

可以从链上或合约调用直接核验的信息包括合约地址、交易哈希、区块高度、调用地址、Gas、字节码、事件日志和函数返回值。

价格、市值、Logo、项目简介、社交链接、风险评分以及“某交易所”“某项目金库”等地址名称，通常由浏览器或外部数据源整理。它们有参考价值，但不是 Ethereum 协议对项目身份或安全性的证明。

记录：

```markdown
### YYYY-MM-DD｜课次 5：ERC-20 合约只读观察

- 核对时间及时区：
- 发行方或项目官方合约地址来源：
- 网络：
- 合约地址（公开示例，不是本人地址）：
- token 名称与 symbol：
- decimals：
- 浏览器是否显示源码已验证：是 / 否 / 不确定
- 能识别的 Read 函数：
- 能识别的 Write 函数：
- 是否看到代理或管理员标签：
- Transfer 事件说明了什么：
- 浏览器标签中哪些只是第三方信息：
- 我没有执行的操作：Connect / Write / Approve / Sign
```

USDC 示例可以记录为：

```markdown
### 2026-08-30｜课次 5：ERC-20 合约只读观察

- 核对时间及时区：2026-08-30，Asia/Shanghai
- 发行方或项目官方合约地址来源：https://developers.circle.com/stablecoins/usdc-contract-addresses
- 网络：Ethereum Mainnet
- 合约地址（公开示例，不是本人地址）：0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48
- token 名称与 symbol：USD Coin / USDC
- decimals：6
- 浏览器是否显示源码已验证：是；页面同时显示代理和实现合约信息
- 能识别的 Read 函数：name、symbol、decimals、totalSupply、balanceOf、allowance、paused
- 能识别的 Write 函数：transfer、approve、transferFrom，以及带权限控制的管理函数
- 是否看到代理或管理员标签：看到了 Proxy 和 Implementation；标签本身不能证明具体权限
- Transfer 事件说明了什么：记录 token 从 from 地址转移到 to 地址，value 是未换算 decimals 的原始整数
- 浏览器标签中哪些只是第三方信息：美元价格、市值、Logo、项目简介、网站链接和地址名称标签
- 我没有执行的操作：Connect / Write / Approve / Sign
```

如果页面弹出连接钱包提示，关闭即可。只读练习不需要钱包。

## 八、200 字输出

题目：**为什么“ETH 不是所有以太坊 token 的统称”？**

至少包含：

1. ETH 是协议原生资产；
2. token 余额由特定合约或协议状态定义；
3. ERC-20 只规定接口；
4. token 交易需要底层执行资源；
5. 同名 token 必须核对网络和合约地址。

## 九、检查问题

1. 区块浏览器显示“Contract Source Code Verified”是否等于安全审计通过？
2. `eth_call` 与写入交易的核心差别是什么？
3. ERC-20 的 `approve` 和 `transfer` 分别做什么？
4. 一个 ERC-20 token 能否保留相同接口同时允许管理员增发？
5. 合约执行失败后为什么仍可能付费？
6. 代理合约为什么使“代码不可更改”这句话需要限定？

<details>
<summary>完成后展开答案要点</summary>

1. 不是。它主要核对公开源码与部署字节码，不能证明逻辑、权限和经济机制安全。
2. `eth_call` 本地模拟读取且不改链上状态；写入交易要被共识执行并付费。
3. `approve` 设置 spender 的代花额度；`transfer` 直接移动调用者的 token 余额。
4. 可以。标准化接口不限制所有供应与管理员逻辑。
5. 网络已执行并消耗资源；状态回退不等于资源未使用。
6. 用户交互地址可能通过代理委托给可升级实现，管理员可改变实际执行逻辑。

</details>

## 十、常见误区

- **智能合约是真实世界合同的自动法律执行。** 它首先是链上程序，法律效力需另行判断。
- **ERC-20 是 Ethereum 官方背书。** 它是接口标准，不是认证。
- **事件就是最终状态。** 事件是日志，完整判断还需查看执行状态和合约存储。
- **读取合约完全没有资源成本。** 不产生链上 Gas，但节点和服务仍承担计算。
- **失败交易会退回全部费用。** 已使用 Gas 通常仍支付。
- **源码公开就没有管理员后门。** 权限必须逐项阅读，代理实现也要核对。

## 十一、完成标准

- [ ] 能画出交易数据进入 EVM 后的状态变化；
- [ ] 能区分代码、存储、函数、事件与 ABI；
- [ ] 能解释 ERC-20 的标准边界；
- [ ] 完成一次不连接钱包的合约观察；
- [ ] 200 字输出包含五个指定要点；
- [ ] 检查题至少答对 5 题。

## 十二、核心资料

- [Ethereum 官方文档：EVM](https://ethereum.org/developers/docs/evm/)
- [Ethereum 官方文档：智能合约简介](https://ethereum.org/developers/docs/smart-contracts/)
- [Ethereum 官方文档：与智能合约交互](https://ethereum.org/developers/docs/smart-contracts/interacting/)
- [ERC-20 标准（EIP-20）](https://eips.ethereum.org/EIPS/eip-20)
- [Ethereum 官方文档：交易](https://ethereum.org/developers/docs/transactions/)
- [Circle 官方文档：USDC 合约地址](https://developers.circle.com/stablecoins/usdc-contract-addresses)
