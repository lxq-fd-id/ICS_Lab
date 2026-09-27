# Lab0: GitLab 实验报告

- 姓名：______
- 学号：______
- 日期：2026-09-27
- 仓库地址：https://github.com/lxq-fd-id/ICS_Lab

## 一、文档问题回答（任务 1，15 分）

### 1. 你之前有过多人协同开发的经历吗？如果有，你们是使用什么方式分工协作的？

（示例一，如之前有过协作经历）有过。在上学期的项目中，我们使用 GitHub 协作，采用"主干分支 + 功能分支"的方式分工：每人从 main 切出自己的 feature 分支负责一个模块，开发自测完成后发起 Pull Request，由另一位同学 review 通过后合并回主干；遇到同时修改同一文件的冲突时，由后合并的一方负责解决。

（示例二，如之前没有协作经历）之前没有正式的多人协同开发经历。通过本次实验我第一次完整经历了"两个分支修改同一文件的同一行 → 合并产生冲突 → 手动解决"的过程，直观理解了版本控制对多人协作的意义：它把"冲突"从人肉逐行对比复制，变成了机器可检测、可定位、可修复的显式事件。

### 2. 思考一下，Git 为什么要设计"暂存-提交"两个步骤？

Git 把文件状态分为工作区、暂存区（Index）和仓库三层，暂存区是工作区与仓库之间的"缓冲地带"，这个设计有几点价值：

1. **挑选与组装**：一次提交可以只包含你想提交的改动。开发中往往同时改了多个文件、甚至一个文件里包含多种性质的修改（如一个 bug 修复、一个新功能），通过 `git add` 可以把相关改动精确归入同一次提交，保证每次提交"原子"且主题单一，便于日后回溯和检索。
2. **提交前检查**：暂存后、提交前，可以用 `git status` / `git diff --cached` 检查即将入库的内容，避免误提交临时文件等无关内容。
3. **降低切换成本**：暂存区与工作区分离，改动到一半时可以只暂存一部分、丢弃另一部分，或安全地切换分支而不丢失半成品状态。
4. **语义对应**：提交是仓库历史的一个节点，一旦提交不应轻易改变；暂存让提交者在"决定记录什么"之前有充分的斟酌空间。

### 3. `git branch` 和 `git branch -a` 的区别是什么？

`git branch` 只列出**本地分支**；`git branch -a`（等价于 `git branch --all`）除了本地分支，还会列出**远程跟踪分支**（形如 `remotes/origin/main`），即本地记录的远程仓库上存在的分支。克隆仓库后本地与远程分支同名显示（如 `main`），用 `git branch` 看不出远程还有哪些分支，用 `-a` 才能看到"本地 + 远程"的完整分支全景。

## 二、实验步骤（任务 2 & 任务 4）

### 2.1 环境与配置
- 使用 WSL（Ubuntu）作为实验环境，`git --version` 确认已安装（git version 2.53.0）。
- 配置全局身份：`git config --global user.name "lobster"`、`git config --global user.email "19175078132@163.com"`、`git config --global init.defaultBranch main`。
- 生成 SSH 密钥 `ssh-keygen -t ed25519`，公钥添加到 GitHub（Settings → SSH and GPG keys），验证 `ssh -T git@github.com` 输出 `Hi lxq-fd-id! You've successfully authenticated`。

### 2.2 建立个人仓库并完成任务 2
- 在模板仓库页面点击 Use this template → Create a new repository，仓库名 `ICS_Lab`。
- 克隆：`git clone git@github.com:lxq-fd-id/ICS_Lab.git`。
- 修改 `main.c`：完成 TODO，将打印内容改为自定义句子，并补上 `return 0;`。
- 提交：`git add main.c && git commit -m "feat: complete the TODO in main.c"`。

### 2.3 分支管理与合并冲突（任务 4）
1. 在 main 分支把 `printf` 句子改为句子 A 并提交；
2. `git switch -c feature` 创建并切换到 feature 分支，把同一行 `printf` 改为句子 B 并提交；
3. 切回 main 后 `git merge feature`，第一次为 Fast-forward 合并（无冲突）——因为 feature 仅领先于 main，分支尚未真正分叉；
4. 继续提交使两个分支真正分叉：main 上改为句子 C、feature 上改为句子 D；
5. `git merge feature` 触发冲突：`CONFLICT (content): Merge conflict in main.c`，`git status` 显示 `Unmerged paths / both modified: main.c`。

   截图 1（遇到冲突）：
   ![遇到冲突](screenshots/conflict.png)

6. 手动编辑 `main.c`，删除 `<<<<<<< HEAD`、`=======`、`>>>>>>> feature` 三行冲突标记，保留 main 分支的句子，然后 `git add main.c && git commit -m "merge feature into main and resolve conflict"`。

   截图 2（解决冲突后）：
   ![解决冲突](screenshots/resolved.png)

7. 最终提交树（`git log --graph --oneline`）清晰可见合并节点：

```
*   020d588 merge feature into main and resolve conflict
|\
| * 858d0dc feature: change print sentence to D
* | 211770b main: change print sentence to C
|/
* a27dc29 feature: change print sentence to B
* 2376ee7 main: change print sentence to A
* fafcab9 feat: complete the TODO in main.c
* 0e56db8 Initial commit
```

## 三、阅读与思考（任务 3，15 分）

### 1. 《Commit Message 规范》（阮一峰）摘要

文章介绍了以 Angular 规范为代表的 Commit Message 写法：格式为 `<type>(<scope>): <subject>`，type 常用 `feat`（新功能）、`fix`（修复 bug）、`docs`（文档）、`style`、`refactor`（重构）、`perf`（性能）、`test`（测试）、`chore`（杂项）等。规范化的好处有三点：① 提供更多历史信息，如 `git log --pretty=format:%s` 一眼可看出每次提交目的；② 可过滤查找（如 `git log --grep feat`）；③ 可以直接从提交记录自动生成 Change log，便于发布时说明与上一版本的差异。

### 2. 《Git Flow 分支控制》摘要

文章介绍 Gitflow 分支管理模型：两条永久分支——`master`（与线上版本一致，只存放发布并打 tag）与 `develop`（日常开发集成分支）；三类临时分支——`feature`（新功能，从 develop 切出、完成后以 `--no-ff` 合并回 develop 并删除）、`release`（提测/发布准备，结束后合并回 master 与 develop）、`hotfix`（线上紧急修复，从 master 切出）。核心理念是"从哪里来，回到哪里去"，让不同分支各司其职，降低多人协作中的代码冲突。

### 3. 为什么要学习 Git？

Git 是当前软件行业事实标准的版本控制工具，它解决的核心问题是"协作"与"回溯"：记录每一次修改、支持随时回到任意历史版本，避免了 `xxx_old.cpp` 式的混乱备份；分支机制让多人、多特性可以并行开发互不干扰；合并与冲突解决机制让团队能在同一份代码上协同工作。对本课程而言，后续每个实验都要通过 Git 交付，掌握它是完成课程的前提；对未来的工程实践而言，Git 是开源协作与团队开发的基础设施，也是开发者必须掌握的基本功。

## 四、建议（可选）

- 建议实验文档补充一份"国内/海外环境 apt 软件源切换"的说明。本次实验在海外环境遇到复旦镜像（mirrors.fudan.edu.cn）DNS 解析失败的问题，切换到官方源 archive.ubuntu.com 后解决。
- 建议文档在 Git 指令部分补充 `git switch` 与 `git checkout` 的对应关系，帮助习惯旧命令的同学平滑过渡。
