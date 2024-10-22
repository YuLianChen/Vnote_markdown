# git常用命令
`git config` 可以设置git的用户信息，第一次使用的时候
```shell
 git config --global user.name 设置用户名
 git config --global user.email 设置用户的邮箱
```
# 仓库的创建
git的本地仓库有三个文件存储区域，分别是“工作区”“暂存区”和“版本库”
1. 使用git init创建本地仓库
2. 使用git add . 添加全部更改文件到暂存区
3. 使用git commit . 提交所有暂存区文件到版本库（要添加版本变化描述，不然不能提交）
# git和远端仓库的同步
```shell
1. 获取最新的远程更新
git fetch origin

2. 切换到你的工作分支（例如 main）
git checkout main

3. 合并远程更新
git merge origin/main

4. 解决冲突（如果有）
 编辑冲突文件，解决冲突后
git add <解决冲突的文件>
git commit -m "解决合并冲突"

5. 推送合并后的更新到远程仓库
git push origin main
```
# 仓库的回退
git有三种回退方法，分别是软、硬、混合模式回退版本
![三种不同的回退模式](vx_images/45742315266922.png "三种不同的回退模式" =654x)
软回退仅删除仓库内保存的保本，硬回退则会删除所有的工作区和暂存区的内容，而默认的回退会删除仓库内的版本和暂存区内的文件，如果使用hard模式误删除了也不用担心，git有相应的操作日志，可以找回失去的版本
```shell
1. 放弃工作区修改的内容
git checkout -- filepathname //放弃修改某个文件
例如： git checkout -- readme.md
git checkout .  //放弃所有修改的文件
git restore . //放弃所有修改的文件
```


# diff 查看版本差异
![版本差异比较](vx_images/247094415259591.png "diff版本差异比较" =658x)
- git diff 命令是比较工作区和暂存区的内容区别
- git diff HEAD 比较工作区和版本库的内容区别
- git diff --cached 比较暂存区和版本库中的内容
- git diff <版本1> <版本2>可以比较两个版本的之间的区别
- git diff <1> <2> file 可以只比较1和2之间这一个文件的区别

git 中HEAD表示当前版本，HEAD~表示上一个版本，HEAD~2，表示上上个版本，后面数字可以表示前几个版本
![git diff的简单用法](vx_images/317914821256146.png "git diff的简单用法" =658x)

# git 从版本库中删除文件
![git rm删除文件](vx_images/241975321251900.png "git rm删除文件" =658x)
# gitignore 忽略文件
 git要忽略哪些文件，将要忽略的文件写入.gitignore文件下并放置于git目录下。
 ![git要忽略哪些文件](vx_images/337875721269780.png "git要忽略哪些文件" =658x)
 
![.gitignore的匹配规则](vx_images/109710623240464.png =658x)
# 本地仓库和远程仓库同步
先创建github的密匙，使用ssh方式同步仓库，其中公钥的内容要复制下来，上传到GitHub，私有的密钥则谁都不能给。
![本地密匙的生成与远程仓库的使用](vx_images/564970800258891.png =658x)

# 本地仓库和远程仓库关联
将本地已有的仓库和github上已有的仓库进行关联
![本地和远程仓库关联](vx_images/540184409240467.png "本地和远程仓库关联" =658x)
git push -u 是 git push --set-upstream 的简写形式，它的作用是将本地分支与远程分支建立追踪关系，并且推送代码到远程仓库。

使用这个命令的典型场景是，当你创建了一个新的本地分支并且希望将它推送到远程仓库，同时设置该分支与远程对应分支关联。这样，下一次你可以直接使用 git push 或 git pull，Git 会自动知道该与哪个远程分支进行交互。

具体步骤如下：

你在本地创建了一个新分支并做了修改。
你使用 git push -u origin <branch-name> 将本地分支推送到远程仓库，并同时设置追踪关系。

''' shell
git push -u origin my-feature-branch
'''
之后，my-feature-branch 本地分支就与远程 origin/my-feature-branch 建立了追踪关系，后续你只需使用 git push 或 git pull，而无需指定远程分支名称。
# 常见好用的GUI工具

# 在VScode中使用插件管理git
![vscode中文件字母表示的含义](vx_images/429855912258893.png =568x)

# git的分支管理
git创建的分支可以用于更改、debug代码，但又不会影响到主线的稳定，等分支开发完毕再合并到主分区
```shell
 git log --graph --oneline --decorate 查看分支图
```
![分支的使用方式](vx_images/262911014246760.png =568x)
# git分支冲突合并
![git冲突合并](vx_images/421571615266926.png "git冲突合并" =568x)

# git 回退和rebase
如果在主分支上使用rebase，则把要接回的分支从分叉点接回后再接主分支上的内容
![不同的分支上使用rebase](vx_images/149030921259595.png "不同的分支上使用rebase" =568x)

# git 提交日志查看
查看历史提交版本：
1.git log 查看历史所有版本信息
2.git log -x 查看最新的x个版本信息
3.git log -x filename查看某个文件filename最新的x个版本信息（需要进入该文件所在目录）
4.git log --pretty=oneline查看历史所有版本信息，只包含版本号和记录描述

回滚版本：
1.git reset --hard HEAD^，回滚到上个版本
2.git reset --hard HEAD^~2，回滚到前两个版本
3.git reset --hard xxx(版本号或版本号前几位)，回滚到指定版本号，如果是版本号前几位，git会自动寻找匹配的版本号
4.git reset --hard xxx(版本号或版本号前几位) filename，回滚某个文件到指定版本号（需要进入该文件所在目录）
