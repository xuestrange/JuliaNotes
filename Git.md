

# Git


这是我第二次学习git...，第一次学了个囫囵吞枣，在使用时出现了很多问题，也不知道怎么解决。这一次参考《ProGit》这本书，侧重于概念理解，尽可能的详细。

## Git管理文件的方式

### 文件快照

如图，Git记录的是文件的**快照**，图中虚线框表示新版本该文件没有发生变化，实线框表示文件有改动，Git并不是记录文件的前后差异，而是每一次提交的快照。 所以当commit记录相当多时，仓库会非常大，例如chromium和AOSP的git仓库可能达到了几百GB。

<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/imgimage-20221029232929622.png" alt="image-20221029232929622" style="zoom:67%;" />

### 文件对象

+   blob: 二进制文件，存储了文件的内容（但不包括文件名、文件路径等）
+   tree: 目录和文件名
+   commit: 记录了对应的tree对象、提交信息（提交人、时间、邮箱等）和父提交的sha-1值

`git add`的过程就是为修改过的文件创建blob对象和tree的过程；而`git commit`则会创建`commit`对象

<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/img/image-20221101161454939.png" alt="image-20221101161454939" style="zoom:50%;" />

上图来源于博客[博客](http://chuquan.me/2022/05/21/understand-principle-of-git/)，描述了Git记录文件的方式，和版本提交的原理。

## 概念

### 工作目录

+ 工作目录添加新文件为Untracked表示该文件尚未被Git追踪
+ `git add`可以将Untracked文件添加到暂存区，但是并没有建立追踪
    + 被commit后,修改该文件会被标记为Modified
    + 被commit后,不修改该文件会被标记为Unmodified(没有标记)
+ 工作目录中的Modified文件需要被再次`git add`后才会被添加到暂存区
+ 当一个文件添加到暂存区后，若再进行修改，则需要重新`add`以更新暂存区中的版本记录，否则`commit`的就是之前`add`的版本

### 暂存区

+ 也叫作index，实际上是快照存放的区域，执行`git add`即生成文件快照

### Git目录（本地仓库）

+ commit对象存放的区域

<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/img/image-20221101162309057.png" alt="image-20221101162309057" style="zoom:67%;" />

### 分支

+   `HEAD`为指针，指向某一次提交记录，工作区内的文件为HEAD指向的提交

+   创建新分支，就是创建一个可以移动的指针，这个指针指向某一个commit


<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/img/image-20221101162349736.png" alt="image-20221101162349736" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/img/image-20221101162540274.png" alt="image-20221101162540274" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/xuestrange/picGoUploader/main/img/image-20221101162607349.png" alt="image-20221101162607349" style="zoom:67%;" />

创建分支就是生成一个新的指针的过程，并不会生成快照，所以速度很快。而切换分支就是将HEAD移动到指定指针的过程，HEAD指向的提交与当前工作区的文件一致（未修改）。

## 常用操作

### 合并分支

+   `merge`：保留分支上的提交，仅在合并时创建一次提交
    +   vscode中，在合并且删除分支后，仍能看到分叉，但分支名丢失；
    +   推送到github上之后，看不到已经被删除的分支，只有main上的提交才能看到

+   `rebase`: 将分支上的提交移动到main分支上，然后合并，看起来就是一条线，更加简洁
    +   注意：只有在本地提交中使用rebase, 不要在已经提交到远程的分支上使用rebase


### 查看变化

+ `git diff`，查看工作目录与暂存区的差异
+ `git diff --cached`，查看暂存区与上一次提交的差异

[git diff的结果怎么看](https://blog.csdn.net/CSDN___LYY/article/details/102555882)

### 删除文件

+   使用`git rm`，不要直接去删除。
    +   直接删除一个文件，git会记录为一次操作，即需要`git add`才可以去追踪这个文件的删除操作，但是问题是，文件都被删除了，我去add谁呢？`git rm`的作用是删除工作目录中的指定文件，并且以后不再跟踪这个文件（在暂存区中记录一次修改（删除了某个文件）），并且包含在下一次提交中。

### 移动文件

+   `git mv`去给文件重命名，和移动文件的位置，原理和删除类似	

### 临时保存文件

**只会保存已经被追踪的文件**

```bash
# 临时保存工作区修改并清除修改
git stash
# 恢复并删除备份
git stash pop 'stash@{1}'
# 恢复备份
git stash apply 'stash@{1}'
```

## 版本回退

### git reset

用于回退版本, 但是会删除之后的提交历史, 虽然也能找回来, 也很麻烦.

```bash
# 回退到HEAD的那个版本, 只回退commit, 不会回退暂存区, 保留工作区修改
git reset --soft HEAD
# 回退到HEAD的那个版本, 回退commit, 回退暂存区, 清空工作区的所有修改, 回退到那个commit的那个版本 
git reset --hard HEAD
# 回退到HEAD的那个版本, 回退commit, 回退暂存区, 保留工作区修改
git reset --mixed HEAD
```

### git checkout

+ `git checkout HEAD`用于将工作目录的所有文件回退到之间提交的某一个版本, 因为他会重置工作区修改, 所以这个命令使用前他会检查你在工作区的修改有没有提交, 如果有没有提交的记录会强制你提交后再`checkout`
+ `git checkout branch`可以用来切换分枝

### git revert

`git revert HEAD`会先找出之前的某一次提交(HEAD), 然后创建一个新的提交, 提交后的内容就是之间的那一次提交的内容(HEAD)

### 总结

+ `git checkout`察看代码最好用, 不会动提交历史, 只会更新工作区;
+ `git revert`和`git reset`用哪个看情况了, 看你之前的提交历史重要不重要了，建议用`git revert`更好一点，毕竟不会丢失提交记录

## 合并分支与解决冲突
考虑一种情况是，我们从github仓库中clone了这个仓库的源代码，然后对其做了修改，现在仓库的原作者更新了代码，我想拉取代码到本地与本地代码合并，如果我们修改部分不重叠，则可以自动合并，如果我们修改了同一行代码，则会出现冲突，git会自动修改这个冲突的文件，**把其中能合并的先合并了，然后把不能合并的用特殊语法标记起来，我需要处理这些有冲突的代码（删除你不想要的和git的标记语法）, 这时冲突就解决完成了**，然后执行以下语句
```bash
git add conflict.txt # 假设confict文件有冲突
git commit -m "handle conflict" # 重新提交
```

参考资料
+ [猴子都懂的git入门](https://backlog.com/git-tutorial/cn/stepup/stepup2_7.html)

## 分支操作

```bash
# 修改本地当前分枝的名字
git branch -m "new name"
# 创建分支
git branch "a name"
# 删除本地分支
git branch -d "branch name"
# 切换分支（分支已存在）
git checkout main # 从当前分支切换到dev分支 
# 创建并切换到分支
git checkout -b "new branch"
# 将当前分枝推送到远程
git push origin -u "new name"
# 删除远程原来的分枝
git push origin --delete "old name"
```

## `.gitignore`文件编写

``` bash
# 忽略项目目录下的所有文件,所有文件夹下的文件(不限深度)
example.txt # ignore all files named example.txt
*.txt # ignore multiple files with the same extension among all folders
example* #ignore multiple files with the same name among all folders


# 指定忽略某个文件夹下的某种文件
/example.txt # ignore single file in root
example/ # ignore the whole folder
example/* #ignore all the files in the folder
**/example.txt # ignore all files named example.txt in everywhere, 与example.txt一样


!example.txt # 这个文件不要忽略
!/foo/**/*.py # 不要忽略根目录的foo文件夹下的py文件以及这个foo文件下的所有子文件夹(递归)的py文件
```

总结

-   `*`表示很所个文件
-   `**`表示很多个文件夹
-   `!`表示这个文件不忽略

`.gitignore`要加入版本控制，上传到远程仓库。