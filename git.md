###git教程
**工作区 -> 仓库区域[暂存区 和 版本区]**

##### 初始化
**配置用户信息**
>git config --global user.name "He Youkang"
>git config --global user.email "kangkang2109@gmail.com"

*git 每次提交的时候都会加上这些信息，如果你想在一个项目单独使用不同的user，可以通过在git config user.name "Smartisan"*

**创建版本库**
把当前目录变成可以管理的仓库
>git init


#### 本地
**基本命令**
>git status
>git log
>git log --pretty=oneline


**添加/撤销 暂存区**
>git add file 添加文件至暂存区
>git reset HEAD file 将暂存区的文件放回到工作区


**版本回滚**
>git reset --hard HEAD^
>git revert HEAD 撤销前一次 commit

*reset 和 revert 区别在于reset只是改变HEAD指针回滚，而revert相当于创建一个新的commit，内容回滚*
*标注：HEAD 代表当前版本，HEAD^代表上一个版本， 每个版本都对应这一个id，可以通过id回滚到指定版本
a版本添加了1.txt文件进入缓存区，当版本回滚后，再回滚到a时，1.txt文件消失，因为回滚时会清空缓存区*

**撤销修改**
>git checkout -- file

*（**将工作区的文件还原到仓库区域的文件**）当文件被修改，如果文件已经添加到版本中，则文件撤销到版本中的file内容
如果文件只是添加到了暂存区，则文件撤销到暂存区中的file内容*

**暂存区提交到版本区**
>git commit -m "message log"	
>git commit -s


#### 远程协助
**将本地仓库提交到远程服务器**
本地Git仓库和GitHub仓库之间的传输是通过SSH加密，所以需要SSH Key
1. 创建SSH Key

>cmd： ssh-keygen -t rsa -C "youremail@example.com"
*此时会生成.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥*

2. 在github中添加ssh key
id_rsa.pub文件的内容(公钥) 添加进入Github中
3. 提交到远程仓库

>git remote add origin git@github.com:kangkang2109/test.git
>git push -u origin master

	origin：远程仓库名，master：主分支
*由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令git push origin master。*

**远程仓库拉到本地**
> git clone //拉取master
> git clone -b <branch_name> //拉取指定分支
> git checkout -b dev origin/dev //拉取远程仓库dev到本地
> git pull //把最新的提交更新到本地
> git fetch origin //获取远程代码

**合并分支**
> git merge master 是一个合并操作，会将两个分支的修改合并在一起，默认操作的情况下会提交合并中修改的内容
> git rebase master 并没有进行合并操作，只是提取了当前分支的修改，将其复制在了目标分支的最新提交后面

**分支操作**
> git branch
> git branch <name>	创建分支
> git checkout <name> 切换分支
> git checkout -b <name> 创建+切换分支
> git merge <name> 合并某分支到当前分支(无记录)
> git branch -d <name> 删除分支
> git merge --no-ff -m <commit_content> <branch_name>	合并某分支到当前分支(有记录)

**冲突解决**
>当合并时冲突产生，git会要求用户重新commit冲突文件，并且在冲突文件中标注哪些部分是冲突部分，直到没有冲突时候才能成功合并。
> git rebase origin/master 合并冲突 ，解决冲突后，add文件到暂存区，不用commit命令，直接调用git rebase --continue或者选择直接覆盖--skip

**保留当前工作区状态**
> git stash //保存当前工作区状态
> git stash list //显示所有stash
> git stash apply //恢复不删除
> git stash drop //删除记录
> git stash pop //恢复并删除
> git git stash apply <stash_id> //恢复指定记录




git pull = git fetch + git rebase

git add
git commit -s
	update：。。。	
	Ticket: bug编号
git push smartsianos HEAD:refs/for/master
gerrit-push-review


git reset --hard 00
git branch -a
git pull
