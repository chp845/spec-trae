
---
name: speckit-specify
description: Create or update feature specifications from natural language feature descriptions.
license: MIT
compatibility: Requires speckit workflow.
metadata:
  author: speckit
  version: "1.0"
---

从自然语言功能描述创建或更新功能规范。

## 用户输入

```
$ARGUMENTS
```

在继续之前，你**必须**考虑用户输入（如果不为空）。

## 概述

用户在 `/speckit.specify` 后输入的文本**就是**功能描述。假设你在此对话中始终可以访问它。

**需求提取源：**
- 如果用户在输入中指定了文件路径（例如 "from docs/req1.pdf, docs/req2.md"），你**必须**读取并分析所有指定文档作为主要需求源
- 整合来自多个来源的信息，识别关联、补充或潜在冲突
- 如果未指定文件，则将用户的自然语言描述作为唯一需求源

## 执行步骤

1. **生成简短名称**（2-4 个词）：
   - 分析功能描述并提取有意义的关键词
   - 创建捕捉功能核心的简短名称
   - 尽可能使用动-名词格式（例如 "add-user-auth"、"fix-payment-bug"）
   - 保留技术术语和缩写（OAuth2、API、JWT 等）

2. **在创建新分支之前检查现有分支**：
   ```bash
   git fetch --all --prune
   git ls-remote --heads origin | grep -E 'refs/heads/[0-9]+-<short-name>$'
   git branch | grep -E '^[* ]*[0-9]+-<short-name>$'
   ```
   从所有来源找到最高的特性编号并使用 N+1。

3. **运行脚本**：
   ```bash
   .specify/scripts/bash/create-new-feature.sh --json --json --number N+1 --short-name "your-short-name" "feature description"
   ```
   JSON 输出包含 BRANCH_NAME 和 SPEC_FILE 路径。

4. **加载** `.specify/templates/spec-template.md` 了解必需章节。

5. **执行工作流**：
   1. 从输入或指定文档解析用户需求
   2. 提取关键概念：参与者、操作、数据、约束
   3. 对于模糊方面：做出有根据的猜测，标记为 `[NEEDS CLARIFICATION：具体问题]`（最多 3 个）
   4. 填写用户场景和测试章节
   5. 生成功能需求（每个必须是可测试的）
   6. 定义成功标准（可测量、技术无关）
   7. 识别关键实体（如果涉及数据）
   8. 返回：成功（规范准备好进行规划）

6. **将规范写入** SPEC_FILE，使用模板结构。

7. **规范质量验证**：
   - 在 `FEATURE_DIR/checklists/requirements.md` 创建清单
   - 根据质量标准运行验证检查
   - **如果所有项目通过**：标记清单完成，继续到步骤 8
   - **如果项目失败**：更新规范，重新验证（最多 3 次迭代）
   - **如果仍有 NEEDS CLARIFICATION 标记**：
     1. 提取所有标记（最多 3 个，保留最关键的）
     2. 以表格格式向用户呈现选项
     3. 等待用户响应的选择
     4. 用用户答案替换标记
     5. 重新验证

8. **报告完成**：包括分支名称、规范文件路径、清单结果、下一阶段（`/speckit.clarify` 或 `/speckit.plan`）的准备状态。

## 快速指南

- 专注于**什么**和**为什么**，而非**如何**（无技术栈、API、代码结构）
- 为业务利益相关者编写，而非为开发者
- 不要创建嵌入式清单（单独命令）

## 成功标准指南

成功标准必须是：
1. **可测量**：包括具体指标（时间、百分比、计数、速率）
2. **技术无关**：无框架、语言、数据库或工具
3. **以用户为中心**：从用户/业务角度描述结果
4. **可验证**：无需实现知识即可测试

**好的示例：**
- "用户可以在 3 分钟内完成结账"
- "系统支持 10,000 个并发用户"
- "95% 的搜索在 1 秒内返回结果"

**坏的示例：**
- "API 响应时间在 200ms 以下"（太技术化）
- "数据库可以处理 1000 TPS"（实现细节）
- "React 组件高效渲染"（框架特定）
