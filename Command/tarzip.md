<!--
 * @Author: your name
 * @Date: 2020-06-15 15:59:29
 * @LastEditTime: 2020-06-15 16:14:41
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\Command\tarzip.md
--> 
# 压缩命令
## zip命令
在linux中只用zip命令压缩时，如果要文件夹中包含软连接（动态链接库那类）的话，会自动断链。为了保持软连接，需要加上-y命令。
```
zip -r -y lib.zip ./lib
```
## tar
tar是可以直接保存和还原软连接
```
打包命令：tar czvf 1.tar.gz ./*
解包命令：tar xzvf 1.tar.gz
```
其中每一个参数的功能如下：
```
-c 建立新的压缩文件
-v 显示操作过程
-f 指定压缩文件
-z 支持gzip解压文件

-x 从压缩的文件中提取文件
```