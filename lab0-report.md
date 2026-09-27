Lab0: GitLab 实验报告

姓名：龙雪琪　　学号：25303020009　　日期：2026-09-27

仓库地址：https://github.com/lxq-fd-id/ICS_Lab

**一、几个问题的思考**

**关于多人协作的经历**

说实话，我以前没有过正式的多人协同开发经历。之前自己写小程序，都是"改一版就整个文件夹复制一份"的笨办法，目录里堆满了 `xxx_v2`、`xxx_final`、`xxx_final2`，过两天连自己都分不清哪个是最新的。这次实验第一次让我完整走了一遍"两个分支各改一处 → 合并报冲突 → 手动解决"的过程，我才真正理解版本控制对协作意味着什么：它不会替你消除分歧，但能把分歧明明白白地摆出来，让你安全地处理它——这比两个人各改各的、最后默默覆盖掉对方的代码要可靠太多了。

**为什么要有"暂存-提交"两步？**

我一开始也觉得多此一举，自己写东西直接提交不就行了。做完实验回头想，这个设计其实很巧妙：Git 把文件分成了工作区、暂存区和仓库三层，暂存区像是仓库门口的一个"缓冲区"。

它解决的最核心的问题，是把"我正在改什么"和"我决定记录什么"分开。实际写代码时经常是改了很多地方、想改的东西五花八门，如果没有暂存区，要么一次提交把无关的改动全混进去，历史变得没法看；要么只能小心翼翼地逐文件提交。有了 `git add`，就能把一次提交精确地组装成"只做一件事"的节点，提交前还能用 `git status`、`git diff --cached` 检查一下要入库的内容。另外暂存区也让切换分支变得安全——改到一半的东西可以先放着，不用担心半成品丢或者被别的操作冲掉。

**git branch 和 git branch -a 的区别**

`git branch` 只列本地分支；`git branch -a`（`--all`）会额外列出远程跟踪分支，也就是形如 `remotes/origin/main` 的那些。远程跟踪分支可以理解为"本地记录的、远程仓库上次同步时的状态"，clone 或 fetch 的时候才会更新。

这次实验里有个很直观的例子：我执行 `git branch -a`，看到本地有 `main` 和 `feature`，而 `remotes/origin/main` 还停在最早的 `Initial commit`——因为我本地已经提交了 7 次但还没推。如果没有 `-a`，我就完全看不出远程仓库和自己差了多少。

**二、实验过程记录**

**环境与配置**

实验文档说本学期大部分实验都在 Linux 上做，所以我选择了 WSL（Ubuntu）而不是直接在 Windows 上操作。环境就绪后配置了 Git 身份（`user.name` 用了 lobster，邮箱和 GitHub 注册邮箱一致，这样提交才能关联到我的账户），生成了 ed25519 密钥并添加到 GitHub，用 `ssh -T git@github.com` 验证通过——选 SSH 是为了以后推送不用反复输密码。

**建仓库、改 main.c**

在模板仓库页面点 Use this template 建了自己的 `ICS_Lab` 仓库，`git clone` 到本地。模板里的 `main.c` 有一个 `// @TODO`，我把它改成打印自己的句子，顺手补了 `return 0;`：

```c
#include <stdio.h>

int main()
{
    // print a sentence you want
    printf("Hello, world! This is lobster's Lab0 submission.\n");
    return 0;
}
```

用 `make` 编译运行正常（也第一次搞明白了 Makefile 在干什么），`make clean` 清理编译产物，然后 `git add main.c && git commit`。提交信息我特意按 `feat: xxx` 的规范写的，这个习惯是任务 3 读那篇文章时学到的。

**分支与冲突（最有意思的部分）**

这一步按文档要求走了完整流程：在 main 上把打印句子改成句子 A 提交，然后 `git switch -c feature` 切到新分支，把**同一行**改成句子 B 提交，再切回 main 执行 `git merge feature`。

第一次合并居然没冲突，只是 Fast-forward——因为 feature 只是领先于 main，两个分支根本没分叉，git 直接把 main 指针推到了 feature 的位置。文档说这种情况不用回滚，继续提交直到冲突出现就行，于是我让两边真正分叉：main 上又提交了句子 C，feature 上又提交了句子 D，再 merge 时冲突如期而至：

```
Auto-merging main.c
CONFLICT (content): Merge conflict in main.c
Automatic merge failed; fix conflicts and then commit the result.
```

打开 `main.c` 能看到那段著名的冲突标记：

```c
<<<<<<< HEAD
    printf("Hello from the main branch again! Resolving Lab0 conflict.\n");
=======
    printf("Hello from the feature branch again! Feature keeps its sentence.\n");
>>>>>>> feature
```

`<<<<<<<` 和 `=======` 之间是当前分支（HEAD）的版本，`=======` 和 `>>>>>>>` 之间是 feature 分支的版本——git 不知道该留哪个，就把决定权交回给人。我删掉三行标记、保留其中一句，`git add` 后提交，合并完成。`git log --graph` 里能看到一个典型的"分叉再汇合"的合并节点，挺有成就感的。

（截图 1：冲突时的 git status；截图 2：解决后干净的 main.c 和提交树）

![遇到冲突](screenshots/conflict.png)

![解决冲突](screenshots/resolved.png)

**三、阅读两篇文章的收获**

**《Commit Message 规范》（阮一峰）**：文章讲的是目前最流行的 Angular 规范，提交信息写成 `<type>(<scope>): <subject>` 的格式，type 有 `feat`、`fix`、`docs`、`refactor`、`perf`、`test`、`chore` 这些。好处是让 `git log` 扫一眼就能看出每次提交在干嘛、可以按类型过滤，还能自动生成 Change log。我的理解是，提交信息本质上是给"这次改动为什么存在"留的索引，写得规范一点，等于在给未来的自己省时间。

**《Git Flow 分支控制》**：这篇文章把分支划分得很清楚：`master` 和 `develop` 两条永久分支（前者永远和线上版本一致，后者做日常集成），`feature`、`release`、`hotfix` 三类临时分支各有各的使命，原则是"从哪里来，回到哪里去"。看完再回头看自己这次实验的 feature 分支，就能体会为什么团队喜欢"一个功能一个分支"了——隔离得干净，合回来才可控。

**为什么值得学 Git**：往小里说，它是个人写代码的"后悔药"和"时光机"，改坏了能回退，想对比随时 diff，不用再靠复制文件夹来备份；往大里说，它是多人协作的地基，GitHub 上几乎所有开源项目都跑在 Git 上，是这行的基本功。对我们来说更实际的是，后面每个实验都要靠它交作业，现在不学会，后面会很难受。

**四、踩过的坑**

1. 装编译工具时 `apt update` 报 `Could not resolve 'mirrors.fudan.edu.cn'`，查了一下是软件源里配了复旦镜像而当时网络解析不了这个域名，把源换成官方 archive.ubuntu.com 后就正常了。
2. 第一次 merge 没冲突（原因见上文），一开始有点慌，后来发现这反而是理解"冲突只发生在分叉之后"的好机会。
3. 一开始不知道从 Windows 怎么打开 WSL 里的文件，后来用 `\\wsl$\Ubuntu\home\longxq\ICS_Lab` 或者在 VSCode 里直接打开 WSL 目录就方便了。

**五、一点总结**

这次实验把"配置环境 → SSH → 建仓库 → 克隆 → 修改提交 → 分支 → 合并冲突 → 写报告 → 推送"整条链路完整走了一遍，收获最大的是暂存区的设计思想和亲手制造并解决一次冲突。以前这些名词只是听说过，现在是真正用过一遍了。希望后续实验还能多来几次这样的动手环节。
