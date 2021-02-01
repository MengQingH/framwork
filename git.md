**工作区**：创建了git仓库的目录。

**版本库(Repository)**：工作区的隐藏目录.git文件夹为git的版本库，Git的版本库里存了很多东西，其中最重要的就是称为**stage（或者叫index）的暂存区**，还有Git为我们自动创建的第一个**分支master**，以及**指向master的一个指针叫HEAD**。

**暂存区(Stage)**：存放未提交的更改，向git中添加文件时，分为下面两步执行：
1. git add添加文件，实际上是把文件添加到暂存区中
2. git commit提交文件，实际上是把暂存区中的内容提交到当前分支。

<br> <img src=./git版本库.jpg><br>

cd:
* **git init**：初始化git，在该文件夹内创建一个git仓库
* **git add 文件名**：向暂存区添加文件，可以添加多个
* **git add .** ：向暂存区添加所有文件
* **git commit -m"提交信息"**：提交修改，提交信息不能少
* **git diff 文件名**：查看文件的改动
* git log：显示最近三次的提交日志，加上--pretty=oneline参数输出信息更加整齐
* git reflog：查看git的历史命令
* **git status**：查看git工作区的状态，可以查看工作区的所有文件
* git pull：更新本地分支。

提交与退回：
* git reset --hard head：返回到原来的版本，git中用head代表当前版本，上个版本为head^，上上个版本为head^^，前一百个版本为head~100
* git reset --hard <版本号前几位>：退回该版本
* **git reset -- <file>**：把该文件从暂存区退回到工作区
* **git checkout -- 文件名**：把该文件在工作区的修改全部撤销，有两种情况。总之，就是让这个文件回到最近一次git commit或git add时的状态。
    * 一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
    * 一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

远程库：
* **git remote add origin <address>**：与远程仓库关联，关联之前需要设置SSH
    * git remote add origin git@github.com:<name>/<repostory>
    * git remote add origin https://github.com:<name>/<repostory>
* **git push origin master**：把master分支的修改提交到远程库，在push后加-u参数可以把本地库的分支和远程库分支关联起来，
* git clone git@<地址>:把远程库中的内容克隆到本地
* git remote set-url url：切换远程库的url

分支：
* git branch <name>：在当前内容上创建一条新的分支，分支上的内容和当前分支相同
* git checkout <name>：把当前分支切换为该分支，切换分支后，工作区的内容变成该分支的内容
* git checkout -b <name>：创建一个新的分支并把当前分支切换为该分支
* git branch：列出所有分支，当前分支前有*号
* git branch -d <name>：删除该分支
* git branch -D <name>：强项删除一个没有被合并过的分支
* git merge <name>：合并指定分支到当前分支

* git stash：保存当前分支的工作
* git stash list：查看所有的保存信息
* git stash apply：恢复最新的stash保存的信息
* git stash apply stash@{0}：恢复指定的stash
* git stash drop：删除最新的保存信息
* git stash pop：恢复stash保存的信息并删除

* git cherry-pick 提交号：复制一个其他分支的修改到当前分支

其他：
* git remote -v：查看远程库信息
* git remote rm origin:移除对应的远程库
* git remote add origin url:添加一个远程库

## 关联远程仓库
1. 在git中输入命令创建SSH key：$ ssh-keygen -t rsa -C "youremail@example.com"。可以选择key的创建位置，默认是在用户目录下的.ssh文件夹中，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。
2. 打开github，在setting中的SSH key中输入刚刚创建的id_rsa.pub文件的内容

## idea中git
1. 撤回git操作，已经commit但还没有push的内容：VSC->Git->reset head
    * head^  退回到上次commit
    * head~2   退回到第二次commit之前
    * head id号  退回到指定的commit版本
2. 在当前分支中应用其他分支的提交：在git中选择其他分支，找到要合并的commit，点击樱桃图标合并。


## idea中rebase
rebase可以把多次提交合并到一次commit中。
1. 在要合并的分支中复制最先提交的commit的version number。
2. vcs-git-rebase，勾选Interactive，onto中粘贴1中的version number。点击rebase
3. 把第一个选择为pick，后面的选择为squash，点击start rebasing。