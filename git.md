# 版本控制
记录文件内容变化，记录修改历史，查看历史版本，合并分支操作

# git工作机制
- 工作区->(add)->暂存区->(commit)->本地库->(push)->远程库

# git命令
- 初始配置
  - git config --global user.name 用户名
  - git config --global user.email 邮箱
- 初始化本地库
  - git init
- 查看库状态
  - git status 
- 添加到暂存区
  - git add xxx  
- 暂存文件提交本地库
  - git commit -m "version_info" xxx
- 查看历史版本
  - git reflog
  - git log
- 版本穿梭(通过7位版本号标记，可以回滚和重做)
  - git reset --hard xxxxxxx

# 分支
- 查看分支
  - git branch -v
- 创建分支
  - git branch xxxx
- 切换分支
  - git checkout xxxx
- 合并分支**到当前分支**
  - git merge xxxx
  - 当存在冲突时(多个文件在相同位置都发生了修改)，需要手动修改被合并的文件中的冲突区域，并重新暂存和提交。此次提交不需要追加文件名

# git团队协作
- 常见远程仓库
