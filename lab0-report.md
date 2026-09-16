# Lab0：GitLab 实验报告

> 课程：计算机系统基础（2026 年秋季学期）
>
> 实验：Lab0：GitLab
>
> GitHub：[`speedwal-spec/lab0-gitlab`](https://github.com/speedwal-spec/lab0-gitlab)

## 1. 实验目标

本实验的目标是熟悉 Git 的基本工作流、分支管理和 GitHub 远程仓库操作，并通过一次真实的分支合并冲突理解 Git 的协作模型。完成的本地仓库位于 `lab0-gitlab`，最终代码和本报告均在 `main` 分支。

## 2. 文档问题回答

### 2.1 多人协同开发经历与分工

我目前没有可以引用的正式多人协同开发项目经历。本次实验用两个本地分支模拟两位开发者：`main` 表示稳定版本维护者，`feature` 表示新功能开发者。实际团队中可以按功能模块拆分任务，每个人从共同的开发分支创建自己的 feature 分支，通过提交记录、代码审查和合并请求协作；遇到同一位置的修改时，由相关开发者共同确认最终行为并解决冲突。

### 2.2 为什么要有“暂存—提交”两个步骤

暂存区提供了一个明确的提交边界。工作区中可能同时存在多个目的不同的修改，例如一个功能实现、一个格式调整和一份文档更新。先用 `git add` 选择本次要提交的文件或修改，再用 `git commit` 固化版本，可以带来以下好处：

1. 一次提交只表达一个相对完整的意图，历史更容易阅读和回退。
2. 可以在提交前检查暂存内容，避免把调试代码或无关文件一并提交。
3. 可以把同一批工作区改动拆成多个逻辑提交，便于代码审查和定位问题。
4. 提交只记录已经确认的快照，而工作区仍然可以继续进行下一项工作。

因此，暂存区不是多余的中转站，而是“准备提交的内容”和“正在编辑的内容”之间的控制层。

### 2.3 `git branch` 与 `git branch -a` 的区别

`git branch` 默认列出本地分支，例如当前仓库会显示 `main` 和 `feature`；带有 `*` 的分支是当前分支。

`git branch -a` 列出所有分支，包括本地分支和远程跟踪分支，例如 `remotes/template/main`。远程跟踪分支是本地对远程分支状态的记录，不等于当前工作区中的本地分支。查看远程分支前通常需要先执行 `git fetch` 或 `git pull` 获取最新信息。

我在本仓库中实际观察到的区别是：前一个命令只显示 `main` 和 `feature`，后一个命令还会显示 `remotes/template/HEAD` 和 `remotes/template/main`。这让我意识到，本地分支、远程仓库和远程跟踪分支是三个相关但不同的概念。

参考：[Git 官方文档：git-branch](https://git-scm.com/docs/git-branch)

## 3. Git 基本操作与实验步骤

### 3.1 初始化与完成 TODO

我从课程模板仓库克隆了初始代码。第一次查看远程仓库时只有默认名称 `origin`；为了提醒自己这是只用于获取课程模板的上游仓库，我将它改名为 `template`，避免误把实验内容推回课程模板仓库：

```bash
git clone https://github.com/ICS-26Fall-FDU/GitLab.git lab0-gitlab
cd lab0-gitlab
git remote rename origin template
git status
git remote -v
```

初始 `main.c` 中的 TODO 是输出一句话。我将其改为：

```c
printf("Every commit tells a story.\n");
return 0;
```

修改后，我先用 `git diff` 确认只改动了目标文件，再完成第一次提交：

```bash
git add main.c
git status
git commit -m "feat(main): complete starter TODO"
```

对应提交为 `cb6790f`。

### 3.2 创建 feature 分支并提交

```bash
git switch -c feature
```

切换后我先运行 `git branch`，确认 `*` 已经出现在 `feature` 前。在 `feature` 分支中，将输出行改为：

```c
printf("Feature branch: experiment safely before release.\n");
```

然后提交：

```bash
git add main.c
git commit -m "feat(main): add feature branch message"
```

对应提交为 `a3f4b4b`。

### 3.3 在 main 分支进行冲突性修改

切回 `main`，并再次用 `git status` 确认当前分支：

```bash
git switch main
git status
```

在 `main` 分支中，将同一行改为另一种内容：

```c
printf("Main branch: keep the released version stable.\n");
```

然后提交：

```bash
git add main.c
git commit -m "feat(main): add stable branch message"
```

对应提交为 `2a4e09d`。由于两个分支都从同一个父提交出发，并且修改了 `main.c` 的同一行，接下来合并时会产生冲突。

### 3.4 合并并解决冲突

在 `main` 分支执行：

```bash
git merge feature
```

Git 报告：

```text
CONFLICT (content): Merge conflict in main.c
Automatic merge failed; fix conflicts and then commit the result.
```

此时 `main.c` 中出现了 `<<<<<<< HEAD`、`=======` 和 `>>>>>>> feature` 三个冲突标记。冲突截图如下：

![Git 合并冲突证据](assets/merge-conflict.png)

为了让图片中的内容仍可搜索和核验，我同时保留了[合并冲突的原始终端文本](assets/merge-conflict.txt)。

我根据两边的意图，保留“feature 工作最终进入稳定 main 分支”的合并语义，将冲突区域改为：

```c
printf("Feature work is merged into the stable main branch.\n");
```

解决后执行：

```bash
git add main.c
git commit -m "merge(main): resolve feature conflict"
```

生成的合并提交为 `1e472cb`。验证结果为工作树干净：

```text
On branch main
nothing to commit, working tree clean
```

并且提交图同时保留了 `main` 和 `feature` 两条历史：

![冲突解决后的提交历史](assets/merge-resolved.png)

对应的可搜索版本见[冲突解决后的原始终端文本](assets/merge-resolved.txt)。从提交图中的分叉、两个父提交和重新汇合可以确认，这不是一次 fast-forward 合并。

### 3.5 连接个人 GitHub 仓库

我一开始先在本地完成了实验，之后才在 GitHub 上从课程模板创建个人仓库。这与文档推荐的“先创建个人仓库、再克隆”顺序不同，因此本地和 GitHub 当时各有一个内容相同、但提交编号不同的模板起点。

我先用 `git bundle` 保存完整本地历史作为备份，再添加个人仓库远程：

```bash
git remote add origin https://github.com/speedwal-spec/lab0-gitlab.git
git fetch origin
```

确认两个起点的文件树完全一致后，我将实验提交重放到个人仓库的初始提交之上，并保留合并结构：

```bash
git rebase --rebase-merges --onto origin/main f67e080 main
```

重放到合并提交时，同一处冲突再次出现。我使用原合并提交中已经确认的最终内容解决冲突，再运行 `git rebase --continue`。最后通过提交图确认 `feature`、`main` 和双父合并提交仍然完整，然后推送两个分支：

```bash
git push -u origin main
git push -u origin feature
```

这次补救让我理解了：工作区文件相同，不代表两条 Git 历史自动相同；下次应先从模板创建个人仓库，再克隆个人仓库开始实验。`rebase` 会改写提交编号，日常协作中不能在未沟通的情况下对共享历史执行这类操作。

## 4. 拓展阅读

### 4.1《Commit message 和 Change log 编写指南》

文章指出，commit message 不只是随手写的一句话，它应该清晰说明本次提交的目的。推荐的 Angular 风格把提交说明分成 Header、Body 和 Footer：Header 使用 `<type>(<scope>): <subject>`，Body 解释修改动机以及与旧行为的差异，Footer 用于记录不兼容变更或关闭 Issue。常见的 `type` 包括 `feat`、`fix`、`docs`、`style`、`refactor`、`test` 和 `chore`；标题应简洁、以动词开头，并避免无意义的句号。

规范化提交信息的价值在于：可以快速浏览历史、按类型筛选提交、自动生成 Change Log，并让代码审查者理解一个提交为什么存在。本实验使用了 `feat(...)` 和 `merge(...)` 风格的提交信息，也是在练习让历史记录具有可读性。

来源：[Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

### 4.2《Gitflow 使用规范》

文章介绍了以长期分支和短期分支配合发布的 Gitflow 工作流。`master`（本课程使用 `main`）保存稳定的线上版本，`develop` 用于日常集成；新功能从 `develop` 创建 `feature` 分支，完成后合并回 `develop`；准备发布时创建 `release` 分支，测试完成后合并到稳定分支并打版本标签；线上紧急问题则从稳定分支创建 `hotfix` 分支，修复后合并回稳定分支和开发分支。

这种模型用分支职责隔离降低协作风险，并通过 `--no-ff` 等策略保留功能或发布分支的边界。它不一定适合所有项目，但展示了“从正确的基线创建分支，完成后回到原来的集成线”的重要思想。

来源：[Gitflow 使用规范](https://www.dafaycoding.com/article/git-gif-flow)

### 4.3 为什么要学习 Git

Git 把代码变化组织成可追踪、可比较、可回退的历史，解决了“改坏后无法恢复”和“多人修改后无法合并”这两类高频问题。更重要的是，Git 让协作过程变得可验证：提交说明表达意图，分支隔离风险，合并记录整合成果，远程仓库保存共享副本。即使是单人项目，分支和提交历史也能帮助自己实验新想法而不破坏稳定版本；在团队项目中，Git 则是代码审查、持续集成和发布流程的基础设施。

## 5. 检查结果

- `main.c` 已完成模板中的 TODO，并已提交。
- 已创建 `feature` 分支，并在 `feature` 与 `main` 上分别提交了对 `main.c` 同一行的不同修改。
- `git merge feature` 已产生真实内容冲突，冲突标记已清理并形成 merge commit。
- `main` 分支最终工作树干净，提交图保留了合并前的两条分支历史。
- 已将实验报告写入 `main` 分支。
- 已用 MinGW GCC 13.2.0 实际完成清理、编译、运行和再次清理，程序正常输出最终合并内容，且没有产生编译警告。完整输出见[编译运行记录](assets/build-verification.txt)。
- 已运行 `git diff --check template/main..HEAD -- . ':!assets/merge-conflict.txt'`、`git fsck --full` 和自动评分等价检查；文本格式、Git 对象和预期输出均符合要求。格式检查只排除了故意保存冲突标记的原始证据文件，实际源码中已经没有冲突标记。

课程模板远程保留为 `template`，个人仓库 [`speedwal-spec/lab0-gitlab`](https://github.com/speedwal-spec/lab0-gitlab) 使用 `origin`。两者分开后，可以在需要时获取模板更新，同时避免向课程模板误推送。

## 6. 建议

后续实验建议保持“先 `git status`，再操作”的习惯；每个提交只完成一个清晰目标；在推送前确认当前分支确实是 `main`，并通过 `git log --graph --oneline --all` 检查提交历史。
