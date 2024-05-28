# git常用命令
`git config` 可以设置git的用户信息，第一次使用的时候
```shell
 git config --global user.name 设置用户名
 git config --global user.email 设置用户的邮箱
```
# 仓库的创建
git的本地仓库有三个文件存储区域，分别是“工作区”“暂存区”和“版本库”

# 仓库的回退
git有三种回退方法，分别是软、硬、混合模式回退版本
![三种不同的回退模式](vx_images/45742315266922.png "三种不同的回退模式" =654x)
软回退仅删除仓库内保存的保本，硬回退则会删除所有的工作区和暂存区的内容，而默认的回退会删除仓库内的版本和暂存区内的文件，如果使用hard模式误删除了也不用担心，git有相应的操作日志，可以找回失去的版本

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