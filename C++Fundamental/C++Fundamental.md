<!--
 * @Author: your name
 * @Date: 2020-05-04 21:58:09
 * @LastEditTime: 2020-05-04 22:53:50
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \StupidBirdFliesFirst\C++Fundamental\C++Fundamental.md
 -->

# C++基础知识

## C++与C语言的不同
- C++是面向对象语言，C语言是面向过程语言
- C++比C语言多了类、模板等部分
- C++有87个标准库，C有29个标准库

## C++的程序创建过程（linux）
1. 编写源代码，使用各种文本编辑器编写；
2. 预处理，处理源代码文件中的宏定义、条件编译文件、头文件等，最终生成一个没有宏定义，没有编译指令，头文件均被展开的文件；
    ```
    gcc -E hello.c -o hello.i
    ```
3. 编译，检测语法的规范性，检测有无语法错误，然后将源码生成汇编代码；
    ```
    gcc -S hello.i -o hello.s
    ```
4. 汇编，将汇编文件生成机器指令文件，即目标文件；
    ```
    gcc -c hello.s -o hello.o
    ```
5. 链接，将目标文件与其他代码链接起来，例如各种库或者启动代码，最终生成可执行文件，
   ```
   gcc hello.o -o hello
   ```
   库分为动态链接库和静态链接库两种：
   - 静态库是在链接过程中把库文件全部加入到可执行文件中，在运行时就不再需要库文件了，后缀为.a
   - 动态库在编译链接时并没有把全部代码都加入到可执行文件中去，在程序执行时再加载库，后缀为.so

    使用以上两种库时在gcc命令中刚加入-L表示查找路径，加入-l表示库名，之后将生成可执行文件。

gcc和g++其实都是可以编译c和cpp文件，但是gcc会把文件内容当成c语言，g++会把内容当成C++语言，所以这里同样需要注意。