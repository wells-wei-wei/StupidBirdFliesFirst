<!--
 * @Author: your name
 * @Date: 2020-05-21 12:03:46
 * @LastEditTime: 2020-05-29 19:39:43
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Command\git.md
--> 
# git命令

## 基本的创建和修改流程
1. 自己创建仓库时先在GitHub的网页上新建一个仓库。或者在本地使用：
   ```
   git init
   ```
2. 将新建的仓库克隆到本地
   ```
   git clone 仓库的网址
   ```
3. 修改后使用：
   ```
   git status
   ```
   这个命令可以查看哪些文件有修改以及增加减少了哪些文件
4. github一般不能上传100m以上的文件，所以这些大文件需要写一个.gitignore文件来忽略他们，这时可以使用：
   ```
   find . -type f -size +100M
   ```
   来筛选出大于100m的文件，然后将他们写入.gitignore文件
5. 使用
   ```
   git add XXX
   ```
   来跟踪某个文件，或者可以直接使用：
   ```
   git add .
   git add -A
   ```
   来跟踪所用文件
6. 使用
   ```
   git commit -m "update"
   ```
   将修改假如本地仓库

   也可以使用：
   ```
   git commit -am 'message'
   ```
   将上面的add和commit合二为一
7. 使用
   ```
   git push
   ```
   将修改上传至github。也可以使用：
   ```
   git push origin XXX
   ```
   添加到指定的分支

如果是多人项目，有人有修改或者有人在网页上修改的话，需要先使用
```
git pull
```
把修改的拉进本地仓库

其他命令：

8. 查看提交日志：
   ```
   git log
   ```
9. 创建分支：
   ```
   git branch XXX
   ```
   创建的新分支的代码一般是来自于master的，所以，比如你创建了新分支test，那么test分支的代码是和master的代码是一样的。

   我们还可以使用
   ```
   git branch
   ```
   查看现在一共有哪些分支以及目前的开发处于哪条分支。

10.  查看本地和远程所有分支：
   ```
   git branch -a
   ```
11. 切换分支：
   ```
   git checkout XXX
   ```
   这样就切换到了XXX分支。然后我们再到XXX分支进行功能的开发工作。

12. 创建分支并且切换分支:
   ```
   git checkout -b XXX
   ```
   这条命令就是执行了前面的两条分支，git branch XXX和git checkout XXX，创建并且直接切换到XXX分支，这个命令的好处在于，当你需要进行新的功能开发的时候，你直接创建新分支，然后直接切换了，就可以直接开搞了。

13. 合并分支：
   ```
   git merge
   ```
14. 删除本地分支
   ```
   git branch -D XXX
   ```
   当我们完成了功能开发，且合并到了master的时候，我们就可以删除我们当前的分支

15. 删除远程分支：
   ```
   git push origin --delete XXX
   ```
   删除远程分支属于危险操作，如果权限不合理，可能会出现大问题。建议：git branch -a 查看所有分支，再进行操作。

16. 版本回退
   ```
   git reset --hard XXX
   ```
   这个命令使用需要注意，会把当前分支的代码全部回退到以前的一个版本，不可逆转，需要谨慎使用。这个命令虽然不太常用，但是，当出现大的问题的时候，却能发挥很大的作用，直接回退到一个以前的版本。