---
description: 通过处理和执行 tasks.md 中定义的所有任务来执行实施计划
scripts:
  sh: scripts/bash/check-prerequisites.sh --json --require-tasks --include-tasks
  ps: scripts/powershell/check-prerequisites.ps1 -Json -RequireTasks -IncludeTasks
---

## 用户输入

```text
$ARGUMENTS
```

在继续之前, 你**必须**考虑用户输入(如果不为空).

## 概要

1. 从仓库根目录运行 `{SCRIPT}` 并解析 FEATURE_DIR 和 AVAILABLE_DOCS 列表. 所有路径必须是绝对路径. 对于参数中的单引号如 "I'm Groot", 使用转义语法: 例如 'I'\''m Groot'(或尽可能使用双引号: "I'm Groot").

2. **检查清单状态**(如果 FEATURE_DIR/checklists/ 存在): 
   - 扫描 checklists/ 目录中的所有清单文件
   - 对于每个清单, 统计: 
     * 总项目数: 所有匹配 `- [ ]` 或 `- [X]` 或 `- [x]` 的行
     * 已完成项目数: 匹配 `- [X]` 或 `- [x]` 的行
     * 未完成项目数: 匹配 `- [ ]` 的行
   - 创建状态表: 
     ```
     | Checklist | Total | Completed | Incomplete | Status |
     |-----------|-------|-----------|------------|--------|
     | ux.md     | 12    | 12        | 0          | ✓ PASS |
     | test.md   | 8     | 5         | 3          | ✗ FAIL |
     | security.md | 6   | 6         | 0          | ✓ PASS |
     ```
   - 计算总体状态: 
     * **PASS**: 所有清单都有 0 个未完成项目
     * **FAIL**: 一个或多个清单有未完成项目

   - **如果任何清单未完成**: 
     * 显示包含未完成项目数的表格
     * **停止**并询问: "Some checklists are incomplete. Do you want to proceed with implementation anyway? (yes/no)"
     * 在继续之前等待用户响应
     * 如果用户说 "no" 或 "wait" 或 "stop", 停止执行
     * 如果用户说 "yes" 或 "proceed" 或 "continue", 继续到步骤 3

   - **如果所有清单都已完成**: 
     * 显示显示所有清单通过的表格
     * 自动继续到步骤 3

3. 加载和分析实施上下文: 
   - **必需**: 读取 tasks.md 获取完整任务列表和执行计划
   - **必需**: 读取 plan.md 获取技术栈、架构和文件结构
   - **如果存在**: 读取 data-model.md 获取实体和关系
   - **如果存在**: 读取 contracts/ 获取 API 规范和测试要求
   - **如果存在**: 读取 research.md 获取技术决策和约束
   - **如果存在**: 读取 quickstart.md 获取集成场景

4. **项目设置验证**: 
   - **必需**: 基于实际项目设置创建/验证忽略文件: 

   **检测和创建逻辑**: 
   - 检查以下命令是否成功以确定是否为 git 仓库(如果是, 创建/验证 .gitignore): 

     ```sh
     git rev-parse --git-dir 2>/dev/null
     ```
   - 检查 Dockerfile* 是否存在或 plan.md 中是否提到 Docker → 创建/验证 .dockerignore
   - 检查 .eslintrc* 或 eslint.config.* 是否存在 → 创建/验证 .eslintignore
   - 检查 .prettierrc* 是否存在 → 创建/验证 .prettierignore
   - 检查 .npmrc 或 package.json 是否存在 → 创建/验证 .npmignore(如果发布)
   - 检查 terraform 文件(*.tf)是否存在 → 创建/验证 .terraformignore
   - 检查是否需要 .helmignore(存在 helm charts)→ 创建/验证 .helmignore

   **如果忽略文件存在**: 验证它包含基本模式, 仅追加缺失的关键模式
   **如果忽略文件缺失**: 为检测到的技术创建完整模式集

   **按技术的通用模式**(来自 plan.md 技术栈):
   - **Node.js/JavaScript**: `node_modules/`, `dist/`, `build/`, `*.log`, `.env*`
   - **Python**: `__pycache__/`, `*.pyc`, `.venv/`, `venv/`, `dist/`, `*.egg-info/`
   - **Java**: `target/`, `*.class`, `*.jar`, `.gradle/`, `build/`
   - **C#/.NET**: `bin/`, `obj/`, `*.user`, `*.suo`, `packages/`
   - **Go**: `*.exe`, `*.test`, `vendor/`, `*.out`
   - **Ruby**: `.bundle/`, `log/`, `tmp/`, `*.gem`, `vendor/bundle/`
   - **PHP**: `vendor/`, `*.log`, `*.cache`, `*.env`
   - **Rust**: `target/`, `debug/`, `release/`, `*.rs.bk`, `*.rlib`, `*.prof*`, `.idea/`, `*.log`, `.env*`
   - **Kotlin**: `build/`, `out/`, `.gradle/`, `.idea/`, `*.class`, `*.jar`, `*.iml`, `*.log`, `.env*`
   - **C++**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.so`, `*.a`, `*.exe`, `*.dll`, `.idea/`, `*.log`, `.env*`
   - **C**: `build/`, `bin/`, `obj/`, `out/`, `*.o`, `*.a`, `*.so`, `*.exe`, `Makefile`, `config.log`, `.idea/`, `*.log`, `.env*`
   - **Swift**: `.build/`, `DerivedData/`, `*.swiftpm/`, `Packages/`
   - **R**: `.Rproj.user/`, `.Rhistory`, `.RData`, `.Ruserdata`, `*.Rproj`, `packrat/`, `renv/`
   - **通用**: `.DS_Store`, `Thumbs.db`, `*.tmp`, `*.swp`, `.vscode/`, `.idea/`

   **工具特定模式**:
   - **Docker**: `node_modules/`, `.git/`, `Dockerfile*`, `.dockerignore`, `*.log*`, `.env*`, `coverage/`
   - **ESLint**: `node_modules/`, `dist/`, `build/`, `coverage/`, `*.min.js`
   - **Prettier**: `node_modules/`, `dist/`, `build/`, `coverage/`, `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`
   - **Terraform**: `.terraform/`, `*.tfstate*`, `*.tfvars`, `.terraform.lock.hcl`
   - **Kubernetes/k8s**: `*.secret.yaml`, `secrets/`, `.kube/`, `kubeconfig*`, `*.key`, `*.crt`

5. 解析 tasks.md 结构并提取: 
   - **任务阶段**: 设置、测试、核心、集成、完善
   - **任务执行约束**: **必须**严格按照 `tasks.md` 中的阶段顺序和任务顺序执行. 严禁跨阶段或跳过未完成任务执行. 
   - **任务详情**: ID、描述、文件路径. (忽略 [P] 并行标记, 始终保持顺序执行以确保稳定性)
   - **执行流程**: 线性执行流程, 每个任务必须在下一个任务开始前完全结束. 

6. 按照任务计划执行实施: 
   - **严格阶段顺序**: 必须在当前阶段所有任务完成后, 才能进入下一阶段. 
   - **严格任务顺序**: 必须按任务在 `tasks.md` 中出现的物理顺序逐一执行. 
   - **任务完成标记**: **每个**任务执行完成后, 必须立即在 `tasks.md` 中将其标记为 `[x]`. 
   - **遵循 TDD 方法**: 确保测试任务在对应的功能实施任务之前完成并标记. 
   - **原子提交/验证**: 在标记任务完成并继续下一个任务之前, 验证当前任务的更改. 

7. 实施执行规则: 
   - **顺序执行保证**: 严禁并行执行任何任务. 
   - **状态同步**: 每次任务完成后立即保存文件, 确保 `tasks.md` 实时反映进度. 
   - **阶段性检查**: 在每个阶段结束时, 确认该阶段所有任务均已标记为 `[x]`. 

8. 进度跟踪和错误处理: 
   - 在每个完成的任务后报告进度, 并确认已更新 `tasks.md`. 
   - 如果任何任务失败, **立即停止**执行. 严禁在有失败任务的情况下继续后续任务. 
   - 提供带有调试上下文的清晰错误消息, 并指明失败的任务 ID. 
   - 在修复失败后, 从该失败的任务开始重新执行. 

9. 完成验证: 
   - 最终检查 `tasks.md`, 验证所有任务均已标记为 `[x]`. 
   - 确认实施的功能与原始规范完全一致. 
   - 验证测试通过且覆盖率满足要求. 
   - 报告最终状态, 提供已完成阶段和任务的详细摘要. 

注意: 此命令要求 `tasks.md` 具有完整的任务分解. 如果任务不完整或缺失, 必须先运行 `/tasks` 重新生成.
