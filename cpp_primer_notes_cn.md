## C++ Primer 笔记



### :rocket: 基础

#### 指针、引用与常量限定

建议：如果不改动入参，就要尽量声明出来。即，用常量类型接收函数的入参，以及，用引用代替指针。

**引用**的本质是没有空指针引用的问题的顶层常量指针，顶层常量就是不指向别的对象的指针。

指针的**常量限定**分两种：顶层和底层（顶层常量限定的`const`写在解引用符的右边：`*const`），分别表示不指向其他地址（类似引用）和不改变所指向地址处的内容。`constexpr`可以看作是一种更智能的`const`，它只要求编译器在编译期能算出值就行。另外，它对指针类型进行的是顶层常量限定，要同时限定底层常量的写法是：`constexpr const int *p`。

#### 型别推导

建议：用`using `（代替过去的`typedef`）来重命名类型。多用`auto`和`decltype`定类型，少人为写死类型。

型别推导是单纯的语法糖，目的是通过自动化获取类型来减轻开发的负担，它的规则有：

1. 当`decltype(expr)`中的表达式，是对指针类型的解引用，或者包了括号，都将被编译器用作声明引用类型。
2. `auto`会获取表达式的底层常量限定，这与`constexpr`被作为顶层常量限定正好相反。
3. 函数指针在赋值时可以不带取地址符，在取函数来调用的时候可以不带解引用符。但`decltype`声明抓取的不是函数指针而是函数，因此，在通过`decltype`获取函数指针类型的时候，要多带一个解引用符号。

#### P1 声明、定义、头文件

1. 为了把程序拆成多个文件来相互引用（易读性、复用），C/C++支持分离式编译。
2. 为了支持**分离式编译**，C/C++将变量和函数的声明和定义分开来。只声明不定义要加关键字`extern`，否则不显式初始化也会被编译器当作定义（就像默认初始化一个类对象不需要带调用符号）。
3. 为了避免预处理头文件引入的时候引入多次声明，C/C++支持头文件保护符`#ifndef`、`#define`和`#endif`，从而使变量和函数只被声明一次。

注：`include`只是在预处理的时候把被导入的文件拷贝到`include`的位置。声明与定义都只能有一次。

建议：类作为一种类型，一般定义在同名头文件中。

建议：定义函数和变量的文件中，一般也要引入其声明所在文件，以利用编译器检查声明和定义的一致性。

```cpp
// a.cpp
#include "a.hpp"

static int i = 56;
int j = 66;
```

```cpp
// a.hpp
#ifndef ACPP
#define ACPP 1
#include "a.cpp"

extern int i;
extern int j;
#endif
```

```cpp
// b.cpp
#include <iostream>

#include "a.hpp"

int main() {
    std::cout << "main()" << std::endl;
    std::cout << i << std::endl;
    std::cout << j << std::endl;
}
```

#### P2 作用域、生命周期

名字有作用域，对象（C++世界中的对象，一般具有超过面向对象编程概念里的对象的更宽泛的指涉，即，它指向程序中所有具体而非模板或类型的数据，地址数据也是数据）有生命周期。声明的是名字，定义的是对象，声明和定义在同一处的叫初始化。

1. 名字的作用域是程序文本的一部分，体现在编译的可见性。
2. 对象的生命周期体现在程序执行过程中的存留时间。

函数与条件、循环语句块类似，也是一种**语句块**。同时，与`for`循环初始条件中的变量定义类似，函数参数也属于自动变量。块内的自动变量作用域限定在块内，生命周期也限定在进程进入和走出块作用域期间。但如果是局部静态（static）变量，就和存在于所有函数体之外的对象一样，生命周期长至进程结束。

对于所有函数之外的`static`变量来说，C语言将其作用域限定在定义它的文件中，而另一个文件中的`extern`声明将无法获取其定义。C++标准消除了这个限制，并用未命名的命名空间来实现它，未命名的命名空间的作用域为其所在作用域。

#### 编译选项、预处理变量

断言`assert(true_expr)`是一种预处理宏。可以通过定义预处理变量`NDEBUG`来关闭断言，编译选项也可以重置该变量。此外，C++也有一些别的由预处理器定义的魔术变量：`__FILE__`、`__LINE__`、`__FUNC__`和`__TIME__`等。

```shell
CC 
 -c # compile only
 -o output_file_name
 -D NDEBUG # 定义预处理变量NDEBUG
```



### :tv: 面向对象

#### 构造、隐式构造与析构

建议：在已定义了构造函数的情况下，使用`= default`来生成所需要的默认构造函数的定义。

构造就是对象的数据成员定义的过程，进程将按照类中成员出现的先后顺序对成员数据进行构造（成员对象的析构将按相反的顺序被调用）。成员`public`的默认构造函数将被调用，除非在初始值列表中有显式的构造调用。如果没有可用的构造函数，成员的值将是未定义的，类似没有赋值的内置类型的自动变量。

建议：优先使用**初始值列表**对成员数据进行构造。初始值列表在进入构造函数函数体之前进行，且它调用的不是赋值（构造函数体内只能赋值了）而是初始化构造，因此，对于数据成员是`const`或引用限定的情况下，就只能使用初始值列表或类内初始化（依赖数据成员的默认构造），而某些编译器可能还不支持类内初始化。

除了显式调用，构造函数经常被隐式调用，**隐式构造**的场景有：函数非引用限定的参数传递（想要在参数传递中禁用参数类型的隐式类型转换可以给形参加一个`explict`限定）与结果返回，数组、聚合类、容器的列表初始化，隐式类型转换。

析构的作用是清理额外资源，比如动态申请的内存与文件操作符，而数据成员所占内存的回收不归开发者管。

析构可以被显式调用，但一般来说，依赖（在**销毁**该类型的对象前）析构函数的自动调用就够了，释放一个对象的场景有：变量离开作用域、成员随其对象被销毁、元素随其容器被销毁、动态内存的`delete`以及临时对象随其所在表达式结束而被销毁。析构、成员对象销毁的顺序是构造顺序的逆序。

析构要定义为`public`的，否则，在该类定义的外部，该类型的对象自动调用其析构函数将产生可访问性的错误。

```cpp
#include <iostream>

class IN {
   public:
    IN() { std::cout << "IN" << std::endl; }
    IN(int i) { std::cout << "IN i" << std::endl; }
    ~IN() { std::cout << "~IN()" << std::endl; }
};
class IN2 {
   public:
    IN2() { std::cout << "IN 2" << std::endl; }
    ~IN2() { std::cout << "~IN2()" << std::endl; }
};
class OUT {
    IN2 in2;
    IN in;

   public:
    OUT() : in(8) { std::cout << "OUT" << std::endl; }
    ~OUT() {
        in.~IN();
        std::cout << "~OUT()" << std::endl;
    }
};

int main() {
    OUT out;
    std::cout << "main()" << std::endl;
}
```

#### 访问控制、友元与继承

注：C++中，`class`与`struct`唯一的区别就是`class`中默认的访问控制权限是`private`，而`struct`是`public`。

注：C++继承的动态绑定是需要用指针、引用类型来实现的。

访问控制限定的是外部类型对本类型的可访问性。

1. 友元是一种指定类型或函数的`public`访问控制。友元声明不是一般意义上的声明，它只是在类定义中声明有哪些外部方法或类可访问本类的内部成员。声明友元函数用函数的签名，声明友元类则用`class`关键字和类名。
2. 继承：子类是外部类型，所以，子类在其内部可以访问其继承自父类的`protected`成员，但不能通过父类来获取父类对象的该成员。子类继承（拥有）父类的全部数据，但在自己的空间内不一定有**访问权限**。
3. 在继承体系内，子类定义起始行的派生列表限定符的作用也是对继承来的成员在被其他外部类访问的时候进行限定，是派生类在基类自有访问控制之上添加的访问控制，如果原来派生类就没有该继承来的成员的访问权限，即`private`限定，则新限定也就无从谈起。可以使用`using`声明屏蔽访问说明符的作用并重新规划访问控制权限，类似于在`lambda`表达式的隐式捕获的基础上，特别指定个别参数的显式捕获。

#### 面向对象的作用域**

当编译器要找一个在成员函数中出现的名字时，先查看它是否为局部变量，不是局部变量再看类内数据成员，如果不在类里面再从成员函数定义点（成员函数可能定义在类外部）往前看是否存在该名字的声明。除了让编译器去识别，也可以通过作用域限定符来指明：`ClassName::MemberName`和`::GlobalName`，因为，类其实是一个**命名空间**，比如`std::vector<T>::size_type`就是在`vector`类内通过`using`定义的一个`unsigned`的类型别名。

#### P1 拷贝、赋值运算符与拷贝控制

WHAT 拷贝构造函数的第一个参数是（一般为非`explict`、`const`限定，可指定默认参数的）自身类类型的引用（如果不是引用就存在无限递归）；拷贝赋值运算符接受一个与其所属类相同类型的参数（可以不为引用，一般为`const`）并返回指向其左侧类型的引用，它的本质是析构（、销毁）与构造的结合。

类似默认构造函数，如无定义，编译器默认会合成拷贝构造函数与拷贝赋值运算符。因为，拷贝是隐式调用默认的手段，而隐式调用就是一些经常发生的隐式构造、赋值。

需要析构（释放动态内存或其他类外资源）的类，一般也需要定义拷贝，来避免指针地址（指向**资源**）的简单拷贝：不管是类值还是类指针的（两者的区别就是如何拷贝指针成员，前者`new`一个，而后者只是指针地址的赋值，但需要计数，也即模仿`shared_ptr`）；对于类指针的，如果使用智能指针，析构和拷贝就都不需要自定义了。

建议：很多算法（比如原址排序）要用到`swap`操作（它的本质是一次初始化和两次赋值），为了避免默认泛型交换函数执行默认的拷贝操作，我们通常自定义`swap`接口以减少内存分配操作（尤其是类值的）。

注：很多时候，我们习惯于用`std::`标明任何来自标准库的名字以降低冲突的可能性，但此处，我们习惯于在使用交换操作的块作用域中的首先声明`use std::swap;`，然后使用不加限定的`swap`以使自定义接口获得优先匹配。

赋值操作的本质是释放原对象、持有一个新对象，“拷贝并赋值”（选择普通可修改参数来定义拷贝赋值）使用自定义的`swap`接口可极大地简化其实现，并潜在地具有效率优势，不论是类值的还是类指针的，它的原理是：将`*this`置换到作为自动变量的入参上以实现隐式的资源释放。

```cpp
#include <iostream>
#include <string>

class Exclusive {  // contains exclusive(value like) resource
    friend void swap(Exclusive &, Exclusive &);

   public:
    // normal constructor
    Exclusive(const std::string &s = "") : res(new std::string(s)), i(0) {}
    Exclusive(const std::string &s, const int i) : res(new std::string(s)), i(i) {}
    // copy constructor
    Exclusive(const Exclusive &hp) : res(new std::string(*hp.res)), i(hp.i) {}
    // move constructor
    Exclusive(Exclusive &&hp) : res(hp.res), i(hp.i) { hp.res = nullptr; }
    // copy assignment operator
    // Exclusive &operator=(const Exclusive &hp) {
    //     delete res;  // gc
    //     // construct
    //     res = new std::string(*hp.res);
    //     i = hp.i;
    //     return *this;
    // }
    Exclusive &operator=(Exclusive hp) {  // construct
        swap(*this, hp);
        return *this;  // gc
    }
    // desctructor
    ~Exclusive() {
        show(false);
        std::cout << ": auto gc" << std::endl;
        delete res;
    }
    // printer
    void show(bool eofl = true) {
        if (res != nullptr)
            std::cout << "show(): " << *res << " with id " << i;
        else
            std::cout << "show(): exclusive res moved";
        if (eofl) std::cout << std::endl;
    }
    // setter
    Exclusive &seti(int i) {
        this->i = i;
        return *this;
    }

   private:
    std::string *res;
    int i;
};
inline void swap(Exclusive &hp, Exclusive &hp1) {
    using std::swap;
    swap(hp.res, hp1.res);
    swap(hp.i, hp1.i);
};

class Shared {  // contains shared(pointer like) resource
    friend void swap(Shared &, Shared &);

   public:
    // constructor
    Shared(const std::string &s = "") : res(new std::string(s)), i(0), use_count(new size_t(1)) {}
    Shared(const std::string &s, const int i) : res(new std::string(s)), i(i), use_count(new size_t(1)) {}
    // copy constructor
    Shared(const Shared &hp) : res(hp.res), i(hp.i), use_count(hp.use_count) { ++*use_count; }
    // move constructor
    Shared(Shared &&hp) : res(hp.res), i(hp.i), use_count(hp.use_count) {
        hp.res = nullptr;
        hp.use_count = new size_t(1);
    }
    // copy assignment operator
    // Shared &operator=(const Shared &hp) {
    //     if (--*use_count == 0) {  // gc
    //         delete use_count;
    //         delete res;
    //     }
    //     // construct
    //     res = hp.res;
    //     use_count = hp.use_count;
    //     i = hp.i;

    //     ++*hp.use_count;
    //     return *this;
    // }
    Shared &operator=(Shared hp) {
        swap(*this, hp);
        return *this;
    }
    // destructor
    ~Shared() {
        show(false);
        if (--*use_count == 0) {
            std::cout << ": auto gc" << std::endl;
            delete use_count;
            delete res;
        } else
            std::cout << std::endl;
    }
    // printer
    Shared &show(bool eofl = true) {
        if (res != nullptr && use_count != nullptr)
            std::cout << "show(): " << *res << " with id " << i << ", cnt " << *use_count;
        else
            std::cout << "show(): shared res moved";
        if (eofl) std::cout << std::endl;
        return *this;
    }
    // setter
    Shared &seti(int i) {
        this->i = i;
        return *this;
    }

   private:
    std::string *res;
    size_t *use_count;
    int i;
};
inline void swap(Shared &hp1, Shared &hp2) {
    using std::swap;
    swap(hp1.res, hp2.res);
    swap(hp1.use_count, hp2.use_count);
    swap(hp1.i, hp2.i);
};

int main() {
    Exclusive hp1("exclusive res 1", 1), hp2("exclusive res 2", 2), hp0("exclusive res 0");
    std::cout << "\n---swap(hp1, hp2); (hp0 = hp1).seti(0);---" << std::endl;
    swap(hp1, hp2);
    (hp0 = hp1).seti(0).show();
    std::cout << "\n---hp0 = std::move(hp2);---" << std::endl;
    hp0 = std::move(hp2);

    Shared hp3("shared res 3", 3), hp4("shared res 4", 4), hp00("shared res 00");
    std::cout << "\n---swap(hp3, hp4); (hp00 = hp3).seti(0);---" << std::endl;
    swap(hp3, hp4);
    (hp00 = hp3).seti(0).show();
    std::cout << "\n---hp00 = std::move(hp4);---" << std::endl;
    hp00 = std::move(hp4);

    std::cout << "\n---exit---\n" << std::endl;
}
```

```
---swap(hp1, hp2); (hp0 = hp1).seti(0);---
show(): exclusive res 2 with id 0
show(): exclusive res 0 with id 0: auto gc

---hp0 = std::move(hp2);---
show(): exclusive res 2 with id 0: auto gc

---swap(hp3, hp4); (hp00 = hp3).seti(0);---
show(): shared res 4 with id 0, cnt 2
show(): shared res 00 with id 0, cnt 1: auto gc

---hp00 = std::move(hp4);---
show(): shared res 4 with id 0, cnt 2

---exit---

show(): shared res 3 with id 3, cnt 1: auto gc
show(): shared res moved: auto gc
show(): shared res 4 with id 4, cnt 1: auto gc
show(): exclusive res 1 with id 1: auto gc
show(): exclusive res moved: auto gc
show(): exclusive res 2 with id 2: auto gc
```

#### P2 移动、右值引用

`std::move`做的是强制类型转换，它将（左值）类型转变成右值引用类型。而右值引用是种类似于引用的**修饰符**，表示对象的内容是可被窃取的，即，在移动拷贝、移动赋值运算符接收到一个右值引用类型的入参时，它们应该窃取这个被引用对象的内容。`std::move`可用在非引用返回值、赋值运算符的右侧对象之上，借用或隐式或显式的移动操作，直接窃取被移动对象的内容。因为被移动的对象可能随即被析构、销毁，所以移动操作（移动拷贝、移动赋值运算符）应该在获取到被移动对象的资源后，将被移动对象的指针置空，也因为需要置空，所以，移动操作要校验自移动。

赋值、下标、解引用和前置增减运算符返回左值引用（用的是对象的身份、内存中的位置），而算术、关系、位及后置增减运算符返回右值（用的是对象的值、内容）。只有`const`左值引用可以持有右值，因为它不会破坏右值的常量属性。

注：当自定义类未定义任何拷贝操作的时候，编译器才会合成移动操作，且要求类的数据成员都可移动（成员是内置类型或是定义了移动操作的类）。移动操作不可用时，对应的拷贝操作就会代替移动操作被调用。

注：移动操作最好加一个`noexcept`关键字，否则，比如，标准库的`vector`还是会调用拷贝操作符以将原数据赋值到扩容的新数组上。因为`vector`调用的是`std::move_if_noexcept`，它依条件调用`std::move`或返回`const &`类型。

#### 成员的`const`、引用限定**

访问控制限定的是外部的类访问类内成员的权限，而引用与`const`限定符是为了声明类方法对类数据成员的修改与否。

常量成员函数（`const`限定）不修改对象的数据成员，要在`const`限定声明的方法内修改一个数据成员，那这个数据成员要有一个`mutable`关键字声明。`&`和`&&`（左值与右值引用，参见拷贝控制章节）限定符声明对象是一个左值或右值。`const`与引用限定都可以区分重载版本，编译器会选出与对象的类型限定最匹配的方法。G. 引用限定符和`const`限定类似地作用于非`const`对象，它的作用是：如果`string`有限定赋值运算符的对象是左值的话，就不会出现`str1+str2="wow"`这种情况了；且一旦一个方法有移动标记，所有方法就都要标记。5 移动迭代器、引用限定与常量：引用限定符和`const`（）一样可以可以区分重载版本，且置于`const`之后，且具有相同的名字和参数列表的两个方法必须同时添加或不添加引用限定符。类似`const`，引用限定符作用于的同样是`this`对象。



#### 类静态成员

每个`static`数据成员可以看成是类的一个对象，而不与该类定义的对象有任何关系。因此，静态成员可以是该类型的对象，而对象的成员只能是指向该类型对象的指针（**不完全类型**的内涵就是对象不能持有另外一个自己）。



### :coffee: 可调用对象

#### functor

这种定义了调用操作`operator()`的类对象就是函数对象 functor。

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

#### $\lambda$ 表达式（谓词）、捕获

建议：凡是需要带捕获的谓词，优先考虑语法简洁的 lambda 表达式（但要掌握 functor，有利于理解 lambda 表达式的本质）。凡是不带捕获的，优先考虑是否有定义一个函数的必要，这样的内存开销也许会更小。

1. 捕获分为值捕获与引用捕获（比如，标准输入对象不支持拷贝，只能被作为引用捕获）。
	1. 捕获的外部变量可以不列出来，默认闭包。类似子类的继承访问控制下的`using`声明，默认引用捕获时，显式声明不加引用符号`[&, ci]`表示值捕获；默认值捕获的时候，显式捕获加引用符号`[=, &os]`表示引用捕获。
	2. 引用捕获时，lambda 表达式中可以通过这个引用修改原来的入参。值捕获时，参数被拷贝作为一个常量成员，如果想在函数体内改动值，可以加`mutable`声明，类似于`const`成员限定与`mutable`声明。
2. `bind`和`placeholder`的作用是将一个函数，通过构造 functor，转换成一个参数数目和顺序变化了的可调用对象。占位符相当于可调用对象的入参，而填入的常量则相当于被捕获到 functor 里的成员。

注：lambda 表达式可以捕获外部变量，本质就是一个临时定义的 functor，即，当编译器遇到 lambda 表达式时会定义一个匿名的类，而捕获的本质就是在这个匿名类对象构造过程中初始化类对象的成员数据。lambda 表达式是一个谓词，有几个捕获参数就是几元谓词。谓词，就是可调用对象，对它的调用也叫做断言。例如，在1小于2的断言中，小于是一个二元谓词，但也可以看作一元谓词。元，即是可调用对象的又名参数，或者是参数绑定中的无名占位符。

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

#### 列表初始化

建议：列表初始化是一种非常好的构造方式，容器、数组和聚合类都支持它，构造、赋值都支持它，`initializer_list`也采用这种语法来实现可变数目的入参（比如，参数绑定的`bind`接口）。

#### 迭代器与实现

C++容器基本操作与迭代器设计风格：正向迭代器的范围是从首到**尾后**节点，反向迭代器是从尾到首前节点。擦除只接收正向迭代器。反向迭代器有一个`base`方法用来转换成对应的正向迭代器。插入操作是往迭代器指向的元素前面插入。单向链表与其他容器的设计是相反的，即，往后插入与删除，并拥有首前迭代器。迭代器一般取前闭后开范围。

1. `vector`（`string`是`char`类型的`vector`）可修改，可随机访问，元素往一个方向追加，存储空间（数组）会重新分配。可变数组`vector`的迭代器就是数组下标的封装。操作位置之后的迭代器、引用和指针会错位。
2. `deque`[STL](https://www.cnblogs.com/ybf-yyj/p/10185711.html) 的实现基于一个主控可变数组（其元素为指针，指向固定大小的数组段），迭代器由主控数组下标、固定数组段的首尾点及当前点构成。首尾操作无需像`vector`那样移动很多元素，且对元素的引用等不会失效。
3. `list`(`forward_list`)，迭代器依赖链表节点指针实现，任何操作都不会使迭代器失效。

建议：除了频繁在首尾操作，一般操作频繁的，都选择性能更高的链表类。只是初始化后取内容看的就用向量类。

#### 泛型算法

建议：泛型算法库中的很多函数是容易实现的，比如`fill` `fill_n` `equal` `accumulate`（`numeric`库） `replace` `for_each` `copy` `replace_copy` `replace_copy_if` `find_if` `unique`等，但了解库函数会极大地提高编程效率，比如，可以用算法库中`partation`来快速实现leetcode 283或快排算法，如下为仿写、实现。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

class Solution { // leetcode 283
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

`tuple`是一种泛化的`pair`，`tie`是一种C++式的多返回类型语法糖。如下为用到了这种语法糖的leetcode 714实现。

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



### :penguin: 字符串

#### 字符串操作

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

#### 文件与流

我们在写算法题的时候，一般不会涉及文件的处理，都是在内存中。但涉及到大型的软件工程，就需要文件或数据库这些外围数据节点了。同时，为了降低对内存空间的消耗，用流来读取文件。所有编程语言的IO设计模式都是大同小异的，文件操作都是在调用操作系统的能力。

注：对于写算法题来说，常用的是字符串相关的IO库。建议：基于`sstream`，`getline`，`copy`和`stream_iteartor`实现C++的`split`与`join`接口。`getline`比较原始，只能接受一个字符类型作为分隔符；而新标准的泛型接口支持C原生字符串类型作为分隔符，且风格将与迭代器更为统一，譬如可以用这样一条语句来打印容器内容：`std::copy(v.begin(), v.end(), std::ostream_iterator<T>(std::cout, "\n"));`。

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



### :bath:  智能指针

#### 动态内存管理

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

#### allocator

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



### 继承与可访问性**

1 继承与动态绑定、可访问性：A. 与Java不同，C++需要显式地将基类方法标记为`virtual`以启用动态绑定，并且要求是指针或引用类型的变量（虚析构函数也利用这一机制）。B. 关于数据成员的继承，派生类完全具备基类的内容，但不一定有访问权限，`protected`则是一种真正意义上的权限“继承”：外部无权访问，但派生类可以访问；同时，派生类中只能访问派生类自己所有的那部分继承过去的成员，而不能获取基类对象的该成员。C. 继承中的方法的实现方式，本质上是这些方法隐式地持有一个`const`的类对象引用。如果将派生类当作/强制转换（非指针或引用）为基类的话，派生类中非基类的部分将被“切掉”。同时，如果想使用特定被继承类的`virtual`方法，则可以加入类作用域符号。D. 在不同作用域中无法重载函数名。派生类的作用域嵌套在其基类的作用域之内（除了部分基类内容受访问控制的限制不能访问外）。因此，当派生类重用基类的名字时，基类的同名（甚至方法参数不一样也如此）内容就被覆盖了，但我们仍然可以通过类作用域符号引用那些被隐藏的内容。

2 阻止继承或阻止生成对象：C++和Java一样用`final`表示类不可继承，但Java的`final`还具有类似C++`const`关键字的功能，而`const`虽然也是Java关键字，却未被使用。Java使用`abstract`表示纯虚函数，而C++使用`=0`，纯虚函数表示该类不定义该方法，该类也无法生成实例对象。

3 访问控制：特别值得注意的是，访问控制限定的是其他类（包括其派生类）的访问权限，类内方法中自然没有任何限制。A. 派生列表限定符的作用是，对继承来的成员在被其他类访问的时候进行限定，是派生类在基类自有访问控制之上添加的访问控制（如果原来派生类就没有该继承来的成员的访问权限，新限定也会无从谈起）。B. 可以使用`using`声明在访问说明符之上重新规划访问控制，类似于在隐式捕获上特别指定一些显式捕获。

4 容器与继承：面向对象特性要依靠指针和引用这件事儿确实很不直接，同时，还导致另一个问题，就是容器中存储在继承体系中的一些对象的时候，要用指针或共享指针，否则，子类对象会被截断成父类对象。可以把一个派生类的智能指针转换成基类的智能指针，内部做的应该是指针类型的强制转换。



### 附录

#### 正则表达式

正则，就是用字符来描述字符。正则的规则在不同编程语言中是通用。

:one: 正则表达式里用的字符，可以分为两类：

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

		`\b` 匹配单词的边界，`\B` 匹配非单词字符的边界，单词为`\w`（包括字母、下划线与`\d`），边界意为 `\s`（包括空格与制表符、换行符等），`.` 意为 `[^\n\r]`（正则表达式一般匹配一行）

:two: 正则表达式中 [不] 捕获子表达式的规则：

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