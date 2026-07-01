下面先按“研究笔记第一版”展开。核心不急着收敛，而是把每组问题继续向外发散成：**人类机制 → Agent Memory 设计启发 → 新机制/架构 → 评测/产品方向**。

---

# 总纲：Agent Memory 不应只是“存储系统”，而应是“改变未来行为的系统”

人类记忆不是为了完整保存过去，而是为了让未来的注意、判断、行动、风险规避、身份连续性变得更好。
所以 Agent Memory 也不应只问：

> 这条信息要不要存？

而应问：

> 这条信息将来会在什么场景下改变 Agent 的行为？
> 它会让 Agent 更快、更稳、更少重复犯错，还是会污染判断？

因此可以把 Agent Memory 拆成七个核心环节：

```text
感知/注意 → 写入候选 → 情景记录 → 巩固抽象 → 召回使用 → 行为改变 → 再验证/更新/遗忘
```

这比“向量库 + RAG”更接近人类记忆。向量库只解决“找相似内容”，但人类记忆还解决：什么值得记、为什么记、何时想起、如何避免误用、如何修正旧经验、如何形成技能、如何形成团队知识。

---

# 一、记忆的本质：从“保存过去”到“塑造未来行为”

人类记忆承担的不只是归档功能，而是五类功能：预测、决策、身份、风险规避、行为自动化。

对 Agent 来说，这意味着 memory 的最小单位不应该是“文本片段”，而应该是：

```text
过去情境 + 当时目标 + 采取动作 + 结果反馈 + 未来适用条件 + 预期行为改变
```

例如 Coding Agent 不应该只记住：

> 这个项目用 pnpm。

而应该记住：

> 在 repo A 中，安装依赖和跑测试必须使用 pnpm；上次误用 npm 导致 lockfile 变化和 CI 失败；未来修改依赖前先检查 pnpm-lock.yaml。

这才是“改变未来行为”的记忆。

由此可以产生一个关键设计方向：**Behavior-Oriented Memory Schema**。每条记忆都要回答三个问题：

```text
它来自哪里？
它适用于哪里？
它将来要改变什么行为？
```

这也解释了为什么人类不会完整保存所有经历。完整保存会造成检索噪声、认知负担和错误迁移。Agent 也是一样：全量工具日志不是记忆，只是原始证据。真正的 memory 应该是经过筛选、压缩、标注边界后的行为资产。

---

# 二、注意与筛选：Memory Admission Gate

人类注意系统不是平均分配的，它会优先关注目标相关、异常、高代价、高奖励、高重复、高社会反馈的信息。

Agent 也需要类似的 **Memory Admission Gate**，否则会出现“什么都记”的污染问题。特别是 Coding Agent 每次任务会产生大量信息：用户需求、工具调用、diff、错误栈、测试输出、模型推理、临时假设、失败尝试。如果全部写入长期记忆，长期看会比没有记忆更糟。

可以设计一套写入候选评分：

```text
memory_score =
  goal_relevance
+ recurrence
+ failure_cost
+ user_correction_weight
+ verification_strength
+ future_triggerability
- locality_penalty
- ambiguity_penalty
- staleness_risk
```

这里最重要的是 **future_triggerability**：这条信息未来能不能被某个明确线索触发？
比如“用户不喜欢太泛的总结”就容易触发，因为当 Agent 下次写总结时可以使用。
但“某次中间日志显示 test failed”如果没有最终归因，就不一定值得长期保留。

新机制可以叫：

```text
Memory Candidate Filter
```

它在 hook 日志和长期记忆之间工作，只产生候选，不直接入库。候选需要经过验证、聚合或人工确认，才能升级成 project memory、team memory 或 skill。

评测方向：构造大量任务日志，让系统判断哪些片段应该成为长期记忆。评价指标不是“召回多少”，而是：

```text
高价值记忆保留率
低价值噪声过滤率
错误记忆写入率
未来任务命中率
```

---

# 三、情景记忆：Agent Episode 不只是日志，而是一次“可复盘经历”

人类记住一次经历时，不是保存孤立事实，而是保存事件结构：时间、地点、人物、目标、动作、情绪/重要性、结果、解释。

Agent 的 episodic memory 也应该如此。一次 coding task 的 episode 可以包含：

```yaml
episode_id:
repo:
branch:
user_goal:
initial_context:
files_touched:
commands_run:
tool_results:
failed_attempts:
successful_steps:
final_diff:
verification:
user_feedback:
risk_events:
decision_rationale:
open_questions:
future_relevance:
```

这里最容易被忽略的是 **decision_rationale**。很多 Agent 记录“做了什么”，但不记录“为什么当时这么判断”。
可没有 rationale，未来 Agent 就无法判断这条经验是否适用，只能机械复用。

例如：

```text
当时选择改 parser 而不是改 validator，是因为测试失败发生在 AST 构建阶段，而不是 schema 校验阶段。
```

这类信息对未来定位问题很关键。

新机制：**Causal Episode Trace**。
它不是全量 chain-of-thought，而是可公开、可验证的“决策因果摘要”：

```text
观察到什么 → 排除了什么 → 选择了什么 → 结果如何 → 证据是什么
```

产品方向：Coding Agent 结束任务时自动生成 **Task Memory Card**，类似人类复盘卡片，不是冗长日志，而是可读、可检索、可升级的 episode。

---

# 四、语义记忆：从 episode 到 project facts / rules / pitfalls

人类会从多次具体经历中抽象稳定知识。Agent 也应该从 episodic memory 中抽象出语义记忆，但抽象必须保留边界。

例如多次 episode 显示：

```text
修改 auth middleware 后，必须同时更新 integration test。
```

这可以升级为 project rule：

```yaml
type: project_rule
rule: 修改 auth middleware 后必须运行 auth integration tests
scope:
  repo: api-service
  paths:
    - src/middleware/auth/**
evidence:
  - episode_001
  - episode_017
  - ci_failure_2026_...
confidence: high
not_apply_when:
  - only docs changed
  - test fixture only
```

关键不是“总结一句话”，而是保留：

```text
适用范围
证据来源
置信度
反例
失效条件
```

人类会过度泛化，Agent 也会。比如一次“npm install 导致问题”不能泛化成“所有 JS 项目都不能用 npm”。所以 Agent 需要 **Generalization Guard**：

```text
单次经验默认不能升级为全局规则
局部经验必须带 scope
跨项目迁移需要显式验证
规则升级需要多个 episode 或强证据
```

评测方向：设计任务集，测试 Agent 是否能从三次类似失败中抽象出规则，但不会把一次偶然失败错误泛化。

---

# 五、程序记忆：从知道规则到自动执行 skill

人类熟练工程师不是每次从零推理，而是拥有大量程序记忆：看到某类错误栈，自然知道先查什么；改某类文件，自然知道跑什么测试。

Agent 的 procedural memory 不是普通文本，而是可执行的工作流、命令、hook、检查器或子 Agent skill。

例如语义记忆是：

```text
修改 migration 后要检查 generated schema。
```

程序记忆则是：

```bash
pnpm db:generate
pnpm test:integration -- schema
git diff -- generated/**
```

也可以是 hook：

```text
当 files_touched 匹配 migrations/** 时，自动提醒并运行 schema check。
```

所以可以设计 **Skill Promotion Pipeline**：

```text
episode success pattern
→ workflow candidate
→ repeated validation
→ human review
→ local skill
→ project skill
→ team skill
```

这对应人类从“知道怎么做”到“自然会这么做”的过程。

但坏习惯也会形成。Agent 可能把错误经验固化成自动化行为，例如每次测试失败就盲目 mock 掉异常。
因此 procedural memory 必须有：

```text
success evidence
failure evidence
rollback mechanism
usage counter
last validation time
negative feedback log
```

产品方向：`agent-external-brain` 可以提供：

```text
aeb skill list
aeb skill promote
aeb skill rollback
aeb skill test
aeb skill explain
```

让 skill 不是黑箱，而是可观察、可退回、可审计。

---

# 六、情绪与重要性：用工程信号模拟情绪权重

Agent 没有情绪，但可以用工程事件模拟“重要性权重”。人类会强烈记住恐惧、奖励、羞耻、惊喜、重大事故，因为这些事件对未来行为有高预测价值。

Coding Agent 中对应的高权重信号包括：

```text
用户明确纠正
CI 失败
线上事故
安全风险
数据损坏风险
重复犯错
高成本回滚
PR review 强烈反对
测试长期失败后终于修复
```

这些事件应该影响写入、巩固、召回和触发强度。

可以设计 **Affective Weight Emulator**：

```yaml
importance:
  severity: high
  reason:
    - user_correction
    - repeated_failure
    - production_risk
  behavioral_effect:
    - warn_before_action
    - require_verification
    - retrieve_early
```

但人类也会被强情绪误导。一次高压失败可能让人过度保守。Agent 也可能因为一次严重失败，未来在所有类似场景都拒绝行动。
因此高权重记忆需要 **counterbalance**：

```text
高权重 ≠ 永久真理
事故记忆必须绑定触发条件
高风险提醒需要可消除
多次成功反例可以降低权重
```

产品方向：**Incident Memory**。
每次严重失败生成事故记忆卡：

```text
事故现象
根因
影响范围
修复方式
未来触发条件
必须检查项
误报风险
```

它比普通 project memory 权重更高，但也更需要边界。

---

# 七、巩固与睡眠：Agent Dream / Consolidation Pipeline

人类睡眠不是简单休息，而是离线重放、压缩、重组、筛选记忆。Agent 也需要“任务结束后”和“空闲时”的离线整理。

这可以成为 Agent Memory 的核心差异化机制：**Dream Pipeline**。

它做的不是生成幻想，而是：

```text
重放最近 episodes
发现重复失败模式
合并重复记忆
检测冲突规则
标记过期 memory
提炼 project facts
发现 skill candidates
生成 future triggers
降低噪声权重
```

一个可实现的 pipeline：

```text
raw traces
→ episode summaries
→ candidate memories
→ evidence clustering
→ semantic extraction
→ procedural candidate extraction
→ conflict detection
→ aging / forgetting
→ memory index rebuild
```

这对应你之前项目中的 checkpoint writer、stop verifier、memory candidates，但可以继续升级成：

```text
Stop Hook = 写入 episode
Post-task Dream = 生成候选
Nightly Dream = 跨任务巩固
Weekly Dream = 规则清理和 skill promotion
```

评测方向：同一个 benchmark，比较三种 Agent：

```text
无记忆 Agent
只存 episode Agent
带 Dream consolidation Agent
```

观察重复任务、相似任务、跨模块任务中，Dream 是否真的减少重复错误、减少用户提醒、提升定位速度。

---

# 八、遗忘：不是删除，而是降低干扰

人类遗忘有积极作用：减少噪声、避免旧经验干扰、促进泛化。Agent 也必须会遗忘，否则长期运行后会形成 memory sludge。

Agent 的遗忘不应只有 delete，而应该有多种形式：

```text
decay: 权重降低
archive: 默认不召回，但可追溯
invalidate: 标记失效
supersede: 被新规则替代
scope_narrow: 缩小适用范围
quarantine: 暂停使用，等待验证
delete: 彻底移除敏感或有害记忆
```

代码场景尤其需要遗忘，因为 repo、依赖、架构、测试都会变。
触发记忆失效的事件包括：

```text
package lock 大幅变化
目录结构变化
测试框架迁移
主要依赖升级
架构重构
CI 配置变化
owner 变化
用户明确否定
```

可以设计 **Memory Half-Life**：

```yaml
decay_policy:
  default_half_life: 90d
  refresh_on_successful_use: true
  decay_on_failed_use: true
  expire_on_file_deleted: true
```

长期产品方向：Memory Dashboard 中显示：

```text
活跃记忆
即将过期记忆
长期未验证记忆
冲突记忆
高风险但陈旧记忆
```

遗忘不是弱化产品能力，而是让记忆系统长期可用。

---

# 九、召回：从全文搜索到线索触发网络

人类召回更像 cue network，而不是全文搜索。地点、人物、气味、任务目标、情绪状态都会触发相关记忆。

Coding Agent 的线索可以非常工程化：

```text
repo
branch
file path
module name
test name
error stack
dependency name
PR label
user intent
command failure
changed file pattern
CI job name
```

所以 retrieval 不应该只做 embedding similarity。它需要混合排序：

```text
retrieval_score =
  semantic_similarity
+ scope_match
+ path_match
+ error_signature_match
+ task_goal_match
+ risk_match
+ recency
+ confidence
+ importance
- staleness
- conflict_penalty
```

尤其重要的是：召回不应只关注“相似”，还要关注“因果相关”和“风险相关”。

例如当前错误是：

```text
TypeError in auth/session.ts
```

相似文本可能召回一堆 TypeError，但真正有价值的是：

```text
上次修改 auth/session.ts 后，忘记更新 token mock，导致 integration test 失败。
```

这不是语义最相似，但任务相关性很高。

新机制：**Contextual Retrieval Router**。
它先判断当前处于什么场景：

```text
debugging
refactoring
dependency update
test repair
security review
release prep
PR response
```

然后使用不同 retrieval profile。Debugging 召回失败路径，Security 召回风险规则，Refactor 召回架构边界。

评测方向：构造“相似但不相关”和“不相似但相关”的记忆，测试 retrieval 是否能召回真正有用的记忆。

---

# 十、再巩固：Memory 不能只追加，必须可修订

人类每次回忆都会修改记忆。Agent Memory 如果只 append，就会积累大量互相冲突的旧结论。

因此 memory 应该支持：

```text
versioning
supersession
conflict links
revision reason
evidence update
confidence change
scope change
```

例如旧记忆：

```text
项目使用 Jest。
```

后来迁移到 Vitest，不能简单新增一条：

```text
项目使用 Vitest。
```

而应该形成：

```yaml
old_memory:
  statement: 项目使用 Jest
  status: superseded
  valid_until: 2026-06-15
  superseded_by: mem_456

new_memory:
  statement: 项目使用 Vitest
  valid_from: 2026-06-15
  evidence: migration PR / package.json
```

这就是 **Memory Reconsolidation**。

特别是 Team Memory 中，历史轨迹很重要。因为有些旧规则在旧分支、旧版本、旧客户部署中仍然有效。
所以不是“删除旧记忆”，而是把它从 active memory 转为 historical memory。

产品方向：

```text
aeb memory revise
aeb memory supersede
aeb memory history
aeb memory conflict
```

让 Agent 可以解释：

> 我以前记得这个项目用 Jest，但这条记忆已被 2026-06-15 的 Vitest 迁移记录覆盖。

这会显著降低“过期记忆伤害”。

---

# 十一、错误记忆：Agent 也会产生 false memory

人类错误记忆来自暗示、压力、重复叙事、来源混淆。Agent 也会产生类似问题：

```text
模型推理被当成事实
用户猜测被当成验证结果
一次失败日志被错误归因
临时 workaround 被总结成规则
prompt injection 写入长期记忆
过度压缩导致条件丢失
```

所以每条 memory 都应该区分：

```text
observed_fact
tool_verified_fact
user_claim
model_inference
hypothesis
preference
rule
skill
```

一个很关键的 schema 字段是：

```yaml
epistemic_status:
  type: observed | inferred | claimed | verified | disputed
  confidence:
  evidence:
  verification_method:
```

例如：

```text
“用户说这个接口不能改”是 user_claim。
“测试证明改接口会导致 12 个用例失败”是 tool_verified_fact。
“可能因为兼容性要求不能改”是 model_inference。
```

三者不能混在一起。

防污染机制：**False Memory Detector**。
它在 consolidation 时检查：

```text
是否缺少证据？
是否来自单次模型猜测？
是否和工具结果矛盾？
是否被用户后来否定？
是否有 prompt injection 特征？
是否过度泛化？
```

评测方向：给 Agent 注入错误用户陈述、误导性日志、过度总结机会，测试它是否会把错误内容写成长期记忆。

---

# 十二、来源监控：Source Monitoring 是 memory schema 的核心

人类经常混淆“我亲眼看到的”和“别人告诉我的”。Agent 如果不做 source monitoring，会把不同来源的可信度混在一起。

Coding Agent 的来源至少包括：

```text
user instruction
system instruction
tool observation
test result
CI result
git diff
PR review
issue description
documentation
model inference
previous memory
external web source
```

不同来源的权重完全不同。
比如：

```text
用户说测试通过了
```

和：

```text
Agent 实际运行 pnpm test 通过
```

不应等价。

Team Memory 更复杂，因为还涉及：

```text
created_by
verified_by
approved_by
owned_by
last_used_by
scope_owner
```

所以 memory schema 需要字段：

```yaml
source:
  kind:
  actor:
  tool:
  timestamp:
  artifact:
  link:
  trust_level:
verification:
  method:
  result:
  verified_at:
  verified_by:
```

产品方向：Memory UI 中每条记忆旁边显示“证据链”，类似：

```text
来自：CI job #1821
验证：2026-06-28 通过
适用：api-service/auth
可信度：高
```

这比单纯展示一句总结更可控。

---

# 十三、元记忆：Agent 要知道“自己知道什么、不知道什么”

人类有 meta-memory：我知道我大概记得，但不确定；我知道这件事可能过时；我知道需要查证。

Agent 也需要 meta-memory。否则它会把所有记忆都当成等价上下文使用。

每条记忆可以增加：

```yaml
confidence:
uncertainty_reason:
last_verified_at:
staleness_risk:
missing_evidence:
needs_revalidation_before_action:
```

例如：

```text
我记得这个项目之前用 pnpm，但 package manager 这类信息容易随时间变化；在执行安装前先检查 lockfile。
```

这就是 meta-memory 影响行动。

更进一步，Agent 还应该知道自己缺少哪些记忆。
比如在 repo 中首次执行任务时，Agent 应该能说：

```text
我还没有这个 repo 的测试命令记忆、部署流程记忆、架构边界记忆。
```

这会启发一个产品方向：**Memory Coverage Map**。

```text
repo 基础事实覆盖度
测试命令覆盖度
模块 owner 覆盖度
已知坑覆盖度
事故记忆覆盖度
skill 覆盖度
```

评测方向：测试 Agent 是否能在记忆不足时主动查证，而不是胡乱复用旧经验。

---

# 十四、反事实与失败路径：Anti-pattern Memory

资深工程师的价值不只在于知道怎么做，还在于知道哪些方案“看起来合理但会失败”。

Agent 应该显式记录 failed attempts，而不是只保留成功路径。
失败记忆可以表示为：

```yaml
type: anti_pattern
attempt:
  action:
  rationale:
failure:
  symptom:
  evidence:
root_cause:
avoid_when:
  conditions:
try_instead:
  alternative:
status:
  temporary_failure | invalid_strategy | risky_without_verification
```

关键是区分：

```text
暂时失败：当时因为环境问题失败，未来可以再试
原则失败：这个方案违反架构或需求，不应再试
高风险失败：可以做，但必须验证
```

例如：

```text
不要通过跳过 failing test 来修复 CI；上次这样导致 PR review 被拒。
```

这类 anti-pattern memory 应该在未来类似场景强触发。

但失败路径也可能让 Agent 过度保守。
所以 anti-pattern 需要反例和解除条件：

```text
not_apply_when:
  - test is obsolete and approved for deletion
  - maintainer explicitly requested removal
```

评测方向：构造重复错误任务，测试 Agent 是否避免再次走同一个失败路径，同时不把合理方案误杀。

---

# 十五、前瞻记忆：从“记住过去”到“未来条件触发”

人类前瞻记忆是：

> 将来遇到某个条件时，要做某件事。

这是 Agent Memory 极容易被忽略但非常重要的方向。它不像普通 recall，而更像 condition-action memory。

例如：

```text
当修改 migrations/** 时，自动检查 generated schema。
当修改 auth middleware 时，必须跑 auth integration tests。
当用户要求写公众号文章时，保持系列标题风格一致。
当切换到 team memory 时，不要把个人经验直接升级为团队规则。
```

这类记忆接近 hook、CI、lint、policy check，但区别是：

```text
hook 是机械触发
CI 是验证结果
lint 是静态规则
前瞻记忆是带来源、边界、意图和历史证据的未来提醒
```

它可以升级为 hook，但不等于 hook。

架构上可设计：

```text
Prospective Memory Engine
```

字段：

```yaml
trigger:
  event:
  pattern:
  context:
action:
  remind | retrieve | run_command | block | ask_confirmation
evidence:
scope:
cooldown:
false_positive_log:
```

为了避免过度触发，需要：

```text
cooldown
severity level
scope match
confidence threshold
user dismiss feedback
```

产品方向：Coding Agent 外置大脑里的“未来提醒器”，比普通 todo 更强，因为它绑定工程条件。

---

# 十六、情境依赖：每条记忆都必须有作用域

这是你之前特别强调的方向：个人记忆、项目记忆、团队记忆、组织记忆必须区分。否则个人经验会被错误放大成团队规则，团队规则也会污染个人 Agent 风格。

人类经验高度依赖情境。一个人在某个公司形成的工程习惯，不一定适合另一个团队。
Agent memory 也必须表达：

```text
apply_when
not_apply_when
scope
role
repo
module
environment
user
team
```

一个强 schema：

```yaml
scope:
  level: personal | session | project | repo | module | team | org
  repo:
  paths:
  branch_pattern:
  environment:
  user:
  role:
apply_when:
not_apply_when:
promotion_policy:
  can_promote_to_team: false
```

最重要的是 **not_apply_when**。很多系统只写适用条件，不写不适用条件，导致误召回。

例如：

```text
这个项目不要自动格式化整个文件。
not_apply_when:
  - 新文件
  - 用户明确要求重排格式
  - formatter config 已统一迁移
```

评测方向：同一记忆在 A repo 有效，在 B repo 有害，测试 Agent 是否能拒绝跨作用域误用。

---

# 十七、事件边界：Event Segmentation 决定 memory 质量

人类会把连续生活切成 episode，因为边界能减少混淆。Agent 长任务也需要边界，否则不同目标、不同 repo、不同风险等级会混在一起。

触发 memory boundary 的信号包括：

```text
用户目标变化
repo 变化
branch 变化
任务阶段变化
从 debug 切到 refactor
从实现切到验证
从个人任务切到团队规则讨论
权限/风险等级变化
重大失败或成功
用户纠正
```

边界出现时应该做：

```text
checkpoint
working memory cleanup
episode summary
candidate memory extraction
open issue preservation
```

这可以和已有 MVP 的 checkpoint writer 结合。
更进一步，checkpoint 不只是保存进度，而是形成可恢复的 cognitive state：

```yaml
current_goal:
done:
remaining:
assumptions:
verified:
unverified:
risk:
next_action:
```

产品方向：`aeb checkpoint` 不只是任务恢复工具，也是 memory boundary writer。

评测方向：长任务中故意插入需求切换，测试 Agent 是否把两个任务的记忆混淆。

---

# 十八、角色记忆：不同 Agent 角色需要不同 memory profile

人类在工程师、父亲、作者、管理者身份下，会注意不同信息、召回不同经验、采取不同行动。
Agent 也应有 role-specific memory。

例如：

```text
Coder Memory: 命令、架构、测试、实现坑
Reviewer Memory: 代码规范、历史 review 意见、风险点
Security Memory: 漏洞模式、敏感文件、权限边界
Planner Memory: 项目目标、路线图、依赖关系
Researcher Memory: 资料来源、论证链、开放问题
Writer Memory: 风格、标题、叙事结构、读者偏好
```

同一条 memory 对不同角色权重不同。
比如“不要偷懒写摘要，要一篇篇写完”对 Writer Agent 权重极高，对 Coder Agent 权重低。

架构方向：**Role Memory View**。

```text
global memory store
→ role-based retrieval profile
→ permission filter
→ task-specific working memory
```

这也能启发 multi-agent memory routing：不是所有 Agent 共享所有内容，而是共享索引、证据和责任范围。

评测方向：同一任务由 Coder、Reviewer、Security 三个 Agent 分别处理，测试它们是否召回不同但合适的记忆。

---

# 十九、社会记忆：Team Memory 不等于所有人共享全部信息

人类团队不是每个人记住全部知识，而是形成 transactive memory：

> 我知道谁知道什么。

这对 Team Memory Infrastructure 非常关键。团队记忆不一定要复制所有内容，而要维护：

```text
谁负责什么
谁验证过什么
哪个模块有哪些历史坑
哪个规则是谁批准的
哪个经验来自哪次事故
哪个 Agent 擅长哪类任务
```

Team Memory 的 schema 应该包括：

```yaml
owner:
expert:
verified_by:
responsible_scope:
approval_status:
team_visibility:
promotion_history:
```

个人经验升级为团队经验必须经过验证。
可以设计 **Memory Promotion Workflow**：

```text
personal memory
→ project candidate
→ evidence required
→ reviewer approval
→ team memory
→ org pattern
```

否则会发生：

```text
某个 Agent 的局部偏好污染整个团队
某次偶然事故被升级成永久规则
未经验证的用户说法变成组织知识
```

产品方向：Team Memory Dashboard：

```text
模块知识地图
专家/owner 路由
事故记忆库
团队规则库
待审批 memory candidates
过期团队规则
```

这比“共享向量库”更像组织知识基础设施。

---

# 二十、记忆与身份：Agent Self Model 要可控、可回滚

人类通过自传式记忆形成“我是谁”。工程师也会通过长期项目、失败、反馈形成风格：谨慎、快、爱抽象、重测试、重产品。

Agent 可能也会通过长期记忆形成工作风格。
但这有风险，因为风格不应不可控地从历史中自然生长。

可以把 Agent self model 拆成：

```yaml
working_style:
  prefers:
  avoids:
  verification_habit:
  communication_style:
  risk_tolerance:
  learned_from:
  editable_by_user: true
  versioned: true
```

关键问题：

```text
哪些风格来自 system prompt？
哪些来自用户偏好？
哪些来自项目经验？
哪些来自团队规范？
哪些只是 Agent 自己形成的习惯？
```

这些来源必须分开。
例如：

```text
“用户要求不要只写摘要，要完整文章”是用户偏好。
“这个 repo 修改前必须跑测试”是项目规则。
“我倾向于先写计划”可能是 Agent 工作风格。
```

产品方向：**Agent Profile Memory**，但必须可观察、可修改、可回滚。
否则“可成长 Agent”会变成“不可控 Agent”。

---

# 二十一、记忆偏差：Memory Retrieval Debiasing

人类会因为某类记忆容易想起而高估其概率，这就是可得性偏差。Agent 也会。
如果某类 memory 经常被召回，Agent 可能总是套用熟悉解释。

例如：

```text
过去多次测试失败是 mock 问题
→ 以后每次测试失败都先怀疑 mock
→ 忽略真实业务 bug
```

所以 retrieval 不应只召回“最像的记忆”，还应召回：

```text
成功案例
失败案例
反例
被推翻案例
近期变化
冲突记忆
```

可以设计 **Balanced Retrieval Pack**：

```text
top relevant memory
top anti-pattern
top counterexample
top stale/conflict warning
top verified project rule
```

这会让 Agent 不容易被单一记忆牵引。

评测方向：
构造一个场景，最相似历史经验其实是误导，真正正确的是一个低相似但因果相关的反例。测试 Agent 是否能检索到反例并调整判断。

产品方向：Memory Retrieval Debugger，展示：

```text
为什么召回这条？
还有哪些反例？
有没有冲突？
有没有过期风险？
```

---

# 二十二、记忆免疫：Memory Immune System

人类和组织都有知识免疫机制：怀疑、审查、同行评议、事实核查、权限控制。Agent Memory 也需要免疫系统。

有害记忆来源包括：

```text
prompt injection
恶意用户输入
错误日志解释
未经验证的模型推断
过度总结
跨项目污染
旧规则未失效
错误 skill promotion
```

Memory Immune System 可以包含几层：

```text
write-time filter
source classification
trust scoring
quarantine
human approval
conflict detection
staleness detection
rollback
audit log
```

进入 quarantine 的记忆包括：

```text
来自网页/issue/comment 的指令性内容
要求绕过安全/测试的内容
和现有规则冲突的内容
缺少证据但影响范围大的内容
准备升级为 team rule 或 skill 的内容
```

Team Memory 中尤其需要审批。
例如：

```text
个人 Agent 发现“跳过某测试可以让 CI 过”
```

这绝不能自动升级成 team skill，必须进入隔离区。

产品方向：

```text
Memory Quarantine Inbox
Memory Approval Workflow
Memory Rollback Timeline
Memory Threat Report
```

这会让 Agent Memory 更像基础设施，而不是玩具功能。

---

# 二十三、记忆迁移：Skill Promotion 与适用边界

人类能把一个项目经验迁移到另一个项目，但优秀迁移依赖抽象能力和边界判断。
Agent 也需要判断：

```text
这条经验是 project-specific，还是 reusable？
```

例如：

```text
“这个 repo 用 pnpm”不可迁移。
“修改数据库 schema 后应检查生成代码和迁移测试”可以迁移。
```

迁移需要把记忆拆成：

```text
表层事实
底层模式
适用条件
验证方法
迁移历史
失败记录
```

可以设计 **Memory Transfer Record**：

```yaml
source_context:
target_context:
transferred_memory:
adaptation:
verification_result:
transfer_success:
scope_adjustment:
```

如果迁移失败，不是简单删除，而是修正边界：

```text
这条经验适用于 Prisma 项目，但不适用于手写 SQL migration 项目。
```

评测方向：跨 repo benchmark。
训练阶段在 repo A 中形成经验，测试阶段到 repo B 中判断哪些能迁移，哪些不能迁移。指标：

```text
positive transfer rate
negative transfer harm
scope adjustment accuracy
verification-before-transfer rate
```

这是评估 Agent 成长能力的核心指标之一。

---

# 二十四、记忆压缩：Memory Distillation 不能丢掉条件和证据

人类会把大量经历压缩成一句经验。Agent 也需要压缩，否则记忆不可用。
但压缩风险是丢掉关键条件。

推荐分层：

```text
raw trace
→ episode summary
→ verified fact
→ project rule
→ anti-pattern
→ skill
→ team policy
```

每一层都要保留到上一层的链接。

例如：

```yaml
rule:
  statement: 修改 auth middleware 后必须运行 auth integration tests
  evidence:
    - episode_12
    - ci_failure_31
  derived_from:
    - raw_trace_...
  confidence: high
  scope:
```

压缩格式最适合 Agent 使用的不是自然语言长文，而是半结构化对象 + 人类可读摘要：

```yaml
memory:
  summary:
  type:
  scope:
  trigger:
  action:
  evidence:
  confidence:
  invalidation:
```

新机制：**Loss-Aware Memory Compression**。
压缩时显式检查：

```text
有没有丢失适用条件？
有没有丢失证据？
有没有把推断写成事实？
有没有把局部经验写成全局规则？
有没有删掉重要反例？
```

评测方向：给系统一组 raw episodes，让它压缩成 memory，再测试未来任务中是否因压缩丢失边界而误用。

---

# 二十五、记忆评估：不要评估“记住多少”，要评估“行动是否更好”

人类记忆的价值体现在行动改善。Agent Memory 的评估也应该如此。

核心指标：

```text
重复错误减少率
任务恢复成功率
首次尝试成功率
用户重复提醒减少率
定位问题时间降低
验证步骤完整度提升
错误记忆伤害率
过期记忆误用率
跨任务迁移收益
团队规则污染率
```

可以设计几类 benchmark：

## 1. Repeated Mistake Benchmark

第一次任务中 Agent 犯错并被纠正。
第二次相似任务测试是否避免重复犯错。

## 2. Recovery Benchmark

任务中断后重新开始，测试 checkpoint + episode memory 是否恢复目标、上下文、未完成事项。

## 3. Similar-but-Different Benchmark

第二次任务和第一次相似但关键条件不同，测试 Agent 是否避免机械套用旧经验。

## 4. Stale Memory Benchmark

旧记忆曾经正确，但 repo 已迁移，测试 Agent 是否查证并失效旧记忆。

## 5. Team Memory Promotion Benchmark

个人经验是否被错误升级为团队规则；团队规则是否有证据和 owner。

## 6. Prospective Memory Benchmark

修改某类文件时，是否触发正确提醒、命令或检查。

## 7. Bad Skill Benchmark

Agent 是否会把错误 workaround 固化为 skill；能否回滚。

最终评估公式可以围绕：

```text
Memory Utility = 行动收益 - 召回噪声 - 错误记忆伤害 - 维护成本
```

---

# 二十六、新方向继续扩展

下面这些方向很值得单独发展成机制、架构或产品模块。

## 1. 情绪记忆 → 事故记忆 / 风险记忆

重大失败应该产生高权重 incident memory。
但要绑定场景，避免“一朝被蛇咬，十年怕井绳”的过度保守。

## 2. 创伤记忆 → 高风险强提醒

线上事故、数据删除、安全漏洞这类事件应有强触发机制：

```text
warn
block
require human approval
require test evidence
```

## 3. 空间记忆 → 代码库结构记忆

人类靠空间感导航。Agent 可以形成 repo map：

```text
哪些目录是核心路径
哪些模块经常一起变化
哪些文件是高风险区域
哪些测试覆盖哪些代码
```

## 4. 导航记忆 → 大型 repo 路径感知

资深工程师知道“大概去哪里找”。Agent 也应记住：

```text
bug 类型 → 可能目录
功能领域 → owner 模块
测试失败 → 相关 fixture
```

## 5. 语境切换 → 多项目 memory isolation

跨项目时必须清理 working memory，切换 retrieval profile，避免项目 A 的规则污染项目 B。

## 6. 记忆衰退 → memory aging

长期不用、未验证、scope 变化大的记忆自动降权。
但高价值历史事故可保留为 archive，不默认召回。

## 7. 儿童学习 → 从低级规则到高级抽象

Agent 初期应多记录具体 episode，随着任务增加再形成语义规则和 skill。
不要一开始就过度抽象。

## 8. 专家记忆 → Expert Agent Memory

不同专家 Agent 应拥有不同 memory graph：安全专家关心威胁模式，架构专家关心边界和依赖，测试专家关心覆盖与 fixture。

## 9. 教学记忆 → Agent-to-Agent Transfer

一个 Agent 的经验可以生成 teaching card 给另一个 Agent：

```text
场景
坑点
证据
操作步骤
适用边界
练习任务
```

这比共享原始日志更有效。

## 10. 集体记忆 → 组织级 Memory Infrastructure

组织级记忆不应该是大杂烩，而应是：

```text
团队规则
事故库
owner map
skill library
架构决策记录
验证证据
失效机制
```

## 11. 创造性联想 → 跨项目 skill recombination

人类创造力来自远距离联想。Agent 可以在 Dream 阶段发现：

```text
项目 A 的 release checklist
可以改造成项目 B 的 deployment guard
```

但必须经过迁移验证。

## 12. 错误归因 → Memory Attribution

很多错误记忆来自错误归因。
Agent 需要记录：

```text
观察到的失败
候选原因
验证过的原因
被排除的原因
最终根因
```

不要只记最后一句“原因是 X”。

## 13. 习惯打破 → Bad Skill Rollback

坏 skill 需要像依赖版本一样可回滚。
每个 skill 都要有：

```text
usage history
failure history
negative feedback
rollback version
```

## 14. 注意偏差 → Retrieval Debiasing

Memory retrieval 不应总是找最熟悉案例。
应主动召回反例、冲突、过期风险。

## 15. 记忆宫殿 → 代码库空间化 Memory Interface

可以把 repo 可视化成 memory palace：

```text
目录 = 房间
模块 = 区域
事故 = 标记点
owner = 人物
skill = 路径
```

这对大型 repo 和团队知识导航很有产品想象力。

## 16. 自传式记忆 → Agent Career / Project History

Agent 可以维护 project history：

```text
参与过哪些项目
解决过哪些问题
形成过哪些 skill
被纠正过哪些习惯
哪些规则被废弃
```

但必须可控、可导出、可清理。

## 17. 闪光灯记忆 → 重大事故快照

重大事故发生时自动保存完整上下文快照：

```text
diff
logs
commands
timeline
decision points
owner
fix
postmortem
future guard
```

## 18. 语义网络 → Memory Graph

Memory 不应只是列表，而是图：

```text
episode → fact → rule → skill
rule ↔ conflict rule
skill → incidents
module → owner
test → source files
```

图结构能支持因果召回和冲突检测。

## 19. 程序自动化 → Hook / Command Promotion

反复成功的手动流程可以升级为 command；反复触发的提醒可以升级为 hook；高风险规则可以升级为 policy check。

## 20. 社会传承 → Team Skill Library

团队中被验证的 workflow 可以沉淀为 skill library：

```text
debug-auth-failure
prepare-release
review-db-migration
triage-ci-failure
handle-user-correction
```

每个 skill 都应有 owner、版本、适用范围、验证任务和失败案例。

---

# 可以形成的总架构

把上面全部机制收敛成一个 Agent Memory Runtime，可以是：

```text
1. Event Capture Layer
   - hooks
   - tool logs
   - diffs
   - test results
   - user feedback

2. Episode Layer
   - task boundary
   - checkpoint
   - causal trace
   - failed attempts
   - verification result

3. Memory Candidate Layer
   - admission scoring
   - source monitoring
   - importance weighting
   - quarantine

4. Consolidation / Dream Layer
   - clustering
   - compression
   - semantic extraction
   - conflict detection
   - skill candidate mining
   - aging

5. Memory Store
   - episodic memory
   - semantic memory
   - procedural memory
   - prospective memory
   - incident memory
   - anti-pattern memory
   - team memory

6. Retrieval Layer
   - context-aware retrieval
   - role-specific profile
   - scope filtering
   - counterexample retrieval
   - staleness warning

7. Action Layer
   - remind
   - run command
   - block risky action
   - ask verification
   - invoke skill
   - update checkpoint

8. Governance Layer
   - source trust
   - approval
   - versioning
   - rollback
   - audit
   - team promotion
```

---

# 最值得优先落地的 10 个机制

按你的项目方向，优先级最高的不是“大而全的记忆库”，而是下面这些能直接提升 Coding Agent 的模块：

```text
1. Memory Admission Gate
2. Episode / Checkpoint Writer
3. Source Monitoring Schema
4. Scope-aware Memory Store
5. Prospective Memory Trigger
6. Anti-pattern / Failed Attempt Memory
7. Dream Consolidation Pipeline
8. Memory Versioning / Supersession
9. Skill Promotion / Rollback
10. Team Memory Promotion Workflow
```

这 10 个机制可以分别支撑两个系列：

```text
给 Coding Agent 装一个外置大脑
→ 个人/项目级 Agent Memory Runtime

给 Agent 团队装一个共享大脑
→ Team Memory Infrastructure / Memory Governance
```

这份清单继续向下写，可以拆成一组更强的文章/论文/专利方向：
**“Agent Memory 的核心不是存储，而是让经验在正确场景下改变未来行为。”**
