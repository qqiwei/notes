## 《C++ Primer 5th》 笔记



### :rocket: 基础

#### 指针、引用与常量

建议：如果不改动入参，就要尽量声明出来，比如，用常量类型接收函数的入参，或者，用引用代替指针。

**引用**和指针区别很小，它的本质是没有空指针引用的问题的顶层常量指针。**指针**分顶层和底层，对应的常量限定符，要分别加在`*`的右边和左边，分别表示不指向其他地址（类似引用）和不改变指针地址所指向的内容。

`constexpr`是一种更智能的`const`，它不要求表达式在编译期一定能获取到结果，但只要调用点在接受常量的地方，编译器就会为我们替换成能够获取到的值。

#### 型别推导

建议：多用`using `（代替`typedef`）来重命名类型。多用`auto`和`decltype`定类型。

`decltype()`中的对指针的解引用以及增加的括号都将被看作是在声明引用类型。`auto`声明的类型，则会提取指针对象的底层类型限定，并舍弃顶层限定。

**函数指针**在赋值时可以不带取地址符，在取函数来调用的时候可以不带解引用符。但是`decltype`声明抓取的不是函数指针而是函数，因此在获取谓词（断言，参见可调用对象章节）的时候，要多带一个解引用符号。

#### 定义与声明

为了把程序拆成多个文件来相互引用（易读性、复用），C/C++支持**分离式编译**。

为了支持分离式编译，C/C++将变量和函数的声明和定义分开来。只声明不定义要加关键字`extern`，否则不显式初始化也会被编译器当作定义。

为了避免预处理的时候引入多次声明，C/C++引入头文件保护符`#ifndef`、`#define`和`#endif`，从而使变量和函数只被声明一次。

建议：类作为一种类型，一般定义在同名头文件中。函数和变量的定义文件中，一般也要引入其声明所在文件，这是为了利用编译器的声明定义一致性检查的功能。

#### 作用域与生命周期

名字有作用域，对象有**生命周期**。C++的对象是一种超过Java那种对象概念的更宽泛的概念，指程序中所有具体的数据。名字的作用域是程序文本的一部分，体现在编译完成前的可见性。对象的生命周期体现在程序执行过程中的存留时间。

函数与条件、循环语句块类似，也是一种语句块。块内的变量作用域限定在块内，生命周期则在进入和走出块作用域的时候自动（auto）生与灭，或者，如果是局部静态（static）变量，就和存在于所有函数体之外的对象一样，生命周期长至程序结束。函数参数也和`for`循环初始条件中的变量类似地都属于自动变量。

#### 编译选项与预处理变量

断言`assert(true_expr)`是一种预处理宏。可以通过定义预处理变量`NDEBUG`来关闭断言。此外，C++也有一些由预处理器定义的魔术变量：`__FILE__`、`__LINE__`、`__FUNC__`和`__TIME__`等。

```shell
CC file file.o
-o output_file_name
-c # compile only
-D NDEBUG # 定义预处理变量NDEBUG
```



### :tv: 面向对象

#### 构造函数与析构

建议：使用`= default`来生成所需要的默认构造函数的定义。

默认构造：首先使用类内初始值（声明数据成员处的初始化）来初始化它的数据成员，然后**默认初始化**不存在类内初始值的其他数据成员。如果数据成员的类型没有默认构造函数的定义，或者是内置类型，数据成员的值将是未定义的。

初始值列表：在默认初始化之后、构造函数的函数体执行前完成的，并按照它们在类中出现的顺序进行。另外，在初始值列表中属于**初始化**，在构造函数体中则属于赋值，因此，如果数据成员是`const`或引用这两种应该初始化的类型，就只能使用类内初始化或者初始值列表的方式。所以，没有任何理由要使用初始值列表之外的其他初始化方式。数据成员初始化的顺序由它们在类中定义的顺序决定，而非列表中的顺序。另外，初始值列表的位置还可以放委托构造函数。

建议：优先使用初始值列表（位于构造函数的参数列表和函数体之间，以冒号开头）来实现构造的定义。因为，部分编译器对类内初始值不提供支持；在初始值列表中属于初始化，在构造函数体中则属于赋值，因此，如果数据成员是`const`或引用类型，就只能使用类内初始值或初始值列表的方式。

析构：先执行析构函数体中的资源清理动作，然后按照初始化的逆序进行对象销毁。

类类型的隐式转换：默认通过构造函数实现，可以声明构造函数为`explicit`来阻止这种隐式转换行为。

#### 访问控制

建议：优先使用`class`而非`struct`。很多人对`struct`有误会，其实它和`class`唯一的区别就是`class`中默认的访问控制权限是`private`，而`struct`是`public`。

友元声明不是一般意义上的声明，它只是在类定义中声明有哪些外部方法或类对象可访问控制本类对象的内部成员。声明友元函数要写函数的签名，声明友元类要写`class`关键字和类名。

#### 面向对象语言中的作用域

当编译器要找一个在成员函数中出现的名字时，先查看它是否为局部变量，不是局部变量再看类内数据成员，如果不再类里面再从成员函数定义点往前看是否存在该名字的声明。如果不想用这种默认的方式，可以自己用作用域限定符（`ClassName::MemberName`和`::GlobalName`）来指明。就此可以看出，类其实可以当成一个**命名空间**来看。可以用`using`来重命名一个类型成员，比如`std::vector<int>::size_type`。

#### 常量成员函数

注：访问控制限定的是外部访问类内部成员的权限。而`const`与引用限定符是为了区分类对象自己的类型限定。

常量成员函数（`const`限定）不修改对象的数据成员，要在`const`限定声明的方法内修改一个数据成员，那这个数据成员要有一个`mutable`关键字声明。`&`和`&&`（左值与右值引用，参见拷贝控制章节）限定符声明对象是一个左值或右值。`const`与引用限定都可以区分重载版本，编译器会选出与**对象**的类型限定最匹配的方法。

#### 类的静态成员

每个`static`数据成员可以看成是类的一个对象，而不与该类定义的对象有任何关系。因此，静态成员可以是该类型的对象，而对象的成员只能是指向该类型对象的指针。**不完全类型**的内涵就是对象不能持有另外一个自己。



### :coffee: 可调用对象

#### functor

这种定义了调用操作`operator()`的类就是函数对象functor。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;
class ShortThan {
    const size_t len;

   public:
    explicit ShortThan(size_t l) : len(l) {}
    bool operator()(const string &str) { return str.length() < len; }
};
int main() {
    vector<string> strs{"apple", "banana", "pear"};
    cout << count_if(strs.begin(), strs.end(), ShortThan(5)) << endl;
}
```

#### $\lambda$ 表达式

建议：lambda 表达式可以捕获外部变量，本质就是一个临时定义的 functor，当编译器遇到 lambda 表达式时，同时定义了一个不知名的类和它的一个对象，**捕获**就是在构造过程中初始化类对象的成员数据，一旦完成拷贝。凡是需要带捕获谓词的地方，优先考虑 lambda 表达式，而非 functor（但要掌握 functor，有利于理解 lambda 表达式的本质）。凡是不带捕获的，优先考虑是否有定义一个函数的必要，这样的内存开销也许会更小。参数绑定（包括占位符）是为了复用已有的多入参函数来实现带捕获的可调用对象，占位就是谓词的元，非占位的参数就是被绑定的参数，如无复用多入参函数的必要，就使用 lambda 表达式。

注：“1小于2”，“小于”是一个二元谓词。“猫吃鱼”，可取二元谓词，也可取一元谓词“吃鱼”，“鱼”可看作是写死的，也可以看作是被捕获的变量。元指的是被断言的输入对象。

捕获分为值捕获与引用捕获（标准输入对象不支持拷贝构造，参考拷贝控制章节）。捕获的外部变量名可以列出来，也可以指示编译器自动识别：指示默认引用捕获时，显式捕获不加引用符号`[&, ci]`表示值捕获；指示默认值捕获的时候，显式捕获加引用符号`[=, &os]`表示引用捕获（类似于子类的继承访问控制与`using`重声明，参见继承的访问控制章节）。

注：引用捕获时，lambda 表达式中可以通过这个引用修改原来的入参（参考前面类对象的初始化章节）。值捕获时，捕获默认作为一个常量数据成员，如果想改动这个值，可以加一个`mutable`关键字（参考前面常量成员函数章节）。

注：从反向编译获得的名字中浏览可调用对象：`nm -C filename.o | grep "void foo"`。

```cpp
template <typename T>
void foo(T f) {}

struct Foo {
    void operator()(bool x) {}
};
struct Bar {
    void operator()(bool x) {}
};

void f1(bool x) {}
void f2(bool x) {}

int main() {
    // no-template class
    foo(Foo());

    foo(Bar());
    foo(Bar());
    // func pointer
    foo(f1);
    foo(f2);
    // lambda expr
    foo([](bool x) {});

    foo([](bool x) {});
}

// $ g++ -c hello.cpp
// $ nm -C hello.o | grep "void foo"

// 0000000000000000 T void foo<Bar>(Bar)
// 0000000000000000 T void foo<Foo>(Foo)
// 0000000000000000 T void foo<void (*)(bool)>(void (*)(bool))
// 0000000000000080 t void foo<main::{lambda(bool)#1}>(main::{lambda(bool)#1})
// 000000000000008a t void foo<main::{lambda(bool)#2}>(main::{lambda(bool)#2})
```



### :knife: 容器

注：基本的容器操作，可以先翻书看再通过做算法题来掌握，再总结一些特殊的要点和C++容器接口的风格来巩固。

#### 列表初始化

建议：列表初始化是一种很好的构造方式，赋值等操作也支持列表初始化（列表初始化的使用场景之广，从`initializer_list`也可以看出来，它的本质和Java的`String`一样，是一种不可变的`vector`，说到底，C++不支持Java那种纯粹的可变数量入参的语法糖`...`）。按照给定个数构造包含重复元素的容器也很方便。

#### 迭代器

C++容器基本操作与迭代器设计风格：正向迭代器的范围是从首节点到**尾后**节点，反向迭代器是从尾节点到首前节点。擦除只接收正向迭代器。反向迭代器有一个`base`方法用来获取对应的正向迭代器。插入操作是往迭代器指向的元素前面插入。单向链表的操作是相反的，即，往后插入或删除，并拥有首前迭代器。迭代器一般取前闭后开范围。

使迭代器失效的操作：

1. 对于`vector`(`string`)，当作容量可变的数组来看，即便不会重新分配数组，插入和删除位置之后的迭代器、指针和引用也都会失效；
2. 对于`deque`，可以当作双端动态数组来看，如果删除收尾元素，所有的迭代器、指针和引用不会失效，如果在首尾位置添加元素，迭代器会失效，已有的指针与引用不会失效。
3. 对于`list`(`forward_list`)，容器的迭代器、指针和引用都不会失效。

建议：除了频繁在首尾操作，一般操作频繁的，都选择性能更高的链表类。只是初始化后取内容看的就用向量类。

#### 泛型算法

建议：泛型算法库中的很多函数是容易实现的，比如`fill` `fill_n` `equal` `accumulate`（`numeric`库） `replace` `for_each` `copy` `replace_copy` `replace_copy_if` `find_if` `unique`等，但了解库函数会极大地提高编程效率，比如，可以用算法库中`partation`来快速实现leetcode 283或快排算法，如下为仿写与实现。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

class Solution {
   public:
    template <class ForwardIt, class UnaryPredicate>
    ForwardIt partition(ForwardIt first, ForwardIt last, UnaryPredicate p) {
        first = std::find_if_not(first, last, p);
        //  [          | f | i    ] l
        if (first == last) return first;
        for (ForwardIt i = std::next(first); i != last; ++i)
            if (p(*i)) {
                std::iter_swap(i, first);
                ++first;
            }
        return first;
    }
    template <class ForwardIt>
    void quicksort(ForwardIt first, ForwardIt last) {
        if (first == last) return;
        auto pivot = *std::next(first, std::distance(first, last) / 2);
        ForwardIt first_not = partition(first, last, [=](const auto& em) { return em < pivot; });
        quicksort(first, first_not);
        first_not = partition(first_not, last, [=](const auto& em) { return em == pivot; });
        quicksort(first_not, last);
    }
    // std::partition is not stable
    void moveZeroes(std::vector<int>& nums) {
        std::stable_partition(nums.begin(), nums.end(), [](const auto& em) { return em != 0; });
    }
};

int main() {
    std::vector<int> v{0, 1, 0, 5, 4, 3, 0, 2};
    Solution().quicksort(v.begin(), v.end());
    for_each(v.begin(), v.end(), [](const auto& em) { std::cout << em << std::ends; });

    v.assign({0, 1, 0, 3, 12});
    Solution().moveZeroes(v);
    for_each(v.begin(), v.end(), [](const auto& em) { std::cout << em << std::ends; });
}
```

1. 部分修改容器元素的库函数，默认不修改容器的容量，可能会需要用到`back_inserter(container)`指示做`push_back`操作。注：插入迭代器的行为不正常。此外，泛型迭代器还有IO迭代器，可用来从流读取/写入特定类型的元素。
2. 部分库函数，接收额外的函数指针等可调用对象，来完成对算法的定制，比如 `sort` `stable_sort`等。
3. 列表类库内有自己的 `merge`, `remove`, `reverse`, `sort` 和 `unique` 函数。

注：几乎除string类既定了使用char作为元素类型外，所有容器及泛型算法都是泛型的，需要指定元素类型。

#### 关联容器

关联是指键与值之间的关联。关联容器对象的下标操作在元素不存在的情况下会执行关联键（值）的默认初始化（相当于插入这个键），对应的`at`接口则会在元素不存在的时候抛出`out_of_range`异常，但后者一般没必要用，因为查找接口很方便。删除方法`erase`接受key，返回被删除的元素个数。查找方法`find`、`count`、`lower_bound`、`upper_bound`（两个`bound`方法，分别表示查找第一个不小于给定值，和大于给定值的元素，可见二者合起来表示一个相等元素的范围）和`equal_range`接受key，返回迭代器类型的`pair`。

#### tuple与tie

`tuple`是一种泛化的`pair`，`tie`是一种C++式的多返回类型语法糖。如下为用到了`tie`的leetcode 714实现。

```cpp
// leetcode 714 买卖股票的最佳时机含手续费
class Solution {
public:
    int maxProfit(vector<int>& prices, int fee) {
        int zero = 0, one = -prices[0]; // not buy or buy
        for (size_t i = 1, size = prices.size(); i < size; ++i) {
            tie(zero, one) = pair<int, int>(max(zero, one+prices[i]-fee), 
                                            max(zero-prices[i], one));
        }
        return zero;
    }
};

int main() {
    vector<int> vec{1, 3, 2, 8, 4, 9};
    int fee = 2;
    cout << (new Solution)->maxProfit(vec, fee);
}
```

#### 字符串

字符串类本质是一个`char`类型`vector`，C风格的字符指针可以自动转换成字符串。与Java不同，C++的字符串不是`const`的，其中的每个字符都可以像数组那样被替换。改变字符串的库函数也可以看作在改变`vector`，`append`在`end`处插入，`replace`先删再插入。`r?find`查找子串（正则表达式规则参考附录），`find_(?:last|first)(?:_not)?_of`查找字符。`to_string`和`sto(:?i|l|ul|ll|ull|f|d|ld)`用于在字符串与数值类型之间转换。正则表达式的规则参见附录章节。

```cpp
#include <iostream>
#include <string>
using namespace std;
int main()
{
    string s(string("lists."), 0);
    s.erase(s.end() - 2, s.end() - 1);

    s.append({ 's' });
    s.replace(0, 1, "a l");

    s.append(string(". end"), 0, 3);
    cout << s; // "a list.s. end"
}
```



### :shell: 文件与流

我们在写算法题的时候，一般不会涉及文件的处理，都是在内存中。但涉及到大型的软件工程，就需要文件或数据库这些外围数据节点了。同时，为了降低对内存空间的消耗，用流来读取文件。所有编程语言的IO设计模式都是大同小异的，文件操作都是在调用操作系统的能力。

注：对于写算法题来说，常用的是字符串相关的IO库。建议：基于`sstream`，`getline`，`copy`和`stream_iteartor`实现C++的`split`与`join`接口。

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <sstream>
#include <vector>
// leetcode_1451: resort words in certain text
using namespace std;
class Solution {
    static void split(const string &text, vector<string> &tokens, const char by = ' ') {
        string token;
        istringstream in(text);
        while (getline(in, token, by)) tokens.emplace_back(std::move(token));
    }
    static void join(const vector<string> &tokens, string &str, const string by = " ") {
        ostringstream out;
        copy(tokens.begin(), tokens.end(), ostream_iterator<string>(out, by.c_str()));
        str = out.str();
    }

   public:
    string arrangeWords(string text) {
        // split
        vector<string> tokens;
        split(text, tokens);
        // sort
        stable_sort(tokens.begin(), tokens.end(), 
                    [](const string &x, const string &y) { return x.size() < y.size(); });
        // join
        join(tokens, text);
        return text;
    }
};
int main() {
    cout << (new Solution())->arrangeWords("keep calm and code on") << endl;
    return 0;
}
```



### :blossom: 动态内存

智能指针是基于`use_count`对指针类型的封装。原生的`new`和`delete`、`delete []`能够完成**动态内存**管理，但智能智能类更加安全和方便，它提供了：`deleter`机制来代替默认的`delete`操作（如果所持有的指针不是动态内存指针会报错）来实现资源的自动释放、解引用`*`、解引用并取成员`->`、`swap`、`reset([p[, d]])`、`get()`等。

建议：可以用原生的`new`创建出的动态内存指针来初始化一个智能指针，但对于共享指针，优先使用`make_shared<T>(args)`方法构造对象。

注：`deleter`类是`unique_ptr`类的一部分，而`shared_ptr`可以在运行时更改它的删除器，它的实现机制可能是这样的：`del ? del(p) : delete p;`，其中`del`是删除器。

```cpp
#include <iostream>
#include <memory>

void called_locally() {
    int i = 2;
    std::shared_ptr<int> p(&i, 
        [](int *ptr) { std::cout << "before killed, I hold a number: " << *ptr << std::endl; });
    std::cout << p.use_count() << std::endl;
}

int main() { called_locally(); }
```

管理动态数组用的`allocator`，一般的使用步骤为：创建管理器、分配内存、构造元素对象、销毁元素、回收内存。

```cpp
#include <algorithm>
#include <iostream>
#include <memory>
#include <vector>
using namespace std;
int main() {
    allocator<string> alloc;
    const int size = 10;
    const auto begin = alloc.allocate(size);

    auto end = begin;
    alloc.construct(end++);
    alloc.construct(end++, "str 1");
    alloc.construct(end++, 3, 's');
    vector<string> vec{"str 3", ", ", "str 5"};
    end = uninitialized_copy(vec.begin(), vec.end(), end);
    for_each(begin, end, [](const auto i) { cout << i << endl; });
    while (end != begin) alloc.destroy(--end);

    alloc.deallocate(begin, size);
}
```

### :herb: 拷贝控制

1 拷贝（构造（初始化）、赋值）：拷贝构造函数的第一个参数是自身类类型的引用（如果不是引用就存在循环自调用），一般为`const`、非`explict`，可以携带默认参数，即使有自定义构造函数，编译器也会帮忙合成拷贝构造函数，合成的拷贝构造函数对于其数组类型的成员，将执行逐元素拷贝。拷贝初始化更重要的作用是实现一种**隐式构造**，包括在函数的非引用参数传递、非引用结果返回与列表初始化（数组或聚合类）的时候。拷贝赋值运算符接受一个与其所属类相同类型的参数（可以不为引用，一般为`const`），返回指向其左侧类型的引用，且编译器同样会默认合成，拷贝赋值运算符一般是析构（`this`）和构造（拷贝初始化）过程的结合。

2 析构函数：被自动调用的地方包括变量离开作用域、成员随其对象被销毁、元素随其容器被销毁、动态内存的`delete`以及临时对象随其所在表达式结束的时候。析构和构造的区别：类的初始化可以在初始值列表种指定，但类的析构是隐式的；同时，在继承体系中，派生类要负责基类成员的构造，但却只负责自己分配资源的回收。

3 避免拷贝：需要析构（释放动态内存或其他类外资源）的类，一般也需要定义拷贝，来避免指针地址的简单拷贝：不管是类值还是类指针的（两者的区别就是如何拷贝指针成员，前者`new`一个，而后者只是指针地址的赋值，但需要计数，也即模仿`shared_ptr`）；对于类指针的，如果使用智能指针，析构和拷贝就都不需要自定义了。同样地，为了避免拷贝，我们通常自定义`swap`接口，以最大限度地利用C++那些常量时间复杂度的`swap`函数（库函数是泛型接口，因而可以先`using std::swap`，以优先匹配非泛型的自定义的接口）。最后，**拷贝并交换**这种写法，使得`swap`接口，非常适合用于对拷贝赋值进行定义，不论是类值还是类指针的类，并同时使类获得了移动赋值运算符。

```cpp
#include "hello.hpp"
using namespace std;
class HasPtr {  // value-like
    friend void swap(HasPtr &, HasPtr &);

   public:
    // default constructor
    HasPtr(const string &s = string()) : ps(new string(s)), i(0) {}
    // common constructor
    HasPtr(const string &s, const int i) : ps(new string(s)), i(i) {}
    // copy constructor
    HasPtr(const HasPtr &hp) : ps(new string(*hp.ps)), i(hp.i) {}
    // move constructor
    HasPtr(HasPtr &&hp) : ps(hp.ps), i(hp.i) { hp.ps = nullptr; }
    // copy assignment operator
    // HasPtr &operator=(const HasPtr &hp) {
    //     auto newStr = new string(*hp.ps);
    //     // destructor
    //     delete ps;
    //     // copy constructor
    //     ps = newStr;
    //     i = hp.i;
    //     return *this;
    // }
    // copy/move assignment operator
    HasPtr &operator=(HasPtr hp) {
        // void swap(HasPtr &, HasPtr &);  // function declaration
        swap(*this, hp);
        return *this;
    }
    // destructor
    ~HasPtr() { delete ps; }
    // a getter
    string str() { return *ps; }

   private:
    string *ps;
    int i;
};
inline void swap(HasPtr &hp, HasPtr &hp1) {
    // using std::swap;  // generic api
    swap(hp.ps, hp1.ps);
    swap(hp.i, hp1.i);
};

class HasPtr2 {  // pointer-like
    friend void swap(HasPtr2 &, HasPtr2 &);

   public:
    // default constructor
    HasPtr2(const string &s = string())
        : ps(new string(s)), i(0), use_count(new size_t(1)) {}
    // common constructor
    HasPtr2(const string &s, const int i)
        : ps(new string(s)), i(i), use_count(new size_t(1)) {}
    // copy constructor
    HasPtr2(const HasPtr2 &hp) : ps(hp.ps), i(hp.i), use_count(hp.use_count) {
        ++*use_count;
    }
    // move constructor: almost like a shared_ptr takes an unique_ptr
    HasPtr2(HasPtr2 &&hp) : ps(hp.ps), i(hp.i), use_count(hp.use_count) {
        hp.ps = new string();
        hp.use_count = new size_t(1);
    }
    // copy assignment operator
    // HasPtr2 &operator=(const HasPtr2 &hp) {
    //     // desctructor
    //     if (--*use_count == 0) {
    //         delete use_count;
    //         delete ps;
    //     }
    //     // copy constructor
    //     ps = hp.ps;
    //     use_count = hp.use_count;
    //     i = hp.i;
    //     ++*hp.use_count;
    //     return *this;
    // }
    HasPtr2 &operator=(HasPtr2 hp) {
        // void swap(HasPtr2 &, HasPtr2 &);
        swap(*this, hp);
        return *this;
    }
    // destructor
    ~HasPtr2() {
        if (--*use_count == 0) {
            delete use_count;
            delete ps;
        }
    }
    // a getter
    string str() { return *ps; }
    int get_use_count() { return *use_count; }

   private:
    string *ps;
    size_t *use_count;
    int i;
};
inline void swap(HasPtr2 &hp1, HasPtr2 &hp2) {
    swap(hp1.ps, hp2.ps);
    swap(hp1.use_count, hp2.use_count);
    swap(hp1.i, hp2.i);
};

int main() {
    HasPtr hp("11", 11), hp1("12", 12), hp0;
    swap(hp, hp1);  // hp: 11->12
    hp0 = hp;       // hp0: 0->12
    cout << hp0.str() << endl;

    HasPtr2 hp3("21", 21), hp4("22", 22), hp2;
    swap(hp3, hp4);  // hp3: 21->22
    hp2 = hp3;       // hp2: 0->22
    cout << hp2.str() << ", " << hp2.get_use_count() << endl;

    cout << endl;
}
```

4 右值引用与移动：A. 赋值、下标、解引用和前置增减运算符都返回左值引用（用的是对象的身份、内存中的位置），而算术、关系、位及后置增减运算符都返回右值（用的是对象的值、内容），后者，我们不能把左值引用绑定到上面，但是，我们可以把一个`const`的左值引用绑定上去。B. 右值引用是为了给移动操作用的，只能绑定在将要销毁的对象上。我们虽然不能把右值引用绑定到左值上，但是，可以`move`可以显式地将一个左值转变成一个右值引用类型。C. 为了使自定义的类能支持像库里的类拥有的那样的移动操作，需要自定义移动构造函数与移动赋值运算符，否则，`move`会调用拷贝接口。移动操作设计的要点是：接收输入对象的内容，将输入对象置空，并且，由于不会分配资源，而是窃取，所以一般不会抛出异常，所以，最好要加上`noexcept`标注，避免标准库做额外的处理，否则，比如说，一个持有自定义类型作为元素的`vector`，它采用移动来实现数组的扩充，拷贝过程中的问题不影响原数据，但是移动会，所以，没有声明自己不会抛出错误的元素类型，`vector`不会调用该元素类型的移动操作，而是调用拷贝。D. 自定义的移动赋值运算符，需要滤过自赋值的情况，因为在接管新资源前，需要释放`this`的旧资源，而我们不能在用之前就释放那些资源。E. 合成移动操作的条件是：未定义任何拷贝操作，且每个非`static`的数据成员都是可以移动的（内置类型可移动，类类型需要有对应的移动操作）。F. 拷贝和移动的参数一般都是`const T&`和`T&&`。G. 引用限定符和`const`限定类似地作用于非`const`对象，它的作用是：如果`string`有限定赋值运算符的对象是左值的话，就不会出现`str1+str2="wow"`这种情况了；且一旦一个方法有移动标记，所有方法就都要标记。

```cpp
#include "hello.hpp"
using namespace std;
class HP {
    int i;

   public:
    int get() { return i; }
    HP() : i(0) {}
    HP(int i) : i(i) { cout << "construct with " << i << endl; }
    HP(const HP &hp) : i(hp.i) { cout << "copy with " << i << endl; }
    HP(HP &&hp) : i(hp.i) {
        hp.i = 100000;
        cout << "move with " << i << endl;
    }
    HP &operator=(HP &hp) {
        i = hp.i;
        cout << "assign with " << i << endl;
        return *this;
    }
    ~HP() { cout << "die with " << i << endl; }
};

int main() {
    HP hp();
    HP hp1 = hp();
    cout << hp1.get() << endl;

    vector<int> a{1, 2, 3};
    auto b = move(a);
    cout << b.size() << " " << a.size() << endl;
}

HP hp() {
    HP hp(9);
    // return hp;
    return move(hp);
}
```

5 移动迭代器、引用限定与常量：引用限定符和`const`（底层`const`表示不更改所引用对象的值，而顶层`const`表示不再改去引用其他对象；在 C++11 新增的型别推导中，`auto`只取底层，`decltype`会连顶层一块取）一样可以可以区分重载版本，且置于`const`之后，且具有相同的名字和参数列表的两个方法必须同时添加或不添加引用限定符。类似`const`，引用限定符作用于的同样是`this`对象。

### 继承与可访问性

1 继承与动态绑定、可访问性：A. 与Java不同，C++需要显式地将基类方法标记为`virtual`以启用动态绑定，并且要求是指针或引用类型的变量（虚析构函数也利用这一机制）。B. 关于数据成员的继承，派生类完全具备基类的内容，但不一定有访问权限，`protected`则是一种真正意义上的权限“继承”：外部无权访问，但派生类可以访问；同时，派生类中只能访问派生类自己所有的那部分继承过去的成员，而不能获取基类对象的该成员。C. 继承中的方法的实现方式，本质上是这些方法隐式地持有一个`const`的类对象引用。如果将派生类当作/强制转换（非指针或引用）为基类的话，派生类中非基类的部分将被“切掉”。同时，如果想使用特定被继承类的`virtual`方法，则可以加入类作用域符号。D. 在不同作用域中无法重载函数名。派生类的作用域嵌套在其基类的作用域之内（除了部分基类内容受访问控制的限制不能访问外）。因此，当派生类重用基类的名字时，基类的同名（甚至方法参数不一样也如此）内容就被覆盖了，但我们仍然可以通过类作用域符号引用那些被隐藏的内容。

2 阻止继承或阻止生成对象：C++和Java一样用`final`表示类不可继承，但Java的`final`还具有类似C++`const`关键字的功能，而`const`虽然也是Java关键字，却未被使用。Java使用`abstract`表示纯虚函数，而C++使用`=0`，纯虚函数表示该类不定义该方法，该类也无法生成实例对象。

3 访问控制：特别值得注意的是，访问控制限定的是其他类（包括其派生类）的访问权限，类内方法中自然没有任何限制。A. 派生列表限定符的作用是，对继承来的成员在被其他类访问的时候进行限定，是派生类在基类自有访问控制之上添加的访问控制（如果原来派生类就没有该继承来的成员的访问权限，新限定也会无从谈起）。B. 可以使用`using`声明在访问说明符之上重新规划访问控制，类似于在隐式捕获上特别指定一些显式捕获。

4 容器与继承：面向对象特性要依靠指针和引用这件事儿确实很不直接，同时，还导致另一个问题，就是容器中存储在继承体系中的一些对象的时候，要用指针或共享指针，否则，子类对象会被截断成父类对象。可以把一个派生类的智能指针转换成基类的智能指针，内部做的应该是指针类型的强制转换。



### 附录

#### 正则表达式

正则，就是用字符来描述字符。正则表达式里用的字符，可以分为两类：

1. 通用编程语言支持的字符：普通字符（包括空格）和 通用编程语言的转义字符（包括制表符、换行符、换页符）

2. 正则表达式占据的字符：

	1. 表示匹配的规则，被正则占用，想表示原字符要加转义符，需要转义的符号有：

		1. 字符个数的限定：`* +` `{n, m}` `?` 

			`?`还用于表示非捕获，可能要结合`: ! <`之一

		2. 字符集：`[]`

		3. 子表达式：`()`

		4. 行首行尾：`^`（还用于表示非匹配）、`$`

		5. 或：`|`

	2. 正则表达式定义的转义符号，类似通用编程语言的，用来表示一类符号（`\w` `\d` `\s` `\b` 等），为了与通用编程语言的转移符号相区别，可使用`raw`字符串以关闭编程语言层面的转义。对于非正则表达式定义的转移符号，反斜杠将会被忽略。

		`\b` 匹配单词的边界，`\B` 匹配非单词字符的边界，单词为`\w`（包括字母和`\d`），边界意为 `\s`（包括空格与制表符、换行符等），`.` 意为 `[^\n\r]`（正则表达式一般匹配一行）

正则表达式中 [不] 捕获子表达式的规则：

1. 在正则表达式中，反斜杠加数字序号，表示被捕获的子表达式（一旦匹配完成了，一般就用美元符号获取该子表达式，比如在IDE的字符串替换中使用）
2. `?`表示**非捕获**，`?`结合`:`表示不捕获但匹配，`?`结合`= ! <`表示不捕获也不匹配（预查是否匹配，不消耗字符）
	0. `(?:exp)` 不捕获但匹配，比如 `fl(?:y|ies)`匹配单复数的苍蝇的英文
	1. `exp1(?=exp2)`： 正向肯定预查（后面是 `exp2` 吧？）
	2. `exp1(?!exp2)`： 正向否定预查（后面不是 `exp2` 吧？）
	3. `(?<=exp2)exp1`： 反向肯定预查（前面是 `exp2` 吧？）
	4. `(?<!exp2)exp1`： 反向否定预查（前面不是 `exp2`）



1. Why use no `namespace std`? - [check it](https://www.bilibili.com/video/BV1VJ411M7WR?p=60)
   1. I could know what do I use from the standard libaray if I use `std::`.
   2. Symbols from different namespaces may conflict with each other if I simply declare `using namespace`.
   3. Use namespaces for personal libraries, smaller scopes and needed only.
ps: namespace is simply created for solving comfliction between names, and the cpp classes are like namespaces as well.