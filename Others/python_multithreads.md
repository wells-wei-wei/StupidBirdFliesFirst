# python多线程的局限性
python实际上没有真正意义上的可并行的多线程，主要是python中自带的GIL造成的

## python多线程写法
```python
from threading import Thread
import time

def my_counter():
    i = 0
    for _ in range(100000000):
        i = i + 1
    return True

def main():
    thread_array = {}
    start_time = time.time()
    for tid in range(2):
        t = Thread(target=my_counter)
        t.start()
        thread_array[tid] = t
    for i in range(2):
        thread_array[i].join()
    end_time = time.time()
    print("Total time: {}".format(end_time - start_time))

if __name__ == '__main__':
    main()
```

## GIL
GIL的全程为Global Interpreter Lock，意即全局解释器锁。首先需要明确的一点是GIL并不是Python的特性，它是在实现Python解析器(CPython)时所引入的一个概念。就好比C++是一套语言（语法）标准，但是可以用不同的编译器来编译成可执行代码。有名的编译器例如GCC，INTEL C++，Visual C++等。Python也一样，同样一段代码可以通过CPython，PyPy，Psyco等不同的Python执行环境来执行。像其中的JPython就没有GIL。然而因为CPython是大部分环境下默认的Python执行环境。所以在很多人的概念里CPython就是Python，也就想当然的把GIL归结为Python语言的缺陷。

GIL主要的作用就是每个线程只有获得了这把锁才能运行。因此一个进程中必然同时只能运行一个线程，无论有几个核心。

GIL的出现是为了保证线程安全，解决线程间数据完整性和状态同步问题。在C++中我们可以使用各种锁来保证线程安全，但是python本着方便开发者的思路，让解释器给整个程序的所有线程加了一个超级大锁，也就是GIL。一开始很多人接受了这种加锁方式，当发现这种方式会造成效率低下时，很多库已经对它严重依赖，因此至今也没有完全去除。

在CPython解释器解释执行任何Python代码时，都需要先获得这把锁才行，在遇到I/O操作时会释放这把锁。如果是纯计算的程序，没有I/O操作，解释器会每隔100次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 sys.setcheckinterval 来调整）。
