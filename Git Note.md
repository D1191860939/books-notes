


1.  在Git版本控制之下，修改文件的操作是在工作区，通过git add 命令将文件就挪动到了暂存区，通过git commit命令就将其挪动到版本库，最后通过git push推送远程仓库
    
2.  工作区和暂存区的切换：
    

-   git add: 工作区 -> 暂存区
    
-   git rm --cached <file> : 暂存区 -> 工作区
    

3.  用户名和邮箱的设置（user.name "xialihao" user.email "[xlh1191860939@163.com](mailto:xlh1191860939@163.com)"）
    

-   git config --global: 全局的系统文件
    
-   Git config --system: 系统的设置
    
-   git config --local: 当前版本库的
    

4.  git rm 与 rm：
    

-   git rm：它实际上会包含两步，第一步删除文件；第二步将被删除的文件纳入到暂存区（stage，index）
    
-   rm：只有git rm的第一步
    

此时，若想恢复被删除的文件，需要进行两个动作：a. 将删除的文件从暂存区恢复到工作区；b. 将工作区中的修改丢弃掉

与此类似，git mv与mv的区别也是一样的，都是重命名或者移动操作，只不过前者会将重命名的状态纳入到暂存区中

5.  git commit -amend "xxx": 修正提交日志
    
6.  git help xxx : 同man 命令
    
7.  分支删除：
    

-   git branch -d ：如果待删除的分支已经合并到其他分支，那么此时可以使用-d来删除分支
    
-   git branch -D ：如果待删除的分支并没有合并到其他分支，那么此时是不允许通过-d来删除分支；如果此时你已经确定要删除该分支，那么可以使用-D强制删除
    

8.  丢弃修改：
    

-   git checkout -- <file>：此时checkout的含义是为了丢弃工作区的修改，使之同之前暂存区的状态一致（丢弃掉相对于），相当于用暂存区的内容来覆盖工作区
    
-   git reset HEAD <file>：丢弃暂存区的修改，使之挪动到工作区，即将版本库中的内容来覆盖暂存区
    

总结起来而言：如果某个文件的改动已经提交到暂存区，此时要想将文件状态恢复到上一次commit提交状态，那么我们需要两步操作：

-   通过git reset HEAD <file>：将提交到暂存区的改动恢复到工作区
    
-   通过git checkout -- <file>：将工作区的改动直接丢弃
    

9.  git checkout <commit_id> 与git reset <commit_id>之间的区别：
    

-   checkout挪动的是HEAD指针所只指向的位置，使之处于游离（detach）的状态
    
-   reset不仅挪动了当前HEAD指针所指向的commit点，还挪动了当前分支所指向的commit点
    

10.  分支重命名：git branch -m <oldBranchName>  <newBranchName>
    
11.  git stash


-   pop/apply：类似于stack的pop和peek操作
    
-   drop：删除某个保存点
    
-   list：显示保存列表
    
-   clear：清空保存列表
    

12.  git diff


-   直接 git diff：比较的是工作区与暂存区的内容，源文件是暂存区，目标文件是工作区
    
-   git diff <commit_id>：比较的是特定提交与工作区之间的差别
    
-   git diff HEAD：比较的是最新的提交与工作区之间的差别
    
-   git diff --cached：比较的是版本库与暂存区之间的差别

13. pull = fetch + merge
14. 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTgyNTQ3MjA0NF19
-->
