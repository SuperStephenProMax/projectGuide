---

### C.2 **Feature 流程**（包含设计讨论与 DoD）

#### **Step 1：创建 Feature Issue（必须）**

* **是的，你需要先在 GitHub 上点击 "Create Issue"，并选择 `Feature` 模板。**
* **必要字段**：你必须填写需求的背景、目标、验收标准（DoD）、依赖与风险。

**Feature Issue 模板（详细）**

```markdown
## 需求描述

简要描述此功能/需求的目标。背景、目的、动机等。

**示例：**
- 我们需要新增一个接口来提供用户订单历史查询，支持分页。

## 验收标准（Definition of Done / DoD）

请列出功能完成的验收标准，所有标准必须可测试、可验证、边界条件明确。

**示例 DoD：**
- [ ] 新增接口返回值和错误码已经文档化。
- [ ] UT 覆盖新增的逻辑，且新增代码增量覆盖率 ≥ 70%。
- [ ] 失败场景（例如：无权限、数据不存在）必须有 UT 覆盖。
- [ ] 功能可回滚：不破坏现有功能或有明确的迁移方案。
- [ ] 接口设计与现有 API 兼容（如果是 API 变更）。

## 范围/不做的事情（Out of Scope）

明确哪些内容不在当前需求范围内，避免范围蔓延。

**示例：**
- 当前仅支持用户查询，不涉及支付功能。

## 风险与依赖

列出当前功能实现的潜在风险以及可能的外部依赖（例如：第三方服务、库）。

**示例：**
- 风险：若查询数据量大，性能可能受影响。
- 依赖：需要与支付服务团队协调返回的数据格式。

## 其他说明

其他对功能开发和测试有帮助的备注、链接或文档。
```

### Step 2：创建 Feature 分支

* 基于 Issue 创建 Feature 分支，命名遵循规范：`feature/<issueId>-<short-description>`
  示例：`feature/123-add-user-history-api`

```bash
git checkout main
git pull --ff-only origin main
git checkout -b feature/123-add-user-history-api
```

---

### **C.1 Bugfix 流程**（从发现 → Issue → 分支 → 开发 → 测试 → PR → 合并 → 发布/回滚）

#### **Step 1：创建 Bug Issue（必须）**

* 必须用 Bug 模板，详细描述复现步骤、期望与实际结果、日志等。

**Bug Issue 模板**

```markdown
## Bug 描述

简要描述此问题的背景和影响。

**示例：**
- 用户在支付时出现了 `NullPointerException`，导致支付失败。

## 复现步骤

列出复现问题的详细步骤。

**示例：**
1. 用户登录
2. 进入支付页面
3. 点击支付，发生异常

## 期望结果

描述预期行为。

**示例：**
- 用户能够顺利支付，并且显示支付成功信息。

## 实际结果

描述实际行为。

**示例：**
- 用户支付时应用崩溃，出现 `NullPointerException`。

## 错误日志

提供完整的错误日志或者异常堆栈。

**示例：**
- `java.lang.NullPointerException: Cannot read property 'amount'`
```

#### **Step 2：创建 Bug 分支**

* 基于 Issue 创建 Bugfix 分支，命名遵循规范：`bugfix/<issueId>-<short-description>`
  示例：`bugfix/456-fix-null-pointer-in-payment`

```bash
git checkout main
git pull --ff-only origin main
git checkout -b bugfix/456-fix-null-pointer-in-payment
```

#### **Step 3：开发与本地自测**

* **运行单元测试：**

```bash
mvn -B -ntp test
```

* **生成 JaCoCo 覆盖率报告（本地验证与 CI 一致）：**

```bash
mvn -B -ntp clean \
  org.jacoco:jacoco-maven-plugin:0.8.12:prepare-agent test \
  org.jacoco:jacoco-maven-plugin:0.8.12:report
```

#### **Step 4：提交并推送分支**

* 提交代码并推送到远程仓库：

```bash
git commit -m "fix(payment): prevent NPE when payment amount is null (fixes #456)"
git push -u origin bugfix/456-fix-null-pointer-in-payment
```

---

### **C.3 Hotfix 流程**（紧急修复、回滚策略）

#### **Step 1：创建 Hotfix Issue（必须）**

* 必须用 Hotfix 模板，描述严重问题、紧急修复的背景与目标。

**Hotfix Issue 模板**

```markdown
## Hotfix 描述

简要描述此问题的背景和紧急性。

**示例：**
- 线上支付接口故障，影响用户支付流程，需要紧急修复。

## 影响范围

列出可能受到影响的用户/系统。

**示例：**
- 所有支付功能受影响，用户无法完成支付。

## 紧急修复步骤

描述紧急修复的步骤和注意事项。

**示例：**
- 临时禁用支付功能，防止更多用户受影响。
- 在修复后恢复功能并重新测试。

## 回滚计划

描述如何回滚此热修复。

**示例：**
- 如果修复失败，可以通过 `git revert` 回滚到最近的提交。
```

#### **Step 2：创建 Hotfix 分支**

* 创建 Hotfix 分支，命名遵循规范：`hotfix/<issueId>-<short-description>`
  示例：`hotfix/789-fix-payment-outage`

```bash
git checkout main
git pull --ff-only origin main
git checkout -b hotfix/789-fix-payment-outage
```

#### **Step 3：修复与验证**

* 与 Bugfix 流程类似，修复后进行本地测试和覆盖率验证。

#### **Step 4：推送并创建 PR**

* 提交并推送分支：

```bash
git commit -m "hotfix(payment): fix payment gateway outage (fixes #789)"
git push -u origin hotfix/789-fix-payment-outage
```

#### **Step 5：合并到 `main`，发布紧急修复**

* 在 GitHub 上创建 PR，完成合并与发布。

#### **Step 6：回滚**

* 如果 Hotfix 引入问题，可以通过以下命令回滚：

```bash
git checkout main
git pull --ff-only origin main
git revert <SHA-of-hotfix-merge>
git push origin main
```

---

### 总结：GitHub 上的 Issue/PR 流程与模板

* **Feature 流程**：

    * 先创建 Feature Issue，定义验收标准（DoD）。
    * 进行设计与开发，完成后创建 PR 并进行 Code Review。
* **Bugfix 流程**：

    * 先创建 Bug Issue，描述复现步骤、期望与实际结果。
    * 完成修复后提交，进行本地自测与覆盖率验证。
* **Hotfix 流程**：

    * 对紧急问题创建 Hotfix Issue。
    * 快速修复并进行验证，PR 合并后发布。

如果你需要更细化的模板或具体实现，可以直接根据你的需求调整。
