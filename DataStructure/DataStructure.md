<!--
 * @Author: your name
 * @Date: 2020-06-15 19:36:05
 * @LastEditTime: 2020-06-19 21:24:24
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \undefinedc:\Users\conan\Desktop\LongTime\StupidBirdFliesFirst\DataStructure\DataStructure.md
--> 
# 数据结构
![](datastructure.png)
## 线性表
线性表就是n个具有相同特性的数据元素的有限序列。
### 数组
c++中的数组一般用int[]来表示，可以存储一组相同类型的数据。数组在声明时必须预留出空间，在使用前要申请空间，所以很有可能浪费内存。数组所占用的空间是一段连续的区域，所以所有的数据的地址都是连续的。数组很难在中间插入数据，但是读取的效率比较高。

初始化数组：
```c++
double balance[5] = {1000.0, 2.0, 3.4, 7.0, 50.0};
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0};
```

读取：
```c++
balance[4] = 50.0;
double salary = balance[9];
```

在STL中，array是可以代替数组的，与数组的基本特点一致。针对数组和array无法变长的缺点，STL中提供了vector。

### 链表
链表虽然也是将相同的数据存储成线性模式，但是它与数组有很大不同，链表的存储空间是离散的，并不会占用一整块连续的空间，所以每个元素除了存储自己的值以外还要存储下一个元素的地址。链表比较容易在中间插入或者删减数据，但是读取效率很低，因为只能从头到尾遍历。

C++链表的一般写法：
```c++
 struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};
```

STL中list可以代替链表。

### 数组和链表的区别
- 存储：数组时连续的内存空间，链表不需要连续
- 长度：数组的长度需要预先确定，若超出数组则会溢出；链表的长度是动态扩展的
- 随机访问：即按照需要访问线性表中第n个元素。数组可以随机访问，时间复杂度为O(1)；链表不支持随机访问，平均需要O(n)
- 插入、删除：数组其实位置的插入和删除， 时间复杂度为O(n)；链表的时间复杂度为O(1)

### 队列
队列的最大特点就是依照存储顺序，先进先出。一般改变队列中元素的操作方法只有两个——push与pop。push是把元素从队尾插入，pop是把元素从队头删除。

#### 队列的数组实现
队列的数组实现有两种形式，一种是线性实现，另一种是循环实现。

线性实现如下所示，front表示当前队列的队首，rear表示当前队列的队尾。因为数组必须要提前声明占用的空间，所以线性实现有一定的局限性，当rear抵达max-1的时候就不能再让元素入队了。此时front前面的空间将被浪费，因为前面的都出去了，然后就没法再被使用。
![](listarray.jpg)

循环实现就是把一个数组看成一个循环的圆，当rear或front抵达max - 1的时候，再前进，将回到0处。这样一来，就能对数组的空间有充分的利用。当然实际上的数组不可能是个环，实现这个环的方式是head和rear对数组长度求余数，这样保证head和rear的序号永远在一个固定范围内。
![](listarrayround.jpg)

在实现的时候，需要区分如何判断队列已满或者队列已空。一般的判断准则是当rear的下一个位置是front的时候，则队列已满。当rear和front的位置重叠，则队列已空。

```c++
template <class T>
class Queue {
private:
    static const int maxqueue = 10; // 所占用的最大空间
    T entry[maxqueue + 1];  // 分配比maxqueue + 1的空间
                            // 实际上队列的可用空间还是maxque
    int head; // 队首序号
    int rear; // 队尾序号
    int queueSize; // 队列尺寸
 
    bool full() const {
        return (((rear + 1) % (maxqueue + 1)) == head) // 求余都是为了让rear和front的范围限定在[0, 10]
            && (queueSize == maxqueue);
    }

public:
    Queue() : head(0), rear(0), queueSize(0) {
        for (int i = 0; i < maxqueue + 1; i++)
            entry[i] = T();
        }

    // 下面求余都是为了让rear和front的范围限定在[0, 10]

    void push(const T &val) {
        if (!full()) {
            entry[rear] = val;
            rear = (rear + 1) % (maxqueue + 1);// 插入数据以后rear再向后一个，为了防止rear+1以后比10大，所以要求余
            ++queueSize;
        }
    }

    void pop() {
        if (!empty()) {
            head = (head + 1) % (maxqueue + 1);
            --queueSize;
        }
    }

    T front() const {
        if (!empty())
        return entry[head];
    }

    T back() const {
        if (!empty())
        // rear-1是队列最后一个元素的位置
        // rear-1可能是-1，所以将其跳转到maxqueue位置
      return entry[(rear - 1 + (maxqueue + 1)) % (maxqueue + 1)];
    }

    bool empty() const {
        return ((rear + 1) % (maxqueue + 1) == head) && (queueSize == 0);
    }

    int size() const {
        return queueSize;
    }
};
```

#### 队列的链表实现
链表的实现总体来说就简单理解一些，利用两个结点，head记录链表头即队头，rear记录链表尾即队尾。
```c++
template <class T>
class Queue {
private:
    struct Node {
        T data;
        Node *next;
    };

private:
    Node *head;
    Node *rear;
    int queueSize;

public:
    Queue() : head(NULL), rear(NULL), queueSize(0) { }

    ~Queue() {
        while (!empty())
        pop();
    }

    void push(const T &val) {
        Node *new_node = new Node;
        new_node->data = val;
        new_node->next = NULL;
        if (head == NULL && rear == NULL) {
            head = rear = new_node;
        } else {
            rear->next = new_node;
            rear = rear->next;
        }
        ++queueSize;
    }

    void pop() {
        if (!empty()) {
            Node *temp = head;
            head = head->next;
            delete temp;
            --queueSize;
        }
    }
 
    T front() const {
        if (!empty())
        return head->data;
    }

    T back() const {
        if (!empty()) return rear->data;
    }

    bool empty() const {
        return queueSize == 0;
    }

    int size() const {
        return queueSize;
    }
};
```