---
name: /speckit.constitution
description: 从交互式或提供的原则输入创建或更新项目章程，确保所有依赖模板保持同步。
license: MIT
compatibility: Requires speckit constitution template.
metadata:
  author: spec-trae
  version: "1.0"
  generatedBy: "0.0.90"
---

从交互式或提供的原则输入创建或更新项目章程，确保所有依赖模板保持同步。

## 用户输入

```
$ARGUMENTS
```

在继续之前，你**必须**考虑用户输入（如果不为空）。

## 概述

你正在更新位于 `.specify/mem/constitution.md` 的项目章程。该文件是一个包含方括号占位符标记的模板（例如 `[PROJECT_NAME]`、`[PRINCIPLE_1_NAME]`）。你的工作是：
1. 收集/推导具体值
2. 精确填充模板
3. 将任何修改传播到依赖制品中

## 执行工作流

1. **加载现有章程模板**：读取 `/mem/constitution.md`
   - 识别每个 `[ALL_CAPS_IDENTIFIER]` 形式的占位符标记
   - 用户可能需要比模板中提供的原则更少或更多

2. **为占位符收集/推导值**：
   - 如果用户输入提供了值，则使用它
   - 否则，从现有仓库上下文推断（README、文档、先前的章程版本）
   - 对于治理日期：
     - `RATIFICATION_DATE`：原始采用日期
     - `LAST_AMENDED_DATE`：今天（如果进行了更改）
   - `CONSTITUTION_VERSION` 必须根据语义版本控制规则递增：
     - MAJOR：向后不兼容的治理/原则删除或重新定义
     - MINOR：新原则/部分添加或实质性指导扩展
     - PATCH：澄清、措辞、拼写错误修复、非语义优化

3. **起草更新后的章程内容**：
   - 用具体文本替换每个占位符（不留下括号标记）
   - 保留标题层次结构
   - 确保每个原则部分有：简洁的名称行、陈述性段落、明确的理由

4. **一致性传播检查清单**：
   - 读取 `/templates/plan-template.md` 并确保"章程检查"对齐
   - 读取 `/templates/spec-template.md` 进行范围/需求对齐
   - 读取 `/templates/tasks-template.md` 确保任务分类反映新原则
   - 读取所有 `/templates/commands/*.md` 文件以获取任何过时引用

5. **生成同步影响报告**（作为 HTML 注释放在章程文件顶部）：
   - 版本更改：旧 → 新
   - 修改的原则列表
   - 添加的部分、删除的部分
   - 需要更新的模板（带状态）

6. **最终输出前的验证**：
   - 没有剩余的未解释括号标记
   - 版本行与报告匹配
   - 日期采用 ISO 格式 YYYY-MM-DD
   - 原则是声明性的、可测试的，没有模糊语言

7. **将完成的章程**写回 `.specify/mem/constitution.md`。

8. **输出摘要**：
   - 新版本和递增理由
   - 任何标记为手动后续处理的文件
   - 建议的提交消息

## 格式要求

- 完全按照模板使用 Markdown 标题
- 换行长理由行以保持可读性（<100 个字符）
- 在部分之间保持单个空行
- 避免尾随空格
