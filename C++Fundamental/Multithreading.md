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
```
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

std::thread支持移动的好处是可以创建thread_guard类的实例(定义见 清单2.3)，并且拥有其线程的所有权。当thread_guard对象所持有的线程已经被引用，移动操作就可以避免很多不必要的麻烦；这意味着，当某个对象转移了线程的所有权后，它就不能对线程进行加入或分离。为了确保线程程序退出前完成，下面的代码里定义了scoped_thread类。现在，我们来看一下这段代码：
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