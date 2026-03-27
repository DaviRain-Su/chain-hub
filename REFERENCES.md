# REFERENCES.md — Chain Hub 参考项目与灵感来源

> 记录对 Chain Hub 设计有直接启发的外部项目。
> 开发时遇到相关问题，先来这里查。

---

## lat.md — 代码知识图谱

**仓库：** https://github.com/1st1/lat.md

**一句话：** 把项目知识从"一个扁平文件"变成"互联的维基网络"，让 AI Agent 能真正理解代码库。

### 解决的问题

`AGENTS.md` 的局限：
- 单文件 → 项目大了就爆炸
- 没有链接 → Agent 找不到关联概念
- 和代码脱节 → 文档说一套，代码做一套

`lat.md` 的解决：
```
lat.md/
├── architecture.md  →  [[auth#OAuth Flow]]         链接到
├── auth.md          →  [[src/auth.ts#validateToken]] 链接到
└── src/auth.ts      →  // @lat: [[auth#OAuth Flow]] 反向关联
```

网状知识，双向链接，自动校验。

### 关键特性

| 特性 | 说明 |
|------|------|
| Wiki 链接 | `[[file#section#subsection]]` |
| 代码引用 | `[[src/AgentArena.sol#postTask]]` |
| `@lat` 注释 | `// @lat: [[architecture#Payment Flow]]` 代码声明"我实现了这个设计" |
| `lat check` | 验证所有链接有效，CI 检查文档和代码同步 |
| 语义搜索 | `lat search "how do we pay?"` Agent 用自然语言查知识 |

### 对 Chain Hub 的直接价值

Chain Hub 的核心使命是让 AI Agent 能理解和调用任意区块链协议。
lat.md 可以成为 **Chain Hub 协议注册的原生标准**：

```
每个在 Chain Hub 注册的协议，都附带一张 lat.md 知识图谱：

chain-hub-registry/
└── protocols/
    └── uniswap-v4/
        ├── lat.md/
        │   ├── overview.md       # 协议概述
        │   ├── swap.md           # [[src/PoolManager.sol#swap]]
        │   └── liquidity.md      # [[src/PoolManager.sol#modifyLiquidity]]
        └── manifest.json         # Chain Hub 标准注册文件
```

**效果：** Agent 接到"帮我在 Uniswap V4 上加流动性"的任务时，
先 `lat search "add liquidity uniswap v4"` → 找到 `liquidity.md` → 找到源码锚点 → 直接生成正确的调用代码。

### 实现建议（Chain Hub V2）

```
Phase 1：协议注册支持可选的 lat.md/ 目录
Phase 2：chain-hub search <query> 命令内部调用 lat 语义搜索
Phase 3：Agent 执行任务前自动拉取目标协议的 lat.md 上下文
```

### 与 AgentSoul.md 的关系

| | lat.md | AgentSoul.md |
|-|--------|-------------|
| 描述对象 | 项目/代码库 | 个人/Agent 偏好 |
| 存储位置 | 项目仓库内 `lat.md/` | 本地 `~/.arena/soul.md` |
| 链接目标 | 代码文件、设计文档 | 记忆、偏好、任务历史 |
| 用途 | Agent 理解**项目** | Agent 理解**用户** |

**两者结合（Chain Hub 的长期愿景）：**
```
Agent 接到任务
  → AgentSoul.md：理解"主人喜欢什么风格"
  → lat.md：理解"这个协议怎么设计的"
  → 生成符合两者上下文的正确调用
```

> lat.md 是"协议的 Soul"，AgentSoul.md 是"人的 Soul"。
> Chain Hub 连接两者。

---

## 其他参考（待补充）

_发现新的参考项目时在这里添加。_

---

## Gradience 生态项目（内部）

### Agent Arena — 去中心化 Agent 任务市场

**仓库：** https://github.com/DaviRain-Su/agent-arena
**本地路径：** `/home/mulerun/.openclaw/workspace/agent-arena/`
**状态：** MVP 完成，待部署到 X-Layer 测试网

**一句话：** Agent 竞争完成任务，链上结算 OKB，信誉永久上链——去中心化的 Agent 能力证明市场。

#### 核心机制

```
Task Poster 发布任务 + 锁定 OKB
  ↓
多个 Agent 申请竞争
  ↓
Task Poster 指定执行 Agent
  ↓
Agent 提交结果（IPFS CID）
  ↓
Judge 评分（0-100）→ 自动结算
  ↓
链上信誉更新（avgScore / winRate / completed）
```

#### 技术栈

| 层 | 技术 |
|----|------|
| 合约 | Solidity v0.8.24，X-Layer（chainId 1952），OKB 原生支付 |
| 前端 | Next.js 14，ethers.js，Tailwind，cyberpunk 修仙主题 |
| SDK | TypeScript，`ArenaClient` + `AgentLoop` |
| CLI | TypeScript，OnchainOS TEE 钱包 first，本地 keystore 降级 |
| Indexer | Node.js + SQLite（本地）/ Cloudflare Workers + D1（云端） |

#### 合约核心函数（v1.2）

```solidity
registerAgent(agentId, metadata, ownerAddr)  // ownerAddr=0 → 自己是 owner
postTask(description, evaluationCID, deadline) payable
applyForTask(taskId)
assignTask(taskId, agentWallet)
submitResult(taskId, resultHash)
judgeAndPay(taskId, score, winner, reasonURI)  // onlyJudge
getAgentReputation(wallet) → (avgScore, completed, attempted, winRate)
getMyAgents(ownerAddr) → []agentWallets      // 一主多 Agent
getAgentInfo(wallet) → (wallet, owner, agentId, metadata, registered)
forceRefund(taskId)                           // 超时任何人可调用
```

#### 与 Chain Hub 的关系

- **Chain Hub 是工具层，Agent Arena 是市场层**
- Chain Hub 注册的协议 → Agent Arena 任务描述中引用
- Agent Arena 的链上信誉 → Chain Hub 协议接入的门槛参考
- 长期：Chain Hub 的 Key Vault → Agent Arena CLI 的钱包后端

#### 关键设计文件

| 文件 | 内容 |
|------|------|
| `DESIGN.md` | 22节完整产品设计，含 ADR、竞品分析、ERC-8004、x402、DeFi V3 路线图 |
| `VISION.md` | Gradience Agent Economic Network 愿景 |
| `blueprint/xianxia-mapping.md` | 修仙叙事体系，Hackathon 演示脚本 |
| `blueprint/asset-philosophy.md` | 资产分类原则（本命瓷/元神/功法/法宝） |
| `contracts/AgentArena.sol` | v1.2 合约源码 |
| `artifacts/AgentArena.json` | 编译产物，37 ABI entries，8609 bytes |

#### 核心概念速查

- **境界体系**：avgScore 0-20=练气期 / 21-40=筑基期 / 41-60=金丹期 / 61-80=元婴期 / 81+=化神期
- **Judge 角色**：MVP 阶段为项目方钱包，v2 演化为多节点 DAO
- **ERC-8004**：Agent Arena 是 ERC-8004 信誉字段的数据生产者，`getAgentReputation()` 即标准接口
- **修仙彩蛋**：合约注释 `大道五十，天衍四九，人遁其一。Agent Arena 就是那遁去的一`

