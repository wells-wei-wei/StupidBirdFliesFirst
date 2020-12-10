<!-- TOC -->

- [设计模式分类](#设计模式分类)
- [设计模式的六大原则](#设计模式的六大原则)
    - [单一职责原则](#单一职责原则)
    - [里氏替换原则（Liskov Substitution Principle）](#里氏替换原则liskov-substitution-principle)
    - [依赖倒转原则（Dependence Inversion Principle）](#依赖倒转原则dependence-inversion-principle)
    - [接口隔离原则（Interface Segregation Principle）](#接口隔离原则interface-segregation-principle)
    - [迪米特法则（最少知道原则）（Demeter Principle）](#迪米特法则最少知道原则demeter-principle)
    - [合成复用原则（Composite Reuse Principle）](#合成复用原则composite-reuse-principle)
- [工厂模式](#工厂模式)
    - [简单工厂模式](#简单工厂模式)
    - [工厂方法模式](#工厂方法模式)
    - [抽象工厂模式](#抽象工厂模式)
- [策略模式](#策略模式)
- [适配器模式](#适配器模式)
- [单例模式](#单例模式)
    - [懒汉模式（线程不安全）](#懒汉模式线程不安全)
    - [懒汉模式（线程安全）](#懒汉模式线程安全)
    - [懒汉模式（C++11线程安全）](#懒汉模式c11线程安全)
    - [饿汉模式](#饿汉模式)
- [原型模式](#原型模式)
- [模板模式](#模板模式)
- [建造者模式](#建造者模式)
- [外观模式](#外观模式)
- [代理模式](#代理模式)
- [享元模式](#享元模式)
- [桥接模式](#桥接模式)
- [装饰模式](#装饰模式)
- [备忘录模式](#备忘录模式)
- [中介者模式](#中介者模式)
- [职责链模式](#职责链模式)
- [观察者模式](#观察者模式)

<!-- /TOC -->

# 设计模式分类
总体来说设计模式分为三大类：

创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。

结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。

行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

其实还有两类：并发型模式和线程池模式。

# 设计模式的六大原则
总原则：**开闭原则**，即对扩展开放，对修改关闭。在程序需要进行拓展的时候，不能去修改原有的代码，而是要扩展原有代码，实现一个热插拔的效果。

## 单一职责原则
每个类应该实现单一的职责，如若不然，就应该把类拆分。
## 里氏替换原则（Liskov Substitution Principle）
任何基类可以出现的地方，子类一定可以出现。 只有当衍生类可以替换掉基类，软件单位的功能不受到影响时，基类才能真正被复用，而衍生类也能够在基类的基础上增加新的行为。里氏代换原则是对“开-闭”原则的补充。实现“开-闭”原则的关键步骤就是抽象化。而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。

另外，子类对父类的方法尽量不要重写和重载。因为父类代表了定义好的结构，通过这个规范的接口与外界交互，子类不应该随便破坏它。（重载：函数名相同，函数的参数个数、参数类型或参数顺序三者中必须至少有一种不同。函数返回值的类型可以相同，也可以不相同。发生在一个类内部。重定义：也叫做隐藏，子类重新定义父类中有相同名称的非虚函数 ( 参数列表可以不同 ) ，指派生类的函数屏蔽了与其同名的基类函数。可以理解成发生在继承中的重载。重写：也叫做覆盖，一般发生在子类和父类继承关系之间。子类重新定义父类中有相同名称和参数的虚函数。(override)）

## 依赖倒转原则（Dependence Inversion Principle）
这个是开闭原则的基础，具体内容：面向接口编程，依赖于抽象而不依赖于具体。写代码时用到具体类时，不与具体类交互，而与具体类的上层接口交互。

## 接口隔离原则（Interface Segregation Principle）
这个原则的意思是：每个接口中不存在子类用不到却必须实现的方法，如果不然，就要将接口拆分。使用多个隔离的接口，比使用单个接口（多个接口方法集合到一个的接口）要好。

## 迪米特法则（最少知道原则）（Demeter Principle）
就是说：一个类对自己依赖的类知道的越少越好。也就是说无论被依赖的类多么复杂，都应该将逻辑封装在方法的内部，通过public方法提供给外部。这样当被依赖的类变化时，才能最小的影响该类。

最少知道原则的另一个表达方式是：只与直接的朋友通信。类之间只要有耦合关系，就叫朋友关系。耦合分为依赖、关联、聚合、组合等。我们称出现为成员变量、方法参数、方法返回值中的类为直接朋友。局部变量、临时变量则不是直接的朋友。我们要求陌生的类不要作为局部变量出现在类中。

## 合成复用原则（Composite Reuse Principle）
原则是尽量首先使用合成/聚合的方式，而不是使用继承。

# 工厂模式
工厂模式其实主要目的就是在程序运行中决定到底需要哪种类别。
## 简单工厂模式
```c++
typedef enum
{
    T80 = 1,
    T99
}TankType;
    
class Tank
{
public:
    virtual void message() = 0;
};
    
class Tank80:public Tank
{
public:
    void message()
    {
        cout << "Tank80" << endl;
    }
};
    
class Tank99:public Tank
{
public:
    void message()
    {
        cout << "Tank99" << endl;
    }
};
    
class TankFactory
{
public:
    Tank* createTank(TankType type)
    {
        switch(type)
        {
        case 1:
            return new Tank80();
        case 2:
            return new Tank99();
        default:
            return NULL;
        }
    }
};
```
从这种形式来说，工厂模式必须与虚函数同时使用。

## 工厂方法模式
因为简单工厂模式其实违反了开闭原则，当需要增加新的类别时必须去修改工厂类的内部实现。所以为了能够遵守设计模式，将工厂类设计成基类，然后派生类为生产每一种类别的“特殊工厂”：
```c++
class Tank
{
public:
    virtual void message() = 0;
};
    
class Tank80:public Tank
{
public:
    void message()
    {
        cout << "Tank80" << endl;
    }
};
    
class Tank99:public Tank
{
public:
    void message()
    {
        cout << "Tank99" << endl;
    }
};
    
class TankFactory
{
public:
    virtual Tank* createTank() = 0;
};
    
class Tank80Factory:public TankFactory
{
public:
    Tank* createTank()
    {
        return new Tank80();
    }
};
    
class Tank99Factory:public TankFactory
{
public:
    Tank* createTank()
    {
        return new Tank99();
    }
};
```

## 抽象工厂模式
以上的工厂都是只生产一类产品，抽象工厂就是指一个工厂可以生产不同种类的产品，而每个产品都有一个派生类工厂去生产他：
```c++
class Tank
{
public:
    virtual void message() = 0;
};
    
class Tank80:public Tank
{
public:
    void message()
    {
        cout << "Tank80" << endl;
    }
};
    
class Tank99:public Tank
{
public:
    void message()
    {
        cout << "Tank99" << endl;
    }
};
    
class Plain
{
public:
    virtual void message() = 0;
};
    
class Plain80: public Plain
{
public:
    void message()
    {
        cout << "Plain80" << endl;
    }
};
    
class Plain99: public Plain
{
public:
    void message()
    {
        cout << "Plain99" << endl;
    }
};
    
class Factory
{
public:
    virtual Tank* createTank() = 0;
    virtual Plain* createPlain() = 0;
};
    
class Factory80:public Factory
{
public:
    Tank* createTank()
    {
        return new Tank80();
    }
    Plain* createPlain()
    {
        return new Plain80();
    }
};
    
class Factory99:public Factory
{
public:
    Tank* createTank()
    {
        return new Tank99();
    }
    Plain* createPlain()
    {
        return new Plain99();
    }
};
```
说实话，我真觉得抽象工厂没啥用，类别多了，但是依然破坏了设计原则。

# 策略模式
策略模式是指定义一系列的算法，把它们一个个封装起来，并且使它们可以互相替换。使得算法可以独立于使用它的客户而变化，也就是说这些算法所完成的功能是一样的，对外接口是一样的，只是各自现实上存在差异。

主要解决：在有多种算法相似的情况下，使用 if…else 所带来的复杂和难以维护。

何时使用：一个系统有许多许多类，而区分它们的只是他们直接的行为。

如何解决：将这些算法封装成一个一个的类，任意地替换。

关键代码：实现同一个接口。

缺点： 1、策略类会增多。 2、所有策略类都需要对外暴露。
策略模式共有五种实现方法：①将算法封装成类内的一个成员函数，然后用指针传入并调用 ②传入带参数的标签，跟工厂模式相似 ③使用模板类，跟第①种相似 ④前面几种都需要把函数编程类内的成员函数，最后一种是单纯地使用函数指针
```c++
//传统策略模式实现
class Hurt
{
public:
    virtual void redBuff() = 0;
};
    
class AdcHurt:public Hurt
{
public:
    void redBuff()
    {
        cout << "Adc hurt" << endl;
    }
};
    
class ApcHurt:public Hurt
{
public:
    void redBuff()
    {
        cout << "Apc hurt" << endl;
    }
};
    
//方法1：传入一个指针参数
class Soldier
{
public:
    Soldier(Hurt* hurt):m_hurt(hurt)
    {
    }
    ~Soldier()
    {
    }
    void beInjured()
    {
        m_hurt->redBuff();
    }
private:
    Hurt* m_hurt;
};
    
//方法2：传入一个参数标签
typedef enum
{
    adc,
    apc
}HurtType;
    
class Master
{
public:
    Master(HurtType type)
    {
        switch(type)
        {
        case adc:
            m_hurt = new AdcHurt;
            break;
        case apc:
            m_hurt = new ApcHurt;
            break;
        default:
            m_hurt = NULL;
            break;
        }
    }
    ~Master()
    {
    }
    void beInjured()
    {
        if(m_hurt != NULL)
        {
            m_hurt->redBuff();
        }
        else
        {
            cout << "Not hurt" << endl;
        }
    }
private:
    Hurt* m_hurt;
};
    
//方法3：使用模板类
template <typename T>
class Tank
{
public:
    void beInjured()
    {
        m_hurt.redBuff();
    }
private:
    T m_hurt;
};
//END
    
//使用函数指针实现策略模式
void adcHurt(int num)
{
    cout << "adc hurt:" << num << endl;
}
    
void apcHurt(int num)
{
    cout << "apc hurt:" << num << endl;
}
    
//普通函数指针
class Aid
{
public:
    typedef void (*HurtFun)(int);
    
    Aid(HurtFun fun):m_fun(fun)
    {
    }
    void beInjured(int num)
    {
        m_fun(num);
    }
private:
    HurtFun m_fun;
};
    
//使用std::function , 头文件：#include<functional>
class Bowman
{
public:
    typedef std::function<void(int)> HurtFunc;
    
    Bowman(HurtFunc fun):m_fun(fun)
    {
    }
    void beInjured(int num)
    {
        m_fun(num);
    }
    
private:
    HurtFunc m_fun;
};
//END
```

# 适配器模式
适配器模式：将一个类的接口转换成客户希望的另一个接口，使得原本由于接口不兼容而不能一起工作的哪些类可以一起工作。

主要解决：主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

何时使用： 1、系统需要使用现有的类，而此类的接口不符合系统的需要。 2、想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。 3、通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）

如何解决：继承或依赖（推荐）。

关键代码：适配器继承或依赖已有的对象，实现想要的目标接口。

缺点：1、过多地使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是 A 接口，其实内部被适配成了 B 接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

```c++
//使用复合，对象模式
class Deque  //双端队列，被适配类
{
public:
    void push_back(int x)
    {
        cout << "Deque push_back:" << x << endl;
    }
    void push_front(int x)
    {
        cout << "Deque push_front:" << x << endl;
    }
    void pop_back()
    {
        cout << "Deque pop_back" << endl;
    }
    void pop_front()
    {
        cout << "Deque pop_front" << endl;
    }
};
    
class Sequence  //顺序类，目标类，也就是说最终希望使用的是这个接口
{
public:
    virtual void push(int x) = 0;
    virtual void pop() = 0;
};
    
class Stack:public Sequence   //栈, 适配类
{
public:
    void push(int x)
    {
        m_deque.push_back(x);
    }
    void pop()
    {
        m_deque.pop_back();
    }
private:
    Deque m_deque;
};
    
class Queue:public Sequence  //队列，适配类
{
public:
    void push(int x)
    {
        m_deque.push_back(x);
    }
    void pop()
    {
        m_deque.pop_front();
    }
private:
    Deque m_deque;
};
//EN

//使用继承,类模式
class Deque  //双端队列，被适配类
{
public:
    void push_back(int x)
    {
        cout << "Deque push_back:" << x << endl;
    }
    void push_front(int x)
    {
        cout << "Deque push_front:" << x << endl;
    }
    void pop_back()
    {
        cout << "Deque pop_back" << endl;
    }
    void pop_front()
    {
        cout << "Deque pop_front" << endl;
    }
};
    
class Sequence  //顺序类，目标类
{
public:
    virtual void push(int x) = 0;
    virtual void pop() = 0;
};
    
class Stack:public Sequence, private Deque   //栈, 适配类
{
public:
    void push(int x)
    {
        push_back(x);
    }
    void pop()
    {
        pop_back();
    }
};
    
class Queue:public Sequence, private Deque  //队列，适配类
{
public:
    void push(int x)
    {
        push_back(x);
    }
    void pop()
    {
        pop_front();
    }
};
//END
```

# 单例模式
单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

主要解决：一个全局使用的类频繁地创建与销毁。

何时使用：想控制实例数目，节省系统资源的时候。

如何解决：判断系统是否已存在单例，如果有则返回，没有则创建。

关键代码：构造函数是私有的。

单例大约有两种实现方法：懒汉与饿汉。

懒汉：故名思义，不到万不得已就不会去实例化类，也就是说在第一次用到类实例的时候才会去实例化，所以上边的经典方法被归为懒汉实现；

饿汉：饿了肯定要饥不择食。所以在单例类定义的时候就进行实例化。

特点与选择：由于要进行线程同步，所以在访问量比较大，或者可能访问的线程比较多时，采用饿汉实现，可以实现更好的性能。这是以空间换时间。

在访问量较小时，采用懒汉实现。这是以时间换空间。

## 懒汉模式（线程不安全）
```c++
class Singleton{
private:
    Singleton(){};
    Singleton(const Singleton& obj) = delete;  //明确拒绝
	Singleton& operator=(const Singleton& obj) = delete; //明确拒绝

    static const Singleton* _singleton;
public:
    static const Singleton* getintance(){
        if(_singleton==nullptr) _singleton=new Singleton();//这里在当前这种懒汉模式中是必须要加判断的
        return _singleton;
    }
};
const Singleton* Singleton::_singleton=nullptr;//static必须在类外有初始化，否则会报错
int main(){
    const Singleton* single=Singleton::getintance();
}

```
懒汉模式线程不安全，原因是假如现在同时有两个线程都要使用getintance创建实例，其中一个正准备创建实例，另一个也通过了if判断，这时就会创建两个实例。

## 懒汉模式（线程安全）
```c++
class Singleton{
private:
    Singleton(){};
    Singleton(const Singleton& obj)=delete;
    Singleton& operator=(const Singleton& obj)=delete;

    static const Singleton* _singleton;
public:
    static const Singleton* getinstance(){
        if(_singleton==nullptr){
            mutex.lock();
            if(_singleton==nullptr){
                _singleton=new Singleton();
            }
            mutex.unlock();
        }
        return _singleton;
    }
};
const Singleton* Singleton::_singleton=nullptr;

int main(){
    const Singleton* single=Singleton::getinstance();
}
```
这里为了线程安全必须加两层if判断来看实例是不是空的。因为这里有可能同时有两个线程都通过了第一层if，然后一个抢到了锁，另一个阻塞。这时假如没有第二个if的话，等第一个线程创建完了以后第二个紧接着就又会创建一个，这就又生成了两个实例。

## 懒汉模式（C++11线程安全）
```c++
class Singleton{
private:
    Singleton(){};
    Singleton(const Singleton& obj)=delete;
    Singleton& operator=(const Singleton& obj)=delete;

    static const Singleton* _singleton;
public:
    static const Singleton* getinstance(){
        static const Singleton* _singleton=new Singleton();
        return _singleton;
    }
};

int main(){
    const Singleton* single=Singleton::getinstance();
}
```
在c++11中要求必须保证类内的静态变量的线程安全，因此这种情况下也只会创建一个实例，且代码数量最少。

## 饿汉模式
饿汉模式线程安全。
```c++
class Singleton{
private:
    Singleton(){};
    Singleton(const Singleton& obj)=delete;
    Singleton& operator=(const Singleton& obj)=delete;

    static const Singleton* _singleton;
public:
    static const Singleton* getinstance(){
        return _singleton;
    }
};

const Singleton* Singleton::_singleton=new Singleton();

int main(){
    const Singleton* single=Singleton::getinstance();
}
```

# 原型模式
原型模式：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

主要解决：在运行期建立和删除对象。

何时使用：1).当我们的对象类型不是开始就能确定的，而这个类型是在运行期确定的话，那么我们通过这个类型的对象克隆出一个新的对象比较容易一些；2).有的时候，我们需要一个对象在某个状态下的副本，此时，我们使用原型模式是最好的选择；例如：一个对象，经过一段处理之后，其内部的状态发生了变化；这个时候，我们需要一个这个状态的副本，如果直接new一个新的对象的话,但是它的状态是不对的，此时，可以使用原型模式，将原来的对象拷贝一个出来，这个对象就和之前的对象是完全一致的了；3).当我们处理一些比较简单的对象时，并且对象之间的区别很小，可能就几个属性不同而已，那么就可以使用原型模式来完成，省去了创建对象时的麻烦了；4).有的时候，创建对象时，构造函数的参数很多，而自己又不完全的知道每个参数的意义，就可以使用原型模式来创建一个新的对象，不必去理会创建的过程。

适当的时候考虑一下原型模式，能减少对应的工作量，减少程序的复杂度，提高效率。

如何解决：利用已有的一个原型对象，快速地生成和原型对象一样的实例。

关键代码：拷贝，return new className(*this);
```c++
class Clone
{
public:
    Clone()
    {
    }
    virtual ~Clone()
    {
    }
    virtual Clone* clone() = 0;
    virtual void show() = 0;
};
    
class Sheep:public Clone
{
public:
    Sheep(int id, string name):Clone(),m_id(id),m_name(name)
    {
        cout << "Sheep() id add:" << &m_id << endl;
        cout << "Sheep() name add:" << &m_name << endl;
    }
    ~Sheep()
    {
    }
    
    Sheep(const Sheep& obj)
    {
        this->m_id = obj.m_id;
        this->m_name = obj.m_name;
        cout << "Sheep(const Sheep& obj) id add:" << &m_id << endl;
        cout << "Sheep(const Sheep& obj) name add:" << &m_name << endl;
    }
    
    Clone* clone()
    {
        return new Sheep(*this);
    }
    void show()
    {
        cout << "id  :" << m_id << endl;
        cout << "name:" << m_name.data() << endl;
    }
private:
    int m_id;
    string m_name;
};
    
int main()
{
    Clone* s1 = new Sheep(1, "abs");
    s1->show();
    Clone* s2 = s1->clone();
    s2->show();
    delete s1;
    delete s2;
    return 0;
}
```
这里注意一下，派生类sheep在自己的构造函数中还调用了基类的构造函数Clone，一般来说，派生类在构造的时候会先构造基类对象，假如在派生类中不显式说明的话，就会自动调用基类的默认构造函数，如果是想用基类其他的构造函数，就必须显示在构造列表中使用才行。所以这里其实是不用显示说明的，因为基类构造函数根本就没有主动定义。

# 模板模式
模板模式：定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

主要解决：多个子类有相同的方法，并且逻辑相同，细节有差异。

如何解决：对重要，复杂的算法，将核心算法设计为模板方法，周边细节由子类实现，重构时，经常使用的方法，将相同的代码抽象到父类，通过钩子函数约束行为。

关键代码：在抽象类实现通用接口，细节变化在子类实现。

缺点：每一个不同的实现都需要一个子类来实现，导致类的个数增加，使得系统更加庞大。
```c++
class Computer
{
public:
    void product()
    {
        installCpu();
        installRam();
        installGraphicsCard();
    }
    
protected:
    virtual    void installCpu() = 0;
    virtual void installRam() = 0;
    virtual void installGraphicsCard() = 0;
    
};
    
class ComputerA:public Computer
{
protected:
    void installCpu() override
    {
        cout << "ComputerA install Inter Core i5" << endl;
    }
    
    void installRam() override
    {
        cout << "ComputerA install 2G Ram" << endl;
    }
    
    void installGraphicsCard() override
    {
        cout << "ComputerA install Gtx940 GraphicsCard" << endl;
    }
};
    
class ComputerB:public Computer
{
protected:
    void installCpu() override
    {
        cout << "ComputerB install Inter Core i7" << endl;
    }
    
    void installRam() override
    {
        cout << "ComputerB install 4G Ram" << endl;
    }
    
    void installGraphicsCard() override
    {
        cout << "ComputerB install Gtx960 GraphicsCard" << endl;
    }
};
```
这就是普通的继承+重定义，没啥可说的。

# 建造者模式
建造者模式：将复杂对象的构建和其表示分离，使得同样的构建过程可以创建不同的表示。

主要解决：一个复杂对象的创建工作，由各个部分的子对象用一定的算法构成；由于需求变化，这个复杂对象的各个部分经常面临变化，但将它们组合在一起的算法却相对稳定。

如何解决：将变与不变分开

关键代码：建造者：创建和提供实例，Director：管理建造出来的实例的依赖关系。

缺点：1、产品必须有共同点，范围有限制。 2、如内部变化复杂，会有很多的建造类。
```c++
typedef enum
{
    type1,
    type2
}ProductType;
    
class Product   //产品
{
public:
    void setNum(int num);
    void setColor(string color);
    void setType(ProductType type);
    
    void showProduct();
private:
    int m_num;
    string m_color;
    ProductType m_type;
    
};
    
void Product::setNum(int num)
{
    m_num = num;
}
    
void Product::setColor(string color)
{
    m_color = color;
}
    
void Product::setType(ProductType type)
{
    m_type = type;
}
    
void Product::showProduct()
{
    cout << "Product: " << endl;
    cout << "       num  : " << m_num << endl;
    cout << "       color: " << m_color.data() << endl;
    cout << "       type : " << m_type << endl;
}
    
//建造者父类，定义接口
class Builder
{
public:
    Builder(){}
    virtual ~Builder(){}
    virtual void buildNum(int num) = 0;
    virtual void buildColor(string color) = 0;
    virtual void buildType(ProductType type) = 0;
    virtual void createProduct() = 0;
    virtual Product* getProduct() = 0;
    virtual void show() = 0;
};
    
//建造者A
class BuilderA:public Builder
{
public:
    BuilderA(){}
        ~BuilderA(){}
    void buildNum(int num) override;
    void buildColor(string color) override;
    void buildType(ProductType type) override;
    void createProduct() override;
    Product* getProduct() override;
    void show() override;
private:
    Product* m_product;
};
    
void BuilderA::buildNum(int num)
{
    cout << "BuilderA build Num: " << num << endl;
    m_product->setNum(num);
}
    
void BuilderA::buildColor(string color)
{
    cout << "BuilderA build color: " << color.data() << endl;
    m_product->setColor(color);
}
    
void BuilderA::buildType(ProductType type)
{
    cout << "BuilderA build type: " << type << endl;
    m_product->setType(type);
}
    
void BuilderA::createProduct()
{
    cout << "BuilderA CreateProduct: " << endl;
    m_product = new Product();
}
    
Product* BuilderA::getProduct()
{
    return m_product;
}
void BuilderA::show()
{
    m_product->showProduct();
}
    
//建造者B
class BuilderB:public Builder
{
public:
    BuilderB(){}
        ~BuilderB(){}
    void buildNum(int num) override;
    void buildColor(string color) override;
    void buildType(ProductType type) override;
    void createProduct() override;
    Product* getProduct() override;
    void show() override;
private:
    Product* m_product;
};
    
void BuilderB::buildNum(int num)
{
    cout << "BuilderB build Num: " << num << endl;
    m_product->setNum(num);
}
    
void BuilderB::buildColor(string color)
{
    cout << "BuilderB build color: " << color.data() << endl;
    m_product->setColor(color);
}
    
void BuilderB::buildType(ProductType type)
{
    cout << "BuilderB build type: " << type << endl;
    m_product->setType(type);
}
    
void BuilderB::createProduct()
{
    cout << "BuilderB CreateProduct: " << endl;
    m_product = new Product();
}
    
Product* BuilderB::getProduct()
{
    return m_product;
}
void BuilderB::show()
{
    m_product->showProduct();
}
    
//管理类，负责安排构造的具体过程
class Director
{
public:
    Director(Builder* builder):m_builder(builder)
    {
    }
    void construct(int num, string color, ProductType type)
    {
        m_builder->createProduct();
        m_builder->buildNum(num);
        m_builder->buildColor(color);
        m_builder->buildType(type);
    }
    
private:
    Builder* m_builder;
};
```
其实就是用一个类去以不同的方式建造另一个类

# 外观模式
外观模式：为子系统中的一组接口定义一个一致的界面，外观模式提供了一个高层接口，这个接口使得这一子系统更加容易被使用；对于复杂的系统，系统为客户提供一个简单的接口，把复杂的实现过程封装起来，客户不需要了解系统内部的细节。

主要解决：客户不需要了解系统内部复杂的细节，只需要一个接口；系统入口。

如何解决：客户不直接与系统耦合，而是通过外观类与系统耦合。

关键代码：客户与系统之间加一个外观层，外观层处理系统的调用关系、依赖关系等。

缺点：需要修改时不易继承、不易修改。
```c++
class Cpu
{
public:
    void productCpu()
    {
        cout << "Product Cpu" << endl;
    }
};
    
class Ram
{
public:
    void productRam()
    {
        cout << "Product Ram" << endl;
    }
};
    
class Graphics
{
public:
    void productGraphics()
    {
        cout << "Product Graphics" << endl;
    }
};
    
class Computer
{
public:
    void productComputer()
    {
        Cpu cpu;
        cpu.productCpu();
        Ram ram;
        ram.productRam();
        Graphics graphics;
        graphics.productGraphics();
    }
};
    
int main()
{
    //客户直接调用computer生产函数，无需关心具体部件的生产过程。也可直接单独生产部件
    Computer computer;   
    computer.productComputer();
    
    Cpu cpu;
    cpu.productCpu();
    
    return 0;
}
```
# 代理模式
代理模式：为其它对象提供一种代理以控制对这个对象的访问。

主要解决：在直接访问对象时带来的问题，比如：要访问的对象在远程服务器上。在面向对象系统中，有些对象由于某些原因，直接访问会给使用者或系统带来很多麻烦，可以在访问此对象时加上一个对此对象的访问层。

如何解决：增加中间代理层。

关键代码：实现与被代理类组合。
```c++
class Gril
{
public:
    Gril(string name = "gril"):m_string(name){}
    string getName()
    {
        return m_string;
    }
private:
    string m_string;
};
    
class Profession
{
public:
    virtual ~Profession(){}
    virtual void profess() = 0;
};
    
class YoungMan:public Profession
{
public:
    YoungMan(Gril gril):m_gril(gril){}
    void profess()
    {
        cout << "Young man love " << m_gril.getName().data() << endl;
    }
private:
    Gril m_gril;
};
    
class ManProxy:public Profession
{
public:
    ManProxy(Gril gril):m_man(new YoungMan(gril)){}
    void profess()
    {
        cout << "I am Proxy" << endl;
        m_man->profess();
    }
private:
    YoungMan* m_man;
};
    
int main(int argc, char *argv[])
{
    Gril gril("hei"); //代理
    Profession* proxy = new ManProxy(gril);
    proxy->profess();
    delete proxy;
    return 0;
}
```
# 享元模式
享元模式：运用共享技术有效地支持大量细粒度的对象。

主要解决：在有大量对象时，把其中共同的部分抽象出来，如果有相同的业务请求，直接返回内存中已有的对象，避免重新创建。

如何解决：用唯一标识码判断，如果内存中有，则返回这个唯一标识码所标识的对象。

关键代码：将内部状态作为标识，进行共享。
```c++
//以Money的类别作为内部标识，面值作为外部状态。
enum MoneyCategory   //类别，内在标识，作为标识码
{
    Coin,
    bankNote
};
    
enum FaceValue      //面值，外部标识，需要存储的对象
{
    ValueOne = 1,
    ValueTwo
};
    
class Money      //抽象父类
{
public:
    Money(MoneyCategory cate):m_mCate(cate){}
    virtual ~Money(){ cout << "~Money() " << endl; }
    virtual void save() = 0;
private:
    MoneyCategory m_mCate;
};
    
class MoneyCoin:public Money    //具体子类1
{
public:
    MoneyCoin(MoneyCategory cate):Money(cate){}
    ~MoneyCoin(){ cout << "~MoneyCoin()" << endl; }
    void save()
    {
        cout << "Save Coin" << endl;
    }
};
    
class MoneyNote:public Money   //具体子类2
{
public:
    MoneyNote(MoneyCategory cate):Money(cate){}
    ~MoneyNote(){ cout << "~MoneyNote()" << endl; }
    void save()
    {
        cout << "Save BankNote" << endl;
    }
};
    
class Bank
{
public:
    Bank():m_coin(nullptr),m_note(nullptr),m_count(0){}
    ~Bank()
    {
        if(m_coin != nullptr)
        {
            delete m_coin;
            m_coin = nullptr;
        }
        if(m_note != nullptr)
        {
            delete m_note;
            m_note = nullptr;
        }
    }
    void saveMoney(MoneyCategory cate, FaceValue value)
    {
        switch(cate)    //以类别作为标识码
        {
        case Coin:
        {
            if(m_coin == nullptr)  //内存中存在标识码所标识的对象，则直接调用，不再创建
            {
                m_coin = new MoneyCoin(Coin);
            }
            m_coin->save();
            m_vector.push_back(value);
            break;
        }
        case bankNote:
        {
            if(m_note == nullptr)
            {
                m_note = new MoneyNote(bankNote);
            }
            m_note->save();
            m_vector.push_back(value);
            break;
        }
        default:
            break;
        }
    }
    
    int sumSave()
    {
        auto iter = m_vector.begin();
        for(; iter != m_vector.end(); iter++)
        {
            m_count += *iter;
        }
        return m_count;
    }
    
private:
    vector<FaceValue> m_vector;
    Money* m_coin;
    Money* m_note;
    int m_count;
};
    
int main()
{
    Bank b1;
    b1.saveMoney(Coin, ValueOne);
    b1.saveMoney(Coin, ValueTwo);
    b1.saveMoney(Coin, ValueTwo);
    b1.saveMoney(bankNote, ValueOne);
    b1.saveMoney(bankNote, ValueTwo);
    cout << b1.sumSave() << endl;
    
    return 0;
}
```
# 桥接模式
桥接模式：将抽象部分与实现部分分离，使它们都可以独立变换。

主要解决：在有很多中可能会变化的情况下，用继承会造成类爆炸问题，不易扩展。

如何解决：把不同的分类分离出来，使它们独立变化，减少它们之间的耦合。

关键代码：将现实独立出来，抽象类依赖现实类。
```c++
//将各种App、各种手机全部独立分开，使其自由组合桥接
class App
{
public:
    virtual ~App(){ cout << "~App()" << endl; }
    virtual void run() = 0;
};
    
class GameApp:public App
{
public:
    void run()
    {
        cout << "GameApp Running" << endl;
    }
};
    
class TranslateApp:public App
{
public:
    void run()
    {
        cout << "TranslateApp Running" << endl;
    }
};
    
class MobilePhone
{
public:
    virtual ~MobilePhone(){ cout << "~MobilePhone()" << endl;}
    virtual void appRun(App* app) = 0;  //实现App与手机的桥接
};
    
class XiaoMi:public MobilePhone
{
public:
    void appRun(App* app)
    {
        cout << "XiaoMi: ";
        app->run();
    }
};
    
class HuaWei:public MobilePhone
{
public:
    void appRun(App* app)
    {
        cout << "HuaWei: ";
        app->run();
    }
};
    
int main()
{
    App* gameApp = new GameApp;
    App* translateApp = new TranslateApp;
    MobilePhone* mi = new XiaoMi;
    MobilePhone* hua = new HuaWei;
    mi->appRun(gameApp);
    mi->appRun(translateApp);
    hua->appRun(gameApp);
    hua->appRun(translateApp);
    delete hua;
    delete mi;
    delete gameApp;
    delete translateApp;
    
    return 0;
}
```
# 装饰模式
装饰模式：动态地给一个对象添加一些额外的功能，就新增加功能来说，装饰器模式比生产子类更加灵活。

主要解决：通常我们为了扩展一个类经常使用继承的方式，由于继承为类引入静态特征，并且随着扩展功能的增多，子类会很膨胀。

如何解决：将具体的功能划分，同时继承装饰者类。

关键代码：装饰类复合和继承组件类，具体的扩展类重写父类的方法。
```c++
class Dumplings    //抽象类   饺子
{
public:
    virtual ~Dumplings(){}
    virtual void showDressing() = 0;
};
    
class MeatDumplings:public Dumplings    //现实类  肉馅饺子
{
public:
        ~MeatDumplings(){ cout << "~MeatDumplings()" << endl; }
    void showDressing()
    {
        cout << "Add Meat" << endl;
    }
};
    
class DecoratorDumpling:public Dumplings    //装饰类
{
public:
    DecoratorDumpling(Dumplings* d):m_dumpling(d){}
    virtual ~DecoratorDumpling(){ cout << "~DecoratorDumpling()" << endl; }
    void showDressing()
    {
        m_dumpling->showDressing();
    }
private:
    Dumplings* m_dumpling;
};
    
class SaltDecorator:public DecoratorDumpling   // 装饰类  加盐
{
public:
    SaltDecorator(Dumplings* d):DecoratorDumpling(d){}
    ~SaltDecorator(){ cout << "~SaltDecorator()" << endl; }
    void showDressing()
    {
        DecoratorDumpling::showDressing();   //注意点
        addDressing();
    }
    
private:
    void addDressing()
    {
        cout << "Add Salt" << endl;
    }
};
    
class OilDecorator:public DecoratorDumpling   //装饰类  加油
{
public:
    OilDecorator(Dumplings* d):DecoratorDumpling(d){}
    ~OilDecorator(){ cout << "~OilDecorator()" << endl; }
    void showDressing()
    {
        DecoratorDumpling::showDressing(); //注意点
        addDressing();
    }
    
private:
    void addDressing()
    {
        cout << "Add Oil" << endl;
    }
};
    
class CabbageDecorator:public DecoratorDumpling  //装饰类   加蔬菜
{
public:
    CabbageDecorator(Dumplings* d):DecoratorDumpling(d){}
    ~CabbageDecorator(){ cout << "~CabbageDecorator()" << endl; }
    void showDressing()
    {
        DecoratorDumpling::showDressing(); //注意点
        addDressing();
    }
    
private:
    void addDressing()
    {
        cout << "Add Cabbage" << endl;
    }
};
    
int main()
{
    Dumplings* d = new MeatDumplings;       //原始的肉饺子
    Dumplings* d1 = new SaltDecorator(d);   //加盐后的饺子
    Dumplings* d2 = new OilDecorator(d1);   //加油后的饺子
    Dumplings* d3 = new CabbageDecorator(d2);  //加蔬菜后的饺子
    d3->showDressing();
    delete d;
    delete d1;
    delete d2;
    delete d3;
    return 0;
}
```
# 备忘录模式
备忘录模式：在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态。这样以后就可以将该对象恢复到原来保存的状态。

如何解决：通过一个备忘录类专门存储对象状态。

关键代码：备忘录类、客户类、备忘录管理类；客户类不与备忘录类耦合，而是与备忘录管理类耦合。
```c++
typedef struct  //需要保存的信息
{
    int grade;
    string arm;
    string corps;
}GameValue;
    
class Memento   //备忘录类
{
public:
    Memento(){}
    Memento(GameValue value):m_gameValue(value){}
    GameValue getValue()
    {
        return m_gameValue;
    }
private:
    GameValue m_gameValue;
};
    
class Game     //客户类 游戏
{
public:
    Game(GameValue value):m_gameValue(value)
    {}
    void addGrade()  //等级增加
    {
        m_gameValue.grade++;
    }
    void replaceArm(string arm)  //更换武器
    {
        m_gameValue.arm = arm;
    }
    void replaceCorps(string corps)  //更换工会
    {
        m_gameValue.corps = corps;
    }
    Memento saveValue()    //保存当前信息
    {
        Memento memento(m_gameValue);
        return memento;
    }
    void load(Memento memento) //载入信息
    {
        m_gameValue = memento.getValue();
    }
    void showValue()
    {
        cout << "Grade: " << m_gameValue.grade << endl;
        cout << "Arm  : " << m_gameValue.arm.data() << endl;
        cout << "Corps: " << m_gameValue.corps.data() << endl;
    }
private:
    GameValue m_gameValue;
};
    
class Caretake //备忘录管理类
{
public:
    void save(Memento memento)  //保存信息
    {
        m_memento = memento;
    }
    Memento load()            //读已保存的信息
    {
        return m_memento;
    }
private:
    Memento m_memento;
};
    
int main()
{
    GameValue v1 = {0, "Ak", "3K"};
    Game game(v1);    //初始值
    game.addGrade();
    game.showValue();
    cout << "----------" << endl;
    Caretake care;
    care.save(game.saveValue());  //保存当前值
    game.addGrade();          //修改当前值
    game.replaceArm("M16");
    game.replaceCorps("123");
    game.showValue();
    cout << "----------" << endl;
    game.load(care.load());   //恢复初始值
    game.showValue();
    return 0;
}
```
# 中介者模式
中介者模式：用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显示地相互引用，从而使其耦合松散，而且可以独立地改变它们之前的交互。

主要解决：对象与对象之前存在大量的关联关系，这样势必会造成系统变得复杂，若一个对象改变，我们常常需要跟踪与之关联的对象，并做出相应的处理。

如何解决：将网状结构分离为星型结构。

关键代码：将相关对象的通信封装到一个类中单独处理。
```c++
class Mediator;
    
class Person   //抽象同事类
{
public:
    virtual ~Person(){}
    virtual void setMediator(Mediator* mediator)
    {
        m_mediator = mediator;
    }
    virtual void sendMessage(const string& message) = 0;
    virtual void getMessage(const string& message) = 0;
protected:
    Mediator* m_mediator;
};
    
class Mediator    //抽象中介类
{
public:
    virtual ~Mediator(){}
    virtual void setBuyer(Person* buyer) = 0;
    virtual void setSeller(Person* seller) = 0;
    virtual void send(const string& message, Person* person) = 0;
};
    
class Buyer:public Person   //买家类
{
public:
    void sendMessage(const string& message)
    {
        m_mediator->send(message, this);
    }
    void getMessage(const string& message)
    {
        cout << "Buyer Get: " << message.data() << endl;
    }
};
    
class Seller:public Person  //卖家类
{
public:
    void sendMessage(const string& message)
    {
        m_mediator->send(message, this);
    }
    void getMessage(const string& message)
    {
        cout << "Seller Get: " << message.data() << endl;
    }
};
    
class HouseMediator:public Mediator  //具体的中介类
{
public:
    HouseMediator():m_buyer(nullptr),m_seller(nullptr){}
    void setBuyer(Person* buyer)
    {
        m_buyer = buyer;
    }
    void setSeller(Person *seller)
    {
        m_seller = seller;
    }
    void send(const string& message, Person* person)
    {
        if(person == m_buyer)
        {
            m_seller->getMessage(message);
        }
        if(person == m_seller)
        {
            m_buyer->getMessage(message);
        }
    }
private:
    Person* m_buyer;
    Person* m_seller;
};
    
int main()
{
    Person* buyer = new Buyer;
    Person* seller = new Seller;
    Mediator* houseMediator = new HouseMediator;
    buyer->setMediator(houseMediator);
    seller->setMediator(houseMediator);
    houseMediator->setBuyer(buyer);
    houseMediator->setSeller(seller);
    buyer->sendMessage("1.5?");
    seller->sendMessage("2!!!");
    return 0;
}
```
# 职责链模式
职责链模式：使多个对象都有机会处理请求，从而避免请求的发送者和接收者之前的耦合关系，将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。

主要解决：职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无需关心请求的处理细节和请求的传递，所有职责链将请求的发送者和请求的处理者解耦了。

如何解决：职责链链扣类都现实统一的接口。

关键代码：Handler内指明其上级，handleRequest()里判断是否合适，不合适则传递给上级。
```c++
enum RequestLevel
{
    One = 1,
    Two,
    Three
};
    
class Leader
{
public:
    Leader(Leader* leader):m_leader(leader){}
    virtual ~Leader(){}
    virtual void handleRequest(RequestLevel level) = 0;
protected:
    Leader* m_leader;
};
    
class Monitor:public Leader   //链扣1
{
public:
    Monitor(Leader* leader):Leader(leader){}
    void handleRequest(RequestLevel level)
    {
        if(level < Two)
        {
            cout << "Mointor handle request : " << level << endl;
        }
        else
        {
            m_leader->handleRequest(level);
        }
    }
};
    
class Captain:public Leader    //链扣2
{
public:
    Captain(Leader* leader):Leader(leader){}
    void handleRequest(RequestLevel level)
    {
        if(level < Three)
        {
            cout << "Captain handle request : " << level << endl;
        }
        else
        {
            m_leader->handleRequest(level);
        }
    }
};
    
class General:public Leader   //链扣3
{
public:
    General(Leader* leader):Leader(leader){}
    void handleRequest(RequestLevel level)
    {
        cout << "General handle request : " << level << endl;
    }
};
    
int main()
{
    Leader* general = new General(nullptr);
    Leader* captain = new Captain(general);
    Leader* monitor = new Monitor(captain);
    monitor->handleRequest(Three);
    return 0;
}
```
# 观察者模式
观察者模式：定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都要得到通知并自动更新。

主要解决：一个对象更新，其它对象也要更新。

如何解决：目标类通知函数通知所有观察者自动更新。

关键代码：在目标类中增加一个ArrayList来存放观察者们。

```c++
//数据模型为目标类，视图为观察者类。当数据模型发生改变时，通知视图类更新
class View;
    
class DataModel   //目标抽象类   数据模型
{
public:
    virtual ~DataModel(){}
    virtual void add(View* view) = 0;
    virtual void remove(View* view) = 0;
    virtual void notify() = 0;   //通知函数
};
    
class View      //观察者抽象类   视图
{
public:
    virtual ~View(){ cout << "~View()" << endl; }
    virtual void update() = 0;
};
    
class IntModel:public DataModel   //具体的目标类， 整数模型
{
public:
    ~IntModel()
    {
        clear();
    }
    void add(View* view)
    {
        auto iter = std::find(m_list.begin(), m_list.end(), view); //判断是否重复添加
        if(iter == m_list.end())
        {
            m_list.push_back(view);
        }
    }
    void remove(View* view)
    {
        auto iter = m_list.begin();
        for(;iter != m_list.end(); iter++)
        {
            if(*iter == view)
            {
                delete *iter;        //释放内存
                m_list.erase(iter);  //删除元素
                break;
            }
        }
    }
    void notify()  //通知观察者更新
    {
        auto iter = m_list.begin();
        for(; iter != m_list.end(); iter++)
        {
            (*iter)->update();
        }
    }
private:
    void clear()
    {
        if(!m_list.empty())
        {
            auto iter = m_list.begin();
            for(;iter != m_list.end(); iter++)  //释放内存
            {
                delete *iter;
            }
        }
    }
private:
    list<View*> m_list;
};
    
class TreeView:public View  //具体的观察者类   视图
{
public:
    TreeView(string name):m_name(name),View(){}
    ~TreeView(){ cout << "~TreeView()" << endl; }
    void update()
    {
        cout << m_name.data() << " : Update" << endl;
    }
private:
    string m_name;
};
    
int main()
{
    View* v1 = new TreeView("view1");
    View* v2 = new TreeView("view2");
    View* v3 = new TreeView("view3");
    View* v4 = new TreeView("view4");
    DataModel* model = new IntModel;
    model->add(v1);
    model->add(v2);
    model->add(v3);
    model->add(v2);
    model->add(v4);
    model->notify(); //更新所有观察者对象(View指针指向的TreeView对象)
    cout << "----------" << endl;
    model->remove(v2);
    model->notify(); //更新所有观察者对象(View指针指向的TreeView对象)
    delete model;
    return 0;
}
```