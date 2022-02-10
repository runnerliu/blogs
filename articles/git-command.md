## Git 常用命令

### git add

`git add` 命令将文件内容添加到索引（将修改添加到暂存区）。也就是将要提交的文件的信息添加到索引库中。

#### 描述

将要提交的文件的信息添加到索引库中（将修改添加到暂存区），以准备为下一次提交分段的内容。 它通常将现有路径的当前内容作为一个整体添加，但是通过一些选项，它也可以用于添加内容，只对所应用的工作树文件进行一些更改，或删除工作树中不存在的路径了。

该命令可以在提交之前多次执行。它只在运行 `git add` 命令时添加指定文件的内容; 如果希望随后的更改包含在下一个提交中，那么必须再次运行 `git add` 将新的内容添加到索引。

#### 示例

```
git add .             # 将所有修改添加到暂存区
git add *             # Ant风格添加修改
git add *Controller   # 将以Controller结尾的文件的所有修改添加到暂存区
git add Hello*        # 将所有以Hello开头的文件的修改添加到暂存区
git add Hello?        # 将以Hello开头后面只有一位的文件的修改提交到暂存区 例如:Hello1.txt,HelloA.java
```

### git commit

`git commit` 命令用于将更改记录提交到存储库。将索引的当前内容与描述更改的用户和日志消息一起存储在新的提交中。

#### 描述

`git commit` 命令将索引的当前内容与描述更改的用户和日志消息一起存储在新的提交中。

要添加的内容可以通过以下几种方式指定：

1. 在使用 `git commit` 命令之前，通过使用 `git add` 对索引进行递增的“添加”更改（注意：修改后的文件的状态必须为`added`）；
2. 通过使用 `git rm` 从工作树和索引中删除文件，再次使用 `git commit` 命令;
3. 通过将文件作为参数列出到 `git commit` 命令（不使用 `--interactive` 或 `--patch` 选项），在这种情况下，提交将忽略索引中分段的更改，而是记录列出的文件的当前内容（必须已知到Git的内容）；
4. 通过使用带有 `-a` 选项的 `git commit` 命令来自动从所有已知文件（即所有已经在索引中列出的文件）中添加“更改”，并自动从已从工作树中删除索引中的 `rm` 文件 ，然后执行实际提交；
5. 通过使用 `--interactive` 或 `--patch` 选项与 `git commit` 命令一起确定除了索引中的内容之外哪些文件应该是提交的一部分，然后才能完成操作。

如果提交后立即发现错误，可以使用 `git reset` 命令恢复。

#### 示例

```
git commit -m "the commit message" # git add 存储到暂存区之后将内容提交
```

### git push

`git push` 命令用于将本地分支的更新，推送到远程主机。

#### 描述

使用本地引用更新远程引用，同时发送完成给定引用所需的对象。可以在每次推入存储库时，通过在那里设置挂钩触发一些事件。当命令行不指定使用 `<repository>` 参数推送的位置时，将查询当前分支的 `branch.*.remote` 配置以确定要在哪里推送。 如果配置丢失，则默认为`origin`。

#### 示例

```
git push origin master # 将本地的master分支推送到origin主机的master分支。如果master不存在，则会被新建。
git push origin # 将当前分支推送到origin主机的对应分支。如果当前分支只有一个追踪分支，那么主机名都可以省略。
git push -u origin master # 将本地的master分支推送到origin主机，同时指定origin为默认主机。
git push --all origin # 将所有本地分支都推送到origin主机。
```

### git clone

`git clone` 命令将存储库克隆到新目录中。

#### 描述

将存储库克隆到新创建的目录中，为克隆的存储库中的每个分支创建远程跟踪分支，并从克隆检出的存储库作为当前活动分支的初始分支。

#### 示例

```
git clone <版本库的网址>
如：git clone http://github.com/jquery/jquery.git
```

该命令会在本地主机生成一个目录，与远程主机的版本库同名。

```
git clone <版本库的网址> <本地目录名>
```

该命令可指定不同的目录名。

### git fetch

`git fetch` 命令用于从另一个存储库下载对象和引用。

#### 描述

从一个或多个其他存储库中获取分支或标签以及完成其历史所必需的对象。远程跟踪分支已更新，需要将这些更新取回本地，这时就要用到`git fetch` 命令。

#### 示例

更新远程代码到本地仓库：

```
方式一：
1. 查看远程仓库：git remote -v
2. 从远程origin仓库的master分支获取最新版本到本地：git fetch origin master
3. 比较本地仓库和远程仓库的区别：git log -p master.. origin/master
4. 远程仓库和本地仓库的合并：git merge origin/master

方式二：
1. 查看远程仓库：git remote -v
2. 从远程的origin仓库的master分支下载到本地并新建一个分支temp：git fetch origin master:temp
3. 比较本地仓库和远程仓库的区别：git diff temp
4. 合并temp分支到master分支：git merge temp
5. 删除temp分支：git branch -d temp
注意：如果该分支没有合并到主分支会报错，可以用以下命令强制删除 git branch -D <分支名>
```