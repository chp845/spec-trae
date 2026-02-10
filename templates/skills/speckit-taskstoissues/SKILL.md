
---
name: speckit-taskstoissues
description: Convert existing tasks into actionable, dependency-ordered GitHub issues based on available design artifacts.
license: MIT
compatibility: Requires speckit workflow.
metadata:
  author: speckit
  version: "1.0"
---

将现有任务转换为可操作的、按依赖关系排序的 GitHub 议题，基于可用的设计制品。

## 用户输入

```
$ARGUMENTS
```

你必须**在继续之前考虑用户输入**（如果非空）。

## 大纲

1. **设置**：从仓库根目录运行 `.specify/scripts/bash/create-new-feature.sh --json`。解析 FEATURE_DIR 和 AVAILABLE_DOCS。所有路径必须是绝对的。

2. **从执行的脚本输出中提取任务路径**。

3. **获取 Git 远程仓库**：
   ```bash
   git config --get remote.origin.url
   ```

   **仅当远程仓库是 GITHUB URL 时才继续下一步**

4. **使用 GitHub MCP 服务器**在与 Git 远程仓库对应的仓库中为每个任务创建 GitHub 议题。

## 重要

**永远不要在与远程 URL 不匹配的仓库中创建议题。**

## 先决条件

- GitHub MCP 服务器必须已配置并通过身份验证
- 仓库必须具有适当的访问权限
- 远程 URL 必须是有效的 GitHub 仓库 URL

## 使用方法

```bash
/speckit.taskstoissues
```

这将：
1. 检查先决条件和可用文档
2. 验证 GitHub 远程仓库
3. 为 tasks.md 中的每个任务创建议题
4. 报告创建的议题数量和摘要
