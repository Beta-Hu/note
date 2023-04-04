# 版本控制
记录文件内容变化，记录修改历史，查看历史版本，合并分支操作

# git工作机制
- 工作区->(add)->暂存区->(commit)->本地库->(push)->远程库

# git命令
- 初始配置
  -  git config --global user.name 用户名
  - git config --global user.email 邮箱
- 初始化本地库
  - git init
- 查看库状态
  - git status 
    - 红色: 未被追踪
    - 绿色: 已暂存
- 添加到暂存区
  - git add xxx  
- 暂存文件提交本地库
  - git commit -m "version_info" xxx
- 查看历史版本
  - git reflog
  - git log
- 版本穿梭(通过7位版本号标记，可以回滚和重做)
  - git reset --hard xxxxxxx
- 
