C/C++

- 野指针 : 代码中某个位置释放了内存，而另外一些地方还在使用指向这块内存的指针，那么这些指针就变成了所谓的"野指针"或"悬空指针"。
- 智能指针 : 使用了内存引用计数的方法，类似于netty的refCnt
  - scoped_ptr : 所指向的对象在作用域之外会自动得到析构
  - shared_ptr : shared_ptr中所实现的本质是引用计数(reference counting)，也就是说shared_ptr是支持复制的，复制一个shared_ptr的本质是对这个智能指针的引用次数加1，而当这个智能指针的引用次数降低到0的时候，该对象自动被析构
  - weak_ptr : weak_ptr在指向一个对象的时候不会增加其引用计数，因此你可以用weak_ptr去指向一个对象并且在weak_ptr仍然指向这个对象的时候析构它
- 菱形继承是多重继承中跑不掉的，Java拿掉了多重继承，辅之以接口。C++中虽然没有明确说明接口这种东西，但是只有**纯虚函数的类可以看作Java中的接口**。在多重继承中建议使用“接口”，来避免多重继承中可能出现的各种问题
- 虚继承是一种机制，类通过虚继承指出它希望共享虚基类的状态。对给定的虚基类，**无论该类在派生层次中作为虚基类出现多少次，只继承一个共享的基类子对象，共享基类子对象称为虚基类。**虚基类用virtual声明继承关系就行了。这样一来，D就只有A的一份拷贝。**最典型的应用就是iostream继承于istream和ostream，而istream和ostream虚继承于ios**
- 虚继承只是解决了菱形继承中派生类多个基类内存拷贝的问题，并没有解决多重继承的二义性问题
- C++**的编译器应该是保证虚函数表的指针存在于对象实例中最前面的位置**
- java语言中不存在全局变量或全局函数，而c++兼具面向过程和面向对象编程的特点，可以定义全局变量和全局函数
- 与c/c++语言相比，java语言中没有指针的概念，这有效防止了c/c++语言中操作指针可能引起的系统问题，从而使程序更安全
- 与c/c++语言相比，java语言不支持多重继承，但是java语言引入了接口的概念，可以同时实现多个接口，其中内部类也可以实现多重继承。
- OOP vs GP

  - OOP将 datas 和 methods 关联在一起
  - GP是将 datas 和 methods 分开来，Container和Algorithm可以分开开发再由iterator连接起来
- 操作符重载 : 四个操作符无法被重载 ( '::' ，'.' ，'.*' ，'?:' )       oprator++()
- 模板 template<typename T> == template<class T>

  - 类模板
  - 函数模板
  - 成员模板
  - 特化 : __STL_TEMPLATE_NULL == template<>    为了特定元素的效率
    - 全特化 : 全部绑定
    - 偏特化 : 绑定其中一个，或指针
- C++标准库

  - 容器(Containers)

    - 序列式容器 : size(), front(), back(),  

      - array : 固定大小数组 (c++11)，不可扩充

      - vector : 当容量不够时可以利用Allocator自动扩充，双倍扩充，搬运

        - heap
      - priority_queue
      
    - list : 链表 : 双向(环形)链表
      
    - forward-List : 单向链表
      
    - deque : 双向队列前后都可以添加，buffer : 前后添加buffer来扩充
      
      - queue : 利用deque实现，容器适配器
      
      - stack : 利用deque实现，容器适配器
      
    - 关联式容器 : 查找，Multi表示允许重复key，insert()

      - rb_tree
      
        - begin(), end(), insert_unique(), insert_equal()
      
        - Set/Multiset : 在元素数量超过 hash bucket 后，hash bucket 就会扩充。而且肯定有很多 bucket 会为空，可能有些 bucket 上会有很多元素
        - Map/Multimap
      
      - hashtable
      
        - unordered_ 

  - 分配器(Allocators) : 替容器分配内存，下面都是GNU C 的分配器 : __gnu_xcc::

    - array_allocator
    - mt_allocator
    - debug_allocator
    - pool_allocator ---> alloc : 8的倍数的16条内存链表
    - bitmap_allocator
    - malloc_allocator
    - new_allocator
    - operator new(), operator delete() 分别调用 malloc() 和 free()
    - gnu分配器   allocator -> alloc -> std::allocator

  - 迭代器(Iterators) : 是容器和算法之间的桥梁

    - traits : 萃取机，5种回答算法的类型，class和指针有不同的操作
      - iterator_catoragy : 分类(单向、双向)
      - difference_type : 距离(两个iterator距离)
      - value_type : 迭代器所指元素类型，算法从迭代器类中萃取出容器元素的类型
      - reference_type
      - pointer_type

  - 算法(Algorithms) : Algorithm看不见Containers，对其一无所知。所以他所需要的信息都从iterator获取

    - Algorithm(Iterator it1, Iterator it2)
    - Algorithm(Iterator it1, Iterator it2, Cmp comp)
    - qsort  (C标准库)
    - bsearch  (C) ---> binary_search() (C++)
    - find()
    - sort()
    - count_if(), count()
    - accumulate()
    - for_each()
    - replace(), replace_if(), replace_copy()
    - rbegin(), rend()    反向迭代器

  - 仿函数(Functors) : 提供给算法，重载小括号"()"，是一个对象像一个函数

    - 继承binary_function : 两个操作数，一个返回值，以便适配器改造
    - 继承unary_function : 一个操作数，一个返回值，以便适配器改造

    - 算术类
      - plus
      - minus
    - 逻辑运算类
      - logical_and
    - 相对关系类
      - equal_to
      - less

  - 适配器(Adapters) : 容器适配器，迭代器适配器，仿函数适配器　改造原来的以适配需求
- C++内存管理

  - C++ primitives

    - malloc 	free

    - new 		delete
    - new[]       delete[]
    - new() : placement new  Complex* pc = new(buf)Complex(1, 2); buf为已经分配的内存
    - ::operator new()  ::operator delete() 全局函数
    - allocator.allocate()  allocator.deallocate()
- malloc 分配的内存必须是16的倍数
- 在C中，无符号数与有符号数进行运算时，系统会自动将有符号数直接当做无符号数处理