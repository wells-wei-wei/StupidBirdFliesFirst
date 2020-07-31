所有的多线程编程基本都通过c++11 stl中的std:thread
# 线程管理基础
## 启动线程
使用C++线程库启动线程，可以归结为构造std::thread对象：
```
void do_some_work();
std::thread my_thread(do_some_work);
```
为了让编译器识别std::thread类，这个简单的例子也要包含\<thread\>头文件。如同大多数C++标准库一样，std::thread可以用可调用类型构造，将带有函数调用符类型的实例传入std::thread类中，替换默认的构造函数。
```
class background_task
{
public:
  void operator()() const
  {
    do_something();
    do_something_else();
  }
};

background_task f;
std::thread my_thread(f);
```
有件事需要注意，当把函数对象传入到线程构造函数中时，需要避免“最令人头痛的语法解析”。如果你传递了一个临时变量，而不是一个命名的变量；C++编译器会将其解析为函数声明，而不是类型对象的定义。

例如：
```
std::thread my_thread(background_task());
```
这里相当与声明了一个名为my_thread的函数，这个函数带有一个参数(函数指针指向没有参数并返回background_task对象的函数)，返回一个std::thread对象的函数，而非启动了一个线程。

使用在前面命名函数对象的方式，或使用多组括号①，或使用新统一的初始化语法②，可以避免这个问题。

如下所示：
```
std::thread my_thread((background_task()));  // 1
std::thread my_thread{background_task()};    // 2
```

## 线程的等待与否
线程在建立之后需要明确主线程（也就是main或者调用线程的函数）是否要等待线程结束。这个是必须的，因为如果在std::thread对象销毁之前不做决定的话，thread的析构函数会把整个程序都停了。

等待线程执行使用join命令，让其自主运行使用detach命令。

### 不等待
使用detach()会让线程在后台运行，这就意味着主线程不能与之产生直接交互。也就是说，不会等待这个线程结束；如果线程分离，那么就不可能有std::thread对象能引用它，分离线程的确在后台运行，所以分离线程不能被加入。不过C++运行库保证，当线程退出时，相关资源的能够正确回收，后台线程的归属和控制C++运行库都会处理。

通常称分离线程为守护线程(daemon threads),UNIX中守护线程是指，没有任何显式的用户接口，并在后台运行的线程。这种线程的特点就是长时间运行；线程的生命周期可能会从某一个应用起始到结束，可能会在后台监视文件系统，还有可能对缓存进行清理，亦或对数据结构进行优化。另一方面，分离线程的另一方面只能确定线程什么时候结束，发后即忘(fire and forget)的任务就使用到线程的这种方式。

如2.1.2节所示，调用std::thread成员函数detach()来分离一个线程。之后，相应的std::thread对象就与实际执行的线程无关了，并且这个线程也无法加入：
```
std::thread t(do_background_work);
t.detach();
assert(!t.joinable());
```
为了从std::thread对象中分离线程(前提是有可进行分离的线程),不能对没有执行线程的std::thread对象使用detach(),也是join()的使用条件，并且要用同样的方式进行检查——当std::thread对象使用t.joinable()返回的是true，就可以使用t.detach()。

试想如何能让一个文字处理应用同时编辑多个文档。无论是用户界面，还是在内部应用内部进行，都有很多的解决方法。虽然，这些窗口看起来是完全独立的，每个窗口都有自己独立的菜单选项，但他们却运行在同一个应用实例中。一种内部处理方式是，让每个文档处理窗口拥有自己的线程；每个线程运行同样的的代码，并隔离不同窗口处理的数据。如此这般，打开一个文档就要启动一个新线程。因为是对独立的文档进行操作，所以没有必要等待其他线程完成。因此，这里就可以让文档处理窗口运行在分离的线程上。

下面代码简要的展示了这种方法：

清单2.4 使用分离线程去处理其他文档
```
void edit_document(std::string const& filename)
{
  open_document_and_display_gui(filename);
  while(!done_editing())
  {
    user_command cmd=get_user_input();
    if(cmd.type==open_new_document)
    {
      std::string const new_name=get_filename_from_user();
      std::thread t(edit_document,new_name);  // 1
      t.detach();  // 2
    }
    else
    {
       process_user_input(cmd);
    }
  }
}
```
如果用户选择打开一个新文档，需要启动一个新线程去打开新文档①，并分离线程②。与当前线程做出的操作一样，新线程只不过是打开另一个文件而已。所以，edit_document函数可以复用，通过传参的形式打开新的文件。

这个例子也展示了传参启动线程的方法：不仅可以向std::thread构造函数①传递函数名，还可以传递函数所需的参数(实参)。C++线程库的方式也不是很复杂。当然，也有其他方法完成这项功能，比如:使用一个带有数据成员的成员函数，代替一个需要传参的普通函数。

如果不等待线程，就必须保证线程结束之前，可访问的数据得有效性。这不是一个新问题——单线程代码中，对象销毁之后再去访问，也会产生未定义行为——不过，线程的生命周期增加了这个问题发生的几率。

这种情况很可能发生在线程还没结束，函数已经退出的时候，这时线程函数还持有函数局部变量的指针或引用。下面的清单中就展示了这样的一种情况。

函数已经结束，线程依旧访问局部变量：
```
//这种写法就是仿函数的写法，能够让一个类看起来就像是运行了一个函数一样
struct func
{
  int& i;
  func(int& i_) : i(i_) {}
  void operator() ()
  {
    for (unsigned j=0 ; j<1000000 ; ++j)
    {
      do_something(i);           // 1. 潜在访问隐患：悬空引用
    }
  }
};

void oops()
{
  int some_local_state=0;
  func my_func(some_local_state);
  std::thread my_thread(my_func);
  my_thread.detach();          // 2. 不等待线程结束
}                              // 3. 新线程可能还在运行
```
这个例子中，已经决定不等待线程结束(使用了detach()②)，所以当oops()函数执行完成时③，新线程中的函数可能还在运行。如果线程还在运行，它就会去调用do_something(i)函数①，这时就会访问已经销毁的变量。如同一个单线程程序——允许在函数完成后继续持有局部变量的指针或引用；当然，这从来就不是一个好主意——这种情况发生时，错误并不明显，会使多线程更容易出错。

处理这种情况的常规方法：使线程函数的功能齐全，将数据复制到线程中，而非复制到共享数据中。如果使用一个可调用的对象作为线程函数，这个对象就会复制到线程中，而后原始对象就会立即销毁。但对于对象中包含的指针和引用还需谨慎，例子所示。使用一个能访问局部变量的函数去创建线程是一个糟糕的主意(除非十分确定线程会在函数完成前结束)。此外，可以通过join()函数来确保线程在函数完成前结束。

*另外注意这个仿函数的写法*

### 等待
如果需要等待线程，相关的std::thread实例需要使用join()。上述例子中，将my_thread.detach()替换为my_thread.join()，就可以确保局部变量在线程完成后，才被销毁。在这种情况下，因为原始线程在其生命周期中并没有做什么事，使得用一个独立的线程去执行函数变得收益甚微，但在实际编程中，原始线程要么有自己的工作要做；要么会启动多个子线程来做一些有用的工作，并等待这些线程结束。

join()是简单粗暴的等待线程完成或不等待。当你需要对等待中的线程有更灵活的控制时，比如，看一下某个线程是否结束，或者只等待一段时间(超过时间就判定为超时)。想要做到这些，你需要使用其他机制来完成，比如条件变量和期待(futures)。

调用join()的行为，还清理了线程相关的存储部分，这样std::thread对象将不再与已经完成的线程有任何关联。这意味着，只能对一个线程使用一次join();一旦已经使用过join()，std::thread对象就不能再次加入了，当对其使用joinable()时，将返回false。

### 特殊情况下的等待
如果打算等待对应线程，则需要细心挑选调用join()的位置。当在线程运行之后产生异常，在join()调用之前抛出，就意味着这次调用会被跳过。

避免应用被抛出的异常所终止，就需要作出一个决定。通常，当倾向于在无异常的情况下使用join()时，需要在异常处理过程中调用join()，从而避免生命周期的问题。下面的程序清单是一个例子。
```
struct func
{
  int& i;
  func(int& i_) : i(i_) {}
  void operator() ()
  {
    for (unsigned j=0 ; j<1000000 ; ++j)
    {
      do_something(i);           // 1. 潜在访问隐患：悬空引用
    }
  }
};

void f()
{
  int some_local_state=0;
  func my_func(some_local_state);
  std::thread t(my_func);
  try
  {
    do_something_in_current_thread();
  }
  catch(...)
  {
    t.join();  // 1
    throw;
  }
  t.join();  // 2
}
```
代码使用了try/catch块确保访问本地状态的线程退出后，函数才结束。当函数正常退出时，会执行到②处；当函数执行过程中抛出异常，程序会执行到①处。try/catch块能轻易的捕获轻量级错误，所以这种情况，并非放之四海而皆准。

一种方式是使用“资源获取即初始化方式”(RAII，Resource Acquisition Is Initialization)，并且提供一个类，在析构函数中使用join()，如同下面清单中的代码。看它如何简化f()函数。
```c++
class thread_guard
{
  std::thread& t;
public:
  explicit thread_guard(std::thread& t_):
    t(t_)
  {}
  ~thread_guard()
  {
    if(t.joinable()) // 1
    {
      t.join();      // 2
    }
  }
  thread_guard(thread_guard const&)=delete;   // 3
  thread_guard& operator=(thread_guard const&)=delete;
};

struct func
{
  int& i;
  func(int& i_) : i(i_) {}
  void operator() ()
  {
    for (unsigned j=0 ; j<1000000 ; ++j)
    {
      do_something(i);           // 1. 潜在访问隐患：悬空引用
    }
  }
};

void f()
{
  int some_local_state=0;
  func my_func(some_local_state);
  std::thread t(my_func);
  thread_guard g(t);
  do_something_in_current_thread();
}    // 4
```
当线程执行到④处时，局部对象就要被逆序销毁了。因此，thread_guard对象g是第一个被销毁的，这时线程在析构函数中被加入②到原始线程中。即使do_something_in_current_thread抛出一个异常，这个销毁依旧会发生。

在thread_guard的析构函数的测试中，首先判断线程是否已加入①，如果没有会调用join()②进行加入。这很重要，因为join()只能对给定的对象调用一次，所以对给已加入的线程再次进行加入操作时，将会导致错误。

拷贝构造函数和拷贝赋值操作被标记为=delete③，是为了不让编译器自动生成它们。直接对一个对象进行拷贝或赋值是危险的，因为这可能会弄丢已经加入的线程。通过删除声明，任何尝试给thread_guard对象赋值的操作都会引发一个编译错误。想要了解删除函数的更多知识，请参阅附录A的A.2节。

如果不想等待线程结束，可以分离(_detaching)线程，从而避免_异常安全*(exception-safety)问题。不过，这就打破了线程与std::thread对象的联系，即使线程仍然在后台运行着，分离操作也能确保std::terminate()在std::thread对象销毁才被调用。

## 向线程函数传递参数
正常的向线程函数输入参数的方法如下：
```c++
void f(int i, std::string const& s);
std::thread t(f, 3, "hello");
```
如果是传递指针一般情况下是可行的，线程会对传入的指针进行浅拷贝，主线程和新建线程共享当前指针。需要特别要注意下：
```c++
void f(int i,std::string const& s);
void oops(int some_param)
{
  char buffer[1024]; // 1
  sprintf(buffer, "%i",some_param);
  std::thread t(f,3,buffer); // 2
  t.detach();
}
```
这里在子线程的函数参数中使用的是const string的引用，这样就首先在函数中将char*隐式转换为string，然后将string传送到子线程中进行操作

但是如果使用detach函数，还是有可能会产生问题，因为不确定什么时候进行隐式转换，可能在主线程结束后，隐式转换还没有开始，buffer就已经失效了，这样依然会产生不可预料的结果。

解决方法：

在主线程中进行显示转换，这样可以保证一定可以在主线程中进行显示转换：
```c++
void f(int i,std::string const& s);
void not_oops(int some_param)
{
  char buffer[1024];
  sprintf(buffer,"%i",some_param);
  std::thread t(f,3,std::string(buffer));  // 使用std::string，避免悬垂指针
  t.detach();
}
```

还可能遇到相反的情况：期望传递一个引用，但整个对象被复制了。当线程更新一个引用传递的数据结构时，这种情况就可能发生，比如：
```c++
void update_data_for_widget(widget_id w,widget_data& data); // 1
void oops_again(widget_id w)
{
  widget_data data;
  std::thread t(update_data_for_widget,w,data); // 2
  display_status();
  t.join();
  process_widget_data(data); // 3
}
```
虽然update_data_for_widget①的第二个参数期待传入一个引用，但是std::thread的构造函数②并不知晓；构造函数无视函数期待的参数类型，并盲目的拷贝已提供的变量。当线程调用update_data_for_widget函数时，传递给函数的参数是data变量内部拷贝的引用，而非数据本身的引用。因此，当线程结束时，内部拷贝数据将会在数据更新阶段被销毁，且process_widget_data将会接收到没有修改的data变量③。可以使用std::ref将参数转换成引用的形式，从而可将线程的调用改为以下形式：
```c++
std::thread t(update_data_for_widget,w,std::ref(data));
```
还可以传递一个成员函数指针作为线程函数，并提供一个合适的对象指针作为第一个参数：
```c++
class X
{
public:
  void do_lengthy_work();
};
X my_x;
std::thread t(&X::do_lengthy_work,&my_x); // 1
```
这段代码中，新线程将my_x.do_lengthy_work()作为线程函数；my_x的地址①作为指针对象提供给函数。也可以为成员函数提供参数：std::thread构造函数的第三个参数就是成员函数的第一个参数，以此类推。
```c++
class X
{
public:
  void do_lengthy_work(int);
};
X my_x;
int num(0);
std::thread t(&X::do_lengthy_work, &my_x, num);
```

## 转移线程所有权
假设要写一个在后台启动线程的函数，想通过新线程返回的所有权去调用这个函数，而不是等待线程结束再去调用；或完全与之相反的想法：创建一个线程，并在函数中转移所有权，都必须要等待线程结束。总之，新线程的所有权都需要转移。

这就是移动引入std::thread的原因，C++标准库中有很多资源占有(resource-owning)类型，比如std::ifstream,std::unique_ptr还有std::thread都是可移动，但不可拷贝。这就说明执行线程的所有权可以在std::thread实例中移动，下面将展示一个例子。例子中，创建了两个执行线程，并且在std::thread实例之间(t1,t2和t3)转移所有权：
```c++
void some_function();
void some_other_function();
std::thread t1(some_function);            // 1
std::thread t2=std::move(t1);            // 2
t1=std::thread(some_other_function);    // 3
std::thread t3;                            // 4
t3=std::move(t2);                        // 5
t1=std::move(t3);                        // 6 赋值操作将使程序崩溃
```
当显式使用std::move()创建t2后②，t1的所有权就转移给了t2。之后，t1和执行线程已经没有关联了；执行some_function的函数现在与t2关联。

然后，与一个临时std::thread对象相关的线程启动了③。为什么不显式调用std::move()转移所有权呢？因为，所有者是一个临时对象——移动操作将会隐式的调用。

t3使用默认构造方式创建④，与任何执行线程都没有关联。调用std::move()将与t2关联线程的所有权转移到t3中⑤。因为t2是一个命名对象，需要显式的调用std::move()。移动操作⑤完成后，t1与执行some_other_function的线程相关联，t2与任何线程都无关联，t3与执行some_function的线程相关联。

最后一个移动操作，将some_function线程的所有权转移⑥给t1。不过，t1已经有了一个关联的线程(执行some_other_function的线程)，所以这里系统直接调用std::terminate()终止程序继续运行。这样做（不抛出异常，std::terminate()是noexcept函数)是为了保证与std::thread的析构函数的行为一致。

std::thread支持移动，因此在thread_guard中可能会产生一些问题，当thread_guard对象所持有的线程已经被转移了线程的所有权后，thread_guard就不能对线程进行加入或分离。为了确保线程程序退出前完成，下面的代码里定义了scoped_thread类。现在，我们来看一下这段代码：
```c++
class scoped_thread
{
  std::thread t;
public:
  explicit scoped_thread(std::thread t_):                 // 1
    t(std::move(t_))
  {
    if(!t.joinable())                                     // 2
      throw std::logic_error(“No thread”);
  }
  ~scoped_thread()
  {
    t.join();                                            // 3
  }
  scoped_thread(scoped_thread const&)=delete;
  scoped_thread& operator=(scoped_thread const&)=delete;
};

struct func; // 定义在清单2.1中

void f()
{
  int some_local_state;
  scoped_thread t(std::thread(func(some_local_state)));    // 4
  do_something_in_current_thread();
}                                  
```
不过这里新线程是直接传递到scoped_thread中④，而非创建一个独立的命名变量。当主线程到达f()函数的末尾时，scoped_thread对象将会销毁，然后加入③到的构造函数①创建的线程对象中去。而在清单2.3中的thread_guard类，就要在析构的时候检查线程是否"可加入"。这里把检查放在了构造函数中②，并且当线程不可加入时，抛出异常。

std::thread对象的容器，如果这个容器是移动敏感的(比如，标准中的std::vector<>)，那么移动操作同样适用于这些容器。了解这些后，就可以写出类似清单2.7中的代码，代码量产了一些线程，并且等待它们结束。
```c++
void do_work(unsigned id);

void f()
{
  std::vector<std::thread> threads;
  for(unsigned i=0; i < 20; ++i)
  {
    threads.push_back(std::thread(do_work,i)); // 产生线程
  } 
  std::for_each(threads.begin(),threads.end(),
                  std::mem_fn(&std::thread::join)); // 对每个线程调用join()
}
```
我们经常需要线程去分割一个算法的总工作量，所以在算法结束的之前，所有的线程必须结束。清单2.7说明线程所做的工作都是独立的，并且结果仅会受到共享数据的影响。如果f()有返回值，这个返回值就依赖于线程得到的结果。在写入返回值之前，程序会检查使用共享数据的线程是否终止。操作结果在不同线程中转移的替代方案，我们会在第4章中再次讨论。

将std::thread放入std::vector是向线程自动化管理迈出的第一步：并非为这些线程创建独立的变量，并且将他们直接加入，可以把它们当做一个组。创建一组线程(数量在运行时确定)，可使得这一步迈的更大，而非像清单2.7那样创建固定数量的线程。

## 运行时决定线程数量
std::thread::hardware_concurrency()在新版C++标准库中是一个很有用的函数。这个函数将返回能同时并发在一个程序中的线程数量。例如，多核系统中，返回值可以是CPU核芯的数量。返回值也仅仅是一个提示，当系统信息无法获取时，函数也会返回0。

以下程序实现了一个并行版的std::accumulate（累加求和）。代码中将整体工作拆分成小任务交给每个线程去做，其中设置最小任务数，是为了避免产生太多的线程。程序可能会在操作数量为0的时候抛出异常。比如，std::thread构造函数无法启动一个执行线程，就会抛出一个异常：
```c++
template<typename Iterator,typename T>
struct accumulate_block
{
  void operator()(Iterator first,Iterator last,T& result)
  {
    result=std::accumulate(first,last,result);
  }
};

template<typename Iterator,typename T>
T parallel_accumulate(Iterator first,Iterator last,T init)
{
  unsigned long const length=std::distance(first,last);

  if(!length) // 1
    return init;

  unsigned long const min_per_thread=25;
  unsigned long const max_threads=
      (length+min_per_thread-1)/min_per_thread; // 2

  unsigned long const hardware_threads=
      std::thread::hardware_concurrency();

  unsigned long const num_threads=  // 3
      std::min(hardware_threads != 0 ? hardware_threads : 2, max_threads);

  unsigned long const block_size=length/num_threads; // 4

  std::vector<T> results(num_threads);
  std::vector<std::thread> threads(num_threads-1);  // 5

  Iterator block_start=first;
  for(unsigned long i=0; i < (num_threads-1); ++i)
  {
    Iterator block_end=block_start;
    std::advance(block_end,block_size);  // 6
    threads[i]=std::thread(     // 7
        accumulate_block<Iterator,T>(),
        block_start,block_end,std::ref(results[i]));
    block_start=block_end;  // 8
  }
  accumulate_block<Iterator,T>()(
      block_start,last,results[num_threads-1]); // 9
  std::for_each(threads.begin(),threads.end(),
       std::mem_fn(&std::thread::join));  // 10

  return std::accumulate(results.begin(),results.end(),init); // 11
}
```
函数看起来很长，但不复杂。如果输入的范围为空①，就会得到init的值。反之，如果范围内多于一个元素时，都需要用范围内元素的总数量除以线程(块)中最小任务数，从而确定启动线程的最大数量②，这样能避免无谓的计算资源的浪费。比如，一台32芯的机器上，只有5个数需要计算，却启动了32个线程。

计算量的最大值和硬件支持线程数中，较小的值为启动线程的数量③。因为上下文频繁的切换会降低线程的性能，所以你肯定不想启动的线程数多于硬件支持的线程数量。当std::thread::hardware_concurrency()返回0，你可以选择一个合适的数作为你的选择；在本例中,我选择了"2"。你也不想在一台单核机器上启动太多的线程，因为这样反而会降低性能，有可能最终让你放弃使用并发。

每个线程中处理的元素数量,是范围中元素的总量除以线程的个数得出的④。对于分配是否得当，我们会在后面讨论。

现在，确定了线程个数，通过创建一个std::vector<T>容器存放中间结果，并为线程创建一个std::vector<std::thread>容器⑤。这里需要注意的是，启动的线程数必须比num_threads少1个，因为在启动之前已经有了一个线程(主线程)。

使用简单的循环来启动线程：block_end迭代器指向当前块的末尾⑥，并启动一个新线程为当前块累加结果⑦。当迭代器指向当前块的末尾时，启动下一个块⑧。

启动所有线程后，⑨中的线程会处理最终块的结果。对于分配不均，因为知道最终块是哪一个，那么这个块中有多少个元素就无所谓了。

当累加最终块的结果后，可以等待std::for_each⑩创建线程的完成(如同在清单2.7中做的那样)，之后使用std::accumulate将所有结果进行累加⑪。

结束这个例子之前，需要明确：T类型的加法运算不满足结合律(比如，对于float型或double型，在进行加法操作时，系统很可能会做截断操作)，因为对范围中元素的分组，会导致parallel_accumulate得到的结果可能与std::accumulate得到的结果不同。同样的，这里对迭代器的要求更加严格：必须都是向前迭代器，而std::accumulate可以在只传入迭代器的情况下工作。对于创建出results容器，需要保证T有默认构造函数。对于算法并行，通常都要这样的修改；不过，需要根据算法本身的特性，选择不同的并行方式。算法并行会在第8章有更加深入的讨论。需要注意的：因为不能直接从一个线程中返回一个值，所以需要传递results容器的引用到线程中去。另一个办法，通过地址来获取线程执行的结果；第4章中，我们将使用期望(futures)完成这种方案。

当线程运行时，所有必要的信息都需要传入到线程中去，包括存储计算结果的位置。不过，并非总需如此：有时候这是识别线程的可行方案，可以传递一个标识数，例如清单2.7中的i。不过，当需要标识的函数在调用栈的深层，同时其他线程也可调用该函数，那么标识数就会变的捉襟见肘。好消息是在设计C++的线程库时，就有预见了这种情况，在之后的实现中就给每个线程附加了唯一标识符。

## 识别线程
线程标识类型是std::thread::id，可以通过两种方式进行检索。第一种，可以通过调用std::thread对象的成员函数get_id()来直接获取。如果std::thread对象没有与任何执行线程相关联，get_id()将返回std::thread::type默认构造值，这个值表示“没有线程”。第二种，当前线程中调用std::this_thread::get_id()(这个函数定义在\<thread\>头文件中)也可以获得线程标识。

std::thread::id对象可以自由的拷贝和对比,因为标识符就可以复用。如果两个对象的std::thread::id相等，那它们就是同一个线程，或者都“没有线程”。如果不等，那么就代表了两个不同线程，或者一个有线程，另一没有。

线程库不会限制你去检查线程标识是否一样，std::thread::id类型对象提供相当丰富的对比操作；比如，提供为不同的值进行排序。这意味着允许程序员将其当做为容器的键值，做排序，或做其他方式的比较。按默认顺序比较不同值的std::thread::id，所以这个行为可预见的：当a<b，b<c时，得a<c，等等。标准库也提供std::hash<std::thread::id>容器，所以std::thread::id也可以作为无序容器的键值。

std::thread::id实例常用作检测线程是否需要进行一些操作，比如：当用线程来分割一项工作(如下)，主线程可能要做一些与其他线程不同的工作。这种情况下，启动其他线程前，它可以将自己的线程ID通过std::this_thread::get_id()得到，并进行存储。就是算法核心部分(所有线程都一样的),每个线程都要检查一下，其拥有的线程ID是否与初始线程的ID相同。
```c++
std::thread::id master_thread;
void some_core_part_of_algorithm()
{
  if(std::this_thread::get_id()==master_thread)
  {
    do_master_thread_work();
  }
  do_common_work();
}
```
另外，当前线程的std::thread::id将存储到一个数据结构中。之后在这个结构体中对当前线程的ID与存储的线程ID做对比，来决定操作是被“允许”，还是“需要”(permitted/required)。

同样，作为线程和本地存储不适配的替代方案，线程ID在容器中可作为键值。例如，容器可以存储其掌控下每个线程的信息，或在多个线程中互传信息。

std::thread::id可以作为一个线程的通用标识符，当标识符只与语义相关(比如，数组的索引)时，就需要这个方案了。也可以使用输出流(std::cout)来记录一个std::thread::id对象的值。
```c++
std::cout<<std::this_thread::get_id();
```
具体的输出结果是严格依赖于具体实现的，C++标准的唯一要求就是要保证ID比较结果相等的线程，必须有相同的输出。

# 共享数据
当线程在访问共享数据的时候，必须定一些规矩，用来限定线程可访问的数据位。还有，一个线程更新了共享数据，需要对其他线程进行通知。从易用性的角度，同一进程中的多个线程进行数据共享，有利有弊。错误的共享数据使用是产生并发bug的一个主要原因

## C++中使用互斥量
C++中通过实例化std::mutex创建互斥量。一般来说要建立在所有要在线程中使用的函数的可见区域（比如代码块外，类内定义成成员变量等）。mutex的作用并不是给某一个或某几个变量上锁，而是当我使用mutex.lock()的时候，就会去检查是不是有其他的线程函数内已经用过mutex.lock()了，如果发现有人已经用过，即mutex已经被锁住了，那么当前的mutex变量所在的线程就会阻塞，停止活动，直到刚才那个上锁的人解锁了（mutex.unlock()），那么本线程才重新开始。也就是说可以把我们想要上锁的操作放在mutex.lock()和mutex.unlock()之间，这样就能保证永远只有一个线程在进行操作。

不过，不推荐实践中直接去调用lock和unlock成员函数，因为调用成员函数就意味着，必须记住在每个函数出口都要去调用unlock()，也包括异常的情况。C++标准库为互斥量提供了一个RAII语法的模板类std::lock_guard，其会在构造的时候提供已锁的互斥量，并在析构的时候进行解锁，从而保证了一个已锁的互斥量总是会被正确的解锁。下面的程序清单中，展示了如何在多线程程序中，使用std::mutex构造的std::lock_guard实例，对一个列表进行访问保护。std::mutex和std::lock_guard都在<mutex>头文件中声明。
```c++
#include <list>
#include <mutex>
#include <algorithm>

std::list<int> some_list;    // 1
std::mutex some_mutex;    // 2

void add_to_list(int new_value)
{
  std::lock_guard<std::mutex> guard(some_mutex);    // 3
  some_list.push_back(new_value);
}

bool list_contains(int value_to_find)
{
  std::lock_guard<std::mutex> guard(some_mutex);    // 4
  return std::find(some_list.begin(),some_list.end(),value_to_find) != some_list.end();
}

int main ()
{
  std::thread threads[10];
  // spawn 10 threads:
  for (int i=0; i<10; ++i)
    threads[i] = std::thread(add_to_list,i+1);

  for (auto& th : threads) th.join();

  return 0;
}
```
程序中有一个全局变量①，这个全局变量被一个全局的互斥量保护②。add_to_list()③和list_contains()④函数中使用std::lock_guard\<std::mutex\>，使得这两个函数中对数据的访问是互斥的：list_contains()不可能看到正在被add_to_list()修改的列表。

简单总结一些std::mutex：
1. 对于std::mutex对象，任意时刻最多允许一个线程对其进行上锁
2. mtx.lock()：调用该函数的线程尝试加锁。如果上锁不成功，即：其它线程已经上锁且未释放，则当前线程block。如果上锁成功，则执行后面的操作，操作完成后要调用mtx.unlock()释放锁，否则会导致死锁的产生
3. mtx.unlock()：释放锁
4. std::mutex还有一个操作：mtx.try_lock()，字面意思就是：“尝试上锁”，与mtx.lock()的不同点在于：如果上锁不成功，当前线程不阻塞。

lock_guard是非常典型的RAII风格：
```c++
template<typename _Mutex>
    class lock_guard
    {
    public:
      typedef _Mutex mutex_type;

      explicit lock_guard(mutex_type& __m) : _M_device(__m)
      { _M_device.lock(); }//作者注:构造对象时加锁(申请资源),构造函数结束，就可以正常使用资源了

      lock_guard(mutex_type& __m, adopt_lock_t) : _M_device(__m)
      { } // calling thread owns mutex

      ~lock_guard()
      { _M_device.unlock(); }//作者注:析构对象时解锁(释放资源)
      // 作者注:禁用拷贝构造函数
      lock_guard(const lock_guard&) = delete;
      // 作者注:禁用赋值操作符
      lock_guard& operator=(const lock_guard&) = delete;

    private:
      mutex_type&  _M_device;
    };
```

虽然某些情况下，使用全局变量没问题，但在大多数情况下，互斥量通常会与保护的数据放在同一个类中，而不是定义成全局变量。这是面向对象设计的准则：将其放在一个类中，就可让他们联系在一起，也可对类的功能进行封装，并进行数据保护。在这种情况下，函数add_to_list和list_contains可以作为这个类的成员函数。互斥量和要保护的数据，在类中都需要定义为private成员，这会让访问数据的代码变的清晰，并且容易看出在什么时候对互斥量上锁。当所有成员函数都会在调用时对数据上锁，结束时对数据解锁，那么就保证了数据访问时不变量不被破坏。

当然，也不是总是那么理想，聪明的你一定注意到了：当其中一个成员函数返回的是保护数据的指针或引用时，会破坏对数据的保护。具有访问能力的指针或引用可以访问(并可能修改)被保护的数据，而不会被互斥锁限制。互斥量保护的数据需要对接口的设计相当谨慎，要确保互斥量能锁住任何对保护数据的访问，并且不留后门。

传递出了保护数据的引用：
```c++
class some_data
{
  int a;
  std::string b;
public:
  void do_something();
};

class data_wrapper
{
private:
  some_data data;
  std::mutex m;
public:
  template<typename Function>
  void process_data(Function func)
  {
    std::lock_guard<std::mutex> l(m);
    func(data);    // 1 传递“保护”数据给用户函数
  }
};

some_data* unprotected;

void malicious_function(some_data& protected_data)
{
  unprotected=&protected_data;
}

data_wrapper x;
void foo()
{
  x.process_data(malicious_function);    // 2 传递一个恶意函数
  unprotected->do_something();    // 3 unprotected把类内的protected_data数据提取出来并在外界进行使用，这就是在无保护的情况下访问保护数据
}
```
例子中process_data看起来没有任何问题，std::lock_guard对数据做了很好的保护，但调用用户提供的函数func①，就意味着foo能够绕过保护机制将函数malicious_function传递进去②，在没有锁定互斥量的情况下调用do_something()。

这段代码的问题在于根本没有保护，只是将所有可访问的数据结构代码标记为互斥。函数foo()中调用unprotected->do_something()的代码未能被标记为互斥。这种情况下，C++线程库无法提供任何帮助，只能由程序员来使用正确的互斥锁来保护数据。从乐观的角度上看，还是有方法可循的：切勿将受保护数据的指针或引用传递到互斥锁作用域之外，无论是函数返回值，还是存储在外部可见内存，亦或是以参数的形式传递到用户提供的函数中去。

## 接口内的条件竞争问题
假设使用stl中的stack写以下这样的程序：
```c++
stack<int> s;
if (! s.empty()){    // 1
  int const value = s.top();    // 2
  s.pop();    // 3
  do_something(value);
}
```
由于stl中的容器没有线程安全的，所以如果写成多线程的形式，每个线程都对同一个stack实例s进行同样的操作的话，可能出现下面这样的问题：
|  Thread A   | Thread B  |
|  ----  | ----  |
| if (!s.empty);  |   |
|    | if(!s.empty); |
| int const value = s.top();| |
||int const value = s.top();|
|s.pop();||
|do_something(value);	|s.pop();|
||do_something(value);|
|||
也就是说，在两个线程的运行中，线程A首先用s.pop()将顶部元素出栈，这时出栈的肯定是线程A前面int const value = s.top()的这个元素；但是之后线程B也进行了s.pop()，这时出栈的可就不是xianchengB前面int const value = s.top()的那个元素了，因为前面线程A已经出栈一回了，线程B出去的就不是前面输出的顶部那个，而是又下一个元素，这样实际出栈的元素和前面输出的就不是同一个，这就是stl线程不安全的体现。其他例子比如还有链表在添加元素的时候，需要先断链，假如就在这个断链的时候另外一个线程也来插入的话，那肯定是要出问题的。

这种时候首先想到的第一种方法就是给这几部操作都加上锁：
```c++
void stackTest(stack<int> s)
{
  std::lock_guard<std::mutex> guard(some_mutex);    // 3
  if (! s.empty()){    // 1
    int const value = s.top();    // 2
    s.pop();    // 3
    do_something(value);
  }
}
```
这种一般来说是可以的，但是在多线程并发的场景中，每一个锁所掌管的程序区域应该需要尽量小，也就是**锁粒度尽量小**，因为这样可以减少持有锁的时间，降低其他线程的阻塞时间。所以现在就需要在接口层面来解决竞争问题。

>为什么要加锁？加锁是为了防止不同的线程访问同一共享资源造成混乱。
打个比方：人是不同的线程，卫生间是共享资源
你在上洗手间的时候肯定要把门锁上吧，这就是加锁，只要你在里面，这个卫生间就被锁了，只有你出来之后别人才能用。想象一下如果卫生间的门没有锁会是什么样？

>什么是加锁粒度呢？所谓加锁粒度就是你要锁住的范围是多大。
比如你在家上卫生间，你只要锁住卫生间就可以了吧，不需要将整个家都锁起来不让家人进门吧，卫生间就是你的加锁粒度。

>怎样才算合理的加锁粒度呢？
其实卫生间并不只是用来上厕所的，还可以洗澡，洗手。这里就涉及到优化加锁粒度的问题。
你在卫生间里洗澡，其实别人也可以同时去里面洗手，只要做到隔离起来就可以，如果马桶，浴缸，洗漱台都是隔开相对独立的，实际上卫生间可以同时给三个人使用，
当然三个人做的事儿不能一样。这样就细化了加锁粒度，你在洗澡的时候只要关上浴室的门，别人还是可以进去洗手的。如果当初设计卫生间的时候没有将不同的功能区域划分
隔离开，就不能实现卫生间资源的最大化使用。这就是设计架构的重要性。

在接口层面解决问题：
```c++
#include <exception>
#include <memory>
#include <mutex>
#include <stack>

struct empty_stack: std::exception
{
  const char* what() const throw() {
    return "empty stack!";
  };
};

template<typename T>
class threadsafe_stack
{
private:
  std::stack<T> data;
  mutable std::mutex m;

public:
  threadsafe_stack()
    : data(std::stack<T>()){}

  threadsafe_stack(const threadsafe_stack& other)
  {
    std::lock_guard<std::mutex> lock(other.m);
    data = other.data; // 1 在构造函数体中的执行拷贝
  }

  threadsafe_stack& operator=(const threadsafe_stack&) = delete;

  void push(T new_value)
  {
    std::lock_guard<std::mutex> lock(m);
    data.push(new_value);
  }

  std::shared_ptr<T> pop()
  {
    std::lock_guard<std::mutex> lock(m);
    if(data.empty()) throw empty_stack(); // 在调用pop前，检查栈是否为空

    std::shared_ptr<T> const res(std::make_shared<T>(data.top())); // 在修改堆栈前，分配出返回值
    data.pop();
    return res;
  }

  void pop(T& value)
  {
    std::lock_guard<std::mutex> lock(m);
    if(data.empty()) throw empty_stack();

    value=data.top();
    data.pop();
  }

  bool empty() const
  {
    std::lock_guard<std::mutex> lock(m);
    return data.empty();
  }
};
```

## 线程池
```c++
#pragma once
#ifndef THREAD_POOL_H
#define THREAD_POOL_H

#include <vector>
#include <queue>
#include <thread>
#include <atomic>
#include <condition_variable>
#include <future>
#include <functional>
#include <stdexcept>

namespace std
{
#define  MAX_THREAD_NUM 256

//线程池,可以提交变参函数或拉姆达表达式的匿名函数执行,可以获取执行返回值
//不支持类成员函数, 支持类静态成员函数或全局函数,Opteron()函数等
class threadpool
{
    //这里的using的作用与typedef和#define一样，就是给std::function<void()>起个别名
    using Task = std::function<void()>;//①
    // 线程池
    std::vector<std::thread> pool;
    // 任务队列
    std::queue<Task> tasks;
    // 同步
    std::mutex m_lock;

    // 条件阻塞
    std::condition_variable cv_task;
    // 是否关闭提交
    std::atomic<bool> stoped;//②
    //空闲线程数量
    std::atomic<int>  idlThrNum;

public:
    inline threadpool(unsigned short size = 4) :stoped{ false }
    {
        idlThrNum = size < 1 ? 1 : size;
        for (size = 0; size < idlThrNum; ++size)
        {   //初始化线程数量
            pool.emplace_back(//③
            // 工作线程函数
                [this]//④
                {   //只要不关闭提交就一直运行
                    while(!this->stoped)
                    {
                        //定义一个
                        std::function<void()> task;
                        {   // 获取一个待执行的 task
                            std::unique_lock<std::mutex> lock{ this->m_lock };
                            // unique_lock 相比 lock_guard 的好处是：可以随时 unlock() 和 lock()
                            this->cv_task.wait(lock,//⑤
                                [this] {
                                    return this->stoped.load() || !this->tasks.empty();//.load()表示以原子方式加载并返回值
                                }
                            ); // wait 直到有 task
                            if (this->stoped && this->tasks.empty())
                                return;
                            task = std::move(this->tasks.front()); // 取一个 task
                            this->tasks.pop();
                        }
                        idlThrNum--;
                        task();
                        idlThrNum++;
                    }
                }
            );
        }
    }
    inline ~threadpool()
    {
        stoped.store(true);
        cv_task.notify_all(); // 唤醒所有线程执行
        for (std::thread& thread : pool) {
            //thread.detach(); // 让线程“自生自灭”
            if(thread.joinable())
                thread.join(); // 等待任务结束， 前提：线程一定会执行完
        }
    }

public:
    // 提交一个任务
    // 调用.get()获取返回值会等待任务执行完,获取返回值
    // 有两种方法可以实现调用类成员，
    // 一种是使用   bind： .commit(std::bind(&Dog::sayHello, &dog));
    // 一种是用 mem_fn： .commit(std::mem_fn(&Dog::sayHello), &dog)
    template<class F, class... Args>//class... Args是一个参数包，这个参数包中可以包含0到任意个模板参数
    auto commit(F&& f, Args&&... args) ->std::future<decltype(f(args...))>//⑥
    {
        if (stoped.load())    // stop == true ??
            throw std::runtime_error("commit on ThreadPool is stopped.");

        using RetType = decltype(f(args...)); // typename std::result_of<F(Args...)>::type, 函数 f 的返回值类型
        auto task = std::make_shared<std::packaged_task<RetType()> >(
            std::bind(std::forward<F>(f), std::forward<Args>(args)...)
            );    //⑦
        std::future<RetType> future = task->get_future();
        {    // 添加任务到队列
            std::lock_guard<std::mutex> lock{ m_lock };//对当前块的语句加锁  lock_guard 是 mutex 的 stack 封装类，构造的时候 lock()，析构的时候 unlock()
            tasks.emplace(
                [task]()
                { // push(Task{...})
                    (*task)();
                }
            );
        }
        cv_task.notify_one(); // 唤醒一个线程执行

        return future;
    }

    //空闲线程数量
    int idlCount() { return idlThrNum; }

};

}

#endif
```
```c++
#include "threadpool.h"
#include <iostream>

void fun1(int slp)
{
    printf("  hello, fun1 !  %d\n" ,std::this_thread::get_id());
    if (slp>0) {
        printf(" ======= fun1 sleep %d  =========  %d\n",slp, std::this_thread::get_id());
        std::this_thread::sleep_for(std::chrono::milliseconds(slp));
    }
}

struct gfun {
    int operator()(int n) {
        printf("%d  hello, gfun !  %d\n" ,n, std::this_thread::get_id() );
        return 42;
    }
};

class A {
public:
    static int Afun(int n = 0) {   //函数必须是 static 的才能直接使用线程池
        std::cout << n << "  hello, Afun !  " << std::this_thread::get_id() << std::endl;
        return n;
    }

    static std::string Bfun(int n, std::string str, char c) {
        std::cout << n << "  hello, Bfun !  "<< str.c_str() <<"  " << (int)c <<"  " << std::this_thread::get_id() << std::endl;
        return str;
    }
};

int main()
    try {
        std::threadpool executor{ 50 };
        A a;
        std::future<void> ff = executor.commit(fun1,0);
        std::future<int> fg = executor.commit(gfun{},0);
        std::future<int> gg = executor.commit(a.Afun, 9999); //IDE提示错误,但可以编译运行
        std::future<std::string> gh = executor.commit(A::Bfun, 9998,"mult args", 123);
        std::future<std::string> fh = executor.commit([]()->std::string { std::cout << "hello, fh !  " << std::this_thread::get_id() << std::endl; return "hello,fh ret !"; });

        std::cout << " =======  sleep ========= " << std::this_thread::get_id() << std::endl;
        std::this_thread::sleep_for(std::chrono::microseconds(900));

        for (int i = 0; i < 50; i++) {
            executor.commit(fun1,i*100 );
        }
        std::cout << " =======  commit all ========= " << std::this_thread::get_id()<< " idlsize="<<executor.idlCount() << std::endl;

        std::cout << " =======  sleep ========= " << std::this_thread::get_id() << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(3));

        ff.get(); //调用.get()获取返回值会等待线程执行完,获取返回值
        std::cout << fg.get() << "  " << fh.get().c_str()<< "  " << std::this_thread::get_id() << std::endl;

        std::cout << " =======  sleep ========= " << std::this_thread::get_id() << std::endl;
        std::this_thread::sleep_for(std::chrono::seconds(3));

        std::cout << " =======  fun1,55 ========= " << std::this_thread::get_id() << std::endl;
        executor.commit(fun1,55).get();    //调用.get()获取返回值会等待线程执行完

        std::cout << "end... " << std::this_thread::get_id() << std::endl;


        std::threadpool pool(4);
        std::vector< std::future<int> > results;

        for (int i = 0; i < 8; ++i) {
            results.emplace_back(
                pool.commit([i] {
                    std::cout << "hello " << i << std::endl;
                    std::this_thread::sleep_for(std::chrono::seconds(1));
                    std::cout << "world " << i << std::endl;
                    return i*i;
                })
            );
        }
        std::cout << " =======  commit all2 ========= " << std::this_thread::get_id() << std::endl;

        for (auto && result : results)
            std::cout << result.get() << ' ';
        std::cout << std::endl;
        return 0;
    }
catch (std::exception& e) {
    std::cout << "some unhappy happened...  " << std::this_thread::get_id() << e.what() << std::endl;
}
```
细节解释：

①处的std::function是对于所有可调用对象的统一操作。所谓可调用对象大概有以下几种形式：
```c++
// 普通函数
int add(int a, int b){return a+b;} 

// lambda表达式
auto mod = [](int a, int b){ return a % b;}

// 函数对象类
struct divide{
    int operator()(int denominator, int divisor){
        return denominator/divisor;
    }
};
```
上述三种可调用对象虽然类型不同，但是共享了一种调用形式：
```c++
int(int ,int)
```
std::function就可以将上述类型保存起来，如下：
```c++
std::function<int(int ,int)>  a = add; 
std::function<int(int ,int)>  b = mod ; 
std::function<int(int ,int)>  c = divide(); 
```
总结来说，std::function 是一个可调用对象包装器，是一个类模板，可以容纳除了类成员函数指针之外的所有可调用对象，它可以用统一的方式处理函数、函数对象、函数指针，并允许保存和延迟它们的执行。

std::function可以取代函数指针的作用，因为它可以延迟函数的执行，特别适合作为回调函数使用。它比普通函数指针更加的灵活和便利。

②处的std::atomic表示原子操作，所有其所定义的变量的相关操作都是原子性的。

③处建立了一个线程队列Pool，其中的每个元素类型都是std::thread。emplace_back和push_back在效果上相同，但是使用push_back()向容器中加入一个右值元素（临时对象）的时候，首先会调用构造函数构造这个临时对象，然后需要调用拷贝构造函数将这个临时对象放入容器中。原来的临时变量释放。这样造成的问题是临时变量申请的资源就浪费。引入了右值引用，转移构造函数后，push_back()右值时就会调用构造函数和转移构造函数。在这上面有进一步优化的空间就是使用emplace_back()，这个函数将原地构造元素，不需要触发拷贝构造和转移构造。而且调用形式更加简洁，直接根据参数初始化临时对象的成员：
```c++
struct President  
{  
    std::string name;  
    std::string country;  
    int year;  

    President(std::string p_name, std::string p_country, int p_year)  
        : name(std::move(p_name)), country(std::move(p_country)), year(p_year)  
    {  
        std::cout << "I am being constructed.\n";  
    }
    President(const President& other)
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)
    {
        std::cout << "I am being copy constructed.\n";
    }
    President(President&& other)  
        : name(std::move(other.name)), country(std::move(other.country)), year(other.year)  
    {  
        std::cout << "I am being moved.\n";  
    }  
    President& operator=(const President& other);  
};  

std::vector<President> elections;  
std::cout << "emplace_back:\n";  
elections.emplace_back("Nelson Mandela", "South Africa", 1994); //形式上与President的构造函数相同，但是没有类的创建，就是在vector里原地构造
```

④处就是将要进入Pool的内容，Pool的元素类型是thread，在没有参数的情况下，直接向thread里放入一个函数名就行。这里使用的是lambda函数代替函数名。所谓lambda就是一个临时的函数，也就是匿名函数，本质上也是一个函数，只是还没有名字。

lambda表达式：
```c++
[capture list] (params list) mutable exception-> return type { function body }
```
各项具体含义如下
- capture list：捕获外部变量列表
- params list：形参列表（没有形参的可以省略）
- mutable指示符：用来说用是否可以修改捕获的变量（可省）
- exception：异常设定（可省）
- return type：返回类型（没有返回值可省）
- function body：函数体

⑤处表示条件变量，当 std::condition_variable对象的某个wait 函数被调用的时候，它使用 std::unique_lock(通过 std::mutex) 来锁住当前线程，当前线程会一直被阻塞，直到另外一个线程在相同的std::condition_variable对象上调用了 notification 函数来唤醒当前线程。std::condition_variable提供了两种 wait() 函数。当前线程调用wait()后将被阻塞，直到另外某个线程调用notify_*唤醒了当前线程。在第二种情况下除了锁以外还有第二个参数（bool），只有当第二个参数为false时调用 wait()才会阻塞当前线程，并且在收到其他线程的通知后，只有当第二参数为true时才会被解除阻塞。本线程池中使用的就是第二种。

⑥处的std::future提供了一种访问异步操作结果的机制。

同步就是整个处理过程顺序执行，当各个过程都执行完毕，并返回结果。是一种线性执行的方式，执行的流程不能跨越。一般用于流程性比较强的程序，比如用户登录，需要对用户验证完成后才能登录系统。异步则是只是发送了调用的指令，调用者无需等待被调用的方法完全执行完毕；而是继续执行下面的流程。是一种并行处理的方式，不必等待一个程序执行完，可以执行其它的任务，比如页面数据加载过程，不需要等所有数据获取后再显示页面。

通常一个异步操作我们是不能马上就获取操作结果的，只能在未来某个时候获取。我们可以以同步等待的方式来获取结果，可以通过查询future的状态（future_status）来获取异步操作的结果。future_status有三种状态：
deferred：异步操作还没开始
ready：异步操作已经完成
timeout：异步操作超时

获取future结果有三种方式：get、wait、wait_for，其中get等待异步操作结束并返回结果，wait只是等待异步操作完成，没有返回值，wait_for是超时等待返回结果。
```c++
//查询future的状态
std::future_status status;
do {
    status = future.wait_for(std::chrono::seconds(1));
    if (status == std::future_status::deferred) {
        std::cout << "deferred\n";
    } else if (status == std::future_status::timeout) {
        std::cout << "timeout\n";
    } else if (status == std::future_status::ready) {
        std::cout << "ready!\n";
    }
} while (status != std::future_status::ready);
```

⑥处的->是一种新的函数声明方式，目前函数的声明方式主要就是两种：
```c++
return-type identifier ( argument-declarations... )
```
```c++
auto identifier ( argument-declarations... ) -> return_type
```
第二种表达式主要是为了解决模板函数的推断问题。因为在定义有返回值的函数模板的时候是不知道返回值的类型的，比如说一个模板函数要计算a+b，a和b是两个模板类型，这样的话是不知道最终能返回什么的。所以这里也引入了一个自动推断函数decltype，由->和decltype配合就能完成这种需要返回值的模板函数：
```c++
template <typename T1, typename T2>
auto compose(T1 a, T2 b) -> decltype(a + b);
```

⑦处比较复杂，这里分别解释一下：
- std::make_shared可以返回一个指定类型的std::shared_ptr
- std::packaged_task 包装一个可调用的对象，并且允许异步获取该可调用对象产生的结果，从包装可调用对象意义上来讲，std::packaged_task 与 std::function 类似，只不过 std::packaged_task 将其包装的可调用对象的执行结果传递给一个 std::future 对象（该对象通常在另外一个线程中获取 std::packaged_task 任务的执行结果）。std::packaged_task 对象内部包含了两个最基本元素，一、被包装的任务(stored task)，任务(task)是一个可调用的对象，如函数指针、成员函数指针或者函数对象，二、共享状态(shared state)，用于保存任务的返回值，可以通过 std::future 对象来达到异步访问共享状态的效果。
```c++
#include <iostream>     // std::cout
#include <future>       // std::packaged_task, std::future
#include <chrono>       // std::chrono::seconds
#include <thread>       // std::thread, std::this_thread::sleep_for

// count down taking a second for each value:
int countdown (int from, int to) {
    for (int i=from; i!=to; --i) {
        std::cout << i << '\n';
        std::this_thread::sleep_for(std::chrono::seconds(1));
    }
    std::cout << "Finished!\n";
    return from - to;
}

int main ()
{
    std::packaged_task<int(int,int)> task(countdown); // 设置 packaged_task
    std::future<int> ret = task.get_future(); // 获得与 packaged_task 共享状态相关联的 future 对象.

    std::thread th(std::move(task), 10, 0);   //创建一个新线程完成计数任务.

    int value = ret.get();                    // 等待任务完成并获取结果.

    std::cout << "The countdown lasted for " << value << " seconds.\n";

    th.join();
    return 0;
}
```
- std::bind，可将std::bind函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。

std::bind将可调用对象与其参数一起进行绑定，绑定后的结果可以使用std::function保存。std::bind主要有以下两个作用：将可调用对象和其参数绑定成一个防函数；只绑定部分参数，减少可调用对象传入的参数。
```c++
double my_divide (double x, double y) {return x/y;}
auto fn_half = std::bind (my_divide,_1,2);  
std::cout << fn_half(10) << '\n';           
```
bind的第一个参数是函数名，普通函数做实参时，会隐式转换成函数指针。因此std::bind (my_divide,_1,2)等价于std::bind (&my_divide,_1,2)；_1表示占位符，位于\<functional\>中，std::placeholders::_1；

- forward() 函数，类似于 move() 函数，后者是将参数右值化，前者是... 肿么说呢？大概意思就是：不改变最初传入的类型的引用类型(左值还是左值，右值还是右值)；