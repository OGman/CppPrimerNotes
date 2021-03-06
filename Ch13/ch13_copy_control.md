# 拷贝控制
## 13.1 拷贝、赋值与销毁
### 13.1.1 拷贝构造函数
如果一个构造函数其第一个参数是所属类类型的引用，且其他参数都有默认值，则这个构造函数是拷贝构造函数。
拷贝构造函数的参数必须是所属类类型的引用。
- 合成拷贝构造函数
即使我们定义了其他拷贝构造函数，编译器也会为我们定义一个。
- 拷贝初始化
拷贝初始化不仅在使用=定义变量时发生。以下情况也会发生：
1. 传递一个非引用的形参
2. 返回一个非引用的对象
3. 用列表初始化数组或聚合类的成员
- 参数和返回值
- 拷贝初始化的限制
-编译器可以绕过拷贝构造函数
可以略过但仍需要提供拷贝构造函数。
### 13.1.2 拷贝赋值运算符
- 重载赋值运算符
重载运算符本质上是函数。需要返回类型和参数列表。
某些运算符包括成员函数必须定义为成员函数，定义为成员函数的运算符其左侧运算对象绑定到this指针。
赋值运算符一般返回其左侧运算对象的引用。标准库通常要求保存在容器中的类型有赋值运算符，且返回值为左侧运算对象的引用。
- 合成拷贝赋值运算符
### 13.1.3 析构函数
构造函数初始化非static数据成员。析构函数释放对象使用的资源，并销毁非static数据成员。
析构函数：
```C++
class Foo
{
  ~Foo();
};
```
由于析构函数不接受参数，因此它不能被重载。一个类只能有唯一一个析构函数。
- 析构函数做什么工作
- 什么时候会调用析构函数
对象被销毁的时候执行析构函数：
1. 变量在离开其作用域时被销毁
2. 类的对象被销毁时，其成员被销毁
3. 容器被销毁时，其元素被销毁
4. 动态分配对象的指针使用delete时，对象被销毁
5. 临时对象当语句执行完毕时被销毁
### 13.1.4 三/五法则
### 13.1.5 使用=default
### 13.1.6 阻止拷贝
- 定义删除的函数
```C++
struct Nocopy {
  Nocopy() = default;
  Nocopy(const Nocopy&) = delete;
  Nocopy &operator=(const Nocopy&) = delete;
  ~Nocopy() = default;
}
```
- 析构函数不能是删除的成员
析构函数如果是删除的成员，则对象无法被销毁。对于没有析构函数的类，编译器不允许创建对象。不允许定义该类的变量和创建临时对象。如果一个类的成员类型删除了析构函数，则无法定义该类型
- 合成的拷贝控制成员可能是删除的
如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则对应的成员函数将被定义为删除的。
- private拷贝控制
可以通过将拷贝构造函数声明为private来阻止拷贝，此时用户代码无法拷贝这个对象，但是友元和成员函数可以，我们将这些拷贝控制成员声明但不去定义它们，这样一旦执行拷贝操作，在链接阶段就会报出未定义的函数的错误。
声明一个成员函数但不去定义它是合法的。
用=delete的方法更先进
## 13.2 拷贝控制和资源管理
通常，定义类外资源的类必须定义拷贝控制成员。一旦一个类需要析构函数，则他一定需要一个拷贝构造函数和一个拷贝赋值运算符。一般来说，拷贝的方式有两种可选类型：使类的行为看起来像一个值或者像一个指针。类的行为像值则类在拷贝时会将内部所有的资源拷贝出一份新的，传递给新的对象，原对象与新的对象相互独立。像指针则将原有的资源传递给新的对象，二者的部分数据是共享的。
### 13.2.1 行为像值的类
行为像值的类，对于类管理的资源，每个对象都应该有一份自己的拷贝。
HasPtr需要：
定义一个拷贝构造函数，完成string的拷贝而不是拷贝指针。
定义一个析构函数来释放string
定义一个拷贝赋值运算符来释放当前对象的string，并从右侧对象拷贝string
``` C++
class HasPtr {
public:
  HasPtr(const std::string &s = std::string()):
    ps(new std::string(s)), i(0) {}
  HasPtr(const HasPtr &p):
    ps(*(p.ps)), i(p.i) {}
  HasPtr& operator=(cosnt HasPtr &);
  ~HasPtr() {delete ps;}
private:
  std::string *ps;
  int i;
};
```
- 类值拷贝赋值运算符
赋值运算符通常组合了析构函数和拷贝构造函数的操作，类似析构函数，赋值操作会销毁左侧对象的资源，类似拷贝构造函数，赋值操作从右侧对象拷贝数据。赋值运算符必须满足一种特殊情况：自赋值时不出错。这种情况的问题在于，如果我们首先析构左侧，则对象的资源将被释放，再拷贝就丢失了原有的资源。
```C++
HasPtr& operator=(const HasPtr &rhs)
{
  auto newp = new string(const HasPtr *rhs);
  delete ps;
  ps = newp;
  i = rhs.i;
  return *this;
}
```
本例使用了临时变量来保存对象内的资源
### 13.2.2 定义行为像指针的类
创建像指针的类时，我们要考虑资源的释放方式，由于可能会有很多对象共享共同的资源，因此不能随意释放指针指向的资源。
智能指针完美契合这一使用场景，资源释放由智能指针来完成可以极大地简化我们的工作。
如果我们需要直接管理资源，使用引用计数是比较合适的方式。
- 引用计数
引用计数的工作方式如下：
1. 构造函数需要创建引用计数
2. 拷贝构造函数递增引用计数
3. 析构函数递减引用计数，当引用计数为0时，释放资源
4. 赋值运算符递增右侧引用计数，递减左侧对象的引用计数
- 定义一个使用引用计数的类
```C++
class HasPtr {
public:
  HasPtr(const std::string &s = std::string()):
    ps(new std::string(s)), i(0), use(new std::size_t(1)) {}
  HasPtr(const HasPtr &p):
    ps(p.ps), i(p.i), use(p.use) {++*use;}
  HasPtr& operator=(const HasPtr&);
  ~HasPtr();
private:
  std::string *ps;
  int i;
  std::size_t *use;
};
```
其中的拷贝构造函数拷贝了类的三个数据成员，并且递增了引用计数
- 类指针的拷贝成员“篡改”引用计数
析构函数递减引用计数，如果引用计数为0，则释放ps指向的string：
```C++
HasPtr::~HasPtr()
{
  if (--*use == 0)
  {
    delete ps;
    delete use;
  }
}
```
拷贝赋值运算符递增右侧对象的引用计数，递减左侧对象的引用计数，如果引用计数为0，需要释放左侧对象的资源。
```C++
HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
  ++ *rhs.use;
  if (--*use = 0)
  {
    delete use;
    delete ps;
  }
  use = rhs.use;
  i = rhs.i;
  ps = rhs.ps;
  return *this;
}
```
## 13.3 交换操作
  对于管理资源的类通常应当定义自己的swap。对于与重排元素顺序算法一起使用的类swap是非常重要的。
  交换操作通常要做一次拷贝，两次赋值，对于两个指针：
  ```C++
    HasPtr c = a; // 拷贝
    a = b; // 赋值
    b = c; // 赋值
  ```
- 编写我们自己的swap函数
```C++
class HasPtr {
  friend void swap(HasPtr &lhs, HasPtr &rhs);
};
inline
void swap(HasPtr &lhs, HasPtr &rhs)
{
  using std::swap;
  swap(lhs.ps, rhs.ps);
  swap(lhs.i, rhs.i);
}
```
- swap函数应调用swap，而不是std::swap
## 13.5 动态内存管理类
需要动态管理内存的类一般可以使用标准库容器来实现。如果需要自己分配内存，则需要定义自己的拷贝控制成员来进行。本小节将实现一个只能存放string的vector类。
- StrVec类的设计
每个StrVec对象有三个指针：
1. elements指向分配内存中的首元素
2. first_free 指向最后一个实际元素的下一位置
3. cap 指向分配的内存末尾的下一位置
```C++
class StrVec{
  public:
    StrVec():
      elements(nullptr), first_free(nullptr), cap(nullptr) {}
    StrVec(const StrVec&);
    StrVec &operator=(const StrVec &)；
    ~StrVec();
    void push_back(const std::string &);
    size_t size() const {return first_free - elements;}
    size_t capacity() const {return cap - elements;}
    std::string *begin() const {return elements;}
    std::string *end() const {return first_free;}
  private:
    static std::allocator<std::string> alloc;
    void chk_n_alloc()
      {if (size() == capacity()) rellocate();}
    std::pair<std::string*, std::stirng*> alloc_n_copy
      (const std::string*, const std::string*);
    void free();
    void reallocate();
    std::string *elements;
    std::string *first_free;
    std::string *cap;
};
```
- 使用construct
```C++
void StrVec::push_back(const std::string &s)
{
  chk_n_alloc();
  alloc.construct(first_free++, s);
}
```
- alloc_n_copy成员
我们的StrVec类似标准库中的vector，是类值的类。
```C++
pair<string*, string*>
StrVec::alloc_n_copy(const string *b, const string *e)
{
  auto data = alloc.allocate(e - b);
  return {data, uninitialized_copy(b, e, data)};
}
```
- free成员
```C++
void StrVec::free()
{
  if (elements)
  {
    for (auto p = first_free; p != elements;)
      alloc.destory(--p);
    alloc.deallocate(elements, cap - elements);
  }
}
```
free成员有两个职责，释放所有元素，释放对象之前分配的空间。
- 拷贝控制成员
```C++
StrVec::StrVec(const StrVec &s)
{
    auto new_data = alloc_n_copy(s.begin, s.end);
    elements = new_data.first;
    first_free = cap = new_data.second;
}

StrVec::~StrVec() { free(); }

StrVec &StrVec::operator=(const StrVec &rhs)
{
    auto new_data = alloc_n_copy(rhs.begin, rhs.end);
    free();
    elements = new_data.first;
    first_free = cap = new_data.second;
    return *this;
}
```
- 在重新分配的过程中移动而不是拷贝元素
string拷贝时，会将里面的内容也进行拷贝，这时如果能避免拷贝，我们的分配效率将极大提高
- 移动构造函数和`std::move`
string定义了移动构造函数
std::move，定义在utility头文件中，移动构造函数必须与std::move配合使用，使用std::move时我们不使用using声明，而是直接使用std::move
- reallocate成员
```C++
void StrVec::reallocate()
{
    auto newcapacity = size() ? 2 * size() : 1;
    auto newdata = alloc.allocate(newcapacity);
    auto dest = newdata;
    auto elem = elements;
    for (size_t i = 0; i !=size(); ++i)
        alloc.construct(dest++, std::move(*element++));
    free();
    elements = newdata;
    first_free = dest;
    cap = elements + newcapacity;
}
```
## 13.6 对象移动
不必要的对象拷贝会耗费大量的时间，C++11标准中引入了对象移动的新特性来避免这点。标准库容器、string和shared_ptr类既支持移动也支持拷贝，unique_ptr类只支持移动。
### 13.6.1 右值引用


