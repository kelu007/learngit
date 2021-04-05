# Git 常用命令

## 基本命令

* 初始化一个Git仓库，使用`git init`命令。

* 添加文件到Git仓库，分两步：

1. 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件；
2. 使用命令`git commit -m <message>`，完成。

## 时光机穿梭

* 要随时掌握工作区的状态，使用`git status`命令。
* 如果`git status`告诉你有文件被修改过，用`git diff <file>`可以查看修改内容。

### 版本回退

* `HEAD`指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令`git reset --hard commit_id`。
* 穿梭前，用`git log`可以查看提交历史，以便确定要回退到哪个版本。
* 要重返未来，用`git reflog`查看命令历史，以便确定要回到未来的哪个版本。

### 工作区和暂存区

如下图所示，stage 即为暂存区。

![Working&StageSpace](imgs/Working&StageSpace.png)

### 管理修改

每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中。

### 撤销修改

* 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git restore file`。

* 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git restore --staged <file>`，就回到了场景1，第二步按场景1操作。

* 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考 版本回退 一节，不过前提是没有推送到远程库。

### 删除文件

命令`git rm`用于删除一个文件。如果一个文件已经被提交到版本库，那么你永远不用担心误删，但是要小心，你只能恢复文件到最新版本，你会丢失**最近一次提交后你修改的内容**。

## 远程仓库

### 添加远程库

* 要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`。

* 关联一个远程库时必须给远程库指定一个名字，`origin`是默认习惯命名。

* 关联后，使用命令`git push -u origin master`第一次推送 master 分支的所有内容至远程库 origin。

  此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改。

* 删除远程库：`git remote rm <name>` ，删除前，建议先用`git remote -v`查看远程库信息。

  此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到GitHub，在后台页面找到删除按钮再删除。

### 从远程库克隆

要克隆一个仓库，首先必须知道仓库的地址，然后使用`git clone address`命令克隆。

Git支持多种协议，包括`https`，但`ssh`协议速度最快。

## 分支管理

### 创建与合并分支

* 查看分支：`git branch`

* 创建分支：`git branch <name>`

* 切换分支：`git checkout <name>`或者`git switch <name>`

* 创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

* 合并某分支到当前分支：`git merge <name>`

* 删除分支：`git branch -d <name>`

### 解决冲突

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用`git log --graph`命令可以看到分支合并图。

### 分支管理策略

合并分支时，加上`--no-ff`参数就可以用普通模式合并，Git 就会在 merge 时生成一个新的 commit，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。

```
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
 
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master) merge with no-ff
|\  
| * f52c633 (dev) add merge
|/  
*   cf810e4 conflict fixed
...
```

### Bug 分支

* 修复 bug 时，我们会通过创建新的 bug 分支（如 issue-101）进行修复，然后合并，最后删除。

* 当手头工作（如 dev 分支）没有完成时，先把工作现场`git stash`一下，然后去 master 分支修复 bug，修复后，回到 dev 分支再`git stash pop`或`git stash apply stash@{number}`，恢复之前的工作现场。可以用`git stash list`查看所做的 stash 列表。

* 在 master 分支上修复的 bug，想要合并到当前 dev 分支，可以用`git cherry-pick <commit number>`命令，把 bug 提交的修改“复制”到当前分支，避免重复劳动。

### Feature 分支

* 开发一个新feature，最好新建一个分支。

* 如果要丢弃一个没有被合并（merge）过的分支，可以通过`git branch -D <name>`强行删除。

### 多人协作

**多人协作**的工作模式通常是这样：

1. 首先，可以试图用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

**抓取**：

* 当你的小伙伴从远程库 clone 时，默认情况下，你的小伙伴只能看到本地的`master`分支。现在，你的小伙伴要在`dev`分支上开发，就必须创建远程`origin`的`dev`分支到本地，于是他用这个命令创建本地`dev`分支：

  ```
  $ git checkout -b dev origin/dev
  ```

  现在，他就可以在`dev`上继续修改，然后，时不时地把`dev`分支`push`到远程。

**小结**：

* 查看远程库信息，使用`git remote -v`；
* 查看远程分支，使用`git branch -a`；
* 本地新建的分支如果不推送到远程，对其他人就是不可见的；
* 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
* 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
* 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
* 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。

### Rebase

* rebase操作可以把本地未push的分叉提交历史整理成直线。
* rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

## 标签管理

### 创建标签

* 命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id。

* 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息。

* 命令`git tag`可以查看所有标签。

  注意：标签总是和某个 commit 挂钩。如果这个 commit 既出现在 master 分支，又出现在 dev 分支，那么在这两个分支上都可以看到这个标签。

### 操作标签

* 命令`git push origin <tagname>`可以推送一个本地标签。
* 命令`git push origin --tags`可以推送全部未推送过的本地标签。
* 命令`git tag -d <tagname>`可以删除一个本地标签。
* 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。

## 自定义 Git

### 忽略特殊文件

* 忽略某些文件时，需要编写`.gitignore`。

* 需要找出来到底哪个规则写错了，可以用`git check-ignore`命令检查：

  ```
  $ git check-ignore -v App.class
  .gitignore:3:*.class	App.class
  ```

  Git会告诉我们，`.gitignore`的第3行规则忽略了该文件，于是我们就可以知道应该修订哪个规则。

* `.gitignore`文件本身要放到版本库里，并且可以对`.gitignore`做版本管理！

忽略文件的原则是：

1. 忽略操作系统自动生成的文件，比如缩略图等；
2. 忽略编译生成的中间文件、可执行文件等，也就是如果一个文件是通过另一个文件自动生成的，那自动生成的文件就没必要放进版本库，比如Java编译产生的`.class`文件；
3. 忽略你自己的带有敏感信息的配置文件，比如存放口令的配置文件。

