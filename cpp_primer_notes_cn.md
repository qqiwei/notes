## C++ Primer 的 YI 点点干货 ​

==待复习== 

### 类的构造与数据初始化

1 **默认构造函数**：合成的默认构造函数，首先使用**类内初始值**来初始化它的数据成员，然后默认初始化不存在类内初始值的其他数据成员，但这种默认初始化只有对**含有默认构造函数的类**才有用，而内置类型的数据成员的值将是未定义的。

2 **构造函数初始值列表**：对于被构造函数初始值列表忽视的数据成员，也将先以与默认构造函数相同的方式隐式地初始化，才进入构造函数体。某些编译器不支持类内初始值，因此，最好使用初始值列表显示地初始化所有内置类型的成员。另外，在初始值列表中属于初始化，在构造函数体中则属于赋值，因此，如果数据成员是`const`或引用这两种应该初始化的类型，就只能使用类内初始化或者初始值列表的方式。所以，没有任何理由要使用初始值列表之外的其他初始化方式。数据成员初始化的顺序由它们在类中定义的顺序决定，而非列表中的顺序。

### 容器、迭代器、列表初始化与增删改

1 **容器初始化**：容器可以默认构造、使用同类型对象构造、使用任意容器对象的迭代器构造以及通过列表初始化构造。顺序容器还可以按照给定个数进行默认值初始化与给定值的重复初始化。

2 **容器的赋值操作**：容器对象的赋值运算符`=`可以接受同类型对象与初始化列表。赋值方法`assign`则可以接受任意容器对象的迭代器、初始化列表以及给定个数和对象的初始化。`assign`方法不接受同类型对象，而赋值运算符则不能接受用以描述范围的两个迭代，但它们都接纳**初始化列表**。另外，交换操作`swap`类似于赋值运算符，仅限于同类型容器对象之间。

3 **增删改与单向链表**：对于往容器对象中添加元素，C++的逻辑是，方法insert和emplace都是往给定迭代器的前面插入，只是emplace使用其元素类型的构造方法，而insert还可以接受赋值方法`assign`可以接受的参数。`forward_list`是一个比较特殊的极简的容器，它的迭代器包括`before_begin`和`begin`，且没有`end`，插入方法是`insert_after`，删除也不是删除迭代器所指的元素（包括一个元素或者一个左闭右开的范围），而是`erase_after`。反向迭代器的递增方向则相反，通过`base`方法可转变成右移一位的正向迭代器。同时，在C++中，`stack`和头文件`queue`中的`priority_queue`的`pop`仅弹出顶部元素，返回用`top`。

### 字符串方法、拼接、分割

1 **构造**：除了本质是一个`char`类型`vector`之外，字符串的构造还支持从一个字符指针/数组拷贝指定个数的字符，以及从一个字符串的给定位置开始拷贝到结尾或者拷贝指定的长度。这也体现了字符串在普通顺序容器之上增加的操作支持：字符串不但可以接受一对迭代器，而且可以接受一个给定位置加一个长度；字符串不但可以接受字符串类型，还可以接受C风格的字符指针。此外，与Java不同，C++的字符串不是`const`的，其中的每个字符都可以像数组那样被替换。

2 **字符串方法**：`append`方法本质是在`end`处插入，而`replace`本质上是先删除一段，再执行插入。查找方法`find`和`rfind`查找字符串，`find_first_not_of`和`find_last_not_of`查找字符集中的字符，而且在查找中，`string::npos`之于字符串类似`end`之于一般容器。而字符串和数值类型的转换，可以使用`to_string`和`stoi`方法。

3 **拆分、拼接**：最后，C++没有提供`join`和`split`的库函数，我们可以基于`stringstream`和`getline`等机制自己写一个。只需要记住`getline(istream, string, char)`和`copy(iterator, iterator, ostream_iterator(ostream, char *))`这两句即可。

```cpp
int main()
{
    string s(string("lists."), 0); // diff: string s("lists.", 0);
    s.erase(s.end() - 2, s.end() - 1);

    s.append({ 's' });
    s.replace(0, 1, "a l");

    s.append(string(". end"), 0, 3);
    cout << s;
}
```

```cpp
// leetcode#1451 重新排列 text 中的单词，使所有单词按其长度的升序排列
class Solution { 
public:
    static void split(const string &text, vector<string> &tokens, 
                      const char by = ' ')
    {
        string token;
        istringstream in(text);
        while (getline(in, token, by)) tokens.emplace_back(std::move(token));
    }
    static void join(const vector<string> &tokens, string &str, 
                     const string by = " ")
    {
        ostringstream out;
        copy(tokens.begin(), tokens.end(), 
             ostream_iterator<string>(out, by.c_str()));
        str = out.str();
    }
    string arrangeWords(string text)
    {
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
int main()
{
    cout << (new Solution())->arrangeWords("keep calm and code on") << endl;
    return 0;
}
```

### $\lambda$表达式、`functor` 与可调用对象

1 **隐式捕获**：默认引用捕获的显式捕获不加引用符号`[&, ci]`表示值捕获，默认值捕获的显式捕获加引用符号`[=, &os]`表示引用捕获，类似于继承中的访问说明符与`using`重声明。除了引用捕获，还可以标记`mutable`以做修改。

==待复习==

2 **functor**：参数绑定与占位符解决这样的问题：以泛型算法为例，其参数可能包括可调用对象，而可调用对象的接口参数是既定的，那么要给这样的可调用对象传递其他参数，就可以用参数绑定与占位符，占的位置就是既定的可调用对象的接口参数，其他参数则是想要传入的其他参数。`functor`的作用也类似，且`functor`的写作方式更清晰。

3 **可调用对象**：可以使用`nm -C hello.o`查看可调用对象（模板）的实例化个数，如结果所示，相同签名的函数（指针）实例化一次，同一个类实例化一次，而每用一次$\lambda$表达式就会有一个可调用对象的实例。为了以统一的方式使用这些不同的可调用对象，C++提供了`function<int(int, int)>`，它接收一个函数类型的类型参数，可以用来引用普通函数、匿名的`lambda`函数对象以及函数对象类`functor`。

4 泛型算法库：针对的基本是基于数组的容器，列表类有自己的 `merge`, `remove`, `reverse`, `sort` 和 `unique` 方法。

```
// g++ -c hello.cpp -o hello.o
// nm -C hello.o | grep foo

template <typename T>
void foo(T f){}

struct Bar { void operator()(bool x) {} };

void f1(bool x) {}
void f2(bool x) {}

int main()
{
    foo(f1);
    foo(Bar());
    foo(Bar());
    foo(f2);
    foo([](bool x) {});
    foo([](bool x) {});
}
```

```cpp
#include <algorithm>
// never see this part of note again, Feb 1, 2020
int main()
{
    vector<int> vec{1, 2, 3}, vec_another(3), vec_empty;

    cout << accumulate(vec.begin(), vec.end(), 0) << endl;  // accumulate from 0

    fill(vec.begin(), vec.end(), 0);  // reset to 0
    // fill_n(vec.begin(), vec.size(), 0);
    if (equal(vec.begin(), vec.end(), vec_another.begin())) 
        cout << "all zeros" << endl;

    fill_n(back_inserter(vec_empty), 10, 0);  // insert ten 0
    replace(vec_empty.begin(), vec_empty.end(), 0, 1);
    cout << vec_empty.size() << endl;

    vec_empty.clear();
    vec_another.assign({1, 2, 3});
    replace_copy_if(  // replace into 2, if it's even
        vec_another.begin(), vec_another.end(), back_inserter(vec_empty), 
        [=](const int i) { return i % 2 == 1; }, 2);
    for_each(vec_empty.begin(), vec_empty.end(), 
             [=](const int i) { cout << i << ends; });
    cout << endl;

    vec_another.insert(vec_another.end(), vec_empty.begin(), vec_empty.end());
    for_each(vec_another.begin(), vec_another.end(), 
             [=](const int i) { cout << i << ends; });
    cout << endl;
    sort(vec_another.begin(), vec_another.end());
    auto uniqueEnd = unique(vec_another.begin(), vec_another.end());
    vec_another.erase(uniqueEnd, vec_another.end());
    for_each(vec_another.begin(), vec_another.end(), 
             [=](const int i) { cout << i << ends; });
    
    cout << *(find_if(vec_another.begin(), vec_another.end(), 
                      [](const int i) { return i % 2 == 0; }));
}
```

### 关联容器的下标、删除和查找操作

1 **tuple**：关联容器里存放的是`pair`类型，这很大程度上是因为C++和Java这些语言不像Python或Go那样支持多返回类型。但是，新标准增加的`tuple`（可以看作是一种泛化的`pair`）和`tie`会更加类似多返回类型的用法。

2 关联容器对象的下标操作在元素不存在的情况下会执行关联值的默认初始化。删除方法`erase`接受key，返回被删除的元素个数。查找方法`find`、`count`、`lower_bound`、`upper_bound`（表示查找第一个不小于、大于给定值的元素，上下界方法仅有序容器支持）和`equal_range`（仅支持重复键的有序关联容器所有）接受key，返回迭代器类型。

```cpp
// leetcode#714 买卖股票的最佳时机含手续费
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

### 智能指针的构造、赋值与`deleter` 

1 原生的动态内存管理：使用`new`和`delete`、`delete []`，但智能指针类提供了自动释放资源等机制，将更加安全，而智能指针也提供了符合直觉的接口，比如解引用`*`、解引用并取成员`->`和交换函数`swap`等。

2 智能指针：可以用`new`创建出的指针来初始化一个智能指针，而共享指针`shared_ptr`还支持唯一指针`unique_ptr`所不具有的`make_shared<T>(args)`方法。用另一个共享指针初始化或赋值给一个共享指针时，会增加另一个共享指针的`use_count`（共享指针的`unique`方法基于`use_count`做判断，唯一指针自然不需要提供这两个接口），如果是使用一个唯一指针`unique_ptr`初始化一个共享指针时，符合直觉地，唯一指针的所有权将被接管。两种智能指针都支持类似容器`assign`的重置方法`reset`，额外地，`unique_ptr`的`release`之于`reset()`类似`string`的`length`之于`size`方法。

3 `deleter`：默认机制将使用`delete`释放内部指针，即默认持有的是动态内存，如果持有的指针指向的不是动态内存，销毁就会报错，但附加一个`deleter`就能用于其他资源的释放。唯一指针附加`deleter`还需要提供可调用类`unique_ptr<T, D> u(p)`，或者还有一个额外对象`unique_ptr<T, D> u(p, d)`。==待复习==。

4 弱指针`weak_ptr`：好比寄生在`shared_ptr`上，而`shared_ptr`浑然不知的情况。如果说共享指针的`unique`基于`use_count`，那么这种弱共享的`use_count`和`expired`将基于其寄主。同时，一个名字特别的方法`lock`将返回所基于的共享指针。

管理动态数组用的`allocator`类同样也在`memory`包中，一般的使用步骤如下。

```cpp
int main()
{
    allocator<string> alloc;  // 0.

    int size = 10;  // 1.
    auto const begin = alloc.allocate(size);
    auto end = begin;

    alloc.construct(end++);  // 2a.
    alloc.construct(end++, "str");
    alloc.construct(end++, 3, 'a');

    vector<string> vec{"hello", ", ", "world."};  // 2b.
    end = uninitialized_copy(vec.begin(), vec.end(), end);

    for (size_t i = 0, size = end - begin; i < size; ++i)
        cout << i << " " << begin[i] << endl;

    while (end != begin) alloc.destroy(--end);  // 3.

    alloc.deallocate(begin, size); // 4. 
}
```

```cpp
int main()
{
    shared_ptr<int> sp;
    if (!sp) cout << "empty ptr" << endl;

    void called_locally(); // 函数声明
    called_locally();
}

void called_locally()
{
    int i = 2;
    // shared_ptr<int> p(&i); // using `delete` by default, raising error
    shared_ptr<int> p(  // replace `delete` by a lambda
        &i, [](int *i) { cout << "before killed, the ptr had a number " << *i; });
}
```

### 拷贝、析构、交换与移动操作

1 **拷贝**：拷贝构造函数的第一个参数是自身类类型的引用（如果不是引用就存在循环自调用），一般为`const`、非`explict`，可以携带默认参数。即使有自定义构造函数，编译器也会帮忙合成拷贝构造函数，可见其特殊性。合成的拷贝构造函数对于其数组类型的成员，将执行逐元素拷贝。除了直接使用，拷贝初始化更重要的作用是实现一种**隐式构造**，包括在函数的非引用参数传递、非引用结果返回与列表初始化（数组或聚合类）的时候。拷贝赋值运算符接受一个与其所属类相同类型的参数（可以不为引用，一般为`const`），返回指向其左侧类型的引用，且编译器同样会默认合成。

2 析构函数：被自动调用的地方包括变量离开作用域、成员随其对象被销毁、元素随其容器被销毁、动态内存的`delete`以及临时对象随其所在表达式结束的时候。需要析构（释放动态内存或其他类外资源）的类，一般也需要定义拷贝，来避免指针地址的简单拷贝：不管是如下代码所示的类值的，还是类指针的，两者的区别就是如何拷贝指针成员，前者`new`一个，而后者只是指针地址的赋值，对于类指针的，如果使用智能指针，析构和拷贝就都不需要自定义了。

3 交换函数：`swap`使得**拷贝并交换**这种写法，天然地适合对拷贝赋值进行定义，不论是类值还是类指针的类。PS 我们可以为任何函数提供`=delete`限定，而只能对可以合成的构造或拷贝操作提供`=default`限定。==待复习==

```cpp
class HasPtr {
    friend void swap(HasPtr&, HasPtr&);

public:
    string str() { return *ps; }
    HasPtr(const string& s = string()) : ps(new string(s)), i(0) {}
    HasPtr(const string& s, const int i) : ps(new string(s)), i(i) {}
    ~HasPtr() { delete ps; }
    // need copy-definition to new a string
    HasPtr(const HasPtr& hp) : ps(new string(*hp.ps)), i(0) {}
    // HasPtr& operator=(const HasPtr& hp)
    // {
    //     auto newStr = new string(*hp.ps);
    //     delete ps;  // manually
    //     ps = newStr;
    //     i = hp.i;
    //     return *this;
    // }
    // copy and swap
    HasPtr& operator=(HasPtr hp)
    {
        void swap(HasPtr&, HasPtr&);  // 友元声明不是声明
        swap(*this, hp);
        return *this;
    }

private:
    string* ps;
    int i;
};
inline void swap(HasPtr& hp1, HasPtr& hp2)
{
    swap(hp1.ps, hp2.ps);
    swap(hp1.i, hp2.i);
};

class HasPtr2 {
    friend void swap(HasPtr2&, HasPtr2&);

public:
    string str() { return *ps; }
    HasPtr2(const string& s = string()) : 
        ps(new string(s)), i(0), use_count(new size_t(1)) {}
    HasPtr2(const string& s, const int i) : 
        ps(new string(s)), i(i), use_count(new size_t(1)) {}
    ~HasPtr2()
    {
        if (--*use_count == 0) {
            delete ps;
            delete use_count;
        }
    }
    HasPtr2(const HasPtr2& hp) : 
        ps(hp.ps), i(hp.i), use_count(hp.use_count) { ++*use_count; }
    // HasPtr2& operator=(const HasPtr2& hp)
    // {
    //     ++*hp.use_count;
    //     if (--*use_count == 0) {  // manually
    //         delete ps;
    //         delete use_count;
    //     }
    //     ps = hp.ps;
    //     i = hp.i;
    //     use_count = hp.use_count;
    //     return *this;
    // }
    HasPtr2& operator=(HasPtr2 hp)
    {
        void swap(HasPtr2&, HasPtr2&);
        swap(*this, hp);
        return *this;
    }

private:
    string* ps;
    int i;
    size_t* use_count;  // 类指针的引用计数
};
inline void swap(HasPtr2& hp1, HasPtr2& hp2)
{
    swap(hp1.ps, hp2.ps);
    swap(hp1.i, hp2.i);
    swap(hp1.use_count, hp2.use_count);
};

int main()
{
    HasPtr hp11("11", 11), hp12("12", 12), hp10;
    swap(hp11, hp12);
    hp10 = hp11;
    cout << hp10.str() << endl;

    HasPtr2 hp21("21", 21), hp22("22", 22), hp20;
    swap(hp21, hp22);
    hp20 = hp21;
    cout << hp20.str() << endl;
}
```

4 **右值引用**：赋值、下标、解引用和前置增减运算符都返回**左值引用**，而算术、关系、位及后置增减运算符都返回**右值**。右值是类值的、表达式的、短暂和临时的、非变量的和非引用的。C++11 新增的右值引用绑定在右值上，或者绑定到`const`左值或被`move`的左值（变量算是一种左值，即便是一个右值引用类型的变量）。拷贝赋值有手动释放原有资源的责任，类似地，移动操作也要确保被移动的右值对象后续**可安全释放**（使该右值像一个临时计算值一样，只被使用一次，“阅后即焚”）。以及，还有一个标准库函数`make_move_iterator`可以改变迭代器的解引用操作为右值解引用。

书中以一个`StrVec`的例子来展示包括移动和`allocator`用法在内的一系列拷贝控制与内存管理操作。引用限定符和`const`（复习：底层`const`表示不更改所引用对象的值，而顶层`const`表示不再改去引用其他对象；在 C++11 **新增**的型别推导中，`auto`只取底层，`decltype`会连顶层一块取）一样可以可以区分重载版本，且置于`const`之后，且具有相同的名字和参数列表的两个方法必须同时添加或不添加引用限定符。类似`const`，引用限定符作用于的同样是类对象。

### C++的继承

1 **继承**：与Java不同，C++需要显式地将基类方法标记为`virtual`以启用动态绑定，并且要使用指针或引用类型的变量才有效果。关于继承，派生类完全具备基类的内容，但不一定有访问权限，`protected`则是一种真正意义上的权限“继承”（外部无权访问，但派生类可以访问）。C++和Java一样用`final`表示类不可继承，但Java的`final`还具有类似C++`const`关键字的功能，而`const`虽然也是Java关键字，却未被使用。继承中的方法的实现方式，本质上是这些方法隐式地持有一个`const`的类对象引用。如果将派生类当作/强制转换（非指针或引用）为基类的话，派生类中非基类的部分将被“切掉”。同时，如果想使用特定被继承类的`virtual`方法，则可以插入类作用域符号。Java使用`abstract`表示纯虚函数，而C++使用`=0`，纯虚函数表示该类不定义该方法，该类也无法生成实例对象。

2 **访问控制**：特别值得注意的是，访问控制限定的是其他类（包括其派生类）的访问权限，类内方法中自然没有任何限制。A. 基类的`protected`限定有一个特别的点，是指在其派生类中只能访问派生类自己所有的那部分继承过去的成员，而不能获取基类对象的该成员。B. 派生列表限定符的作用是，对继承来的成员在被其他类访问的时候进行限定，是派生类在基类自有访问控制之上添加的访问控制（如果原来派生类就没有该继承来的成员的访问权限，新限定也会无从谈起）。C. 可以使用`using`声明在访问说明符之上重新规划访问控制。