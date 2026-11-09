---
name: testcase-step-alignment
description: 评审脚本每个Step是否符合测试用例（含Expected Result），并按 Playwright PR Review 口径输出稳定性/可排障问题（Blocker/Major/Minor）与可复制修复片段。
---

# 测试用例 Step 对齐 + Playwright PR Review

## 适用场景

- 评审每个 Step 的脚本是否符合测试用例（含 Expected Result）
- 输出稳定性、可排障、可维护性、项目规则方面的改进点（按 Blocker/Major/Minor）

## 输入要求

- 测试用例的 Steps + Expected Result
- 对应脚本路径（`*.spec.ts`）与必要的 POM/Helper/fixtures 路径
- 可选：失败日志/截图/trace 线索

## 默认行为

- 默认只做 Review，不自动修改代码
- 只有用户明确表达授权修改意图时，才允许进入自动修复

---

## 1. Review Framework

### Step 0：快速定界

- 变更范围：新增/改动了哪些 spec、POM/Helper
- 风险等级：是否涉及登录/权限/创建删除/上传等高危链路

### Step 1：用例 Step 对齐

对每个 Step 逐一检查：

- 动作是否实现：用例动作是否在脚本中出现且顺序合理
- 期待结果是否可验证：是否有断言/验证点
- 断言内容是否匹配：文案、状态、列表项、选中态、弹窗标题等
- 脚本头部目的是否一致
- 关键状态是否等待：断言前是否已等待页面稳定

### Step 2：项目约定检查

检查是否符合 Repository Conventions（详见第 4 节）。

### Step 3：稳定性检查

检查是否符合 Repository Conventions 中的稳定性相关规则（等待策略、断言模式、错误处理等）。

### Step 4：可执行性检查

- 检查用例/断言是否被"屏蔽"（注释掉、skip/only 等）
- 关键断言或业务步骤被注释
- 大量固定等待：`waitForTimeout` / `customWaitForTime` / `waitForTime`

### Step 5：输出结论

按以下格式输出：

**Step 对齐结果表**：逐步判定"通过/部分覆盖/不通过"

**结论汇总**：
- 通过：Step ...
- 部分覆盖：Step ...
- 不通过：Step ...

**PR Review 问题清单**：按 Blocker/Major/Minor 输出，提供可复制修复片段

**需人工确认**：
- 证据不足
- 存在多种合理解释
- 无法建立完整证据链
- 关键代码未提供

---

## 2. Core Principles

### 2.1 证据链完整性（Evidence Chain Completeness）

任何 Review 结论必须建立在完整证据链上。

对于任何用于判断的值：

- 定义
- 赋值
- 覆盖
- 转换
- 使用

都必须追踪。追踪深度以完成当前判断所需为准。

禁止基于局部代码片段直接下结论。

### 2.2 证据充分性原则（Evidence Sufficiency）

当现有证据已经能够稳定、单义地证明某个结论时：

- 不得因缺少某种特定形式的证据而判定覆盖不足
- 不得机械要求额外验证点
- 应先评估现有证据是否足以支撑结论

禁止将：

- 缺少直接证据
- 缺少某类 UI 断言

等同于：

- 缺少证据

### 2.3 最小假设原则（Minimal Assumption）

当代码存在多种解释时：

- 不得选择最容易产生问题的解释
- 不得基于猜测输出缺陷
- 应继续寻找证据
- 无法确认时标记为「需人工确认」

禁止用推测替代证据。

---

## 3. Self Check

在输出每个问题前，执行以下自查：

1. 结论依据是什么？
2. 是否查看了完整上下文？
3. 是否存在反例？
4. 是否存在另一种解释？
5. 如果用户质疑，我能否给出完整证据链？
6. 如果错了，会不会成为误报？

若存在明显证据缺失、上下文不完整或无法自证的问题，应标记为「需人工确认」而非直接输出结论。

---

## 4. Repository Conventions

### 4.1 分层约束

- testcase 只保留业务动作编排与断言
- 底层能力（clickByCss、waitForSelector 等）必须下沉到 Page Object / BasePage
- Page Object / Helper 不得直接出现原生 Playwright 调用
- 若 BasePage 尚未提供能力，应优先建议补充 BasePage 后复用

### 4.2 Fixture 注入

- GUI testcase 从 `@setup/testSetup` 导入 `test` / `expect`
- Page Object / Helper 优先通过 `testSetup.ts` 中已注册的 fixture 注入获取
- 新增 fixture 应遵循 `createPageFixture`、`createEnumFixture` 等既有模式

### 4.3 等待策略

- 避免无条件固定等待（`waitForTimeout` / `customWaitForTime`）
- 优先显式等待元素/对话框/消息/网络稳定点
- 语义化方法（如 `loading()`）内部是否仍依赖固定等待需验证

### 4.4 数据隔离

- 使用随机后缀避免命名冲突
- 清理失败也能执行（finally/afterEach）

### 4.5 断言模式

- 动态 UI 状态优先使用 web-first assertions
- 稳定值读取可使用 `expect(await ...).toBe(...)`
- 避免在动态 UI 状态上大量使用 `expect(await pageObject.isXxx()).toBeTruthy()`

### 4.6 错误处理

- 检查 Page Object 是否存在"错误吞没"模式（catch 后返回 false/null/''）
- 评估是否会导致 spec 只能看到表面断言失败

---

## 5. Examples（最多 3 个，不再扩张）

### 5.1 证据链追踪示例

**反例**：`TITLE_CUT = TITLE_CHARACTER.substring(0, 59)` → 直觉判定"59 字符，少 1 个"

**正例**：追踪到调用处 `i + TITLE_CUT` → `i` 占 1 字符 + 59 = 60 字符 → 与用例"max 60"一致 → 正确

### 5.2 证据充分性示例

**反例**：没有 preview 直接断言 → 判定"位置没验证"

**正例**：章节范围已经稳定、单义地证明位置正确 → 已覆盖

### 5.3 最小假设示例

**反例**：`attributes` 默认值有换行 → 直接判定调用处拼接错误

**正例**：追踪到调用处传入 3 个参数会覆盖默认值 → 实际值与 DOM 一致 → 无问题

---

## 6. 证据引用要求

所有 Major/Blocker 结论必须提供对应代码证据（文件路径 + 行号范围）。

证据优先于结论。若无法提供证据，则不得输出确定性结论。

---

## 优先级定义

### Blocker

- spec 中直接使用 `page.waitForTimeout()`
- 大量使用固定等待
- 存在"屏蔽代码"导致用例实际不执行
- 低质量 selector 导致高不稳定
- PageObject/Helper 职责混乱
- testcase 直接调用基础类底层方法

### Major

- 关键链路主要依赖固定等待
- 等待策略不完整：断言前未确保页面稳定
- 断言/错误信息不可读
- 数据隔离不足
- 动态 UI 状态未使用 web-first assertion
- Page Object 吞没真实异常
- GUI testcase 未使用 fixture 注入体系

### Minor

- 命名/结构不一致
- 可提炼复用的重复片段