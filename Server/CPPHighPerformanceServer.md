# 基本组成
- 日志系统：设计服务器首先就要设计好日志系统，方便统计和调试
- 协程库封装：将异步操作封装成库，方便程序员以写同步的方式写异步程序
- socket函数库开发：网络开发
- HTTP协议开发
- 分布式协议

# 日志系统
## log4j
log4jj是一个基于Java的日志记录工具，这里仿照其框架进行开发：
- LogEvent类：记录每一项日志的详细信息（文件名，行号，线程id，协程id等）
- LogLevel类：定义不同的日志等级（DEBUG，INFO，WARN，ERROR等）
- logger类：定义日志类别（框架级，业务级）
- appender类：日志输出（输出到命令行或者文件，也可以选择输出到log server）
- formatter类：日志格式（由日志时间决定）

## Logger类
Logger准确地说就是当现在程序需要输出一个日志的时候，只要依据当前的情况（等级和事件）调用一个logger类中的类成员就可以生成日志了，一般并不需要再调用appender等类。

可以将一个logger类的一个对象想象成一个“日志本”，调用其中的函数就是往这个日志本里加入新的日志条目，并且依据当前对象的设定，当前日志本只记录某几种级别的日志，且可以固定输出到某些地方（命令行或问价）。

