# Chain Hub — 产品设计文档

> **全链 AI Agent 服务入口**
>
> 一个 CLI，接入所有区块链服务，Agent 直接用，不管 SDK、API Key、认证。

_版本：v0.1 — 2026-03-27_

---

## 一、为什么需要 Chain Hub

### 现在的问题

每个 AI Agent 想使用链上服务，都要面对这些障碍：

```
注册账号 → 找 API Key 页面 → 设置权限 → 存进环境变量
→ 搞清楚认证方式（Bearer? HMAC? OAuth?）
→ 处理 Key 过期、轮换、撤销
→ 每个服务格式完全不同，重复这一套
```

这些流程是给人设计的，不是给 Agent 设计的。结果是：

- Agent 要持有大量第三方 API Key（安全风险）
- 每接入一个新服务就要重新开发（开发成本高）
- 链上服务对 Agent 来说是一堵一堵的墙

### Chain Hub 的答案

> **Agent 只需要有自己的链上钱包，就能使用所有注册在 Chain Hub 的区块链服务。**

```
现在：Agent → API Key A → 服务 A
             → API Key B → 服务 B
             → API Key C → 服务 C

Chain Hub：Agent → 钱包签名 → Chain Hub → 服务 A / B / C / ...
```

---

## 二、产品定位

**双边平台：协议开发者 ←→ Chain Hub ←→ AI Agent**

```
开发者（协议方）              Chain Hub              Agent（消费方）
─────────────              ──────────             ────────────────
注册协议                    协议注册表               发现协议
上传 ABI                   路由引擎                 执行操作
托管 API Key               Key Vault               不持有任何 Key
获取流量                   信誉系统                 按需选择最优协议
```

类比：**App Store**——开发者上架应用，用户消费，平台做中间路由。
区别：这里的"用户"是 AI Agent，"应用"是区块链协议。

---

## 三、两类核心用户

### 用户 A：协议开发者

**他们是谁：** DeFi 协议团队、链上工具开发者、数据服务提供商。

**他们的需求：** 让自己的协议被 AI Agent 发现和使用，获取流量。

**接入流程（极简）：**

```
1. 用钱包签名注册（无需邮箱账号）
2. 提交 protocol.yaml
3. Chain Hub 自动验证合约可调用性
4. 审核通过 → 上线
5. Agent 可以发现并使用你的协议
```

整个过程像发 npm 包一样简单，不需要联系任何人。

**他们能得到什么：**
- 自己的协议在 Chain Hub 注册表中可被所有 Agent 发现
- 流量统计仪表盘（你的协议被调用了多少次）
- 信誉分积累（链上不可篡改）
- 未来：流量带来的手续费分润

---

### 用户 B：AI Agent

**他们是谁：** 运行在各种 AI 框架上的自主 Agent（OpenClaw、Claude Code、Codex 等）。

**他们的需求：** 一行命令完成链上操作，不需要管底层细节。

**调用示例：**

```bash
# 查余额
chainhub balance --chain xlayer --address 0x...
chainhub balance --chain solana --address 7xKX...

# Swap
chainhub swap --chain eth --from ETH --to USDC --amount 1

# 转账
chainhub send --chain xlayer --to 0x... --amount 0.1 --token OKB

# 合约调用
chainhub call --chain xlayer --contract 0x... --fn "deposit(uint256)" --args "1000"

# 跨链
chainhub bridge --from xlayer --to eth --token USDC --amount 100

# 发现协议
chainhub discover --action swap --from OKB --to USDC --chain xlayer

# 链上数据查询（Index）
chainhub query --chain eth --protocol uniswap --event Swap --from-block 19000000
```

**关键设计原则：命令不变，链作为参数。**

Agent 的代码永远不需要为不同链重写，换链只换 `--chain` 参数。

---

## 四、协议注册表设计

### 类比 GitHub，但给 Agent 机读

```
GitHub                         Chain Hub Registry
─────────────────────          ────────────────────────────
仓库 README                    protocol.yaml（协议描述）
源代码                         合约 ABI + 地址
npm 包                         可调用的 CLI 命令
Stars                          信誉分（链上，不可刷）
README 里的 API 文档            commands 字段（Agent 可机读）
Fork                           适配器继承（未来）
```

**核心差异：GitHub 是给人读的，Chain Hub Registry 是给 Agent 读的。**

### 存储分层

```
链上存储（永久、不可篡改）：
  ├── 协议 ID（唯一标识）
  ├── 部署者地址（钱包，即开发者身份）
  ├── 合约地址列表（每条链上的部署地址）
  ├── 注册时间戳
  └── 信誉分（随调用积累，不可篡改）

IPFS 存储（内容寻址、不可篡改）：
  ├── 完整 ABI
  ├── protocol.yaml（命令定义）
  └── 文档

链下存储（可更新，协议方自己维护）：
  ├── RPC 端点
  ├── API 端点
  └── 版本更新
```

### protocol.yaml 格式

```yaml
name: "MyDEX"
version: "1.0.0"
description: "高效的去中心化交易所，支持稳定币和主流代币"
chains: [xlayer, eth, base]
website: "https://mydex.io"
audit: "https://mydex.io/audit.pdf"  # 可选，安全审计报告

commands:
  - name: swap
    description: "代币兑换"
    params:
      - name: from_token
        type: address
        required: true
        description: "源代币合约地址"
      - name: to_token
        type: address
        required: true
      - name: amount
        type: uint256
        required: true
    contract: "0x1234..."
    function: "swap(address,address,uint256)"
    
  - name: price
    description: "查询代币价格（美元）"
    params:
      - name: token
        type: address
        required: true
    endpoint: "https://api.mydex.io/price"
    method: GET

dependencies:
  - service: thegraph
    subgraph: "mydex/mainnet"
    auth: vault  # Key 由 Chain Hub Vault 托管
```

### 协议发现（chainhub discover）

```bash
chainhub discover --action swap --from OKB --to USDC --chain xlayer

# 输出：
┌──────────────┬──────────┬──────────────┬───────────────┐
│ Protocol     │ Fee      │ Liquidity    │ Reputation    │
├──────────────┼──────────┼──────────────┼───────────────┤
│ OKX DEX      │ 0.01%    │ $12,000,000  │ ★★★★★ (9.8) │
│ MyDEX        │ 0.03%    │ $2,000,000   │ ★★★☆☆ (6.2) │
│ UniswapV3    │ 0.05%    │ $8,000,000   │ ★★★★★ (9.5) │
└──────────────┴──────────┴──────────────┴───────────────┘
```

Agent 可以自主选择，或用 `--best` 让 Chain Hub 自动选最优（综合费率 + 流动性 + 信誉）。

---

## 五、Key Vault 设计（Index 服务的认证问题）

### 问题背景

很多链上操作不只是调合约，还需要 Index 服务：

```
The Graph    → 查询历史交易、事件日志
Alchemy      → NFT metadata、余额历史
Dune         → 链上分析数据
Moralis      → 多链索引
Birdeye      → Solana 代币数据
Nansen       → 链上地址标签
```

这些服务都需要 API Key，格式各不相同。这是 Agent 使用链上服务的最大障碍之一。

### 解决方案：三种 Key 来源模式

**模式 A：协议方自带 Key（默认）**

协议开发者注册时，把自己的 Index API Key 加密托管给 Chain Hub Vault。
Agent 调用时，Chain Hub 自动注入，Agent 不持有、不知道 Key 是什么。

```yaml
# protocol.yaml
dependencies:
  - service: thegraph
    endpoint: "https://api.thegraph.com/subgraphs/name/myprotocol"
    auth: vault
    # Chain Hub 在调用时自动注入 Key
```

**优点：** 协议方控制速率限制，Key 泄露责任清晰。

---

**模式 B：Chain Hub 共享 Key 池（开箱即用）**

Chain Hub 官方采购主流 Index 服务的企业级 Key，建立共享池：

```
Chain Hub 统一采购：
  The Graph   → 企业级 Key，高速率限制
  Alchemy     → 企业级 Key，覆盖 20+ 链
  Birdeye     → Solana 生态数据
  Moralis     → 多链统一查询

免费层 Agent：使用共享池，有速率限制
付费层 Agent：专属配额，更高速率
```

**优点：** Agent 什么都不用配置，直接用。Chain Hub 从中赚取合理差价。

---

**模式 C：Agent 自带 Key（高级用户）**

有自己 API Key 的 Agent 运营者，可以注册到自己的 Vault 命名空间：

```bash
# 注册自己的 Key（加密存储在 Vault）
chainhub vault set --service alchemy --key "alchemy_xxxxx"

# 之后调用时自动使用自己的 Key，不走共享池
chainhub nft --chain eth --action list --owner 0x...
```

**优点：** 高级用户有完全控制权，不受共享池速率限制影响。

---

### Key Vault 安全模型

```
Agent 发出请求
    ↓
Chain Hub 验证 Agent 钱包签名（身份确认）
    ↓
路由层确定需要哪个 Index 服务
    ↓
Vault 查询对应的 Key（加密存储，Chain Hub 服务端解密）
    ↓
Key 注入请求头，发给 Index 服务
    ↓
返回数据给 Agent（Key 全程不经过 Agent）
```

**Agent 始终看不到任何第三方 Key。**

---

## 六、认证体系

### Agent 身份 = 链上钱包

Chain Hub 不使用用户名密码，也不使用 API Key，使用**钱包签名**作为身份证明：

```
1. Agent 用私钥签名一条消息（包含时间戳，防重放）
2. Chain Hub 验证签名，确认 Agent 的链上地址
3. 颁发短期 JWT（有效期 24h）
4. 后续请求携带 JWT
```

**好处：**
- Agent 的身份就是它的链上地址，天然防伪
- 不需要注册流程，有钱包就能用
- 和 Agent Arena 的身份体系完全兼容

### 访问控制层级

```
公开访问（无需身份）：
  - chainhub discover（协议发现）
  - 只读数据查询（代币价格、链上事件）

需要 Agent 身份：
  - 写操作（转账、合约调用、Swap）
  - 速率限制基于钱包地址

付费层（链上订阅）：
  - Agent 在链上购买订阅 NFT 或质押代币
  - Chain Hub 合约验证订阅状态，解锁更高配额
```

---

## 七、技术架构

```
chainhub CLI（Rust，单二进制，跨平台）
    │
    │ HTTPS
    ▼
Chain Hub API Server（Go）
    ├── 认证层（钱包签名验证 → JWT）
    ├── 路由层（--chain + --protocol → 适配器选择）
    ├── Key Vault（加密存储，注入 Index Key）
    ├── 缓存层（Redis，价格/元数据缓存）
    └── 适配器注册表
         ├── EVM 适配器（viem）
         │   ├── X-Layer → OnchainOS / 直连 RPC
         │   ├── Ethereum → Alchemy / Infura
         │   ├── Base / Arbitrum / BSC → 各自 RPC
         │   └── Index → The Graph / Alchemy
         ├── Solana 适配器（@solana/web3.js）
         │   └── Index → Birdeye / Helius
         ├── Sui 适配器（@mysten/sui）
         └── Bridge 适配器（LayerZero / Wormhole）

协议注册表
    ├── 链上合约（协议元数据、信誉分）
    └── IPFS（ABI、protocol.yaml、文档）
```

---

## 八、多链支持路线图

按生态影响力分三期，不贪多：

### Phase 1 — EVM 生态（0~3 个月）

聚焦 X-Layer 和主流 EVM 链，把核心功能做精：

| 命令 | 说明 |
|------|------|
| `chainhub balance` | 查询代币余额 |
| `chainhub send` | 原生代币和 ERC-20 转账 |
| `chainhub swap` | DEX 聚合 Swap |
| `chainhub price` | 实时代币价格 |
| `chainhub gas` | Gas 估算 |
| `chainhub call` | 任意合约调用 |
| `chainhub deploy` | 合约部署 |
| `chainhub query` | Index 数据查询 |

支持链：X-Layer、Ethereum、Base、Arbitrum、BSC、Polygon

---

### Phase 2 — 非 EVM 主流链（3~9 个月）

| 链 | 核心 SDK | Index 服务 |
|----|---------|----------|
| Solana | @solana/web3.js | Birdeye / Helius |
| Sui | @mysten/sui | Sui Explorer API |
| Aptos | aptos-sdk | Aptos Indexer |

统一命令接口不变，适配器内部处理各链差异（账户模型、手续费单位、交易格式）。

---

### Phase 3 — 长尾链 + 跨链（9~18 个月）

| 功能 | 技术方案 |
|------|---------|
| Cosmos 生态 | CosmJS |
| TON | ton-core |
| 跨链 Bridge | LayerZero / Wormhole / deBridge |
| 统一 Bridge 命令 | `chainhub bridge --from X --to Y` 自动路由最优桥 |

---

## 九、盈利模式

### 三层收入

**1. 订阅（To Agent 运营者）**

```
免费层：共享 RPC + 共享 Index Key，100 次/天
Pro 层（$49/月）：私有 RPC 节点，10,000 次/天，MEV 保护
企业层（定价）：专属基础设施，SLA 保障，自定义适配器
```

**2. 路由手续费（To 每笔交易）**

```
Swap 路由费：0.01~0.05%（内置在最优报价里，用户无感）
Bridge 路由费：0.05~0.1%
规模做大后，这是主要收入来源
```

**3. 协议接入费（To 开发者）**

```
基础接入：免费（扩大生态）
Premium 展示位：付费（在 discover 结果中靠前展示）
企业定制适配器：定价服务
```

---

## 十、与 Agent Arena 的关系

Chain Hub 和 Agent Arena 是**同一个大愿景的两层**：

```
用户愿景：
"每个人都有自己的 AI Agent，Agent 代替人完成所有链上操作"

产品矩阵：

  Chain Hub（基础设施层）
    └─ 开发者注册协议 → Agent 发现并调用任何链上服务

        +

  Agent Arena（协作层）
    └─ Agent 接任务 → 竞争 → 完成 → OKB 自动结算

        +

  A2A 经济网络协议（未来网络层）
    └─ Agent 身份标准 + 通信协议 + 信任层 + 支付标准
```

**具体协作场景：**

用户在 Agent Arena 发布任务：
> "帮我监控 OKB 价格，低于 50U 时自动买入 100U"

Agent 接单后，用 Chain Hub 完成执行：

```bash
# Agent 用 Chain Hub 监控价格
chainhub price --token OKB --watch --below 50 --trigger

# 触发时自动执行 Swap
chainhub swap --chain xlayer --from USDC --to OKB --amount 100 --slippage 0.5

# 完成后向 Agent Arena 合约提交结果
chainhub call --chain xlayer \
  --contract <AgentArena地址> \
  --fn "submitResult(uint256,string)" \
  --args "7,done:tx:0x..."
```

全程 Agent 自主完成，没有人工参与，没有 API Key 暴露。

---

## 十一、当前阶段与下一步

### 当前状态

- [x] 产品设计完成（本文档）
- [x] Agent Arena MVP 完成（Chain Hub 的第一个垂直应用）
- [ ] Chain Hub CLI 开发（Phase 1）
- [ ] 协议注册表合约开发
- [ ] Key Vault 服务开发
- [ ] Developer Portal 开发

### 近期优先级

1. **Agent Arena 先上线**（Hackathon 截止 3/28）——验证核心模型
2. **Chain Hub MVP**——聚焦 X-Layer，做好 8 个核心命令
3. **开发者接入**——找 2~3 个 DeFi 协议做试点接入

### 技术选型决策

- **CLI 语言**：Rust ✅ — 二进制小、启动快、内存安全，`clap` 做命令行，`tokio` 做多链并发，多链 SDK 均有 Rust 版（solana-sdk、ethers-rs）
- **协议注册表合约**：部署在 X-Layer（和 Agent Arena 生态一致）
- **Key Vault 路线**：Phase 1 中心化（AWS KMS / 自建）→ Phase 2 接 Lit Protocol（Key 分片到去中心化节点）→ Phase 3 完全去中心化（链上验证 MPC 门限签名）
- **信誉分算法**：见下节

---

## 十二、信誉分系统详细设计

### 信誉分的作用

当 Agent 运行 `chainhub discover` 发现了 100 个都支持 OKB→USDC Swap 的协议时，需要一个可信的排序依据：

- 哪个协议调用成功率高？
- 哪个协议流动性充足？
- 哪个协议经过安全审计？
- 哪个协议是钓鱼合约或已跑路？

信誉分就是这个**可信排序依据**——链上存储，不可篡改，Chain Hub 无法造假。

### 算法

```
信誉分（0~100）= 成功率 × 50 + 调用量得分 × 30 + 安全认证 × 20
```

**① 成功率（权重 50%）**

每次 Agent 通过 Chain Hub 调用协议，链上记录结果：

| 情况 | 记录 |
|------|------|
| 调用成功（tx confirmed） | +1 成功 |
| 合约 revert | +1 失败 |
| 超时 / RPC 异常 | +1 失败 |
| 模拟通过但广播失败 | +1 失败 |

```
成功率得分 = (成功次数 / 总调用次数) × 50
```

最低要求：总调用次数 < 100 时，信誉分显示「数据不足」，不参与排序。

**② 调用量（权重 30%）**

反映"有多少 Agent 在用这个协议"，类似 npm 下载量。

防刷量机制：只统计来自**不同钱包地址**的调用，同一个地址重复调用只计一次。

```
调用量得分 = min(unique调用地址数 / 10000, 1) × 30
```

10000 个不同 Agent 调用过 = 满分。

**③ 安全认证（权重 20%）**

人工 + 半自动审核，一次性评定，不会随时间变化：

| 认证项 | 得分 |
|--------|------|
| 合约开源且可验证 | +5 |
| 有第三方审计报告（Certik/SlowMist 等） | +10 |
| 合约不可升级（immutable） | +3 |
| 无管理员提权函数 | +2 |

满分 20 分。未提交任何审计的协议，此项得 0 分。

### 链上存储结构

```solidity
struct ProtocolReputation {
    uint256 totalCalls;        // 总调用次数
    uint256 successCalls;      // 成功次数
    uint256 uniqueCallers;     // 不同调用地址数
    uint8   securityScore;     // 安全认证分（0-20，人工评定）
    uint256 lastUpdated;       // 最后更新时间戳
}
```

信誉分实时计算，不存储最终分数（防止过时数据误导）。

### 恶意协议处理

两种处理机制：

**自动下架**：连续 100 次调用成功率低于 20%，自动暂停展示。

**人工举报**：任何 Agent 可以链上提交举报（需质押少量代币防止滥用举报），治理委员会（初期为 Chain Hub 团队）审核后决定是否下架。下架不等于删除——链上记录永久保留，只是不在 discover 结果中展示。

---

## 十三、Key Vault 分阶段实现

### Phase 1：中心化（立即可做）

```
技术栈：AWS KMS + 自建加密服务
时间线：1 个月内可上线

流程：
  协议方上传 API Key
    → AES-256 加密
    → 存入 PostgreSQL
    → 主密钥托管在 AWS KMS

Agent 调用时：
    → JWT 验证身份
    → 从 DB 取密文
    → 调用 KMS 解密
    → 注入请求
    → Key 不落日志
```

缺点：Chain Hub 团队理论上能看到 Key（中心化风险）。

---

### Phase 2：Lit Protocol MPC（3~6 个月后）

```
技术栈：Lit Protocol（支持 EVM + Solana）
原理：Key 分片存储在 Lit 网络 1000+ 节点
      任何单节点都拿不到完整 Key
      需要门限数量节点共同签名才能解密

接入方式：
  协议方注册时，Key 通过 Lit SDK 加密分片
  Agent 调用时，Chain Hub 向 Lit 网络发起解密请求
  Lit 节点验证 Chain Hub 合约授权后，返回解密结果
  Key 在 Lit 网络内完成注入，不经过 Chain Hub 服务器
```

优点：Chain Hub 服务器也看不到 Key，真正去中心化托管。

其他可选 MPC 服务：
- **Turnkey** — 专门做 Wallet-as-a-Service，API 简洁
- **Shadow** — Solana 原生，延迟低
- **Privy** — 企业版有 MPC，接入成本低

---

### Phase 3：完全去中心化（12 个月后）

```
Chain Hub 合约本身变成去中心化协议
Key Vault 完全由 MPC 节点网络运营
任何人可以运行 Vault 节点，赚取手续费
Chain Hub 团队不再持有任何特权
```

---

_文档持续更新。讨论记录归档在 memory/2026-03-27.md_
