## Git主要命令汇总

Git详细教程主要参照[廖雪峰的官网](https://www.liaoxuefeng.com/wiki/896043488029600)，致谢。如有收获，请打赏原作者。

---

	$ git init	#初始化，生成.git目录

将文件提交到版本库，分两步进行：

	$ git add <file>	#将file添加到暂存区，可以多次添加
	$ git commit -m "<introduction>"	#将文件提交到版本库

版本回退功能：

	$ git status	#查看仓库当前状态，如文件是否有更改，是否准备提交等
	changes not staged for commit	#文件修改后尚未提交到暂存区
	changes to be committed	#暂存区中的文件尚未提交到版本库

	$ git diff <file>	#查看file的改动情况

	$ git log	#查看历史记录（复杂版）
	$ git log --pretty=oneline	#单行显示不同的版本信息
	$ git reflog	#查看执行的所有命令

	$ git reset --hard HEAD^	#返回上一版本
	$ git reset --hard <commit_id>	#返回指定版本
	
撤销修改

	文件尚未add到暂存区：
	$ git checkout -- <file>	#恢复到与版本库一致的状态，即撤销对文件所作的修改

	文件已经add到暂存区，分两步进行撤销：
	$ git reset HEAD <file>	#将撤销暂存区的修改，即恢复到尚未add的状态
	$ git checkout -- <file>	#彻底删除变动，即恢复到版本库状态

	注：若文件add到暂存区后又进行了修改，可用 git checkout -- <file> 将文件恢复到与暂存区一致的状态

删除文件

	本地文件删除以后，想从版本库还原：
	$ git checkout -- <file>	#从版本库中还原文件

	确实要从版本库中删除文件，分两步进行：
	$ git rm <file>	#删除文件
	$ git commit -m "remove file_name"	#提交信息

	注：文件从版本库中删除后，仍可以通过reset命令找回，但还是需要慎重操作。

分支管理

	$ git checkout -b dev	#创建新分支dev，-b 表示创建并切换

	$ git branch	#列出所有分支，当前分支前标注*
	文档内容修改后提交，将提交到当前分支，不会影响其他分支

	$ git checkout master	#切换到分支master（主分支）

	$ git merge dev	#将分支dev的工作成果合并到当前分支（如 master）上，此时使用fast-forward模式，不保存分支信息
	$ git merge --no-ff -m "<MERGE INFO>" dev	#这一合并命令可保存分支信息（但是分支删除后不能恢复）

	$ git branch -d dev	#dev上的工作成果合并到当前分支后，可删除dev分支

	$ git branch -D <branch>	#强制删除未经合并的分支

	注：使用分支dev完成任务，将任务合并至主分支master，删除分支dev。与在主分支master上直接工作的效果相同，但更为安全。

新版Git提供的分支命令（未测试，当前使用的版本中无相关参数）

	$ git switch -c dev  #创建并切换到新分支dev

	$ git switch master  #切换到主分支master

解决合并中的冲突

	当分支dev中的文件修改内容与主分支master中有冲突，比如在两个分支中对同一文件进行了不同修改，在merge时会出现冲突，此时打开文件，可看到<<< === >>>标记的内容，手动更新后重新提交即可（即在master分支下提交最终版本）

	$ git log --graph --pretty=oneline --abbrev-commit	#用可视化的方式查看分支的合并情况

提交部分工作（如修复bug）

	当前情境：正在dev上进行工作，需将另一文件中的bug修复后提交到master

	方法一：切换到master，修复bug后提交，再返回dev
	$ git stash	#将当前未add、未commit的工作储藏起来
	$ git checkout master	#切换到master进行bug修复
	创建新分支，修改文件，提交，返回master合并分支，删除新分支
	$ git checkout dev	#返回dev
	$ git cherry-pick <fix_bug_id>	#在dev上重复一遍bug修复过程，避免重复劳动。如果内容有冲突，手动修改后重新提交即可
	$ git stash list	#查看存储的工作
	$ git stash pop	#恢复之前的工作，同时删除stash的内容
	
	方法二：直接在dev上进行修复、提交，切换到master复制提交内容，再返回dev工作
	修复、add、commit（记录commit_id）
	$ git stash
	$ git checkout master
	$ git cherry-pick <commit_id>	#在master上复制bug的修复内容
	$ git checkout dev
	$ git stash pop	


	恢复stash内容时亦可以通过两步法：
	$ git stash	apply stash@{<id>}	#恢复stash内容
	$ git stash drop	#删除stash的内容

标签管理

	$ git tag <tag_name>	#在最新提交的commit上标注标签
	$ git tag -a <tag_name>	-m "<introduction>" <commit_id>	#在指定的commit上标注标签，并创建说明文字
	$ git tag	#查看已有标签
	$ git show <tag_name>	#查看指定标签的信息
	$ git tag -d <tag_name>	#删除本地标签
	$ git push origin <tag_name>	#将指定标签推送到远程
	$ git push origin --tags	#将所有尚未推送的标签一次性推送
	推送的标签位于GitHub的releases标签下

	删除已推送到远程的标签，分两步：
	$ git tag -d <tag_name>	#删除本地标签
	$ git push origin :refs/tags/<tag_name>	#删除远程标签

Git配置

	$ git config --global color.ui true	#让Git显示颜色
	$ git config --global alias.st status	#创建别名，用st代表status
	$ git config --global alias.last 'log -1'	#用last命令来显示最后一次提交信息

	--global表示配置针对当前用户，不加这一参数将只针对当前仓库起作用
	每个仓库的Git配置文件位于 .git/config 文件中
	当前用户的Git配置文件位于用户主目录下的 .gitconfig 文件中 

	在推送过程中忽略指定类型的文件：
	创建 .gitignore 文件，推送到Git，后续提交任务将忽略相应的文件，相关配置文件可参阅 https://github.com/github/gitignore
	$ git check-ignore -v <file>	#当文件add有误时，可以用这个命令查看原因
	$ git add -f <file>	#强制 add 指定文件

远程仓库（GitHub）

	情景1：已有本地仓库learngit：
	在GitHub上新建同名仓库，与本地关联
	$ git remote add origin <github_links>
	$ git push -u origin master	#将本地仓库的所有内容推送到远程库中，参数 -u 只在初次关联时使用

	情景2：先创建远程仓库：
	在GitHub上新创建仓库gitskills，再克隆一个本地库
	$ git clone <github_links>
	$ git push origin master	#本地修改文件后，可以直接推送到远程库中

在实际开发过程中的分支管理原则：

	1. master为主分支，应当非常稳定，仅用于发布新版本，需时刻与远程库同步
	2. 日常工作应在dev上进行，新版本确定后，发布到master上，也需与远程库同步，方便团队其他成员在上面开展工作
	3. 团队合作时，每个成员从dev上建立自己的分支，将更新合并到dev上，这些分支可根据需要决定是否推送到远程

Git使用的协议：

	1. git协议：git@github.com:USERNAME/REPOSITORY.git 速度快，推荐
	2. ssh协议：https://github.com/USERNAME/REPOSITORY.git 速度慢

团队合作

	$ git remote	#查看远程库的信息
	$ git remote -v	#查看远程库的详细信息，fetch为抓取地址，push为推送地址

	情境：以团队成员的身份，参与到一个现有项目中：
	$ git clone <github_links>	#从远程库克隆一个仓库，默认只能看到master分支
	$ git checkout -b dev origin/dev	#将远程库的dev分支创建到本地（最好创建相同的分支名称）
	之后可以修改、add、commit、push，参与项目合作

	情境：向远程库推送任务时，出现冲突（远程分支比本地更新，如已有其他成员对同一任务提交了修改），此时需先抓取最新的提交，在本地合并后，再次推送
	$ git pull	#从远程库抓取最新的提交版本
	如果抓取失败，可能是由于没有指定本地dev分支与远程origin/dev分支的链接（no tracking information）
	$ git branch --set-upstream-to=origin/dev dev	#设置origin/dev与dev之间的链接
	如果抓取成功，但是提示合并过程中有冲突，需要手动解决文件中的冲突信息，重新提交后，推送到远程

	$ git rebase	#将本地未推送的分支提交历史整理成直线

参与开源项目，如[bootstrap](https://github.com/twbs/bootstrap)

	访问bootstrap项目主页，点击“fork”，在自己的账号下克隆一个bootstrap仓库，以保证拥有推送修改的权限
	$ git clone git@github.com:USERNAME/bootstrap.git	#从自己的账号下克隆项目到本地
	本地修改或添加新功能后，可以推送到自己的仓库
	如果希望官网库接受修改，需要在GitHub发起 pull request

---

后记

本文适用于对Git比较熟悉的人员复习或临时查找命令时使用，不建议作为初学者的学习材料。初学者还请按照原作者的介绍，一步一步实际操作，1-2日便可掌握。切勿偷懒。
