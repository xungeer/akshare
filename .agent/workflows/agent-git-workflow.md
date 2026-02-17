---
description: 版本控制工作流
---

# Antigravity Agent 指令集：akshare 定制化版本控制工作流

## 1. 角色与目标设定 (Context & Objective)

* **Target Repository:** `akshare` (Python 开源金融数据接口库)
* **Execution Environment:** Antigravity IDE 终端及 Git 环境。
* **Primary Objective:** 维护一个 Forked `akshare` 仓库。必须确保能够随时从官方上游（Upstream）拉取最新功能，同时安全地保留、测试并提交本地的自定义修改（如增加新的数据接口或清洗逻辑），且两者不能产生破坏性冲突。

## 2. 状态与变量定义 (Variables)

在执行任何 Git 操作前，Agent 需明确以下默认命名约定：

* `origin`: 用户的远程个人仓库 (例: `https://github.com/xungeer/akshare.git`)。
* `upstream`: 官方远程仓库 (`https://github.com/akfamily/akshare.git`)。
* `main` / `master`: 纯净分支，**仅用于**严格同步 `upstream` 的官方代码，绝对禁止在此分支进行任何本地自定义修改。
* `custom-dev` (或具体功能分支名): 定制化开发分支，所有本地修改必须在此类分支上进行。

## 3. 初始化阶段：建立上游关联 (Initialization)

*触发条件：首次接管该项目，或执行 `git remote -v` 未发现 upstream 时。*

**Agent 执行步骤：**

1. 进入项目根目录。
2. 添加官方仓库为上游：
```bash
git remote add upstream https://github.com/akfamily/akshare.git

```


3. 验证远程分支配置是否正确：
```bash
git remote -v

```


*(期望输出应同时包含 origin 和 upstream 的 fetch/push 地址)*

## 4. 标准开发工作流 (Daily Development SOP)

*触发条件：用户要求对 akshare 源码进行自定义修改时。*

**Agent 执行步骤：**

1. **环境检查：** 确保当前不在 `main` 分支上直接修改。
```bash
git checkout -b custom-dev

```


*(若分支已存在，则使用 `git checkout custom-dev`)*
2. **执行修改：** 根据用户需求修改 Python 源码。
3. **提交变更：** 遵循规范的 Commit Message。
```bash
git add .
git commit -m "feat(custom): 添加了自定义的A股数据处理逻辑"

```


4. **推送到个人云端：**
```bash
git push origin custom-dev

```



## 5. 官方代码同步工作流 (Upstream Sync SOP)

*触发条件：官方 akshare 发布了新版本，或用户明确要求同步最新官方功能时。*

**Agent 执行步骤（严格按顺序执行）：**

**步骤 A：更新本地纯净主分支**

1. 拉取官方最新动态：
```bash
git fetch upstream

```


2. 切换到本地纯净主分支：
```bash
git checkout main

```


3. 将官方更新合并到本地主分支：
```bash
git merge upstream/main

```


4. （可选）将同步后的纯净代码推送到用户的 GitHub `main` 分支：
```bash
git push origin main

```



**步骤 B：将官方更新融合到自定义分支**

1. 切换回用户的开发分支：
```bash
git checkout custom-dev

```


2. 将更新后的 `main` 分支代码合并进来（推荐使用 rebase 以保持提交历史整洁，若 Agent 处理冲突能力受限，可降级使用 merge）：
```bash
# 策略 1: Rebase (推荐，历史更干净)
git rebase main

# 策略 2: Merge (备选，容错率更高)
git merge main

```



## 6. 异常处理：冲突解决指南 (Conflict Resolution)

*触发条件：在执行 `rebase` 或 `merge` 时，Git 抛出 `CONFLICT` 警告。*

**Agent 执行原则：**

1. **中断并分析：** 立即停止自动提交，读取 `git status` 获取冲突文件列表。
2. **代码对齐：** 打开冲突文件，定位 `<<<<<<<`、`=======`、`>>>>>>>` 标记。
3. **解决策略：**
* 如果是官方的数据源接口发生变更，而本地修改的是数据清洗逻辑：保留官方的新接口，重新适配本地的清洗逻辑。
* 如果不确定如何取舍：通过 Antigravity 的对话窗口，向用户输出冲突代码段，并请求用户人工确认保留哪一部分。


4. **恢复流程：** 解决冲突并删除标记符后，执行：
```bash
git add <冲突的文件>
# 若使用的是 rebase:
git rebase --continue
# 若使用的是 merge:
git commit -m "chore: 解决与官方最新代码的合并冲突"

```