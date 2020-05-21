<!--
 * @Author: your name
 * @Date: 2020-05-21 12:03:46
 * @LastEditTime: 2020-05-21 12:13:21
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Command\git.md
--> 
# git命令

## 基本的创建和修改流程
1. 自己创建仓库时先在GitHub的网页上新建一个仓库
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
   git add -A
   ```
   来跟踪文件
6. 使用
   ```
   git commit -m "update"
   ```
   将修改假如本地仓库
7. 使用
   ```
   git push
   ```
   将修改上传至github

如果是多人项目，有人有修改或者有人在网页上修改的话，需要先使用
```
git pull
```
把修改的拉进本地仓库
