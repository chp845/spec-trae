
---
name: speckit-implement
description: Execute implementation plan by processing and completing all tasks defined in tasks.md.
license: MIT
compatibility: Requires speckit workflow.
metadata:
  author: speckit
  version: "1.0"
---

通过处理和执行 tasks.md 中定义的所有任务来执行实施计划。

## 用户输入

```
$ARGUMENTS
```

在继续之前，你**必须**考虑用户输入（如果不为空）。

## 概述

此命令通过处理和完成 `tasks.md` 中的所有任务来执行实施计划。

## 执行步骤

1. **设置**：从仓库根目录运行 `.specify/scripts/bash/create-new-feature.sh --json`。解析 FEATURE_DIR 和 AVAILABLE_DOCS。所有路径必须是绝对的。

2. **清单状态**（如果 FEATURE_DIR/checklists/ 存在）：
   - 扫描 checklists/ 目录中的所有清单文件
   - 统计：总项目数、已完成项目（`- [X]` 或 `- [x]`）、未完成项目（`- [ ]`）
   - 创建状态表显示每个清单的 PASS/FAIL

   **如果任何清单未完成：**
   - 显示包含未完成计数的表格
   - **停止**并询问："一些清单未完成。你想继续实施吗？"
   - 等待用户响应（"yes"/"no"）

   **如果所有清单通过：**
   - 显示通过的表格
   - 自动继续到步骤 3

3. **加载实施上下文**：
   - **必需**：读取 tasks.md 获取完整任务列表
   - **必需**：读取 plan.md 获取技术栈、架构、文件结构
   - **如果存在**：读取 data-model.md、contracts/、research.md、quickstart.md

4. **项目设置验证**：
   - 检测和创建/验证忽略文件：
     - `.gitignore`（用于 git 仓库）
     - `.dockerignore`（如果提到 Docker）
     - `.eslintignore`、`.prettierignore`、`.npmignore`、`.terraformignore`、`.helmignore` 按需

5. **解析 tasks.md 结构**：
   - 任务阶段：设置、测试、核心、集成、完善
   - **必须**完全按照阶段顺序和任务顺序执行
   - 提取：任务 ID、描述、文件路径
   - 忽略 [P] 并行标记——始终顺序执行以确保稳定性

6. **执行实施**：
   - **严格阶段顺序**：在进入下一阶段之前完成当前阶段的所有任务
   - **严格任务顺序**：按 tasks.md 中出现的物理顺序执行任务
   - **标记完成**：每个任务完成后，立即标记为 `[x]`
   - **遵循 TDD 方法**：确保测试任务在对应功能任务之前完成
   - **原子提交**：在标记任务完成之前验证更改

7. **执行规则**：
   - **顺序执行**：永不并行化任务
   - **状态同步**：每次任务后立即保存文件
   - **阶段检查点**：在阶段结束时确认所有任务标记为 `[x]`

8. **进度跟踪和错误处理**：
   - 每个完成的任务后报告进度
   - 如果任何任务失败，**立即停止**
   - 提供带有失败任务 ID 的清晰错误消息
   - 修复后从失败的任务重新执行

9. **完成验证**：
   - 最终检查所有任务标记为 `[x]`
   - 确认实施的功能与原始规范完全一致
   - 验证测试通过且覆盖率满足要求
   - 报告最终状态和完成的阶段/任务摘要

## 重要说明

此命令要求 `tasks.md` 具有完整的任务分解。如果任务不完整或缺失，必须先运行 `/speckit.tasks`。
