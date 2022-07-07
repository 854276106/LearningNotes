# Effective   C++

## 习惯C++

2. ### 使用const , enum , inline替换 #define

   * 使用const常量替换#define宏

   * 使用const来定义==class专属常量==（作用域限制在class内）

   * 编译器在编译期间需要知道数组大小，这时可以使用枚举类型==enum==来替换#define

   * ```c++
     class CostEstimate
     {
     private:
         enum{NumTurns = 5 };
         int scores[NumTurns];
     }
     ```

   * 取一个const地址合法，但通常取一个enum或#define的地址是不合法的
   * 对于本体类似函数的宏，使用==inline==（内联函数）来替换#define

3. ### 尽可能使用const

   * 

4. ### 确定对象使用前已被初始化

   * 永远在使用对象之前先将它初始化

   * 构造函数内的赋值操作并不是初始化，对象的成员变量的初始化动作发生在进入构造函数本体之前

   * 总是使用初始化列表来代替赋值操作（效率高，且会初始化）

   * 总在初始化列表中列出所有成员变量，且其排列次序应与在class中的声明次序相同；

   * static对象：包括global对象，定义于namespace作用域内的对象，在class内、函数内、以及file作用域内被声明为static的对象；

   * 函数内的对象被称为==local static==对象，其他static对象被称为==non-local static==对象；解决“跨编译单元的初始化次序”问题，使用local static对象替换non-local static对象

   * 具体做法为：将每个non-local static对象搬到自己的专属函数内（该对象在此函数内被声明为static），该函数返回一个对象的引用

   * 这个做法的基础在于：C++保证，函数内的local static对象会在“该函数调用期间”“首次遇上该对象之定义式”时被初始化。

   * ```c++
     class FileSystem{...};
     FileSystem& tfS()  //用这个函数来替换tfs对象；
     {
         static FileSystem fs; 
         return fs;
     }
     class Directory{...};
     Directory::Directory(params)
     {
         ...
          std::size_t disks=tfs().numDisks;   //原本的fs对象现在改为tfs()函数
         ...
     }
     Directory& tempDir()     //如果被频繁调用，使用inline更佳
     {
         static Directory td;
         return td;
     }
     ```

   * 

## 构造/析构/赋值运算

5. ### 了解C++默默编写并调用哪些函数

   * 编译器可以暗自为class创建==默认构造函数、拷贝构造函数、重载的operator=、已经非虚析构函数==
   * 在一个内含==引用==的class内进行赋值操作时，必须定义自己的operator=

6. ### 若不想使用编译器自动生成的函数，就该明确拒绝

   * 当不想让对象被拷贝，想要拒绝编译器默认生成的拷贝构造函数时，可将相应的函数声明为private并且不予实现
   * 也可在父类中声明==拷贝构造函数==和==“operator=”==为private然后去继承，只声明不实现（在父类中声明再让子类去继承是为了防止成员函数和友元函数依旧可以操作private的拷贝构造）

7. ### 为多态基类声明virtual析构函数

   * 任何class只要带有virtual函数都几乎确定应该也有一个virtual析构函数
* 只有当class内含至少一个virtual函数，才为他声明virtual析构函数
   * 不要企图继承一个标准容器或其他任何“带有non-virtual析构函数”的class
* 当需要抽象类却又没有纯虚函数可用时，为这个类声明一个纯虚析构并提供一份定义，这个规则只适用于带多态性质的父类
  
8. ### 别让异常逃离析构函数

   *  不要在析构函数中吐出任何异常

   * 如果一个被析构调用的函数可能抛出异常，析构函数应该捕捉所有异常，然后吞下它们（不传播）或结束程序

   * ```c++
     DBConn::~DBConn()
     {
     	try
     	{
     		db.close();
     	}
     	catch(...)
     	{
     		//制作运转记录，记下对close的调用失败
     		std::abort();//结束程序   分情况调用
     	}
     }
     ```

   * 如果客户需要对某个操作函数运行期间抛出的异常做出反应，class应该提供一个普通函数（而非在析构函数中）执行该操作

9. ### 绝不在构造和析构的过程中调用virtual函数

   * 如果一个类有多个构造函数，且每个都需要执行相同的工作；避免代码重复的优秀做法是把共同的初始化代码放进一个初始化函数如init()内；
   * derived class对象内的base class成分会在derived class自身成分被构造之前先构造妥当
   * 如果base class构造函数调用了virtual函数，在实例化derived对象时会引发base class的构造，此时调用的virtual函数是base class的版本（即使要建立的对象类型是derived class的）
   * base class 构造期间的virtual函数绝不会下降到derived class阶层

10. ### 令operator=返回一个reference to *this

   * 所有与赋值相关的运算操作符，都返回一个reference指向操作符的左侧实参
   
11. ### 在operator=中处理"自我赋值"

12. ### 复制对象时勿忘记其每一个成分

   * Copying函数应该确保复制”对象内的所有成员变量“ 及 ”所有base class成分“

   * 如果你为一个class添加一个成员变量，必须同时修改copying函数。

   * 为derived class撰写copying函数，必须很小心地也复制其base class成分，让derived class的copying函数调用相应的base class函数

   * ```c++
     class Customer
     {
     public:
         Customer(const Customer& rhs);
         Customer& operator=(const Customer& rhs);
     };
     
     class PriorityCustomer: public Customer
     {
     public:
         PriorityCustomer(const PriorityCustomer& rhs);
         PriorityCustomer& operator=(const PriorityCustomer& rhs);
     
     private:
         int priority;
     };
     
     PriorityCustomer::PriorityCustomer(const PriorityCustomer &rhs)
         :Customer(rhs),              //调用base class的copy构造函数
         priority(rhs.priority)
     {
         std::cout<<"Log"<<std::endl;
     }
     
     PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer &rhs)
     {
         std::cout<<"Log"<<std::endl;
         Customer::operator=(rhs);         //对base class成分进行赋值动作
         priority=rhs.priority;
         return *this;
     }
     ```

   * 不要尝试以某个copying函数实现另一个copying函数，应该将共同的代码放进一个private的初始化函数init()中，并由两个copying函数共同调用。



## 资源管理

13. ### 以对象管理资源

    * 