
创建一个android studio在线依赖compile   http://blog.csdn.net/qfanmingyiq/article/details/53389361

  Git教程	  https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000  

- HEAD指向的版本就是当前版本,上一个版本就是HEAD^, 上上一个版本就是HEAD^^，或者HEAD~1

指令 | 功能
---|---
git init | 创建一个空仓库
git clone | 从远程库克隆
git add <file> | 提交修改到暂存区
git add . |  提交被修改的和新建的文件，不包括被删除的文件
git add -u --update | 更新所有改变的文件，即提交所有变化的文件
git add -A --all | 提交已被修改和已被删除文件，但是不包括新的文件
git commit -m "说明" | 把暂存区的所有修改提交到分支
git status | 查看工作区状态
git diff | 查看修改内容
git log | 查看提交历史
git reflog | 查看命令历史
git reset --hard commit_id | 版本回退
git reset --soft commit_id | 版本回退到commit前的状态
git checkout -- file | 丢弃工作区的修改（git add 之前）
git reset HEAD file | 撤销掉暂存区的修改(git add之后回到add之前)
git rm | 删除文件
git branch | 查看分支
git branch <name> | 创建分支
git checkout <name> | 切换分支
git checkout -b <name> | 创建+切换分支
git merge <name> | 合并某分支到当前分支
git branch -d <name> | 删除分支
git push origin 分支名 | 提交到分支
