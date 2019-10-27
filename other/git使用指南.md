## git使用指南

### git介绍

> C语言编写的一个分布式版本控制系统

1. git工作原理

![avatar](https://img.mukewang.com/59c31e4400013bc911720340.png)

```

Workspace：工作区
Index / Stage：暂存区
Repository：仓库区（或本地仓库）
Remote：远程仓库

```

2.git和svn区别

+ svn集中式

![avatar](https://www.liaoxuefeng.com/files/attachments/918921540355872/0)

+ git分布式

![avatar](https://www.liaoxuefeng.com/files/attachments/918921562236160/0)


### git常用命令


1. 初始化一个git仓库：git init
2. 克隆仓库：git clone <url>
3. 添加文件（提交到缓存区）：git add <file>
4. 提交文件（提交到HEAD）：git commit -m <message>
5. 推送改动(提交到远程仓库)：git push orgin <branch_name>
6. 查看git仓库状态：git status
7. 查看修改内容：git diff
8. 查看提交日志：git log
9. 回复版本：git reset --hard <commit_id>
10. 创建分支：git branch <branch_name>
11. 切换分支: git check <branch_name> /git switch <branch_name>
12. 查看分支：git branch
13. 合并分支：git merge <branch_name>
14. 删除分支：git branch -d <branch_name>
15. 查看远程库信息：git remore -v
16. 显示标签：git tag


### 参考文档

> https://www.liaoxuefeng.com/wiki/896043488029600
> 
> https://www.bootcss.com/p/git-guide/
 
