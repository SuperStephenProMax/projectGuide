# 《Java_Spring_Boot 项目开发协作与质量保障操作手册》

> 适用默认示例：**MyService（Java + Maven + Spring Boot）**
> 本手册输出的是“可执行、可落地、可审计”的标准流程与仓库文件。若你的项目与默认假设不一致，请在文末“待确认/可配置项表”中更新参数。

---

## A. 概览

### A.1 目标

1. **可追踪**：任何代码变更都能追溯到 Issue/PR/Commit，并能定位负责人、讨论与决策记录。
2. **可回溯**：任意版本可通过 Tag/Release/PR 记录还原；出问题可快速回滚。
3. **质量达标**：

   - A. **所有单元测试（UT）必须成功**
   - B. **增量代码覆盖率（patch/diff coverage）≥ 70%**
   - C. 提供**可审计、可控**的覆盖率豁免机制（有审批、有记录、有“到期/限定条件”）

4. **自动化保障**：GitHub Actions 自动跑 UT、生成覆盖率、计算增量覆盖率并做门禁；失败必须可定位、可复跑。
5. **安全合规**：AI 辅助 Review 默认不外发代码；如需外部 LLM，必须显式批准，并做好 secrets 与 fork 安全策略。

### A.2 适用范围

- 代码仓库：**默认单仓（single repo）**
- 默认主分支：`main`
- 团队规模：`1-2` 人（Maintainer/Developer/Reviewer 角色可由同一人兼任，但流程仍需留痕）
- 期望合并策略：**Squash merge + Linear history**

### A.3 角色与职责

**Maintainers（维护者）**

- 负责 `main` 分支保护、CI 门禁规则、Label 体系、版本发布与回滚决策
- 负责审批覆盖率豁免（只能豁免覆盖率，不豁免 UT）
- 负责 CODEOWNERS 与权限控制（最小权限）

**Developers（开发者）**

- 按流程创建 Issue → 分支 → 开发 → 提 PR
- 本地自测（UT + 覆盖率）后提交
- PR 必须关联 Issue，补齐测试说明、风险与回滚方案

**Reviewers（评审者）**

- 按 Reviewer Checklist 做功能/边界/测试/安全审查
- 确认 CI 全绿、增量覆盖率达标；如豁免，确认理由与到期条件

### A.4 “待确认/可配置项”参数表（先按默认实现落地）

| 参数                      | 默认值（本手册）                                  | 说明                              |
| ------------------------- | ------------------------------------------------- | --------------------------------- |
| 项目名称                  | `Java_Spring_Boot`（示例 MyService）              | **可配置项**                      |
| 仓库类型                  | 单仓                                              | 可配置项                          |
| Java 版本                 | 25                                                | Spring Boot 4 要求；可配置项      |
| 构建工具                  | Maven                                             | 已确认                            |
| 单元测试命令              | `mvn -B -ntp test`                                | 可配置项（如有 IT/UT 分离需调整） |
| 覆盖率工具                | JaCoCo（XML 报告）                                | 默认；可替换                      |
| 覆盖率生成命令（CI 默认） | `mvn ... jacoco:prepare-agent test jacoco:report` | 见 ci.yml，**可配置**             |
| 覆盖率报告文件            | `target/site/jacoco/jacoco.xml`                   | 默认；多模块需调整                |
| 增量覆盖率阈值            | 70（%）                                           | 可配置项                          |
| 允许外部 LLM              | 默认：需显式批准                                  | 通过 PR Label 控制                |
| 合并策略                  | Squash merge + Linear history                     | 默认                              |

---

## B. Git 与 GitHub 规范

### B.1 分支模型（推荐：Trunk-Based + 短生命周期分支）

**规则：**

- `main` 永远保持可构建、可测试、可发布（至少“可回滚”）。
- 开发在短生命周期分支进行，所有变更通过 PR 合并到 `main`。
- 禁止直接 push 到 `main`（通过 Branch Protection 强制）。

**为什么推荐：**

- 团队 1-2 人时最简单可靠：减少长期分支漂移与冲突
- Squash + Linear history 更适合审计：`main` 上每次合并对应一个 PR（一个“变更单元”）

### B.2 分支命名规范

**命名格式：**

- `feature/<issueId>-<short-desc>`
- `bugfix/<issueId>-<short-desc>`
- `hotfix/<issueId>-<short-desc>`
- `chore/<issueId>-<short-desc>`
- `docs/<issueId>-<short-desc>`
- `ci/<issueId>-<short-desc>`
- `refactor/<issueId>-<short-desc>`
- `test/<issueId>-<short-desc>`

**示例：**

- `feature/123-add-rate-limit`
- `bugfix/456-fix-null-pointer`
- `hotfix/789-fix-login-outage`
- `docs/88-update-readme`
- `chore/90-bump-deps`

**硬规则：**

- 分支必须能映射到 Issue：`<issueId>` 必填（若无 Issue，先补 Issue）
- `short-desc` 使用 `kebab-case`，不超过 50 字符

### B.3 Commit 提交规范（Conventional Commits）

**提交格式：**

```
<type>(<scope>): <subject>

<body 可选：说明动机/实现要点/兼容性>
<footer 可选：Refs/Fixes #issue>
```

**type 建议枚举：**

- `feat` 新功能
- `fix` 修复 bug
- `refactor` 重构（不改变外部行为）
- `perf` 性能优化
- `test` 测试相关
- `docs` 文档
- `chore` 杂项（依赖、脚本、非产品代码）
- `ci` CI 配置
- `build` 构建系统相关
- `revert` 回滚

**粒度要求：**

- 一个 commit 解决一件事（避免“又改功能又改格式又改依赖”）
- 尽量让每个 commit **可编译、UT 可跑**（最少保证 PR 合并前可跑）

**Issue 引用要求：**

- 推荐在 footer 使用：`Refs #123`（追踪）或 `Fixes #123`（合并后自动关闭 Issue）
- 小团队也必须留痕：没有 Issue 的提交 = 不可审计

**10 个 commit 示例：**

1. `feat(api): add health endpoint (refs #12)`
2. `fix(auth): prevent NPE when userId missing (fixes #34)`
3. `refactor(service): extract user validation helper (refs #56)`
4. `perf(db): add index for order_lookup (refs #78)`
5. `test(auth): add UT for token expiry edge cases (refs #34)`
6. `docs(readme): document local run and ports (refs #90)`
7. `chore(deps): bump spring-boot to 3.3.x (refs #91)`
8. `ci(actions): add diff coverage gate (refs #92)`
9. `build(maven): enable jacoco xml report (refs #93)`
10. `revert: revert "feat(api): add health endpoint" (refs #12)`

> 建议：PR Title 也遵循 Conventional Commits（Squash merge 时最终落到 `main` 的 commit message 更干净）。

### B.4 PR 规范（标题、描述、关联 Issue、变更范围、证据）

**PR 标题：**

- 必须符合 Conventional Commits（例如 `fix(auth): handle null userId`）
- 必须与变更类型一致（feature/bugfix/hotfix 等）

**PR 描述必须包含：**

- 关联 Issue：`Fixes #123` / `Refs #123`
- 变更说明：What/Why/How（最少 What + Why）
- 测试说明：本地跑过什么、CI 验证什么
- 风险评估：是否影响接口/数据/兼容性
- 回滚策略：如何 revert、是否需要数据回滚
- 如涉及覆盖率豁免：必须填写豁免原因/到期/审批人（见模板）

### B.5 Issue 规范（可追踪可回溯）

**何时必须开 Issue：**

- 任何用户可感知的 bug、功能变更
- 任何影响架构/依赖/CI 的改动
- 任何需要讨论与决策的事项（哪怕最终“不做”也要留痕）

**Issue 字段要求：**

- Bug：复现步骤、期望/实际、环境、日志/截图
- Feature：问题描述、目标、验收标准（DoD）、范围/不做什么、风险与依赖

**推荐 Label 体系（可审计）**

- 类型：`type: bug` / `type: feature` / `type: chore` / `type: docs` / `type: security`
- 优先级：`priority: P0` / `P1` / `P2` / `P3`
- 状态：`status: triage` / `status: ready` / `status: in-progress` / `status: blocked` / `status: done`
- 范围/模块：`area: api` / `area: auth` / `area: db` / `area: ci`（按项目实际增补）
- 风险：`risk: low` / `risk: medium` / `risk: high`

**Milestones（里程碑）**

- 用于版本/迭代（如 `v1.2.0`）
- 规则：PR 必须关联到对应 Milestone 的 Issue（可在 PR 中引用）

**Projects（看板）**

- 小团队可用 GitHub Projects 做轻量追踪：`Backlog → Ready → In Progress → In Review → Done`
- 规则：Issue 创建后进入 Backlog；PR 打开后 Issue 进入 In Review；合并后 Done

### B.6 Code Review 规范（通过条件、最少审批人数、CODEOWNERS）

**通过条件（硬门槛）：**

- CI Required Checks 全绿（UT 必须成功、增量覆盖率达标或已审批豁免）
- PR Checklist 完整（模板字段不允许大量缺失）
- 至少 1 个审批（默认团队 1-2 人：`≥1`）

**1-2 人团队的现实规则（仍需留痕）：**

- 若只有 1 名 Maintainer：允许自审合并，但必须在 PR 评论中写明：
  `Self-reviewed ✅ (reason: solo maintainer)` 并确保 CI 门禁全绿
- 若有 2 人：默认互审；高风险变更（`risk: high`）必须双人确认

**CODEOWNERS 策略：**

- 默认：`*` 由 Maintainers 拥有（强制自动请求 review）
- 可按模块细分（`/src/main/java/...`）

---

## C. 标准开发流程（一步步）

> 目标：新成员照着做，可以完成一次 **bugfix/feature/hotfix** 的全流程，并能满足审计与门禁。

### C.1 Bugfix 流程（从发现到发布/回滚）

#### Step 0：准备本地环境

```bash
java -version
mvn -version
git --version
```

#### Step 1：创建/完善 Issue（必须）

- 使用 Bug 模板创建：填写复现步骤、日志、环境
- 打 Label：`type: bug`、优先级、模块 area
- 写清楚：**验收标准**（什么情况下算修好）

#### Step 2：从 main 拉分支

```bash
git checkout main
git pull --ff-only origin main
git checkout -b bugfix/<issueId>-<short-desc>
# 示例
git checkout -b bugfix/456-fix-null-pointer
```

#### Step 3：开发与本地自测（必须）

**跑 UT：**

```bash
mvn -B -ntp test
```

**生成 JaCoCo 覆盖率（默认 CI 同款命令，推荐本地也跑一次）：**

```bash
mvn -B -ntp clean \
  org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
  org.jacoco:jacoco-maven-plugin:0.8.12:report
# 生成：target/site/jacoco/jacoco.xml
```

（可选）本地算增量覆盖率（与 CI 同逻辑）：

```bash
python -m pip install --upgrade pip
pip install diff-cover
git fetch origin main:refs/remotes/origin/main
diff-cover target/site/jacoco/jacoco.xml \
  --compare-branch origin/main \
  --fail-under=70 \
  --format markdown:diff-cover.md,html:diff-cover.html
```

> diff-cover 支持 JaCoCo XML（以及 Cobertura/Clover/LCov）作为输入格式。

#### Step 4：提交（Conventional Commits）

```bash
git status
git add -A
git commit -m "fix(auth): prevent NPE on null userId (fixes #456)"
```

#### Step 5：推送远端

```bash
git push -u origin bugfix/456-fix-null-pointer
```

#### Step 6：创建 PR（必须使用模板）

- PR 标题：`fix(auth): prevent NPE on null userId`
- PR 描述：包含 `Fixes #456`、测试说明、风险与回滚
- 等待 CI（UT + 增量覆盖率门禁）

#### Step 7：Review & 修正

- Reviewer 按 checklist 审查
- 根据评论补充 UT、改实现、更新文档
- 所有讨论在 PR 内完成（不要私聊“口头通过”）

#### Step 8：合并（Squash merge）

- 合并前确认：

  - Required Checks 全绿
  - 讨论已解决（resolved）
  - PR 描述完整

- 合并方式：Squash merge（线性历史）

#### Step 9：发布与回滚（如需要）

- 发布：打 Tag / GitHub Release（见 Release Checklist）
- 回滚：对 `main` 上的 squash commit 执行 `git revert`

回滚示例：

```bash
git checkout main
git pull --ff-only origin main
git log --oneline  # 找到需要回滚的合并提交 SHA
git revert <SHA>
git push origin main
```

---

### C.2 Feature 流程（包含设计讨论与 DoD）

#### Step 1：创建 Feature Issue（必须）

- 用 Feature 模板
- **必须写验收标准（DoD）**：可测、可验证、边界条件明确
  示例 DoD：

  - 新增接口返回值/错误码已文档化
  - UT 覆盖新增逻辑（新增代码增量覆盖率 ≥ 70%）
  - 失败场景（异常/边界）有 UT
  - 可回滚（不破坏兼容或有迁移方案）

#### Step 2：分支

```bash
git checkout main
git pull --ff-only origin main
git checkout -b feature/<issueId>-<short-desc>
# 示例
git checkout -b feature/123-add-rate-limit
```

#### Step 3：实现（建议小步提交）

- 每个 commit 只做一件事
- 大功能拆成多个 PR 更易审计（推荐）

#### Step 4：测试与覆盖率

同 Bugfix 流程（UT + JaCoCo + diff coverage）

#### Step 5：PR & Review & Merge

同 Bugfix 流程

---

### C.3 Hotfix 流程（紧急修复 + 回滚策略）

**适用场景：**

- 线上故障（P0/P1）、安全漏洞紧急修复

**规则（仍需审计与门禁）：**

- UT **不可豁免**（必须全绿）
- 增量覆盖率原则上仍需 ≥ 70%；如确实来不及，走“覆盖率豁免机制”，并设置到期（短期限）

**步骤：**

```bash
git checkout main
git pull --ff-only origin main
git checkout -b hotfix/<issueId>-<short-desc>
```

- 修复 → 补最小必要 UT → 提 PR → 快速 Review（至少 1 人确认）→ Squash merge
- 发布（tag/release）
- 若 hotfix 引入回归：优先 revert，避免在 `main` 上“边跑边改”

---

### C.4 流程图（Issue → PR → Merge 状态机）

```mermaid
flowchart TD
  A[Issue: status: triage] --> B[Issue: status: ready]
  B --> C[Create Branch<br/>feature/bugfix/...]
  C --> D[Commit(s)<br/>Conventional Commits]
  D --> E[Open PR<br/>Template filled]
  E --> F[CI: UT + Diff Coverage Gate]
  F -->|pass| G[Code Review<br/>>=1 approval]
  F -->|fail| E
  G -->|approved| H[Squash Merge to main]
  H --> I[Release/Tag<br/>optional]
  H --> J[Rollback if needed<br/>git revert]
```

---

## D. CI/CD 与质量门禁（GitHub Actions）

### D.1 CI 触发策略

- `pull_request`：对每个 PR 运行

  - UT（必须成功）
  - 生成 JaCoCo 报告
  - 计算 diff/patch 覆盖率并门禁
  - 产出 artifacts（覆盖率与报告，便于审计）

- `push`（main）：对合并后的主分支运行

  - UT + 生成覆盖率（便于基线留档）

- `workflow_dispatch`：手动触发（用于排查）

### D.2 Required Checks（强制门禁）如何配置

在 GitHub 仓库：

- `Settings` → `Branches` → `Branch protection rules` → 为 `main` 添加规则
- 勾选：

  - Require a pull request before merging
  - Require approvals（建议 ≥ 1）
  - Require conversation resolution
  - Require status checks to pass before merging（选择 `CI` workflow 对应 checks）
  - Require linear history
  - （可选）Restrict who can push to matching branches

> Required Checks 选择：以你在 `ci.yml` 中的 job name 为准（例如 `CI / ci`）。

### D.3 硬指标说明

**硬指标 A：所有 UT 必须成功**

- CI 失败：禁止合并
- 不提供豁免（除非你明确建立更严格的“紧急权限”制度；本手册默认不允许）

**硬指标 B：增量代码覆盖率 ≥ 70%**

- 只针对 PR 的变更行（diff/patch coverage）
- 目标：鼓励“改到的行必须被测试覆盖”，防止靠历史存量覆盖率“粉饰太平”

---

### D.4 覆盖率门禁实现：两套方案（默认推荐方案 2）

#### 方案 1：使用 Codecov / Coveralls 等服务做 patch coverage 门禁

**做法概述：**

1. CI 生成覆盖率报告（JaCoCo XML）
2. 上传到 Codecov
3. 在 Codecov 配置 `patch` 覆盖率阈值 70%
4. 在 GitHub Branch Protection 中把 Codecov 的 `patch` 状态检查设为 required

**GitHub Actions 上传示例（Codecov Action v5）：**
（本手册在 `ci.yml` 中给了可选步骤）

**codecov.yml 示例（建议放仓库根目录，非强制文件清单之一）：**

```yml
coverage:
  status:
    patch:
      default:
        target: 0.70
        threshold: 0.00
    project:
      default:
        target: auto
comment:
  layout: "reach,diff,flags,files"
  behavior: default
```

**优点：**

- 配置简单、可视化强
- patch/project 覆盖率、趋势、PR 注释都很成熟

**缺点：**

- 外部服务依赖（合规/隐私可能不允许）
- 私有仓库通常需要 token；fork 场景的 token 管控要更谨慎

---

#### 方案 2（默认推荐）：纯 GitHub Actions 本地计算 diff coverage（diff-cover + JaCoCo）

**核心工具：**

- JaCoCo：生成 `jacoco.xml`
- diff-cover：读取覆盖率 XML + git diff，计算变更行覆盖率，并支持 `--fail-under` 门禁

**优点（推荐原因）：**

- **不依赖外部服务**：更符合“可审计、可控”的默认目标
- 报告与日志留在 GitHub Actions，可归档 artifacts
- 通过 Label 可实现可控豁免（仍产出报告）

**缺点：**

- 多模块/路径映射可能需要调整 `--src-roots` 或报告路径
- 相比 Codecov 可视化弱一些（但足够可审计）

---

### D.5 豁免机制（默认严格、可审计、可控）

> 原则：**只能豁免“覆盖率门禁”**，不能绕过 UT 全绿。
> 任何豁免必须可追溯：谁批准、为什么、到期时间是什么。

#### 机制设计

- 申请者（Developer）：

  - 给 PR 打标签：`coverage-override-requested`
  - 在 PR 描述中填写：原因、到期时间、风险与补测计划

- 审批者（Maintainer / CODEOWNER）：

  - 将标签改为：`ci-override-coverage`（生效）
  - 必须确认：原因合理、到期不超过 14 天、风险可控

- CI 行为：

  - UT 仍必须成功
  - diff coverage 仍会计算并产出报告
  - 若低于阈值：**不阻断合并**，但会在 PR 留下注释（审计留痕）
  - 超过“到期时间”或缺少字段：CI 直接失败（强制补齐/续期/移除豁免）

> 该机制通过 PR 模板字段 + Actions 校验实现“有记录 + 有到期”。（见下方 `ci.yml`）

#### 何时允许豁免（示例）

- 紧急 hotfix，无法在短时间补齐 UT 覆盖全部新增行，但至少保证核心路径有 UT
- 引入第三方 SDK/不可测的薄封装（但需要在后续迭代补齐更高层测试）
- 纯防御性日志/监控埋点（低风险），并承诺后续补测

#### 谁能豁免

- 仅 Maintainers / CODEOWNERS（具备写权限、可打标签的人）
- 不接受“口头批准”：必须在 PR 中留痕（模板字段 + 标签）

---

### D.6 失败排查指南（常见错误）

1. **UT 失败**

- 本地复现：

  ```bash
  mvn -B -ntp test
  ```

- 查看 `target/surefire-reports/`（CI 会上传 artifact）

2. **找不到 JaCoCo 报告 `jacoco.xml`**

- 确认 CI/本地执行了带 `jacoco:prepare-agent` 的命令
- 默认路径：`target/site/jacoco/jacoco.xml`

3. **diff-cover 报 “unknown revision origin/main...HEAD”**

- 需要 fetch base 分支（CI 已包含）
- diff-cover 官方也建议在 CI 环境 fetch 远端分支后再运行

4. **diff-cover 报 “No lines with coverage information in this diff.”**

- 通常是 coverage XML 中的文件路径与 git diff 路径对不上
- 解决：

  - 确保在仓库根目录运行
  - 调整 `--src-roots`（CI 已给默认 `src/main/java` 等）
  - 多模块项目需调整报告路径/合并报告策略

5. **如何重跑**

- PR 页面 → Checks → Re-run jobs（或重新 push 一次空提交）

---

## E. AI Code Review（Copilot / Claude / Codex 等）

### E.1 启用方式与原则

**原则 1：AI 给建议，不替代人类审批**

- Required：人类 Reviewer 审核通过（≥1）
- AI 评论仅作为建议，不作为“通过门槛”

**原则 2：最小权限**

- AI 工作流仅需：

  - `pull-requests: write`（发评论）
  - `issues: write`（如使用 issue_comment 交互可选）
  - `contents: read`（读取 diff/内容）

- 禁止给 AI 工作流 `contents: write`（避免自动改代码/落盘）

**原则 3：外部 LLM 必须显式批准**

- 默认：代码 **不允许**发送到外部 LLM
- 只有当 PR 同时具备：

  - Label：`ai-review`
  - Label：`external-llm-approved`
    才会触发外部 LLM Review（并且仅限同仓 PR，fork 不运行）

### E.2 推荐集成方式：PR-Agent（可接 OpenAI / Claude）

本手册提供 `ai_review.yml`，基于 PR-Agent GitHub Action（qodo-ai/pr-agent）。该工具支持在 GitHub Actions 内自动对 PR 做 review 并评论。

**你需要配置的 Secrets（二选一）：**

- OpenAI / Codex：`OPENAI_KEY`（PR-Agent 使用该命名）
- Claude：`ANTHROPIC_KEY`（并在 env 中映射为 `ANTHROPIC.KEY`）

**fork 安全策略：**

- 工作流显式限制：`github.event.pull_request.head.repo.full_name == github.repository`
- 结论：来自不可信 fork 的 PR **不会**触发 AI review（避免 secrets 泄露）

### E.3 Review 规则（如何用 AI 建设性落地）

- AI 输出的建议分三类处理：

  1. **必改**：明显 bug、并发/空指针、安全问题（人类确认后改）
  2. **可改**：可读性/风格/抽象（结合团队约定取舍）
  3. **不改**：误报或不符合上下文（在 PR 中回复理由，留痕）

- 对 AI 建议的采纳，最好落到：

  - 新增 UT
  - 补充边界条件
  - 改善日志/错误码/可观测性

- 避免把 secrets、token、内部 URL 粘给 AI（模板中明确提醒）

---

## F. 仓库配置文件（逐个给出：路径 + 完整内容）

> 以下文件内容可直接复制到仓库对应路径。
> 注意：含 `@YOUR_GITHUB_HANDLE`、`Java_Spring_Boot` 等为 **待确认/可配置项**。

---

### F.1 `CONTRIBUTING.md`

````markdown
# Contributing Guide

> Repo: Java_Spring_Boot (default example: MyService)  
> This guide is **executable**: follow steps to complete a bugfix/feature end-to-end.

## 0. Preconditions

- JDK 25 (configurable)
- Maven 3.8+ (recommended 3.9+)
- Git
- (Optional for local diff coverage) Python 3.10+

Verify:

```bash
java -version
mvn -version
git --version
```
````

## 1. Local Build & Test

### 1.1 Run unit tests (UT)

```bash
mvn -B -ntp test
```

### 1.2 Generate coverage report (JaCoCo XML)

Default CI-compatible command (works even if pom.xml doesn't pre-configure JaCoCo):

```bash
mvn -B -ntp clean \
  org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
  org.jacoco:jacoco-maven-plugin:0.8.12:report
```

Expected output:

- `target/site/jacoco/jacoco.xml`
- `target/site/jacoco/index.html`

## 2. Branching Rules

- Base branch: `main` (protected)
- Create short-lived branches:

  - `feature/<issueId>-<short-desc>`
  - `bugfix/<issueId>-<short-desc>`
  - `hotfix/<issueId>-<short-desc>`
  - `docs/<issueId>-<short-desc>`
  - `chore/<issueId>-<short-desc>`

Example:

```bash
git checkout main
git pull --ff-only origin main
git checkout -b bugfix/456-fix-null-pointer
```

## 3. Commit Convention (Conventional Commits)

Format:

```
<type>(<scope>): <subject>

[optional body]
[optional footer: Refs/Fixes #issue]
```

Examples:

- `fix(auth): prevent NPE on null userId (fixes #456)`
- `feat(api): add rate limit interceptor (refs #123)`
- `ci(actions): add diff coverage gate (refs #92)`

Rules:

- One commit = one intent
- Must reference an Issue (Refs/Fixes)

## 4. Pull Request (PR) Process

1. Ensure local tests pass:

```bash
mvn -B -ntp test
```

2. Push your branch:

```bash
git push -u origin bugfix/456-fix-null-pointer
```

3. Open a PR:

- Use PR template (auto-loaded)
- Must link Issue: `Fixes #456` or `Refs #456`
- Fill testing notes, risk, rollback plan

4. CI gates (required):

- UT must pass
- Diff/Patch coverage >= 70% (unless approved override)

## 5. Local Diff Coverage (Optional but recommended)

```bash
python -m pip install --upgrade pip
pip install diff-cover
git fetch origin main:refs/remotes/origin/main

diff-cover target/site/jacoco/jacoco.xml \
  --compare-branch origin/main \
  --fail-under=70 \
  --format markdown:diff-cover.md,html:diff-cover.html
```

## 6. Coverage Override Policy (Audit-required)

Default policy: external LLM is NOT allowed unless explicitly approved.

Coverage override (ONLY for patch coverage, NOT for UT):

- Developer requests: add label `coverage-override-requested`
- Maintainer approves: replace with label `ci-override-coverage`
- PR description MUST include:

  - Override Coverage Reason
  - Override Coverage Expiry (YYYY-MM-DD, <= 14 days)
  - Override Approver (@github-handle)

CI will still generate reports and leave audit comments.

## 7. Definition of Done (DoD) for any PR

- [ ] Linked to an Issue (Fixes/Refs #id)
- [ ] UT passed locally (or explain why not possible)
- [ ] CI passed (required)
- [ ] Diff coverage >= 70% OR approved override with reason+expiry+approver
- [ ] No secrets in code/logs
- [ ] Updated docs if behavior changed
- [ ] Rollback plan described for risky changes

Thanks for contributing!

````

---

### F.2 `.github/PULL_REQUEST_TEMPLATE.md`

```markdown
## Summary
<!-- What & Why. Keep it short but specific. -->

## Related Issue
<!-- Use Fixes to auto-close, or Refs to link. -->
- Fixes #
- Refs #

## Type of Change
- [ ] Bugfix (`bugfix/*`)
- [ ] Feature (`feature/*`)
- [ ] Hotfix (`hotfix/*`)
- [ ] Refactor
- [ ] Docs
- [ ] CI/Build/Chore

## What Changed
-

## How to Test
### Local
```bash
mvn -B -ntp test
````

### Coverage (optional local verification)

```bash
mvn -B -ntp clean \
  org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
  org.jacoco:jacoco-maven-plugin:0.8.12:report
```

## Evidence

- [ ] Logs attached
- [ ] Screenshot attached (if UI)
- [ ] Repro steps included (bugfix)

## Risk & Rollback Plan

**Risk level**: low / medium / high

**Rollback plan** (be specific, include commands if possible):

- Example: revert squash commit on `main`

## Patch Coverage Gate (Required)

- Target: **>= 70%** on changed lines

## Coverage Override (ONLY if you request/approve override)

> ⚠️ Override can ONLY bypass patch coverage gate. UT must still pass.

**Override Coverage Reason:**

<!-- Required if label `ci-override-coverage` is applied -->

**Override Coverage Expiry (YYYY-MM-DD, <= 14 days):**

**Override Approver (@github-handle):**

## AI Review

- [ ] Request AI review (apply label `ai-review`)
- [ ] External LLM approved (apply label `external-llm-approved`) — required by policy

## Checklist (Developer)

- [ ] Issue linked (Fixes/Refs)
- [ ] UT passed locally
- [ ] Added/updated unit tests for new/changed behavior
- [ ] No secrets / tokens / credentials in code or logs
- [ ] Backward compatibility considered
- [ ] Docs updated if needed

````

---

### F.3 `.github/ISSUE_TEMPLATE/bug_report.yml`

```yml
name: "🐞 Bug Report"
description: "报告一个可复现的问题（必须可追踪可回溯）"
title: "[Bug]: "
labels:
  - "type: bug"
  - "status: triage"
body:
  - type: checkboxes
    id: precheck
    attributes:
      label: "提交前检查"
      options:
        - label: "我已搜索现有 Issues，确认未重复"
          required: true
        - label: "我可以提供复现步骤或最小复现代码"
          required: true

  - type: textarea
    id: what_happened
    attributes:
      label: "现象描述"
      description: "发生了什么？"
      placeholder: "简要描述问题现象与影响范围"
    validations:
      required: true

  - type: textarea
    id: expected
    attributes:
      label: "期望结果"
      placeholder: "期望系统表现为何？"
    validations:
      required: true

  - type: textarea
    id: repro_steps
    attributes:
      label: "复现步骤"
      description: "请给出可复现的步骤（越具体越好）"
      placeholder: |
        1. ...
        2. ...
        3. ...
    validations:
      required: true

  - type: textarea
    id: logs
    attributes:
      label: "日志/截图/堆栈"
      description: "请脱敏后贴出关键日志、异常堆栈或截图"
      render: shell
    validations:
      required: false

  - type: input
    id: version
    attributes:
      label: "版本/分支/提交"
      description: "例如：main@<sha> 或 v1.2.0"
      placeholder: "main@abcdef1"
    validations:
      required: true

  - type: dropdown
    id: severity
    attributes:
      label: "严重程度"
      options:
        - "P0 - 线上不可用/重大事故"
        - "P1 - 核心功能受影响"
        - "P2 - 次要功能受影响"
        - "P3 - 体验/边缘问题"
    validations:
      required: true

  - type: textarea
    id: env
    attributes:
      label: "环境信息"
      placeholder: |
        - OS:
        - JDK:
        - Maven:
        - Spring Boot:
        - DB/中间件版本（如适用）:
    validations:
      required: false

  - type: textarea
    id: additional
    attributes:
      label: "补充信息"
      description: "可能的原因、临时绕过方案、相关链接"
    validations:
      required: false
````

---

### F.4 `.github/ISSUE_TEMPLATE/feature_request.yml`

```yml
name: "✨ Feature Request"
description: "提出需求/改进（必须有验收标准 DoD）"
title: "[Feature]: "
labels:
  - "type: feature"
  - "status: triage"
body:
  - type: checkboxes
    id: precheck
    attributes:
      label: "提交前检查"
      options:
        - label: "我已搜索现有 Issues，确认未重复"
          required: true

  - type: textarea
    id: problem
    attributes:
      label: "问题/动机"
      description: "为什么需要这个功能？解决什么痛点？"
      placeholder: "As a ..., I want ..., so that ..."
    validations:
      required: true

  - type: textarea
    id: proposal
    attributes:
      label: "方案建议（可选）"
      description: "你期望怎么做？接口/行为建议是什么？"
    validations:
      required: false

  - type: textarea
    id: acceptance
    attributes:
      label: "验收标准（DoD）"
      description: "必须可测试、可验证"
      placeholder: |
        - [ ] 新增/修改的行为有 UT 覆盖
        - [ ] PR 增量覆盖率 >= 70%
        - [ ] 文档/README 已更新（如适用）
        - [ ] 兼容性/迁移方案明确
    validations:
      required: true

  - type: textarea
    id: out_of_scope
    attributes:
      label: "不做什么（Out of scope）"
      placeholder: "明确本需求不包含的范围，避免范围蔓延"
    validations:
      required: false

  - type: textarea
    id: risks
    attributes:
      label: "风险与依赖"
      placeholder: |
        - 风险点：
        - 依赖（外部系统/库/版本）：
        - 性能/安全影响：
    validations:
      required: false
```

---

### F.5 `.github/CODEOWNERS`（或仓库根目录 `CODEOWNERS`）

```text
# CODEOWNERS
# Replace @YOUR_GITHUB_HANDLE with real maintainers.
# Docs: https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners

* @YOUR_GITHUB_HANDLE

# Examples (optional):
# /src/main/java/ @YOUR_GITHUB_HANDLE
# /.github/ @YOUR_GITHUB_HANDLE
```

---

### F.6 `.github/workflows/ci.yml`（单测 + 覆盖率门禁 + 豁免机制）

> 默认实现：**方案 2（本地 diff coverage）**
> 可选：上传 Codecov（方案 1）
> diff-cover 支持 JaCoCo XML 输入与 `--fail-under` 门禁。
> Codecov Action 推荐使用 `codecov/codecov-action@v5`。

```yml
name: CI

on:
  pull_request:
    types: [opened, synchronize, reopened, ready_for_review, labeled, edited]
  push:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: ci-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

env:
  JAVA_VERSION: "25"                 # configurable
  JACOCO_VERSION: "0.8.12"           # configurable
  PATCH_COVERAGE_MIN: "70"           # configurable (%)
  COVERAGE_XML: "target/site/jacoco/jacoco.xml"
  COVERAGE_REPORT_DIR: "target/site/jacoco"
  DIFF_COVER_MD: "diff-cover.md"
  DIFF_COVER_HTML: "diff-cover.html"

jobs:
  ci:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Java
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: ${{ env.JAVA_VERSION }}
          cache: maven

      - name: Unit tests + JaCoCo report (XML)
        run: |
          set -euxo pipefail
          mvn -B -ntp clean \
            org.jacoco:jacoco-maven-plugin:${{ env.JACOCO_VERSION }}:prepare-agent test \
            org.jacoco:jacoco-maven-plugin:${{ env.JACOCO_VERSION }}:report

      - name: Upload test reports (artifact)
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: surefire-reports
          path: |
            target/surefire-reports/**
          if-no-files-found: warn

      - name: Upload JaCoCo reports (artifact)
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: jacoco-report
          path: |
            ${{ env.COVERAGE_REPORT_DIR }}/**
          if-no-files-found: warn

      - name: Detect coverage override label
        if: github.event_name == 'pull_request'
        id: override
        uses: actions/github-script@v7
        with:
          script: |
            const pr = context.payload.pull_request;
            const labels = (pr.labels || []).map(l => l.name);
            const hasOverride = labels.includes('ci-override-coverage');
            core.setOutput('has_override', hasOverride ? 'true' : 'false');

      - name: Validate override fields (reason/expiry/approver)
        if: github.event_name == 'pull_request' && steps.override.outputs.has_override == 'true'
        run: |
          set -euo pipefail

          BODY="${{ github.event.pull_request.body }}"
          echo "$BODY" > pr_body.txt

          # Require fields in PR template (audit trail)
          grep -qE '^(\*\*Override Coverage Reason:\*\*|Override Coverage Reason:)' pr_body.txt || {
            echo "::error::Missing 'Override Coverage Reason' in PR body."
            exit 1
          }
          grep -qE '^(\*\*Override Coverage Expiry|\*\*Override Coverage Expiry \(YYYY-MM-DD, <= 14 days\):\*\*|Override Coverage Expiry:)' pr_body.txt || {
            echo "::error::Missing 'Override Coverage Expiry' in PR body."
            exit 1
          }
          grep -qE '^(\*\*Override Approver:\*\*|Override Approver:)' pr_body.txt || {
            echo "::error::Missing 'Override Approver' in PR body."
            exit 1
          }

          # Extract expiry date (YYYY-MM-DD)
          EXPIRY=$(grep -E 'Override Coverage Expiry' -n pr_body.txt | head -n1 | sed -E 's/.*([0-9]{4}-[0-9]{2}-[0-9]{2}).*/\1/')
          if ! echo "$EXPIRY" | grep -qE '^[0-9]{4}-[0-9]{2}-[0-9]{2}$'; then
            echo "::error::Override expiry date must be YYYY-MM-DD (found: '$EXPIRY')."
            exit 1
          fi

          TODAY=$(date -u +%F)
          MAX=$(date -u -d "$TODAY +14 days" +%F)

          if [[ "$EXPIRY" < "$TODAY" ]]; then
            echo "::error::Override expiry already passed (expiry=$EXPIRY, today=$TODAY)."
            exit 1
          fi

          if [[ "$EXPIRY" > "$MAX" ]]; then
            echo "::error::Override expiry too long. Must be within 14 days (expiry=$EXPIRY, max=$MAX)."
            exit 1
          fi

          echo "Override expiry validated: $EXPIRY (today=$TODAY, max=$MAX)"

      - name: Setup Python (for diff-cover)
        if: github.event_name == 'pull_request'
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Patch/Diff coverage gate (diff-cover)
        if: github.event_name == 'pull_request'
        run: |
          set -euo pipefail

          # Ensure base branch is available for diff-cover
          BASE_REF="${{ github.base_ref }}"
          git fetch origin "${BASE_REF}:refs/remotes/origin/${BASE_REF}"

          python -m pip install --upgrade pip
          pip install diff-cover

          if [[ ! -f "${{ env.COVERAGE_XML }}" ]]; then
            echo "::error::Coverage XML not found: ${{ env.COVERAGE_XML }}"
            exit 2
          fi

          HAS_OVERRIDE="${{ steps.override.outputs.has_override }}"
          echo "has_override=${HAS_OVERRIDE}" | tee diff-cover.meta

          # Run diff-cover with fail-under gate, and generate reports.
          # If diff coverage < threshold:
          # - fail if no override
          # - pass (with warning) if override label is present
          set +e
          diff-cover "${{ env.COVERAGE_XML }}" \
            --compare-branch "origin/${BASE_REF}" \
            --fail-under="${{ env.PATCH_COVERAGE_MIN }}" \
            --format "markdown:${{ env.DIFF_COVER_MD }},html:${{ env.DIFF_COVER_HTML }}" \
            --exclude "**/generated/**" \
            --total-percent-float \
            --expand-coverage-report \
            2>&1 | tee diff-cover.log
          EXIT_CODE=${PIPESTATUS[0]}
          set -e

          # If report wasn't generated, treat as a hard error (cannot audit)
          if [[ ! -f "${{ env.DIFF_COVER_MD }}" ]]; then
            echo "::error::diff-cover did not generate report. See diff-cover.log"
            exit 2
          fi

          # Add summary to GitHub Actions UI
          {
            echo "## Diff/Patch Coverage"
            echo ""
            echo "- Threshold: **${{ env.PATCH_COVERAGE_MIN }}%**"
            echo "- Override label present: **${HAS_OVERRIDE}**"
            echo ""
            echo "### diff-cover report (markdown)"
            echo ""
            cat "${{ env.DIFF_COVER_MD }}"
          } >> "$GITHUB_STEP_SUMMARY"

          if [[ "${EXIT_CODE}" -ne 0 ]]; then
            if [[ "${HAS_OVERRIDE}" == "true" ]]; then
              echo "::warning::Patch coverage below threshold, but override is approved. (ci-override-coverage)"
              exit 0
            else
              echo "::error::Patch coverage below threshold. Please add tests or request override."
              exit "${EXIT_CODE}"
            fi
          fi

      - name: Upload diff-cover report (artifact)
        if: github.event_name == 'pull_request' && always()
        uses: actions/upload-artifact@v4
        with:
          name: diff-cover-report
          path: |
            diff-cover.log
            diff-cover.meta
            ${{ env.DIFF_COVER_MD }}
            ${{ env.DIFF_COVER_HTML }}
          if-no-files-found: warn

      - name: Comment diff coverage result on PR (audit trail)
        if: github.event_name == 'pull_request' && always()
        uses: actions/github-script@v7
        with:
          script: |
            const fs = require('fs');

            const pr = context.payload.pull_request;
            const hasOverride = (pr.labels || []).some(l => l.name === 'ci-override-coverage');

            let md = '';
            try { md = fs.readFileSync(process.env.DIFF_COVER_MD, 'utf8'); }
            catch (e) { md = '_diff-cover report not found_'; }

            const marker = '<!-- diff-cover-report -->';
            const body =
`${marker}
### 🧪 CI: Diff/Patch Coverage Report

- Threshold: **${process.env.PATCH_COVERAGE_MIN}%**
- Override label: **${hasOverride ? 'true (ci-override-coverage)' : 'false'}**
- Workflow run: ${context.serverUrl}/${context.repo.owner}/${context.repo.repo}/actions/runs/${context.runId}

<details>
<summary>Report (markdown)</summary>

${md}

</details>
`;

            // Upsert a single comment to avoid spamming
            const { data: comments } = await github.rest.issues.listComments({
              owner: context.repo.owner,
              repo: context.repo.repo,
              issue_number: pr.number,
            });

            const existing = comments.find(c => c.body && c.body.includes(marker));
            if (existing) {
              await github.rest.issues.updateComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                comment_id: existing.id,
                body,
              });
            } else {
              await github.rest.issues.createComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                issue_number: pr.number,
                body,
              });
            }
        env:
          DIFF_COVER_MD: ${{ env.DIFF_COVER_MD }}
          PATCH_COVERAGE_MIN: ${{ env.PATCH_COVERAGE_MIN }}

      # Optional: Upload coverage to Codecov (solution 1)
      # Enable by adding CODECOV_TOKEN secret (usually required for private repos).
      - name: Upload coverage to Codecov (optional)
        if: always() && (env.CODECOV_ENABLED == 'true')
        uses: codecov/codecov-action@v5
        with:
          fail_ci_if_error: false
          files: ${{ env.COVERAGE_XML }}
          token: ${{ secrets.CODECOV_TOKEN }}
```

> 说明：Codecov 上传步骤默认关闭（`CODECOV_ENABLED` 未设置）。若你采用方案 1，请在 workflow 或 repo variables 中启用，并在 Branch Protection 中把 Codecov 的 patch check 设为 required。

---

### F.7 `.github/workflows/ai_review.yml`（AI Review 工作流：默认需显式批准）

> 采用 PR-Agent 作为 AI reviewer。其 GitHub Actions 例子与 Claude/OpenAI 的环境变量配置见官方文档。
> PR-Agent 支持用 `OPENAI_KEY` 或 `ANTHROPIC.KEY` 等方式接不同模型。

```yml
name: AI Review (Advisory)

on:
  pull_request:
    types: [opened, reopened, ready_for_review, labeled, synchronize]

permissions:
  contents: read
  pull-requests: write
  issues: write

concurrency:
  group: ai-review-${{ github.workflow }}-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  ai_review_claude:
    name: AI Review (Claude)
    # Security gates:
    # - only run for same-repo PRs (no forks) to avoid secrets exposure
    # - require explicit approval labels
    if: >
      github.event.pull_request.head.repo.full_name == github.repository &&
      contains(join(github.event.pull_request.labels.*.name, ','), 'ai-review') &&
      contains(join(github.event.pull_request.labels.*.name, ','), 'external-llm-approved') &&
      secrets.ANTHROPIC_KEY != ''
    runs-on: ubuntu-latest
    steps:
      - name: PR-Agent (Claude) - review only
        uses: qodo-ai/pr-agent@v0.31
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          # Model selection (configurable)
          config.model: "anthropic/claude-3-opus-20240229"
          config.fallback_models: '["anthropic/claude-3-haiku-20240307"]'
          ANTHROPIC.KEY: ${{ secrets.ANTHROPIC_KEY }}
          # Tool control: advisory review only (no auto-edit)
          github_action_config.auto_review: "true"
          github_action_config.auto_describe: "false"
          github_action_config.auto_improve: "false"
          # Extra instructions (tune to your project)
          pr_reviewer.extra_instructions: >
            Review this Java Spring Boot PR. Focus on correctness, edge cases, null-safety, error handling,
            security (authz/authn), performance, and test adequacy. Suggest missing unit tests and risky changes.

  ai_review_openai:
    name: AI Review (OpenAI/Codex)
    if: >
      github.event.pull_request.head.repo.full_name == github.repository &&
      contains(join(github.event.pull_request.labels.*.name, ','), 'ai-review') &&
      contains(join(github.event.pull_request.labels.*.name, ','), 'external-llm-approved') &&
      secrets.ANTHROPIC_KEY == '' &&
      secrets.OPENAI_KEY != ''
    runs-on: ubuntu-latest
    steps:
      - name: PR-Agent (OpenAI) - review only
        uses: qodo-ai/pr-agent@v0.31
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_KEY: ${{ secrets.OPENAI_KEY }}
          github_action_config.auto_review: "true"
          github_action_config.auto_describe: "false"
          github_action_config.auto_improve: "false"
          pr_reviewer.extra_instructions: >
            Review this Java Spring Boot PR. Focus on correctness, edge cases, null-safety, error handling,
            security (authz/authn), performance, and test adequacy. Suggest missing unit tests and risky changes.
```

**需要的 Secrets：**

- `ANTHROPIC_KEY`（Claude 模式）或 `OPENAI_KEY`（OpenAI/Codex 模式）
- `GITHUB_TOKEN` 为 GitHub 自动提供，无需手工创建

**默认不运行的原因：**

- 必须同时打 Label：`ai-review` + `external-llm-approved`
- 且必须是同仓 PR（fork PR 不运行）

---

### F.8 `.github/dependabot.yml`（推荐：依赖更新自动 PR）

```yml
version: 2
updates:
  - package-ecosystem: "maven"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    labels:
      - "type: chore"
      - "area: deps"

  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
      day: "monday"
    open-pull-requests-limit: 5
    labels:
      - "type: chore"
      - "area: ci"
```

---

### F.9 `.github/workflows/codeql.yml`（可选但推荐：静态安全扫描）

> CodeQL Action 的用法与 `build-mode` 等说明见官方仓库/文档。

```yml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "0 3 * * 1" # every Monday 03:00 UTC
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

jobs:
  analyze:
    name: Analyze (CodeQL)
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: java-kotlin
          build-mode: autobuild

      - name: Autobuild
        uses: github/codeql-action/autobuild@v3

      - name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
```

---

## G. 附录

### G.1 标签字典（建议直接在仓库创建这些 Labels）

**类型**

- `type: bug` 缺陷
- `type: feature` 需求
- `type: chore` 杂项/依赖
- `type: docs` 文档
- `type: security` 安全

**优先级**

- `priority: P0/P1/P2/P3`

**状态**

- `status: triage` 待评估
- `status: ready` 可开发
- `status: in-progress` 开发中
- `status: blocked` 阻塞
- `status: done` 完成

**模块**

- `area: api/auth/db/ci/deps/...`

**风险**

- `risk: low/medium/high`

**门禁与特殊标签**

- `coverage-override-requested`：申请覆盖率豁免（不生效）
- `ci-override-coverage`：覆盖率豁免审批通过（生效，需填写原因+到期+审批人）
- `ai-review`：请求 AI review
- `external-llm-approved`：批准外发给外部 LLM（默认必须）

---

### G.2 PR 模板字段解释（为什么要这些字段）

- **Related Issue**：实现“可追踪可回溯”的核心索引
- **How to Test**：让 reviewer/未来排查者能复现验证
- **Risk & Rollback Plan**：确保线上问题可控（尤其 hotfix）
- **Override Coverage\***：把“破例”制度化、审计化（理由、到期、审批人）

---

### G.3 团队日常执行清单（Checklists）

#### 开发者自检清单（提交前 / 提 PR 前）

- [ ] 分支命名符合规范（包含 issueId）
- [ ] Commit message 符合 Conventional Commits，并引用 Issue
- [ ] 本地 `mvn -B -ntp test` 全绿
- [ ] 新增/修改逻辑有 UT（包含边界与失败场景）
- [ ] 未引入 secrets（token、key、密码、内部 URL 等）
- [ ] PR 模板字段已填完整（测试/风险/回滚/证据）
- [ ] 若请求豁免：已打 `coverage-override-requested`，并写明原因/到期/计划

#### Reviewer 清单（功能、边界、测试、可读性、安全）

- [ ] PR 关联 Issue，需求/修复目标清晰
- [ ] 变更范围合理（是否需要拆 PR）
- [ ] UT 覆盖关键路径与边界条件
- [ ] 错误处理与日志合理（不会泄露敏感信息）
- [ ] API 兼容性（是否需要版本化/迁移）
- [ ] 性能影响（热点路径、N+1、慢查询等）
- [ ] 安全（鉴权/鉴别/越权/输入校验）
- [ ] CI 全绿：UT 必须成功；覆盖率达标或豁免合规

#### Release/回滚清单

- [ ] `main` 当前 commit 对应明确 PR
- [ ] Tag 命名遵循版本规范（如 `v1.2.0`）
- [ ] Release notes 说明变更、风险与升级注意事项
- [ ] 回滚预案明确：`git revert <merge_sha>` 或回滚到上一 tag
- [ ] 若有 DB 变更：有回滚/兼容方案（禁止不可逆迁移无预案）

---

### G.4 示例：覆盖率门禁方案选择建议（默认结论）

- 你希望**最少外部依赖、审计闭环**：选 **方案 2（diff-cover）**（本手册默认已落地）
- 你希望**强可视化 + 团队习惯 SaaS**：选 **方案 1（Codecov/Coveralls）**，并把其 patch check 设为 required

---

如果你后续把 `Java_Spring_Boot` 的真实项目名、Java 版本、是否多模块（multi-module）、以及是否区分 UT/IT（Failsafe）补齐，我也可以在不改变流程原则的前提下，把 `ci.yml` 的 Maven 命令与覆盖率路径进一步“项目定制化”（仍保持可审计与可落地）。

