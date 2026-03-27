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
