# 慕容复 (Murongfu) —— AI Agent 斗转星移技能路由器与合成引擎 (SPEC v1.0)

> **核心理念**：博采众家之长，以彼之道还施彼身。解决多流派 Agent Skills 目录污染、指令打架、Token 爆炸与选择困难症。
> **工程范式**：`Probe observes -> Decision Model (Kev/Jev/Clef) decides -> Weighted Synthesizer harmonizes -> Host Agent acts`

---

## 1. 六大武学源（Target Skill Repositories）与流派基准权重

系统将各大顶流仓库的 `SKILL.md` 解构为最小颗粒度的**原子招式（Skill Atoms）**，并在导入时赋予三维静态基准权重（`0.0 ~ 1.0`）：

| 仓库源 (Source Repo) | 核心流派定位 | 严谨度 (`rigour`) | 敏捷度 (`velocity`) | 领域权威 (`authority`) | 核心提取招式示例 |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **`obra/superpowers`** | 全流程纪律与强制 TDD 派 | 0.95 | 0.40 | 0.85 | `git-worktree` 隔离、红绿重构 TDD、删抢跑代码、7 步工程流 |
| **`mattpocock/skills`** | 决策树拷问与深模块架构派 | 0.88 | 0.60 | 0.90 | `grill-me` 决策树遍历、`domain-modeling`、深模块接口设计 |
| **`cursor/plugins (pstack)`** | `poteto-mode` 极致深潜与并行派 | 0.55 | 0.95 | 0.80 | 跳过繁琐规划直接深潜、多子智能体并行、`comment-sicko` 死代码清理 |
| **`garrytan/gstack`** | 一人创业团队、产品与 QA 派 | 0.70 | 0.85 | 0.92 | Startup/Builder 6 大灵魂拷问、Narrowest Wedge（最小切入点）、4 档 Scope 缩放 |
| **`anthropics/skills`** | 官方标准规范与文档工程派 | 0.85 | 0.70 | 0.95 | 标准结构化交付模板、复杂文档工作流、元技能规范 |
| **`vercel-labs/agent-skills`** | 前端/React/Next.js 架构权威派 | 0.85 | 0.80 | 1.00 (前端域) | React/Next.js 性能与组件规范、技能发现协议 |

---

## 2. 系统架构与双模式设计（Local-First + Cloud Ready）

系统采用同一套协议，支持**纯本地模式（默认开源免费）**与**云端大脑模式（商业化 Pro/Team）**无缝切换：

1. **端侧接入层（`npx murongfu init`）**：
   - 仅向宿主环境（Cursor / Claude Code / Windsurf）注入 `.cursor/mcp.json`（或 `.claude/` 配置）及一段 10 行字的 `murongfu-entry.md` 总纲铁律。
2. **本地状态探针（`src/probe/`）**：
   - 毫秒级采集本地脱敏元数据（绝不上传业务源码）：
     - `gitState`: 当前分支、未提交文件数、变更行数。
     - `docState`: 是否存在 `PLAN.md`、`SPEC.md`、`GLOSSARY.md`、`ADR`。
     - `errorState`: 最近一次测试或构建是否处于失败状态。
     - `techStack`: 解析 `package.json` / `Cargo.toml` / `pyproject.toml` 识别框架（如 React, Next.js, TypeScript）。
     - `userIntent`: 用户当前自然语言指令摘要。
3. **统一决策引擎适配器（`src/decision/`）**：
   - 采用 TypeSafe Jev / 开源 Kev 标准原语，在单次前向推理中输出结构化概率分布：
     - **`Noul` (二元概率门控)**: `is_spec_ready` (`0.0 ~ 1.0`，需求与边界是否已清晰到可直接写代码)。
     - **`Score` (标尺打分)**: `architecture_risk` (`1 ~ 10`，架构破坏性风险)、`urgency_velocity` (`1 ~ 10`，敏捷推进诉求)。
     - **`Choice` (多选一概率分布)**: `lifecycle_stage` $\in$ `['interrogate', 'architect', 'execute', 'debug', 'review']`。
   - **支持 4 种可插拔 Provider（通过 `.murongfu.json` 配置）**：
     - `local-kev`（默认）：优先连接本地开源 `jaredpalmer/kev` (`kev-0.8b` / `kev-4b` on Qwen3.5) 服务端点；若本地未启动模型服务，自动无缝降级到内置的**零依赖贝叶斯启发式计算器（Zero-Dep Heuristic Fallback）**，确保用户 `npx` 开箱 100% 可用。
     - `byok-jev`：用户自备 `JEV_API_KEY`，直连 TypeSafe Jev 官方 API。
     - `byok-compatible`：支持 Cloudflare `Clef` 或 OpenRouter 上的 `Microsoft-Decision-1`。
     - `murongfu-cloud`：将脱敏状态 JSON 发往慕容复云端（云端集中运行微调版决策引擎 + 实时琅嬛玉洞合成），直接返回最终合成 Prompt。

---

## 3. “琅嬛玉洞”原子招式 Schema 与动态加权算法

### 3.1 原子招式数据结构 (`SkillAtom`)

```typescript
export type LifecycleStage = 'interrogate' | 'architect' | 'execute' | 'debug' | 'review';

export interface SkillAtom {
  id: string;                  // 如 "mattpocock:grill-me-tree"
  name: string;                // 如 "决策树穷尽式需求拷问"
  sourceRepo: string;          // 如 "mattpocock/skills"
  stage: LifecycleStage;       // 所属生命周期域
  staticWeights: {
    rigour: number;            // 严谨度 (0.0 - 1.0)
    velocity: number;          // 敏捷度 (0.0 - 1.0)
    authority: number;         // 领域权威度 (0.0 - 1.0)
  };
  techStackFilter?: string[];  // 如 ["react", "nextjs"]，空则通用
  mutexGroup: string[];        // 互斥组标签，如 ["planning:strict"] 与 ["planning:skip"] 互斥
  promptFragment: string;      // 精炼后的可执行指令切片（控制在 40-80 行以内）
}
```

### 3.2 动态加权与互斥消解公式（`src/harmonizer/synthesizer.ts`）

1. **门控拦截（Noul Gate）**：
   - 若用户意图是写代码（`execute`），但 `Noul(is_spec_ready) < 0.60` 且本地无 `PLAN.md`/`SPEC.md` 且 `Score(architecture_risk) > 5.0`，强制将阶段重写为 `interrogate`（需求拷问态）。

2. **动态得分计算**：
   - 记归一化风险权重 $r = \text{Score}(\text{architecture\_risk}) / 10$，敏捷权重 $v = \text{Score}(\text{urgency\_velocity}) / 10$。
   - 对匹配当前阶段与技术栈的每个原子招式 $a$：
     $$\text{Score}(a) = P_{\text{Choice}}(a.\text{stage}) \times \left( r \cdot a.\text{rigour} + v \cdot a.\text{velocity} + 0.5 \cdot a.\text{authority} \right)$$

3. **Top-K 贪心互斥过滤（Mutex Resolution）**：
   - 按 $\text{Score}(a)$ 降序排列，依次选入结果集（最多选 3 个原子招式）；若候选招式的 `mutexGroup` 与已选招式发生碰撞（例如已选 `superpowers:strict-tdd`，则自动过滤掉 `pstack:skip-plan-deep-dive`），彻底杜绝指令精神分裂。

4. **终端可视化遥测头（Viral Telemetry Banner）**：
   - 每次合成返回的 Prompt 顶部附带一行要求宿主原样输出的战报头：
     `> ⚔️ [慕容复·斗转星移] 清晰度: 0.22 | 风险: 8.4 | 阶段: 需求拷问 -> 已激活: gstack/narrow-wedge + mattpocock/grill-me`

---

## 4. 仓库目录结构设计

```text
murongfu/
├── package.json
├── tsconfig.json
├── SPEC.md
├── README.md
├── src/
│   ├── index.ts                 # 核心导出入口
│   ├── cli/
│   │   └── index.ts             # CLI: murongfu init | route | ingest | status
│   ├── probe/
│   │   └── collector.ts         # 本地 Git、文档、报错、技术栈状态采集器
│   ├── decision/
│   │   ├── types.ts             # Choice / Score / Noul 统一接口定义
│   │   ├── adapter.ts           # 统一路由器 (支持 local-kev, byok-jev, clef, cloud)
│   │   └── fallback-engine.ts   # 零依赖本地概率计算器 (保证开箱即用)
│   ├── harmonizer/
│   │   ├── schema.ts            # SkillAtom Zod Schema
│   │   ├── seed-atoms.ts        # 内置6大顶流仓库提炼的黄金种子招式库 (至少12个核心原子)
│   │   ├── ingester.ts          # 离线吸星编译器 (从 GitHub 拉取 SKILL.md 并拆解加权)
│   │   └── synthesizer.ts       # 加权评分、Mutex 互斥消解与 Prompt 合成器
│   └── mcp/
│       └── server.ts            # MCP Server (暴露 murongfu_route 等标准工具)
└── tests/
    └── router.test.ts           # 覆盖三大核心斗转星移场景的自动化测试
```

---

## 5. Task 模式 Vibe Coding 执行里程碑

- **Milestone 1：创建仓库与工程基建**
  - 在 GitHub 创建 `murongfu` 仓库，初始化 `package.json`（TypeScript, `@modelcontextprotocol/sdk` 或 `@modelcontextprotocol/server`, `zod`, `commander`, `vitest`, `tsup`），提交 `SPEC.md`。
- **Milestone 2：打造“琅嬛玉洞”种子库与吸星解析器 (`src/harmonizer/`)**
  - 实现 `SkillAtom` Schema，并在 `seed-atoms.ts` 中高质量编写来自 `obra/superpowers`、`mattpocock/skills`、`pstack`、`garrytan/gstack`、`anthropics/skills`、`vercel-labs/agent-skills` 的至少 12 个真实、精炼的黄金原子招式（涵盖 5 大阶段，明确设置 `rigour`、`velocity`、`authority` 权重与 `mutexGroup`）。
  - 实现 `ingester.ts` 支持拉取外部仓库 Markdown 并解析为原子结构。
- **Milestone 3：实现本地状态探针与统一决策引擎 (`src/probe/` & `src/decision/`)**
  - 实现 `collector.ts` 采集本地目录与 Git 状态。
  - 实现 `adapter.ts`（支持 Kev/Jev 标准 JSON 请求格式及本地 `fallback-engine.ts`），输出校准后的 `Noul`、`Score`、`Choice` 结果。
- **Milestone 4：实现加权合成器、MCP Server 与 CLI (`src/mcp/` & `src/cli/`)**
  - 实现 `synthesizer.ts` 的加权公式与 Mutex 互斥过滤，生成带 `[慕容复·斗转星移]` 战报头的合成指令。
  - 实现 `mcp/server.ts`（提供 `murongfu_route` 工具）和 `cli/index.ts`（支持 `murongfu init` 自动生成 `.cursor/mcp.json` 与 `murongfu-entry.md`，以及 `murongfu route "<prompt>"` 命令行直接预览路由效果）。
- **Milestone 5：自动化测试验证与推送发布**
  - 编写并跑通 `tests/router.test.ts`，验证以下三个场景不会发生指令冲突：
    1. 场景 A（模糊大需求）：输入“帮我做个 SaaS 计费系统”，无 `PLAN.md` -> `Noul` 门控拦截 -> 命中 `gstack:narrow-wedge` + `mattpocock:grill-me`。
    2. 场景 B（已有规格书的高风险开发）：检测到存在 `PLAN.md`，输入“开始实现核心支付模块” -> 门控放行 + 高风险 -> 命中 `superpowers:git-worktree` + `superpowers:strict-tdd`（且排斥 `pstack` 的跳过测试流）。
    3. 场景 C（前端快速迭代）：检测到 React 环境 + 低风险小改动 -> 命中 `pstack:deep-dive` + `vercel:react-best-practices`。
  - 确保构建与测试全部通过，将完整代码推送到 GitHub `murongfu` 仓库。
