---
title: 'Git相关命令'
date: '2018-10-02 13:39:21'
---
git config --list                           查看配置

git config user.name <Name>                 修改用户信息

git remote set-url <url>                    修改远程仓库地址

git add <file> <file>                       向仓库中添加文件

git commit -m<message>                      向仓库中提交文件

git status                                  查看当前仓库状态

git diff <file>                             查看文件前后修改内容

git log                                     从近到远的提交日志
                
git reflog                                  查看命令日志
                                
git reset --hard HEAD^                      回退到上一个版本

git reset --hard HEAD~1                     回退到上一个版本

git checkout --<file>                       撤销修改

git checkout -b dev                         创建dev分支并切换到dev分支

git branch  dev                             创建dev分支

git checkout dev                            切换到dev分支

git branch                                  查看当前分支

git merge dev                               合并dev分支的内容到当前分支上

git branch -d dev                           删除分支dev

git rm <file>                               删除版本库中的文件

git remote add origin git@GitHub.com:SongZhikuan/SongZhikuan.GitHub.io.git 远程仓库连接

git push origin master                      推送到远程仓库

git clone git@github.com:SongZhikuan/SongZhikuan.github.io.git 克隆远程仓库到本地

git remote -v                               查看远程仓库信息

git checkout -b branch-name origin/branch-name  本地创建和远程分支对应的分支

git branch --set-upstream branch-name origin/branch-name  建立本地分支和远程分支的关联

git pull                                    从远程抓取分支      

git push origin dev:dev                     推送本地分支到远程

git checkout -b dev dev                     拉去远程分支到本地

git config http.postBuffer 524288000        设置buffer大小为50M

git config --unset user.name                删除用户信息
