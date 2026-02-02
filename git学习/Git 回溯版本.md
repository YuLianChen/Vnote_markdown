如果现在的代码有问题，但是又很难排查，这个时候回退到之前的版本， 同时保存现在的版本就很有必要，如果你的版本里包含了未提交的文件，那么使用如下方法暂存文件，再回退回去
```shell
git stash push -m "before test"
git branch backup-test
git checkout <commit-id>

#切回测试之前的分支
git checkout backup-test

#舍弃之前的改动
git reset --hard
#把 **未跟踪文件 / 目录** 全部删除，让工作区完全干净
git clean -fd
#切换到主分支
git switch master
#列出当前暂存的内容
git stash list
#恢复当前栈顶的暂存
git stash pop

#查看栈文件改动
git stash show stash@{0}
#查看某一条 stash 的**完整 diff**
git stash show -p stash@{0}
#删除某条
git stash drop stash@{1}
#全部删除
git stash clear
```