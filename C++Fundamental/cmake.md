<!--
 * @Author: your name
 * @Date: 2020-06-06 19:26:09
 * @LastEditTime: 2020-06-07 09:13:59
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\C++Fundamental\cmake.md
--> 
# cmake

这里先引用一个例子，了解初步的CmakeLists.txt的编写过程：
```
PROJECT (HELLO)
SET(SRC_LIST main.c)
MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})
MESSAGE(STATUS "This is SOURCE dir "${HELLO_SOURCE_DIR})
ADD_EXECUTABLE(hello SRC_LIST)
```

## PROJECT
PROJECT指令的语法是
```
project(<PROJECT-NAME> [<language-name>...])
```
关于cmake的语法，其中用<>括起来的是必须要写的，而且名字需要自己起，用[]括起来的是可写可不写的，如果是像这种写法[\<language-name\>...]表示如果这一部分需要写的话，那就也需要自己起个名字，如果是[PARENT_SCOPE]这种写法的话，表示它是一个选项，不用自己起名，要么写PARENT_SCOPE要么就不写。

就是指定本工程的名字以及指定工程支持的语言，但是语言是可以忽略的，一般就按照上面的一样写一个工程名就行。例如上面的例子的工程名就是HELLO。

另外这个命令同时默认了cmake中的常量CMAKE_PROJECT_NAME，值是自己设定的工程名。

还默认了PROJECT_SOURCE_DIR，\<PROJECT-NAME\>_SOURCE_DIR，其中的\<PROJECT-NAME\>也是一开始设定的工程名，这两个变量的值完全一样，是当前CmakeLists.txt所在的目录。

最后还默认了PROJECT_BINARY_DIR, \<PROJECT-NAME\>_BINARY_DIR，其中的\<PROJECT-NAME\>也是一开始设定的工程名，这两个变量的值完全一样，是执行build时的目录（就比如OpenCV中要新建一个build文件夹再进入再执行cmake ..，这时的PROJECT_BINARY_DIR就是这个build文件夹的地址）。

## SET
SET 指令的语法是：
```
set(<variable> <value>... [PARENT_SCOPE])
```
SET命令的主要作用就是设定变量的值，包括自己设计的变量和cmake中的一些常量。第一个\<variable\>表示这个变量的名字，后面的\<value\>表示具体值，如果\<value\>不止一个的话就会将前面的\<variable\>设定成列表。

后面的PARENT_SCOPE表示一个选项，如果没有它就是一个普通的设定语句，带上的话就是还制定了定义域，将在当前作用域上方的作用域中设置变量。每个新目录或函数都会创建一个新作用域。此命令会将变量的值设置到父目录或调用函数中。

这里就是将main.c这个值放进了SRC_LIST的变量之中。

## MESSAGE
message的语法为：
```
message([<mode>] "message text" ...)
``` 
message的作用就是在运行时打印出信息，后面的双引号括起来的是将要打印出的信息，可以打印出好几段，上面例子中的MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})就是在打印This is BINARY dir 之后还要打印HELLO_BINARY_DIR的值，这里的\${}表示从这个变量中取出这个值。

前面的mode表示打印的模式，有以下几种：
- FATAL_ERROR 表示打印将会是错误信息，打印完了就会停止运行
- SEND_ERROR 表示打印将会是错误信息，打印完了会继续运行但是不会生成
- WARNING 表示打印的是警告信息，不会停止
- AUTHOR_WARNING 同上
- DEPRECATION 弃用，暂时没看懂，只有在CMAKE_ERROR_DEPRECATED或者CMAKE_WARN_DEPRECATED被使能时才可以用
- (none) or NOTICE 打印重要信息
- STATUS 没有什么特殊意义，只是打印一段信息
- VERBOSE 打印详细信息
- DEBUG 打印详细信息
- TRACE 暂时没看懂

## ADD_EXECUTABLE
添加一个可执行文件构建目标，其实就是生成一个最后的可执行文件，文件名就是命令的第一个参数。

## cmake_minimum_required
表示当前需要的cmake最低版本：
```
cmake_minimum_required(VERSION 3.10)
```
表示需要最低3.10版本。

## find_package
find_package的主要目的是自动地找到依赖库，它会在某些目录中找到与名字相对应的config文件，在config文件中找到需要的_INCLUDE_DIRS变量和_LIBS变量，这样才能知道编译文件的时候这些模块的头文件和链接库该去哪找。