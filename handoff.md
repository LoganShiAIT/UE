# UE Workspace Handoff Router

## ⚠️ 环境迁移索引（2026-07-31）——所有 Agent 开工前必读

**本机已重装系统，工作区路径已变更。任何文档、脚本或 handoff 中出现的 `C:\Users\Logan\Documents\GitHub\...` 一律作废。**

| 项 | 旧路径（已失效） | 当前路径 |
| --- | --- | --- |
| 工作区根 | `C:\Users\Logan\Documents\GitHub\UE` | `G:\备份\Documents\GitHub\UE` |
| 主线项目 | `…\UE\NiagaraFlocking 5.8` | `G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8` |
| Monolith 源码仓 | `C:\Users\Logan\Documents\GitHub\Monolith` | `G:\备份\Documents\GitHub\Monolith` |

**统一换算规则：`C:\Users\Logan\Documents\GitHub\` → `G:\备份\Documents\GitHub\`**（正斜杠同理）。遇到旧路径请就地换算，不要判定为文件缺失。

### 引擎（重要：有两套，别用错）

| 引擎 | 路径 | 用途 |
| --- | --- | --- |
| **Launcher 安装版 UE 5.8**（BuildId `55116800`） | `G:\UE_5.8` | **日常工作引擎**。Monolith、FlockingToolset、项目模块全部与它匹配。**其路径未随本次迁移改变。** |
| 自编源码版（BuildId `dbbdafe4-…`） | `<workspace>\UnrealEngine` | 仅用于 UE 源码语义核对，不用于启动项目 |

**引擎注册表已随重装丢失**（`HKCU\…\Unreal Engine\Builds`、`HKLM\SOFTWARE\EpicGames`、Epic Launcher 均无）。`.uproject` 双击无法解析引擎，必须直接带参启动：

> **2026-08-01 09:59 修正**：`HKCU\Software\Epic Games\Unreal Engine\Builds` 现已有一条 `{D5944C5E-4E7D-15D4-38AD-199DD5EA67C4} → G:/UE_5.8`（`HKLM\SOFTWARE\EpicGames` 仍不存在）。但 `NiagaraFlocking 5.8\NiagaraFlocking.uproject` 要的是 `{2BB79E4F-456A-4A68-FB5C-BD908A7C4590}`，**该 GUID 未在任何位置注册**，所以下面的带参启动方式**仍然是当前有效解法**。

```
"G:\UE_5.8\Engine\Binaries\Win64\UnrealEditor.exe" "G:\备份\Documents\GitHub\UE\<项目>\<项目>.uproject"
```

### 当前环境缺口

- **Visual Studio / Windows SDK 均未安装** → 无法编译任何 C++。但全部 Niagara 线项目为纯内容项目（无 `Source/`），不受影响；唯 `fishies` 打不开。
- 引擎自带 Python 3.11.8 与 .NET 10.0.203 可用；UnrealBuildTool 需经 `RunUBT.bat` / `Build.bat` 调用，直接调 `UnrealBuildTool.exe` 会报 `.NET location: Not found`。
- 非 ASCII 路径（`备份`）实测未造成故障，UBT 在该路径下正常运行。

### 关于历史文档里的旧路径（不要"顺手修好"）

`HandoffDocs/artifacts/**/test-results/`、`reports/`、`contracts/`、`misc/` 与 `archive/**` 共约 147 个文件仍含旧路径，**这是刻意保留的**：它们记录的是当时实际发生的事，改写等于篡改证据层。只有路由类文件（本文件、`HandoffDocs/handoffs/*.md`）和可执行脚本（`artifacts/**/test-scripts/*`）才应更新。

完整勘察报告：`NiagaraFlocking 5.8\HandoffDocs\artifacts\qoder-test--w-04\reports\20260731-210331-post-reinstall-environment-and-w04-interruption-audit.md`

### 🚚 换机迁移（2026-08-01）——**从另一台机器接手的 Agent 先读这条**

本工作区已完成换机前的完整性核查。**git 侧无工作丢失风险**：superproject `UE`、`NiagaraFlocking 5.8`、`UnrealEngine`、`fishies` 四个仓库本地 HEAD 与远端逐一比对全部一致，主力项目工作区 0 脏文件。

新机落地顺序：

1. `git config --global core.longpaths true`（本仓库历史含长路径，不设会 clone 失败）
2. `git clone --recurse-submodules https://github.com/LoganShiAIT/UE.git`
3. 读本文件上方的路径换算规则与两套引擎定位
4. **读 `NiagaraFlocking 5.8\HandoffDocs\handoffs\workspace-migration-to-new-machine.md`** —— 3 个必坏项的修法、哪些数据 git 永远带不走、逐条落地清单都在那里

三个必坏项摘要：① `.uproject` 的 `EngineAssociation` GUID 未在任何机器注册（改 `"5.8"` 指 Launcher 引擎即可，已验证其自带所需的 `ModelContextProtocol` / `MCPClientToolset`）；② `UnrealEngine` submodule 需 `Setup.bat` + 数小时编译，但它只用于源码核对、可跳过；③ `G:\FlockingC1` 不在任何仓库内且写死绝对路径。

⚠️ **接续 `claude-opus-5-test` 的硬阻塞**：`MonolithSandbox/Plugins/Monolith/Binaries/Win64` 的 20 个 DLL 未跟踪，**必须物理拷贝，重编译会使 hash 偏离 W-03 冻结基线、跨轮对照失效**。

---

- **2026-07-31 21:03:31 +08:00 Qoder test W-04 实际已跑过并被强制中断，判定作废重跑**：文件系统证据显示盲测作者于 2026-07-27 00:05→00:28 实际执行约 23 分钟，产出 19 个作者文件并在 00:18:39 创建资产 `Content/FaithfulFlockingQ1/NS_FaithfulFlocking_Q1.uasset`（419 KB）；Q1 编辑器日志写到 2026-07-29 08:35 且末尾仅剩 EOS 心跳、无退出记录，属重装系统导致的强制终止。该轮**无 final、无 self-check、未进 W-05 冻结**，且 Q1 已非空白，与"从空白制作"成功条件冲突。经 Logan 决定：**该轮作废，重跑一轮全新盲测**。Sealed packet 两份文件 SHA-256 复验与 W-02 记录完全一致，封印仍有效。端口 9318/9316/8000 空闲，Unreal 进程为 0。

- **2026-07-31 21:29:36 +08:00 `claude-opus-5-test` 的 C1 已 provision 并冻结（planned）**：Logan 选定隔离方案 A。C1 落 **`G:\FlockingC1`（刻意在工作区之外）**，已验证从 C1 向上至盘根路径链上无任何 `CLAUDE.md`；端口 **9319**，资产根 `/Game/FaithfulFlockingC1`，Content 资产数 0。Config 三份自 Q1 复制，`DefaultEngine.ini`/`DefaultInput.ini` 与 Q1 逐字节相同，`DefaultMonolith.ini` 仅 `ServerPort` 一行不同；`AdditionalPluginDirectories` 因不再同级而改用绝对路径，解析已验证（uplugin + 20 个模块 DLL 均可达）。**冻结 Monolith 作者面对照 W-03 基线完成迁移后首次复验：功能集 24/24 文件齐全，8 个关键 SHA-256 全部匹配——作者面在重装与迁移中零字节变化，该结论同时适用于 Qoder 轮重跑。** 合同两份与 sealed packet §3 的 9 个允许读文件亦全部复验通过，未重制任何合同。冻结报告：`NiagaraFlocking 5.8\HandoffDocs\artifacts\claude-opus-5-test\reports\20260731-212936-c1-sandbox-provision-and-environment-freeze.md`。启动前仅剩：临时改名用户级 `CLAUDE.md` → 生成 sealed prompt → 开全新 Claude Code 会话。**两轮必须串行。**

- **2026-07-31 21:24:29 +08:00 新增 `claude-opus-5-test` 事项（当时 blocked，现已解除）**：按 Logan 要求建立与 Qoder 轮同口径的 Claude Opus 5 对照测试，handoff 为 `NiagaraFlocking 5.8\HandoffDocs\handoffs\claude-opus-5-test.md`。设计上**直接复用 W-02 的冻结 sealed packet 与 contract 不重制**，保证唯一自变量是模型/harness；拟用独立 sandbox C1、资产根 `/Game/FaithfulFlockingC1`、端口 9319。前置勘察：合同两份 SHA-256 与 sealed packet §3 的 9 个允许读文件哈希**全部复验通过**；Claude Code memory 基线为 0。**阻塞项**：Claude Code 会自动加载 cwd 向上直到用户目录的所有 `CLAUDE.md`，若 C1 落在 `G:\备份\Documents\GitHub\UE\` 下将无条件注入两份 7,826 bytes 的同内容规则文件，其 L50–L53 明令 agent「用 Glob/Read/Grep 浏览项目结构」并创建/读取 `handoff.md` 与 `handoffs/<slug>.md`——**这是把盲测作者直接推去读禁读材料的常驻指令，性质等同于 Qoder 必须关闭的 auto-memory 且更强**。定案前不 provision C1、不启动作者。另记：本实验测量的是「模型＋harness」组合，结论不得简写为模型名对模型名；且 Claude 轮与 Qoder 轮**必须串行执行**，否则争抢同一 Monolith 二进制、引擎与 canonical 保护基线。

- **2026-07-31 21:11:45 +08:00 W-04 首轮收尾完成，Q1 已回到空白，仅剩 Qoder 未装一项阻塞**：经 Logan 授权，首轮全部痕迹（23 个文件：19 个作者产物 + 资产 `NS_FaithfulFlocking_Q1.uasset` + Autosaves 中的资产副本 + Editor 日志）已逐字节冻结至 `NiagaraFlocking 5.8\HandoffDocs\artifacts\qoder-test--w-04\misc\20260731-210949-invalidated-round-01-snapshot\`，复制前后 SHA-256 **23/23 全部匹配**后才执行删除。勘察中新发现的污染源 `Saved/Autosaves/…/NS_FaithfulFlocking_Q1_Auto1.uasset`（548 KB，会在下次开 Editor 时触发恢复写回）已一并清除。清空后核验：Q1 `Content/` = **0 文件**，`Saved/Config/` 三个 ini 经 grep 确认无任何 `FaithfulFlocking` / `Q1` 残留引用。刻意保留：Editor 日志原件（强制终止的关键证据）、`Config/` 三个冻结基线 ini（时间戳仍为 W-03 的 2026-07-26 23:12）。**该快照目录已列入盲测作者禁读清单**——它含首轮的 tools-list、action-schemas、stock-discovery 与 KG1 源码查询答案，读到即污染。

- **2026-07-26 23:53:53 +08:00 Qoder test W-04 已绑定待启动**：已建立 coordinator-only 记录 `NiagaraFlocking 5.8\HandoffDocs\handoffs\qoder-test--w-04.md`，但 blind author 不得读取该 handoff 或完整 task spec。启动前复核 memory=0、自动生成关闭、端口 9318 空闲、无 Unreal 进程；交接只通过 W-02 两份 sealed 文件。Logan 需在 Q1 目录打开全新 Qoder Ultimate 1M 会话并粘贴当前回复中的 prompt。

- **2026-07-26 23:48:56 +08:00 Qoder test memory blocker 已解除**：经 Logan 明确授权，3 条 coordinator 自动 memory 已删除并送入回收站；`C:\Users\Logan\AppData\Roaming\Qoder\User\settings.json` 的 `configMemoryAutoGenerate` 已临时设为 `false`。复核设置 JSON 有效、Qoder memory 文件总数为 0。W-01/W-02/W-03 均完成，W-04 尚未绑定或启动。

- **2026-07-26 23:34:40 +08:00 Qoder test W-04 memory blocker**：插件 provisioning 已用外部冻结目录闭合，但复核发现 coordinator 会话又自动生成 3 条 Qoder memory（`1M_context_档位为实验核心自变量`、`任务协调权威与绑定机制`、`模型官方名称`）。它们会污染全新盲测作者；删除和关闭自动生成尚未获本轮明确授权，故 W-04 暂不绑定、不启动。

- **2026-07-26 23:32:23 +08:00 Qoder test 插件 provisioning 已无拷贝闭合**：4.1 GB 主要是 Saved/PDB/Intermediate，不是 W-04 必需重复数据。Q1 `.uproject` 已使用 UE 5.8 原生 `AdditionalPluginDirectories` 从冻结的 `MonolithSandbox/Plugins` 解析同一 Monolith；descriptor 与 Niagara DLL hash 复验一致，未复制插件、未启动 Editor/MCP。W-01/W-02/W-03 均完成，下一步仅允许全新未读 spec 的 Qoder Ultimate 1M 会话执行 W-04，并在首次连接过 memory、端口、Monolith/live-schema、canonical hash 硬门。

- **2026-07-26 22:46:58 +08:00 Qoder test W-02/W-03 已绑定**：W-01 已完成，实验声明基线固定为 Qoder Ultimate + 手动 1M context；已为当前 Qoder coordinator 建立 `NiagaraFlocking 5.8\HandoffDocs\handoffs\qoder-test--w-02.md`（sealed author packet）与 `qoder-test--w-03.md`（隔离 sandbox/Monolith freeze）。当前会话禁止执行 W-04；盲测作者必须在两项完成后另开未读完整 spec 的全新 Qoder 会话。

- **2026-07-26 15:21:52 +08:00 R2 KB / MCP / Model 归因复盘**：已建立并完成只读复盘事项 `NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-flocking-r2-kb-mcp-model-retrospective.md` 的证据矩阵。完整 KB 对 `ScaledPosition`、`ExecIndex()`、standalone signature、material usage 等已有答案，但 R2 冻结作者包漏投递 persistence/pitfalls 两份关键文档；最终 HLSL 假 PASS 主要是模型合同保持与自检覆盖失败，fresh/DI/material 主要是 MCP 非事务化/结构化写入不足叠加作者包缺口，空间行为主要是 canonical 重建 manifest 缺完整支撑状态且模型未执行邻居密度/力预算预检。未修改知识库、Niagara 资产、插件或既有测试证据。

- **2026-07-26 13:56:09 +08:00 R2 HLSL 逻辑复核**：已对 `NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-flocking-faithful-reference-rebuild-02.md` 的成果完成不含截图/主观视觉的独立 review。warm-process HLSL 硬门本应 FAIL：Body Renderer 绑定 `Particles.ScaledPosition`，但 R2 HLSL 无该字段且作者检查器漏验；运动逻辑由正反馈 Alignment、前置限速、越界后阶跃墙力、无合力限幅/阻尼和非 canonical 支撑状态共同造成同步群与大幅越界；`UniqueID` 被误作 Particle Read 物理数组 index；fresh Output-pin 错误属于 standalone module 输出 signature/call-site 的磁盘重建失败，并伴随 DI/material 持久化缺口。报告：`NiagaraFlocking 5.8\HandoffDocs\artifacts\niagara-flocking-faithful-reference-rebuild-02\reports\20260726-135609-hlsl-logic-review.md`。未修改 Niagara 资产或插件。

## 全工作区强制验收指令：HLSL-first

本指令适用于本工作区内所有能够生成 HLSL 的 Niagara 构建、重建、修复、调参与模型/Agent 模板能力测试，不局限于 Flocking 或 MB01。

1. 验收必须按以下顺序进行：**意图与必需字段清单 → Source/Graph → 真实 Compile/Translate → Generated HLSL 语义硬门 → 平台 Shader 编译 → Runtime/Readback → 视觉验收 → Fresh-load**。
2. **Generated HLSL 是进入 Runtime 和视觉 PASS 之前的强制硬门。** 如果生成 HLSL 未产生、不可读取、仍有编译错误，或没有完整表达预期逻辑，则不得把 Runtime、视觉效果或整体任务判为 PASS；视觉上“很接近”不能补偿语义链缺失。
3. HLSL 检验不得退化为关键词搜索。至少应按任务需要检查：数据集/参数结构中的必需字段、正确 Stage 中的 writer、Parameter Map 与属性 load/store、Data Interface 的声明和真实调用、Stage source/destination 读写闭环、关键计算与积分顺序，以及 Renderer 已绑定字段是否确实由上游生成并持续保留。
4. 每个任务在制作前应建立“必需编译字段/语义清单”，在生成 HLSL 中逐项证明正向签名，并检查会暴露伪实现或漏写的负向签名。应保留可复核的有限 HLSL 摘录、完整输出哈希或等价证据，避免只记录最终 PASS 标签。
5. HLSL 硬门不能替代后续验证：通过 Generated HLSL 后，仍须完成平台 Shader 编译、实时 Runtime/Readback、由 Logan 负责的主观视觉验收，以及任务要求的 Fresh-load；各层结论必须分别记录，不得相互冒充。
6. 若工具缺陷导致无法取得 Generated HLSL，应把该项记为明确阻塞或未验证，不得降级成视觉验收后默认通过。

详细通用规范：`NiagaraFlocking 5.8\HandoffDocs\artifacts\niagara-graph-code-mapping\knowledge-base\niagara-from-blank-agent-build-recovery-and-hlsl-first-acceptance-ue5.8.md`。Simulation Stage 的 Source/HLSL/Runtime 闭环规则：`NiagaraFlocking 5.8\HandoffDocs\artifacts\niagara-graph-code-mapping\knowledge-base\niagara-simulation-stage-source-hlsl-runtime-semantic-closure-ue5.8.md`。

- **2026-07-25 工作区指令升格**：按 Logan 决定，HLSL-first 从知识库与项目 handoff 中的验收规范升格为本主 Handoff 的全工作区强制指令；未写入 `C:\Users\Logan\.codex\AGENTS.md`，因此不会影响 UE 工作区以外的其他任务。
- **2026-07-25 23:25:42 +08:00 Flocking 第二轮测试已初始化**：Logan 确认第一轮候选持续向角落和边缘聚集，故第一轮从 near-pass 纠正为 **FAIL / 未完成半成品**；编译、HLSL、runtime、fresh-load 与 80/81 仅保留为技术证据。新 full handoff `NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-flocking-faithful-reference-rebuild-02.md` 已建立，要求更新 KB-only 作者包、独立上下文/资产根、HLSL-first 硬门及预注册边缘/角落持续性行为门；当前尚未选定作者模型或启动 Editor。
- **2026-07-25 21:35:00 +08:00 HLSL-first 验收与双知识文档**：按用户要求将 GPT-Sol MB01 近一小时试错拆成通用 Agent 构建/恢复流程与 MB01 项目专用知识。验收顺序固定为 Source → 真实 Compile/Translate → Generated HLSL 语义硬门 → 平台 Shader 编译 → Runtime → 视觉 → Fresh-load；HLSL 必须同时覆盖模拟字段、DI、Stage stores 和 Renderer 必需属性。原 `renderer_display_conformant=true` 已被 SpriteSize 漏验推翻，历史报告保留但 handoff 使用修正结论。权威记录：`NiagaraFlocking 5.8\HandoffDocs\handoffs\gemini-test.md`。
- **2026-07-25 21:17:12 +08:00 GPT-Sol MB01 后验可见性修复**：用户报告只能辨认少数对象；确认 Alpha=1、Burst=300、每组 100 个 slot 并未完全同位。根因是原 Relay HLSL 无 `SpriteSize`，UE 5.8 Sprite Renderer 回退 `50×50`，在小轨道上密排后合成三段连续色环。已备份原 System，显式写入 `SpriteSize=(8,8)`，双 emitter Bounds 从 `±1400` 收紧到 `±900`；真实编译 0/0、最终 HLSL load/store/Stage 保留链成立。视觉仍由 Logan review；此人工后验修复不计入模型原始得分。权威记录：`NiagaraFlocking 5.8\HandoffDocs\handoffs\gemini-test.md`。
- **2026-07-25 20:58:10 +08:00 GPT-5.6-Sol High 第二模型轮技术 PASS**：无聊天上下文 subagent 使用冻结非 Flocking MB01 合同和独立资产根完成；作者 final fresh 为 compile 0/0、Stage `WritesParticles=True`、真实 ParticleRead closure、GPU Runtime Drivers=3 / Relays=300 和三时点 finite motion。root 独立 fresh 又复现 Source/HLSL/compile/runtime/readback，并确认写前 39 资产 hash 全匹配。作者用了 338 calls / 57m21.511s；Gemini 首轮为 102 calls / 约5m43s但 FAIL。用户视觉评价和 disabled predecessor 可维护性待 Logan review；所有目标 Editor 已正常退出。权威记录：`NiagaraFlocking 5.8\HandoffDocs\handoffs\gemini-test.md`。
- **2026-07-25 19:12:43 +08:00 Gemini test 首轮完成**：`agy` 1.1.6 已实际运行 `gemini-3.6-flash-high` / effort `high`，按冻结的非 Flocking MB01 Orbital Relay 合同从空白创建 3 个资产。独立验收结论为 **FAIL（非工具面阻塞）**：结构外形成立，但 HLSL 仅写 `OUTPUT_VAR.*`、Relay Stage=`WritesParticles=False`、Particle Reader 函数未声明导致 GPU Shader 2 errors，runtime Disabled 且无粒子数据；有效 fresh-load 在全新 PID 21672 中完整复现相同失败，原 36 沙盒资产 hash 全匹配。Gemini 单模型轮已结束并冻结；Editor 已正常退出，TCP 9316 已释放。权威记录：`NiagaraFlocking 5.8\HandoffDocs\handoffs\gemini-test.md`。
- **2026-07-25 09:29:04 +08:00 Faithful rebuild 改为最大信任执行模式**：`niagara-flocking-faithful-reference-rebuild` 已从 `planned` 改为 `ready-for-execution`。另一 Agent 被指派读取 handoff 后即可自主认领并执行：选择/调整 sandbox、启动 Editor/MCP、创建和迭代任务资产、制作探针、补 task-local 作者能力并完成验证，无需逐项请求授权。authorability 改为优先技术阶段而非审批闸门；仅保留 faithful 目标、canonical/C 不写、scalable 后置和不破坏任务外用户数据。本轮仍只改 handoff，未执行重建。
- **2026-07-25 00:03:56 +08:00 早期知识库正式接入 faithful 路线**：`niagara-flocking-reference-c-gap-analysis` 已新增 Early Knowledge-Base Reuse Contract，将早期 Graph–Code / Flocking Agent Template v1/v2 成果按 faithful 设计、authorability gate、验收护栏三个 phase 精确路由，并冻结当前语义 oracle > 早期 KB > 历史实验的证据优先级。未来仍须另行取得写授权，在新 sandbox 先过 Simulation Stage、DI/pin 与 Body 跨 emitter 显示链 gate；本轮未启动 Editor、未改资产、未开始重建。
- **2026-07-24 23:56:40 +08:00 Canonical Flocking 语义闭环完成**：`niagara-flocking-reference-c-gap-analysis` 的 B0–B2、T1–T7 全部完成，执行全程只读。canonical 的 Asset/Graph/Parameter Map/HLSL/UE runtime/buffer/逐帧数学已冻结；C 已按数值、数学、采样、时序、拓扑、显示链、作者能力七轴完成同 schema 对照。未来路线选择 **faithful reference rebuild first**；C 只作负面对照，scalable 版本必须等 faithful 验收后另立任务。当前没有获得写授权，未启动资产重建。权威 handoff：`NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-flocking-reference-c-gap-analysis.md`。
- **2026-07-24 23:32:25 +08:00 Canonical Flocking 语义闭环已可转交**：在现有 `niagara-flocking-reference-c-gap-analysis` full handoff 中建立正式 Task Board：B0–B2 为已完成参数/HLSL/UE 源码底座，T1–T7 依次覆盖 canonical 拓扑、Graph↔HLSL、Runtime 契约、逐帧语义、理解验证、C 同 schema 对照与重建路线决策。独立 Agent Prompt 已落 `HandoffDocs/artifacts/niagara-flocking-reference-c-gap-analysis/misc/20260724-233225-canonical-flocking-semantic-closure-agent-prompt.md`。任务尚未启动，默认只读，不修改资产。
- **2026-07-24 23:17:54 +08:00 UE 5.8 Niagara 源码交叉确认**：窄读 `NiagaraHlslTranslator.cpp`、`NiagaraGpuComputeDispatch.cpp`、`NiagaraDataInterfaceParticleRead.cpp` 后确认：完整粒子写 Simulation Stage 使用分离 Source/Destination buffer，自身 Particle Attribute Reader 明确读取当前 Stage 的 Source 快照，partial update 自读会被判为竞态风险。当前差距不能归结为“模板库少了一套模板”，而是 Epic 内容依赖缺口、作者工具结构表达缺口和知识语义缺口三层叠加。
- **2026-07-24 22:53:49 +08:00 canonical Flocking HLSL 导读完成**：`niagara-flocking-reference-c-gap-analysis` 已完整阅读 2,133 行生成 HLSL。确认真实帧序为 Stage 0 先限速并积分 Position，Stage 1 再从统一输入快照做 O(N²) 全粒子邻居读取、同时求 Separation/Alignment/Cohesion/硬墙四项力并只写 Velocity；当前代码不消费 NeighborGrid3D。源码导读已落盘，下一步应把函数、Parameter Map 与 buffer 读写映射回 Niagara 图，不修改资产。
- **2026-07-24 22:44:46 +08:00 Flocking 参数对照完成**：`niagara-flocking-reference-c-gap-analysis` 已完成 canonical 与 C 的第一版参数/结构报告。当前资产 SHA 与引用证据完全一致，未启动 Editor、未改资产。首要结论：C 的 `CandidateCount=128` 令 Separation/Alignment/Cohesion 预计只命中约 `1.31/3.81/8.38` 个邻居，吸引与排斥命中不对称，是“聚点”的首要候选；`BoundaryBand=80 + PredictionTime=0.75 + MaxSpeed=120` 使逐轴墙力深入场内并保留切向速度，是“贴面/沿边”的首要候选。canonical 是全粒子扫描、四项力同时求和、`±300` 越界硬墙、local-space + Body 缩放显示；C 是顺序改写 Velocity、预测软墙、world-space 直显，不能靠复制八个数实现等价。
- **2026-07-24 22:31:58 +08:00 新主线**：用户暂停“模板库生产化”，目标回到“理解完整 Niagara 系统”。已建立 full handoff `niagara-flocking-reference-c-gap-analysis`，只读比较 canonical `/Game/BoidSystem` 与模板 C `/Game/TemplateBatch01/C_Flocking/NS_GPUFlocking_Repaired` 的参数、模块次序、数据流、算法语义和运行表现，并解释已观察到的“聚点/贴边”。本任务先产出参数矩阵，将差距分为数值、输入/开关暴露、算法/拓扑三类；原模板与 C 均不修改。
- Active project: `G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8`
- **2026-07-24 22:21 +08:00 权限纠正与客观复核完成**：验收职责正式纠正为“**客观工具/结构/编译/fresh-load 由 Agent；主观观感与完成度由 Logan**”。T3 确认 stock 模块只返回 `OutputMap`；T4 确认 `preview_system` 因 `editor` namespace 未注册而不可用；T5 原“未实现的 stub”结论被推翻，已成功创建并在全新进程 fresh-load 为 `NiagaraStatelessEmitter`。沙盒新增1个 T5 客观探针，原19资产 hash 全匹配；无活动 Editor/目标端口。`niagara-template-batch-01` 第一组收尾全部完成，可归档。
- **2026-07-24 22:06 +08:00 收尾进展**：继续使用现有 `niagara-template-batch-01`，不新建 handoff。Logan 已完成 C 门 6观察，结论为“有问题：部分复刻，运动聚集在点和边上”；铁律 2 已定稿；所有涉及资产的核验今后统一由 Logan 执行，Codex 只提供规范与记录。T3/T4/T5 用户验收规范已落地，等待 Logan 实测，其中 T5 的“stub”描述与冻结源码中的完整创建/保存实现矛盾，结论已降级为待验。本轮未启动 Editor、未调用资产端点、未修改或创建资产。
- **2026-07-24 18:33 +08:00 接力校准**：当前事项仍为 `niagara-template-batch-01`。P0/P1/P2 已全部完成，四个模板与收口报告均已落地，**无阻塞，可直接结项**。四条遗留待办均不阻塞结项：① C 需按门 6 的观察卡补看；② 铁律 2 在下批开工前修订；③ 工具面笔记 T0–T13 整体复核，尤其 T3/T4/T5 尚未由 harness 独立复核；④ 模板库长期归宿未定。权威详情只读 `NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-template-batch-01.md` 的 `Handoff Back`；下方较早的 P0/P1 路由信息仅作历史背景，不得覆盖本条。
- Latest completed task: `flocking-blindtest01-overview-repair` — 固定 BlindTest01 Overview 已持久化为且仅为 `System + FlockingBlind01`，fresh-load/Stack/编译/GPU 短跑/全 Content 审计 14/14 PASS（2026-07-18 23:20）。**该结论只覆盖那个固定资产**，不等于通用可编辑性已解决。
- Task routing: **当前主线是 `niagara-template-batch-01`**（2026-07-22 立项）——用冻结的 KB + 冻结的 Monolith 作者面产 3 个模板并量测速度，生产即盲测。`agent-authoring-sandbox` 与 `niagara-field-semantics-kb` 降为支撑：前者 paused 且作者面冻结，只作已证事实的只读来源；后者只欠 P0-b 一条待补事实。`monolith-source-index-sandbox` 的 M1/M2 已完成并冻结为 observe Source 证据面；`flocking-agent-visual-authorability-exploration` 保持 paused（E0–E4-A 成果保留，E4-B/E5 不自动继续，新线路启动**不构成**恢复它的理由）。
- 2026-07-22 纠偏：`agent-authoring-sandbox` 的 S1 从「验证 Monolith 够不够用」滑成了「把 Monolith 做够用」。两条复盘规律——**能力缺口只能在生产中撞出来，零个来自规划推导**；**以能力验证为完成条件的 handoff 全部中断，以交付物为完成条件的全部完成**。故主线改为产模板，能力缺口只作副产物记录。Flocking 原始 S1 的 UE 5.8 发行内容缺陷已两路验证，属**已关闭的外部因素**，不再阻塞模板库路线。
- Status: active。`niagara-template-batch-01` 的 **P0 已全部完成，下一步直接开工模板 A（喷泉/爆发）**。已就位：G2 源码归档；决策收口（模板在沙盒生产并逐个回流、**沙盒本批期间不可删**、本批只做结构/编译/fresh-load/可编辑四项不做视觉）；KB 补 F12（standalone module script 保存前须 `RequestCompile`）与 `remove_emitter` 禁用规避，现为 12 条事实 + 2 条操作规避；基线冻结（KB 18 份 SHA-256 + schema niagara=133，静态计数已与两次历史 live probe 对账吻合）。
- **2026-07-23 关键教训：验收独立性要独立到方法，不只是独立到人。** 模板 C 的作者与 harness 各自独立工作、互不通气，但都用了 `get_system_diagnostics` 的 `compile_first=false`（只读磁盘上持久化的编译状态快照，而非发起真实编译），于是产出两份一致的错误 PASS。真实编译报 **9 error**。推翻它的是 Logan 在编辑器 UI 里打开资产——**最朴素的路径反而是唯一有效的证伪手段**。「不触发编译更干净」这个直觉是反的：真实编译是整套验收里唯一能证伪的步骤。已固化为 KB **F13**、KB README 防误用规则 4→6 条、工具面笔记 **T0**。A、B 经真实编译复核不受影响。
- **Flocking 已进入本批，排为模板 C**（2026-07-22 22:50）。批次 3 → 4 个模板，排期 **A（喷泉/爆发）→ B（噪声力场）→ C0 探针 → C（Flocking）→ D（自建 graph-native 软边界模块，加到 C 上作墙力）**。走「只替换采样器不替换行为」路线绕开 Epic 内容缺陷：用现成 `create_module_from_hlsl` 写邻居采样叶子输出 `Particles.SampleNeighbors.*` 四属性，`NeighborBehaviours / Drag / LimitForce / SolveForcesAndVelocity` 原样可用。**重建 `AverageNeighborVector` / `FilterNeighborsByDirection` 仍是 Out of Scope**——需 function-call/DI/convert 节点，开工即触发停止条件。C0 探针先验证「HLSL 能否写入该命名空间并被 stock 模块读到」，不过则当场否掉该路线。`niagara-field-semantics-kb` **内容已完备**（16→18 份，持久化层文档含 11 条双锚点事实），只欠 G2 的 `RequestCompile` 一条。`agent-authoring-sandbox` **paused**——S0/S1 已建成独立控制台优雅退出、只读 Overview 探针、5 个通用 graph-native 作者动作（G2 PASS），沙盒 4 个资产；已实测三个缺陷：Monolith 本就是 dispatcher 形态（M3 无需自研）、`remove_emitter` 留持久化孤儿节点（**未修，本批禁用该动作**）、NeighborQuery 缺 NiagaraFluids 插件依赖会崩溃编辑器（已入 KB F11）。
- **仍未成立的通用命题**：「仅凭知识库独立完成可编辑最终资产」。BlindTest01 的 `editor_authorability` 只在那个固定资产上 PASS；KB 虽已补上 editor-only 持久化层这根轴，但**从未被一个只读它的新 Agent 验证过**——原计划的 S2/E5 盲测因 S1 受阻从未执行。`niagara-template-batch-01` 把盲测折叠进生产，本批即是该命题的首次真实检验。
- Updated: 2026-07-22 22:10 +08:00（本轮只读沙盒源码、写 artifacts 归档与 handoff/索引；未启动 Editor、未构建、未改沙盒或任何 Content。基线仍为 `b351e47`，未 push。）
- Current milestone: **产模板并量速度**（`niagara-template-batch-01`）。四条铁律：本批不改 Monolith 一行代码；新增通用能力需 ≥2 个互不相关模板被同一缺口卡住；新动作命名须来自 Niagara 自身词汇（node/pin/module/stage）而非效果词汇；禁用 `remove_emitter`（孤儿节点缺陷未修，改用重建 System）。
- 历史 milestone（2026-07-20）: 主机制从「加验收门」改为「补字段语义」；作者能力实验搬出真实项目，安全性改由一次性项目隔离 + git 基线提供，不再由 per-call 七件套围栏提供。已定：作者面纯 Monolith、零 FlockingToolset 端点；验收观测仪器自留不给盲测 Agent；fresh-load 保留；E5 允许查 UE 源码但每次查询记账并回流成知识库缺口清单。真实项目继续 `observe` only。

本文件只做入口路由。不要依赖旧聊天记录，也不要默认加载旧 roadmap、timestamped artifacts 或其他任务 handoff。

## 默认阅读顺序

0. **接续当前主线时只读这一份**：`NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-template-batch-01.md`。它的 Context Panel 已列出所需材料，并明确了哪些旧文档只能当只读证据、哪些默认不读。`niagara-field-semantics-kb.md` 与 `agent-authoring-sandbox.md` 现为支撑线，按该 Context Panel 指示按需展开。
1. `G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\handoffs\niagara-graph-code-mapping.md`
2. 该 handoff 要求的四份 Context Panel。
3. `G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\artifacts\niagara-graph-code-mapping\reports\20260716-224535-flocking-agent-template-v2-consolidated-manifest.md`
4. `G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\artifacts\niagara-graph-code-mapping\knowledge-base\README.md`
5. 需要执行空白生成时读取：`flocking-from-blank-quality-contract-ue5.8.md` 和 `flocking-agent-generation-acceptance-template-v2.md`。
6. 需要复核已实现 V2 时读取：`G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\archive\2026-07\flocking-knowledge-only-quality-remediation.md`
7. 需要复核 Template v2 盲测结论或其中间失败经验时读取：`G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\archive\2026-07\flocking-agent-template-v2-blind-validation-01.md` 及其最终报告 `HandoffDocs/artifacts/flocking-agent-template-v2-blind-validation-01/reports/20260717-173905-final-blind-validation-report.md`
8. 仅在需要查看项目任务列表或追溯已归档任务时读取：`G:\备份\Documents\GitHub\UE\NiagaraFlocking 5.8\HandoffDocs\handoff.md`（Archived 表 → `HandoffDocs/archive/2026-07/<slug>.md` 全文与 Archive Summary）

当前主线进入第三阶段：**让知识库能教会 Agent 字段语义**。未来 Agent 仍须先选择 `existing_asset_analysis_or_tuning` 或 `from_blank_generation`；既有 V2 是算法/运行质量契约的已验收实例，但在完整作者 API 手册与字段持久化层文档落地前，不得当成“完全自包含、可编辑性已全部通过”的通用模板，也不是其他效果的默认参数来源。BlindTest01 的 Overview 修复已完成，但那是单个固定资产的回归结论，不构成通用可编辑性证明。

## 当前范围

Template v2 已把 `/Game/BoidSystem` 的既有资产知识和 `/Game/FlockingAgentGenerated/NS_FlockingKnowledgeOnlyV2` 的空白生成质量证据整合为设计前硬门、失败分类、操作顺序和总体通过矩阵；`flocking-agent-template-v2-blind-validation-01` 已实证算法/运行/视觉等门可执行，但历史 11/11 因 Overview 缺陷不再是总体结论。固定路径 Overview 修复已于 2026-07-18 完成并 PASS；此后不应再修改 V2 或 BlindTest01 资产。新路线的作者能力实验一律在一次性沙盒项目中进行，不在本项目内写入；任何新目标仍应创建独立 task handoff。

注意：现存已保存用户资产不止 canonical 两个——`Content/FlockingAgentGenerated`（KnowledgeOnly、Remediated、BlindTest01 共 11 个）、`Content/FlockingGenerated/NS_FlockingStandalone`，以及 `M_Fish`、`BodyEmitter`、GridVisualizer、`NiagaraFlocking.umap` 等 8 个支撑资产。未来任何写任务的受保护基线应默认覆盖**全部既有 Content 资产**，而不是只挑 Niagara System（盲测轮只冻结 10 个，事后核验其余 8 个 mtime 未变，属侥幸而非设计保证）。

语义正确性和可复核行为优先；低 token 是紧凑默认视图与渐进展开的结果。不要恢复旧的通用 recipe executor、Leader/Follower 或全局 Niagara 扩展为默认计划，除非用户重新授权。

## 安全提醒

- 除当前任务明确授权的单一 Flocking 目标外，所有 Niagara 写实验仍限内存并必须 exact rollback；canonical `/Game/BoidSystem` 与 `/Game/SimulateBoids` 永不保存；
- 不修改 UE 引擎源码，不做 Tier 2，不允许 CPU fallback；
- 不 reset/move/delete/stage/commit/push；其他任务 handoff 可按用户授权只读取证，未经新授权不修改；
- 当前易变状态：2026-07-20 17:20 复查 UnrealEditor、UnrealEditor-Cmd、CrashReportClient 均为 0，TCP 8000/8001/8002/9316 均未监听；下一次需要 Editor/MCP 的任务仍应先 fresh probe。最近 E4-A live schema 为 105 endpoints，以新会话 live probe 为准。
- 围栏放松只适用于一次性沙盒项目。本项目内的任何写任务，写前全 Content 基线、exact rollback 与 canonical never-save 纪律**不变**；`b351e47` 是 git 回滚基线，不是省略这些的理由。
- 工作区卫生（2026-07-17）：外层 `C:\Users\Logan\Documents\GitHub\UE\.git` 空目录已删除——真正的 git 仓库只有 `NiagaraFlocking 5.8\.git`，不要被外层误判"clean"；2026-07-16 会话散落在 `GitHub\`、`GitHub\UE\` 和项目根的 24 个 IPC token/自检日志已归档到 `NiagaraFlocking 5.8\Logan的文档\debug\20260716-editor-ipc-residue\`（含成因说明）。未来 editor 握手脚本必须用绝对路径指向任务 artifact 目录，禁止依赖进程 CWD。
- 同粒子关联只允许完整 `Index + AcquireTag`；GPU buffer/sample/array index 与 `UniqueID` 禁止作为 Persistent ID。

上一个 Persistent-ID 任务的 39/39 schema、三时点 tuple、rollback、fresh no-save barrier 和 completion audit 仍是已接受底座；记录身份只用于漂移检测。若重新开启 construction-plane 或持久化任务，必须从 fresh probe、写前冻结和新的明确授权开始。

旧 Graph–Code Bridge 里程碑已由任务索引路由到 historical compact report，成果保留但不占默认上下文。
