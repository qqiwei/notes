## C++ Primer 5th 笔记



#### I

###### 常量限定、常量区、`const &`

本质上，**常量限定** 是一种声明以使编译器代为检查一个变量不会被修改的语法糖，而 C++ 也提供了`const_cast`强制转换以支持在必要的时候剥去常量限定，不管是底层还是顶层的常量限定。有了常量限定这样的语法特性和编译器检查支持，我们在定义一个变量（函数参数）的时候，如果不改动就最好标示出来，即：加`const`限定，以及，用引用代替指针。

1. 指针的常量限定分两种：顶层和底层，分别表示不指向其他地址（比如，`this`指针本质上就是一个顶层常量指针，类似`ClassType *const this;`；而引用，本质上也就是一个没有空指针解引用问题的、不需要显式解引用的顶层常量指针，因为它在定义的时候强制要求初始化，且后续不许改引其他对象）和不改变所指向地址处的内容。语法为：`const *const int ci`，其中，解引用符右边的`const`表示顶层常量（一般直接用引用做参数的顶层限定更加方便，比如`const &`）。
2. 常量限定，可以使变量被更清晰地表达、描述和限定；同时，既能接受一个非常量类型，又能接收非常量限定的类型所不能接受常量限定类型 —— 比如，函数的形参类型经常要首先考虑是否可以被设计为 **`const &`** 这种双重常量限定：它的引用（它本身作为顶层常量限定并不否认会修改所引的对象的值）表达了这个变量在函数的作用域中将一直指向一个那个初始化给它去引用的对象（内存块），而它也因为底层`const`限定声明不修改所引用的对象而既可以持有一个不管只是常量限定声明过的底层常量，还是一个字面值（字面值就是常量的意思，而且往往用来表示特定数据类型的底层常量，比如一个在内存布局上分配在无法编辑的 **常量区** 的字符串字面值；常量区是底层的操作系统分配并限定了的只读内存段，而常量限定只是用户态的编译器提供的语法支持。用`const_cast`强制转换一个字符串字面值并编辑会导致运行时错误），也可以持有一个无限定的变量。只有常量引用`const &`可以持有一个右值（比如一个数值字面量），这使它就像C++ 11引入的右值引用（巧的是，也就只有模板或`using`别名定义中的右值引用`&&`才既可以绑定到右值又可以绑定到左值引用上，与其说存在一个叫引用折叠的机制，不如说程序语言在设计上的预期就是为了赋予它持有任何类型的能力，只不过模板参数上还能保留了原有的类型限定）一样。

###### 指针、数组、结构体、字符串、传值

如果将引用（或指针）看作一种表示地址的数据类型，那么底层和顶层常量就只是数据类型是地址还是其他类型之间的区别。但是，这种间接引用的能力是必要的，尤其是对于庞大的结构体来说，**间接引用** 可以避免函数调用传参过程中大量的值拷贝（我们可以把C系语言的传参都看作是传值的，只不过有些值是表示地址的，需要二次地、间接地引用才能操作到真正的数据），这同 Java 中一切（非内置类型的）对象都是引用的出发点是一致的。此处，参考作为 “21世纪C语言” 的 Go，可以更好地理解数组、字符串、函数、引用、面向对象等概念在C系语言中存在的意义。

1. 数组的维度 —— 首先，数组可以看作是一个成员（元素）类型一致的数据结构（或类、结构体），它的 **维度** 是数组这个数据结构类型的一部分，就像C++ 11引入了显式指明容量的`std::array<T, N>`与 Go 的`make(chan)`等语法暗示的那样。有时候，我们认为数组就是指向它底层元素数据类型的指针，数组名和顶层常量指针是一样的；除了它是一种内置的基本的数据结构而我们对它太过熟悉之外，C/C++ 在数组（或函数）与指针之间做的隐式转换加深了的这种错觉。实际上，数组是一个复合类型，它比其底层数据高一个维度；如果对一个指向数组的指针做自增，将移动到数组的`end`处（这个跨度是可获知的原因，也是因为数组的维度是数组类型的一部分，因而事实上不需要真正拿到一个运行期的数组对象，而是在编译期就能静态地获取到数组的整体大小：`SizeOfArrayMemberType * SizeOfTheArray`），这也是`sizeof`有能力取了数组大小（而非元素指针的大小）的缘故。新版本的数组可以动态地初始化其大小，毕竟，数组不是非得定义在栈上，也可以动态地定义在堆上，它仍然是有大小的，仍然是连续的。

    ```cpp
    // test_sizeof_array.cpp
    
    #include <iostream>
    
    /* [see] what means an array in c/cpp. */
    
    constexpr int power(int i) { return i * i; };
    
    int main() {
        // +--------+--------+-----+--------+-----+
        // |  a[0]  |  a[1]  | ... | a[N-1] | ... |
        // +--------+--------+-----+--------+-----+
        // ^        ^              ^
        // &a[0]    &a[1]          &a[N-1]  &a[N]
        //  ^                                ^
        // (&a)                           (&a) + 1
        long long a[power(2)]{0, 1, 2};
        
        // [statically] array name as a pointer(top-level const)
        std::cout << *(&a + 1) - a << ", " << sizeof a / sizeof *a << std::endl;
    
        int N;
        std::cin >> N;
        
        // [dynamically] new an array
        int ad[N];
        std::cout << *(&ad + 1) - ad << ", " << sizeof ad / sizeof *ad << std::endl;
    }
    
    // 4, 4
    // 5
    // 5, 5
    ```

2. 动态数组 —— Go 的字符串和切片基于动态数组，它封装了有关容量、长度的信息（可以参考`reflect.SliceHeader`中对切片类型的定义：它包含容量`Cap`、长度`Len`和一个指向底层数组的`Data uintptr`），本质上，它与其他高级语言（C++中的`std::vector`、`std::string`，Java中的`ArrayList`，它们都是从堆上申请空间来完成底层数组“ **动态** ”扩缩容的）差不多，只是把程序语言发展过程的经验内化到Go的语法里而已。语法上，Go中的切片（由于也内置地用了标签对来表示，而不像C++或Java那样是标准库）和数组很容易混淆。其中，数组标签对中必须有东西（“维度”，`[...]type`表示由编译器推导数组的长度），而切片没有维度，毕竟，它的长度是动态变化的。

3. 结构体（类） —— Go 看着像引用的行为都是建立在底层是一个指向数据块内容的指针之上的，比如切片（和字符串）或者字典类型作为函数参数传递的时候，是整个数据结构的值拷贝，只不过拷贝 **指向底层数组的指针**（和数组长度值）可以避免真正去拷贝大段数组里的数据。这也是`append(sliceObj, sliceObjOrVal)`要返回一个新切片的缘故，因为除了底层数组指针`Data`外，入参切片的`Len`和`Cap`表示两个长度值的成员也是传值而不是传引用进去的，所以，需要组装一个新的结构体返回给调用者。最后，在这一点上，它与Python中的同名接口是不一样的，`pySliceObj.append(obj)`是切片对象的方法，与Go这种C函数的风格不同。这并不是说Go不支持面向对象，而是说在处理切片和字典这些内置数据类型的时候，并没有将它看作类，而是像处理C系语言原生的数组一样，虽然它本质上就是一个数据结构。类似地，Go中类的方法也是要显式地指出其接收器对象的，这就像是以结构体指针为首参数来实现面向对象编程思想的那种C语言的风格（而C++和Java等语言有 **隐式的 `this`** 指针，当然，C的结构体是没有继承与动态绑定能力的），虽然它确实是面向对象的（甚至它部分地实现了C++的多继承与Java的接口特性），而且也具有类的命名空间那种隔离名字污染的能力（否则，函数名就要变得很冗长）。而在C++中也还能看到这种实现风格的“遗迹” —— 在需要通过捕获或者需要指明成员函数指针的所属类以实现一个可调用对象的时候，比如，`function<callableObjReturnT (BoundClassT *)> fp2mf = &BoundClassT::mf`获取到被绑定类的成员函数的指针以作可调用对象（因为类的非静态方法不同于普通函数，需要显式提供一个对象才可用，这里的`BoundClassT *`就是`this`指针的显式版；而类方法之所以能被找到，是因为存在一个潜在的名字查找机制，即，当参数是类的时候，**类的命名空间** 也会被纳入到查找的范围中）。

4. 函数的值传递 —— Go 的零长数组也是数组作为数据结构（至少包含一个长度信息）的另一个暗示。此外，由于 Go 有了切片类型，所以数组传参可以被设计为 **数组内容的值拷贝**，使其数组更像是一个成员是一些重复类型元素紧挨在一起（外加数组长度）的数据结构。C/C++的数组隐式地不做内容拷贝，而是在传参的时候自动退化成或者说仅取其指针，就像复杂结构体在函数之间传递往往用指针（或引用）一样，C系语言函数将数组类型自动转换成对应元素类型的指针，以此来避免大量的值拷贝；这种自动退化也导致C/C++一般需提供额外参数以表示数组的大小。可以认为这是C/C++定义在编译器上的机制，即在将数组名传给函数的时候，编译器将它降维转成了数组这个数据类型底层的成员数据块的首地址，也可以认为是有能力覆盖一个类的拷贝和赋值接口的默认定义的C++对数组类型的接口的一种重定义。

5. 字符串的可编辑性 —— 常量数组相当于将每一个元素都做了底层常量限定（但通过强制类型转换依然是可编辑其元素的；除非，底层数据块是存于常量区的，写操作会在运行时抛出异常），就像对象或类方法的`const`限定声明了对象也就是说它的成员不能被编辑一样。

    1. 与字符串字面值位于常量区，或者局部的数组分配在栈上不同的是，字符串`std::string`本质上其实是一个`char`类型的`std::vector`，它的数据分配在堆栈上，且基于动态数组的字符串类是可修改的。
    2. 对比之下，Go 同样基于动态数组的字符串类本质上也是可编辑的，只不过该类并没有提供 **修改接口**，所以才说它不可变的。但我们仍然可以通过`unsafe`（我是不可变的，你非要修改，我也是有办法的，但我提醒你这是危险的）库直接操作底层数据，只不过有时候这种强行重新解释数据类型的操作很可能在将错误掩盖并延后到运行时，因为可能持有的字符串字面值依然是存放在不可修改的常量区的。P.S. 强类型的 Go ，其强制类型转换依赖于“中介”`unsafe.Pointer`，还需要地址计算的话就再加一个“中介”`uintptr`。
    3. Go与C++在字符串可变性设计上的区别 —— C++ `std::string`对其构造和赋值函数的定义是对入参做 **逐字符拷贝**，这里的入参包括任何有意义的字符序列，比如说，C风格的字符串指针，即，以`\n`结尾的字符指针，也包括字符串字面值（也是C风格的字符数组，只是分配在常量区），以及，其他字符串对象（拷贝构造、赋值）。因为C++标准库对字符串做了这种自定义，所以，使用C++的字符串做函数参数时常常伴随着引用绑定，而Go往往直接传字符串而不必通过指针来传递（类似切片结构体的值传递，而且它又没提供修改接口，所以直接值拷贝就可以了，毕竟，拷贝一两个表示长度的整型不算事儿）。

    ```go
    // const_string_test.go
    
    package testconststring
    
    import {
    	"reflect"
    	"testing"
    }
    
    // [edit] the data underground
    	// through a pointer
    	// if you could
    
    func TestConstString(t *testing.T) {
    	s := "hello" // immutable
    	bs := []byte{'h', 'e', 'l', 'l', 'o'} // mutable
    
    	// s[:][1] = 'E'
    	bs[1] = 'E'
    	t.Log(s[1:], string(bs)[1:])
    
    	var str string
    	hds := (*reflect.StringHeader)(unsafe.Pointer(&str))
    	hds.Data = uintptr((*reflect.StringHeader)(unsafe.Pointer(&s)).Data) // s
    	hds.Len = len(s)
    	// *(*byte)(unsafe.Pointer(hds.Data + 1)) = 'E'
    	hds.Data = uintptr((*reflect.SliceHeader)(unsafe.Pointer(&bs)).Data) // bs
    	*(*byte)(unsafe.Pointer(hds.Data + 1)) = 'e'
    	t.Log(string(bs)[1:], str[1:])
    }
    
    func innerTestArray(a [3]int) {
    	a[0] = 8
    }
    func TestArray(t *testing.T) {
    	a := [...]int {1,2,3}
    	innerTestArray(a)
    	t.Log(a[0])
    }
    ```

    ```cpp
    // test_const_char.cpp
    
    #include <iostream>
    #include <string>
    
    void pointer_to_another_literal(const char *&s) { s = "1"; }
    // void pointer_locally_to_another_literal(const char *s) { s = "2"; }  // no use
    
    void edit_the_array(char *s) { *s = '3'; }
    void edit_the_array_or_re_point(const char *&s, bool edit = false) {
        if (edit) {
            *const_cast<char *>(s) = '4';
        } else {
            s = "5";
        }
    }
    void mod_std_string(std::string &str) { str[0] = '6'; }
    void mod_std_string(const std::string &str) { const_cast<std::string &>(str) = "7"; }
    
    int main() {
        auto s = "0";  // pointer to a string literal(immutable) "0"
        pointer_to_another_literal(s);
        std::cout << s << std::endl;
    
        // [edit] if you can, or you may just be able to re-point
    
        const char a[2] = {0};  // (mutable)array
        const char *const &p = a;
        // edit_the_array(const_cast<char *>(a));
        edit_the_array(const_cast<char *>(p));
        std::cout << a << std::endl;
    
        // edit the array
        edit_the_array_or_re_point(const_cast<const char *&>(p), true);
        std::cout << a << ", " << p << std::endl;
        // re-point to a string literal
        edit_the_array_or_re_point(const_cast<const char *&>(p));
        std::cout << a << ", " << p << std::endl;
        // now you could not edit the data underground anymore
        // edit_the_array_or_re_point(const_cast<const char *&>(p), true);
    
        const std::string str("0");  // string(array) is mutable
        mod_std_string(const_cast<std::string &>(str));
        std::cout << str << std::endl;
        mod_std_string(str);
        std::cout << str << std::endl;
    }
    ```

###### 函数式、闭包、捕获、可调用对象

新式的编程语言往往生来就支持很多有趣的特性，而一些老的语言可能需要一些特别的设计才能达到想要的效果，比如，Java的泛型、函数式编程、流、模块等机制就是在语言发展过程中逐渐引入的。此处，再以Go为例，来对比研究C++在函数式编程上的机制与特点。

1. 函数指针类型的声明 —— C++函数的型别（函数签名）由函数名及其参数决定，而C和Go（它认为虽然有时候有用，但复杂性让重载这一机制得不偿失）都不支持重载。如果不是有`typedef`这样的别名机制的话，C那些混杂了指针、数组与函数指针的复杂声明就是很难阅读的，部分原因是C的函数符号`()`、指针符号`*`与数组符号`[]`的优先级与结合性设计得并不合理，而阅读复杂声明的技巧也就同样就是要把握住运算符之间的优先级。尤其是，指针符号`*`是一个前缀运算符，其优先级低于函数符号`()`，这也就是 **函数指针的声明** 中加额外括号的原因；否则，名字是先与指针符号结合的。比如，在读`char(*(*x[3])())[5]`这样的声明时，额外括号即表明为函数指针类型；所以，`x`就被声明为"array[3] of pointer to function returning pointer to array[5] of char"。而Go类型声明的顺序中，数组和指针是位于实际类型前面的，而且Go有`func`关键字，上面的例子在Go中会是`var x [3]func() *[5]byte`，不需要加一个额外的括号。而 Go 另一个函数声明的有用设计是尾置返回类型：类的作用域与函数的块作用域，是从类名与函数名开始的，对于想利用在类的作用域中可以省略模板类型参数这样的机制，或者想使用基于函数参数类型的推导来定义返回值类型的情况来说，C++ 11引入的 **尾置返回类型** 就很有用了；而Go本身就是尾置返回类型的。

    ```go
    // go test -v -run TestPointertofunc pointer_to_func_test.go
    
    package testpointertofunc
    
    import "testing"
    
    func pointertofunc() *[5]byte {
    	var ans *[5]byte = &[5]byte{}
    	ans[3] = '3'
    	return ans
    }
    
    func TestPointertofunc(t *testing.T) {
        var x [3]func() *[5]byte
    	x[2] = pointertofunc
    	t.Log(x[2]())
    }
    ```

2. Go 函数字面值与闭包、捕获 —— 闭包是指函数与参数的绑定，而捕获看上去似乎是在直接引用外围作用域中的对象。Go的实现其实是基于局部变量可以依据需要 **逃逸** 到堆上这个机制的（而C++ 局部对象的作用域和生命周期绑定的。P.S. 另一个你无法获知一个变量具体地址的原因是Go的函数栈也是基于动态数组的，这样就不会有栈溢出的问题了；也因此，一个持久化的地址值可能是无意义的），C++ 则基于类的构造提供了引用与值传递两种捕获方式。这里引用Go语言规格中对[`defer`](https://go.dev/ref/spec#Defer_statements)和函数字面值（类似函数指针）的说明，以更深入地理解古老的函数这一概念：`defer`语句中被声明执行的函数，其实先是被作为函数字面量连带着参数被计算并保存在`_defer`栈中的，而函数体内非局部的变量可以依据函数被执行（在`defer`场景下是在函数返回时）的位置被逃逸到堆上。P.S. deferred 函数以入`_defer`栈相反的顺序执行时不会被个别函数报错打断，错误栈信息打印在所有函数执行完之后；毕竟，Go的异常处理其实就是返回一个`string`，就像C返回一个整型`-1`一样。

    > Each time a "defer" statement executes, **the function value and parameters to the call** are evaluated as usual and saved anew but the actual function is not invoked. Instead, deferred functions are invoked immediately **before the surrounding function returns**, in the reverse order they were deferred. That is, if the surrounding function returns through an explicit return statement, deferred functions are executed *after* any result parameters are set by that return statement but *before* the function returns to its caller. If a deferred function value evaluates to nil, execution panics when the function is invoked, not when the "defer" statement is executed.
    >
    > Function literals are *closures*: they may refer to **variables defined in a surrounding function**. Those variables are then shared between the surrounding function and the function literal, and they survive as long as they are accessible.

    ```go
    // [see] what's deferred execution
    
    func innerTestDefer() int {
    	var fs [4]func()
    	defer fmt.Println("first defer")
    	for i := 0; i < 3; i++ {
    		defer fmt.Println("defer fmt.Println(i)", i, &i)  // escaped referenced i
    		defer func(i int) {
    			fmt.Println("defer func(i) { fmt.Println(i) }(i)", i, &i)
    		}(i)  // local i, calculated already
    
    		fs[i] = func() {
    			fmt.Println("func() { fmt.Println(i) }()", i, &i)
    		}
    		// fs[i]()
    	}
    
    	var p uintptr
    	p = 0xc000014300
    	ip := (*int)(unsafe.Pointer(p))
    	*ip = 42
    
    	for i := 0; i < 3; i++ {
    		fs[i]()
    	}
    	defer fs[3]()  // nullptr panic
    	defer fmt.Println("last defer")
    	return func() int {
    		fmt.Println("innerTestDefer() return")
    		return 8
    	}()
    }
    func TestDefer(t *testing.T) {
    	fmt.Println("innerTestDefer() returned: ", innerTestDefer())
    }
    
    /* output
    === RUN   TestDefer
    func() { fmt.Println(i) }() 3 0xc0001182d0
    func() { fmt.Println(i) }() 3 0xc0001182d0
    func() { fmt.Println(i) }() 3 0xc0001182d0
    innerTestDefer() return
    last defer
    defer func(i) { fmt.Println(i) }(i) 2 0xc0001182d8
    defer fmt.Println(i) 2 0xc0001182d0
    defer func(i) { fmt.Println(i) }(i) 1 0xc0001182e0
    defer fmt.Println(i) 1 0xc0001182d0
    defer func(i) { fmt.Println(i) }(i) 0 0xc0001182e8
    defer fmt.Println(i) 0 0xc0001182d0
    first defer
    --- FAIL: TestDefer (0.00s)
    panic: runtime error: invalid memory address or nil pointer dereference [recovered]
    */
    ```

3. `operator()` —— 定义了调用操作`operator()`的类对象就是函数对象（functor）。

    ```cpp
    #include <algorithm>
    #include <iostream>
    #include <vector>
    
    /* functor */
    
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

4. 可调用对象 —— lambda 表达式是一种 **局部临时定义** 的匿名的functor类，在每一处出现lambda表达式的作用域中都有一个它的定义；因此，对于有重复调用价值的操作，最好还是定义一个函数，而非这样的局部类。P.S. 可调用对象又叫谓词，对它的调用也叫做断言。例如，在1小于2的断言中，小于是一个二元谓词。

    ```cpp
    // lambda_functor.h
    
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
        foo(Foo());    foo(Bar());    foo(Bar());
        foo(f1);    foo(f2);
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

    1. lambda表达式与捕获 —— lambda 表达式可以捕获外部变量，而捕获的本质就是在这个匿名类对象构造过程中初始化类对象的成员数据。捕获可以分为值捕获与引用捕获（比如，标准输入对象不支持拷贝，只能被作为引用捕获），或分为显式与隐式捕获（被捕获的外部变量可以不列出来，由编译器代为在外围作用域中查找）。与子类通过`using`声明对继承的对象进行访问控制类似地，设置默认引用捕获时，显式声明不加引用符号`[&, ci]`以表示值捕获；默认值捕获的时候，显式捕获加引用符号`[=, &os]`以表示引用捕获，默认与单独对象的设置用逗号分隔开。引用捕获时，lambda 表达式中可以通过这个引用修改原来的入参。值捕获时，参数被默认地拷贝作为一个常量成员，所以，如果想在函数体内改动值需要加一个`mutable`声明，类似`mutable`声明之于`const`成员限定。

    2. `bind`与`placeholder` —— 有时候，我们想或者需要重新包装一个函数（为它指定某些默认的参数，或调整参数顺序），就可以用到它们。

    3. `functional`的`function`类是可调用对象的代名词，它可以指向普通的函数指针，或lambda表达式，也可以通过在模板参数中声明自己是一个接受一个对象指针的函数指针以指向一个类成员函数指针。函数式标准库中还有像`plus`、`less`（泛型算法库中的许多涉及元素之间比较的接口都是以`less`为一个默认模板参数的，它要求元素类型对`<`的实现）、`greater`这样的范型可调用类，在简化代码时可使用这些语言特性和标准库。

        ```cpp
        int first[] = {1, 2, 3};
        int second[] = {3, 2, 1};
        auto firstEnd = first + *(&first + 1) - first;
        auto binaryOp = std::plus<int>;
        int result[firstEnd - first];
        std::transform(first, firstEnd, second, result, binaryOp);  // transform in <algorithm>
        ```

###### 编译期、模板实例化、别名、RTTI

1. 编译期

    1. 就像在每一个调用点展开`inline`一样，常量表达式`constexpr`（对指针类型进行的是顶层常量限定，语法示例：`constexpr const Type *p`）也是一种在编译期就要明确的字面值。另外，`enum`（它和`union`都是特殊的结构体、类，而C++在C中两种`enum`之外新增了一种具有命名空间的`enum`）对象也是常量表达式。
    2. 编译期，除了要算出常量以外，很重要的一个部分是明确类型（而有的，比如数组类型和模板中的带非类型参数，以常量为其类型定义的一部分；而模板类不称其为类型，实例化之后才是）和符号（全局变量和函数。可以说，**符号** 是一些属于某个类型的全局的对象，比如说，不同函数或方法可能属于同一个函数类型）。
    3. 习惯上，C/C++ 被看作是一种 静态（要求在编译期就明确类型，依赖虚函数的动态绑定和父类转子类`reinterpret_cast`等机制部分地减弱了它，就像golang也会依赖`interface`来提供的运行时类型的灵活性一样。但就像切片一样，数据结构所占空间的大小是固定的，只是成员包括一个指向可能发生变化的内存块的指针而已。而汇编语言直接按字节来操作内存因而被看作是无类型的）、弱类型（偏向于容忍隐式转换，C++经常通过非`explict`的构造函数达到目的；Go 是强类型的，只能显式地通过`unsafe.Pointer`实现类型转换）语言。类的大小是固定的（泛型类也能在编译期间就将模板参数确定下来；像`string`和`vector`这样的可变容器类的大小也是固定的，只不过其所持有的可变数据段是动态分配的），这就是`sizeof(Type or object)`、`sizeof object`能获取类型（对象）大小的缘由。`sizeof`返回的类型是一个`unsigned`整型的别名`size_t`，其长度决定于具体机器（为了节省内存的机器）；类似这样直接继承自最早版本的C的 **不可移植的特性** 还有位域（字段排序可能会不一致）和`volatile`（不做暂时从寄存器取用的优化）。

2. 编译期要实例化模板，要求：

    1. 类模板需要显式给出类型参数，非类型模板参数则必须提供常量表达式。模板函数的类型参数则可以通过调用点的实参推断出来的，而函数的返回值模板类型可能需要指定，部分情况下可能需要尾置返回类型，以利用实参类型来声明返回类型；因为，前置返回类型不属于模板定义的作用域内。
    2. 非显式实例化只有使用了才会实例化的特点，导致不能仅凭借类模板定义中的方法声明就通过编译：需要实例化被调函数，就需要给出函数定义，因此，方法定义与类模板定义需要放在同一头文件中。默认的实例化是`static`到源文件的，显式实例化和`extern`声明，可以让所有方法指向同一个实例化；同时，可多处声明，但只能在定义一次。P.S. 我们可以显式实例化某个特定类型参数的类的方法，相当于重定义了这个方法。
    3. 模板友元声明前需要有友元模板的声明：编译器需要检查友元的模板是否可实例化，因此，声明也有实例化要求；除非是对模板友元的任何实例化版本都声明友好性，但依然，需要以一个与类模板本身不同的模板参数标记出来，以表示它支持友元任何类型的实例化。

3. 实现可变参数的方式

    1. 可变参数模板类似`std:initializer_list<T>`，二者都是在编译期就已经确定了类或函数的模板类型。可变参数模板支持多种参数类型自动的打包、解包（通过`...`，包括通过`sizeof...(T or t)`获取可变参数长度），而初始器列表限定了一种类型参数，本质上，都是编译期代为实例化了不同类型的参数或者同一类型的一个列表。
    2. 实现可变长参数列表的另一种实现与这些模板的方式不同，就是继承自C的 **`va_list`**，其原理是：函数从右往左的参数是从依次压入栈中的，从高地址到低地址，因此，可以通过从最低地址的可变长参数列表的位置（最后一个非可变长参数后面那个参数，通过``va_start(last_not_va_last)`获取）往回加地址来获取可变参数，但它依赖一个`fmt`参数来获取每一个可变参数的类型，C的`printf`与Go的`fmt.Printf`的实现就是这样的方式。

4. 别名、`auto`与`decltype`：一种编译器提供的型别推导机制。

    1. `using `可代替 C 原生的`typedef`来给类型起别名，它支持模板，而`typedef`只能引用实例化过的类型；它与模板都有很大的灵活性，而模板类虽不是类型，但它至少是个 **名字**；再比如，直接定义引用的引用是非法的，但是`using`可以像模板参数的引用折叠那样将多次引用都“折叠”掉；而模板的灵活性还体现在模板实参推断的多样性上（与Java泛型限制的繁琐类似，C++ 定义模板及其中涉及到的声明也很繁琐，二者都在将困难的泛型编程留给库的编写者，但泛型库的使用者会轻松很多；而泛型编程的复杂性也是所有编程语言草创之初不回去实现的特性，包括Go）。
    2. `auto`提供了同其他现代编程语言类似的在演化过程中渐渐引入的自动获取类型的能力，与`constexpr`相反，它会获取赋值符号右侧表达式的底层常量限定，而弃掉顶层常量限定。
    3. `decltype(expr)`也是重要的名字声明机制，且与引用或者说“对象别名”（如果说`using`重命名了类型，那么引用可以看作给对象起别名，二者都事关名字）有着很“亲近”的关系：当`decltype(expr)`中的表达式是对指针类型的解引用，或者多包了一层括号，都将被编译器视作要声明引用类型；函数与函数指针（函数指针类型的形参，在被赋值时可以不带取地址符，在取函数指针来调用的时候可以不带解引用符）、数组和指针之间都有着很多自动转换机制，但`decltype`就取函数和数组类型本身（在做函数指针类型的声明时要搭配解引用符号，即`*decltype(func_name)`），而不会转成指针。

    ```cpp
    #include <iostream>
    
    /* auto decltype */
    
    void print(const int *a) { std::cout << a[0] << std::endl; } // same as `*a`
    void printPtr(int *&a) { print(a); }
    // ref to array, not `int &a[]`(array of refs) or `int (*f) `(func pointer, which returns an int)
    void printArr(int (&a)[5]) {
        auto p = a;
        printPtr(p);
    }
    int main() {
        int a[] = {1, 2, 3, 4, 5};
        int *p = a;
        // print(a);
        printPtr(p);
        printArr(a);
    
        auto b(a);  // int *
        print(b + 1);
    
        decltype(a) c;  // (int)[5]
        std::cout << sizeof c / sizeof(int) << std::endl;
    }
    ```

5. RTTI：运行时类型标识

    就像Java语言中对象的`getClass`接口一样，C++有一个类似的`typeid`接口用于返回一个常量的`typeinfo`类型。同时，就像Java中通过`Class`对象比较、或者`instanceof`来实现`equals`接口，C++通过`typeid`、以及`reinterpret_cast`接口（两个重载版本在失败时分别抛出`bad_cast`或返回`null`）来实现它的比较操作。特别地，C++的运行时类型识别只有在类拥有至少一个`virtual`方法时才会真正发挥作用（具有多态性）。也同样类似Java那样，`typeinfo`对象有一个`name`接口，返回一个可能略显奇怪的类名（就像Java数组名的反射也很特别）。

    ```cpp
    #include <iostream>
    
    class T {
        virtual void one(){};
    };
    class TT : public T {};
    int main() {
        const T &t = TT();
        const auto &type = typeid(t);
        std::cout << std::boolalpha << (typeid(t) == typeid(TT)) << std::endl;
        std::cout << type.name() << std::endl;
    }
    
    // true
    // 2TT
    ```

###### 声明、内存分布、预处理

1. 声明、分离式编译、内存分布

    1. 一般来说，类定义（一般以类名命名并独占一个头文件）与函数声明放在头文件中，普通函数和类成员函数（类模板除外，要将方法定义也放在同一个头文件中）的定义放在源文件中；而定义所在的文件中也可引入其声明文件，以利用编译器检查声明和定义的一致性。以及为了避免预处理头文件引入的时候引入多次声明（或导致循环引用），C/C++支持头文件保护符`#ifndef`、`#define`和`#endif`，从而使变量和函数只被声明一次。在没有包、模块（Java 11 在package之上又引入了module概念；而 C++ 20 也已经引入了`import`、`export`等模块化管理机制了）等源码管理机制之前，C/C++ 只能将源文本`include`进来（未使用的，其实也不会编进来），或者将编译好的中间文件链接起来（又或者通过`make`、`cmake`等工具来组织引入与链接等行为）。如下所示，通过`include a.hpp`，`b.cpp`只是引入了`j`的 **声明** 就知足了。被调对象的定义可以通过链接加入进来的，而链接就是将这些 **未定义的名字** 重定位到一个对象定义上，这是典型的分离式编译。但是，没有了直接引入以后，`static`对象将无法被其他文件通过链接与`extern`（只声明而不定义）获取到，比如`i`被限制在了`a.cpp`中。`extern`的含义是只声明，但不论定义在哪，那么，如果有调用点却真的没有定义过的话，就会在链接期报错，比如，如果在`b.cpp`中用了`i`，那么就会因为获取不到`static`的`i`而报未定义错误。

        ```cpp
        /* 分离式编译 */
        
        // a.hpp
        #ifndef AHPP
        #define AHPP
        
        // you may use linker, like:
        // g++ -c *pp
        // g++ *.o -o main
        
        // but you cann't get the static `i` in a.cpp anyway, without:
        // #include "a.cpp"
        
        extern int i;
        extern int j;
        extern int p;
        extern int q;
        
        extern char *s;
        
        template <int T>
        int echoTP() {
            return T;
        }
        #endif
        ```

        ```cpp
        // a.cpp
        
        // may conflicts between extern and static with:
        // #include "a.hpp"
        
        static int i = 100;
        int j = i;
        
        static int p = j;  // static same as `i`
        int q = 101;
        
        auto s = "hello";
        ```

        ```shell
        $ echo a.o b.o c.o main.* | xargs nm -nC | egrep "f[bc]| \w [ijpq]"
        # capital letter for non-static(open), [b]ss i.e. block-start-by-symbol for un-[d]efined
        0000000000000000 d i  // global static, [d]efined and initialized
        0000000000000000 B j  // global non-static [B]ss, defined but not initialized
        0000000000000004 b p  // global static [b]ss, defined but not initialized
        0000000000000004 D q  // global non-static, [D]efined and initialized
        ```

        ```cpp
        // b.cpp
        #include "a.hpp"
        
        int fb() { return echoTP<'b'>(); }
        ```

        ```shell
        0000000000000000 T fb()
        0000000000000000 T int echoTP<98>()
        ```

        ```cpp
        // c.cpp
        #include <iostream>
        
        #include "a.hpp"
        
        // un-included `extern int fb()`
        extern int fb();
        int fc() { return echoTP<'c'>(); }
        
        int main() {
            // if you want to use `i`, you have to offer an accessable definition
            // std::cout << i << std::endl;
            std::cout << j << std::endl;
            // std::cout << p << std::endl;s
            std::cout << q << std::endl;
            std::cout << s << std::endl;
        
            return fb() + fc();
        }
        ```

        ```shell
                         U fb()  # undefined by `extern int fb()`
                         U j     # undefined by `#include "a.hpp"`
                         U q
        0000000000000000 T fc()
        0000000000000000 T int echoTP<99>()
        ```

        ```shell
        # symbols in the final linked executable file
        00000000004015b0 T fb()
        00000000004015d0 T fc()
        0000000000402dd0 T int echoTP<98>()
        0000000000402de0 T int echoTP<99>()
        0000000000403010 d i
        0000000000403014 D q
        0000000000407030 B j
        0000000000407034 b p
        ```

    2. 是否静态限定到某一个文件或类中，与这个标识符是否是初始化有区别，这个区别类似常量限定与`register`类型限定、常量区的区别：前者只影响编译期对访问权限的校验，而是后者则影响变量被分配的 [内存布局](https://www.geeksforgeeks.org/memory-layout-of-c-program/) 区域（可以用`size obj.f`查看不同内存区域的分配情况），而未初始化的全局变量所在的BSS段的存在就是为了减小程序大小，只有在程序执行的时候才初始化为零值或计算得出，而已初始化的全局变量的初始值则可以看作是程序文本的一部分。PS. 函数的局部变量位于函数栈区，而动态内存分配在堆中。`new`和`delete`则是在类似C原生的`malloc`与`free`的用于动态堆内存分配与回收的`operator new`与`operator delete`之上，附加了类的构造和析构，以初始化类的对象、管理类所引用但位于类外的一般是操作系统管理的资源。

2. 宏、预定义变量、编译选项

    1. 宏定义有一些处理字符串的复杂语法，还有一些习惯写法，比如，用`do while(0)`编写代码块。

    2. 断言`assert(true_expr)`是一种预处理宏。可以通过定义预处理变量 `NDEBUG`来关闭断言。此外，C++也有些由预处理器定义的魔术变量：`__FILE__`、`__LINE__`、`__FUNC__`和`__TIME__`等，可用来打日志。

    3. 有些编译选项能提供堆栈保护（`fstack-protect-*`）和性能优化（`-O2`）等功能，而有些编译选项一般是默认要开启的（**`-g -O2`**，比如 CGO 在生成C接口的时候会自动为C编译期添加这两个编译选项，而自定义操作可能覆盖掉原有的默认配置），否则可能会造成问题。曾经遇到过的一个问题就是：在自定义CGO的`CFLAGS`变量的时候，覆盖了原有的`-O2`配置，造成的问题是调用点无法展开`inline`函数，`inline`函数变成一次普通的函数调用，但对应的函数定义又不会生成，因此，造成符号未定义的报错。通过为`go build`附加`-work`选项将中间文件的细节输出到`/tmp`下能了解更多CGO的生成细节，搭配`nm -C`和`gdb`的反汇编命令`disassemble`可以获知编译生成的库中各符号定义和使用的情况。P.S. 经常遇到`inline`等引发的告警，因此提醒自己使用这些特性前要多了解。

        ```shell
        CC 
         -c # compile only
         -o output_file_name
         -D NDEBUG # 定义预处理变量NDEBUG
        ```

###### 作用域、生命周期、命名空间

名字有作用域，对象（C++ 中的对象，具有超过面向对象编程概念里的对象的更宽泛的指涉，即，它意指程序中所有具体而非模板的数据，指针地址、函数指针也都算是数据、对象）有生命周期。声明的是名字，定义的是对象，声明和定义在同一处的叫初始化（构造等同于初始化，并暗含类类型成员的初始化）。名字的作用域是程序文本的一部分，影响编译期间名字在程序中的可见性。对象的生命周期体现在程序执行过程中对象在内存中的存留时间。

1. 块（局部）作用域 —— 花括号（包括条件、循环语句块与函数，而`for`循环初始化处的定义与函数参数一样都属于自动变量）内为 **块作用域**，可能是最重要的一种作用域了。宽泛地理解，C系语言只有一种块作用域，块作用域可以嵌套而且块作用域内能访问作用域外的标识符，全局作用域只不过是所有块作用域之外而已，而函数由于是一等公民，所以函数是最外层的块作用域，而不是嵌入在调用上下文中的作用域。块内的变量作用域限定在块内，包括函数非引用限定的参数的局部变量的生命周期也限定在进程进入和走出块作用域期间（这是C/C++不许在局部或块作用域内返回局部对象地址的原因，抛出这样的异常也不行，因为被分配在栈上，它们在栈展开的时候就都被销毁了，引用将是无效的，这与会自动逃逸到堆上的Go不一样）。
2. `static`变量 —— 不管是局部还是类的静态成员，都和存在于所有函数体之外的对象一样，生命周期长至进程结束，但作用域可能会被限定在“块”中。原生C曾使用**`static`**以将全局变量的作用域限定在所在文件或块中，C++ 11 后，代之以未命名的命名空间（与全局对象的生命周期相同，但作用域限定在命名空间所在的作用域中）。
3. 命名空间 —— 命名空间是一种作用域，而类是一种 **命名空间**。全局命名空间可以用`::g_Name`来引用，而类中可以用`using`重命名一个类型，即为类的成员类型。除了未命名的命名空间之外，命名空间只能用于包含像全局变量、类定义、函数这样的全局对象，因为它是一种用于防止大型程序中的名字污染的机制，对于原本就外部不可见的局部对象来说，命名空间也是多此一举的，所以C++规定它只能收纳全局的名字，并且不能定义在函数与类内部。P.S. 通过`using`直接引用到具体的函数不会导致同类型函数的冲突（使用时加上命名空间即可），但`using`指示一个函数名会将所有同名函数加入同一个重载集合中，就会导致同类型函数的冲突。



#### II

###### 构造、隐式构造、析构、`new`

1. 构造与析构的次序 —— 构造，就是对象的数据成员定义的过程，数据成员将按照其在类中出现的顺序进行构造（成员对象的析构将按相反的顺序被调用）。P.S. 参数的构造与析构发生于此函数调用者的上下文之中，而不是这个函数的上下文，但局部变量的销毁在将要退出栈的时候。而在继承体系内，子类成员先构造，但析构情况稍微有些复杂：如果父类的析构函数是虚函数，则析构按照构造的逆序进行，即先析构（销毁从）父类（继承来的成员），再析构子类，且析构具有多态性；如果父类的析构不是虚函数，则父类指针或引用将不具有多态性，但子类对象的析构依然会去析构父类。另外，关于构造与虚函数有一个原则是不要在构造函数中调用虚函数，因为在子类的父类部分构造的时候，由于子类尚未构造完成而不具有多态性，可能造成与预期不符合的情况。PS. 在已定义了构造函数的情况下，可以用`= default`来生成所需要的默认构造函数的定义。

2. 初始化方式

    1. 初始值列表中的显式构造将优先被调用，否则查看类内初始化（某些编译器可能不支持类内初始化，所以，通常不在类内初始化，除非是常量静态成员才有必要在 **类内初始化**），再否则就尝试调用成员`public`的默认构造函数，再再否则，成员的值将是未定义的，只能依赖构造函数函数体中的赋值操作。
    2. 对于只能进行初始化的`const`或引用限定的数据成员来说，**初始值列表** 是必须的。另外，初始值列表位于构造函数体外，为了处理初始值列表操作过程中可能产生的异常，可以用一种特殊的`T:T(arg) try: member(arg) {} catch {}`语法；而如果是构造函数的参数构造失败，则异常处理的责任属于调用者，因为填充实参的操作就发生在调用者的上下文，此时还没有被调者。 常量限定的对象，只能调用其`const`限定的方法。

3. 隐式构造 —— 除了显式定义一个类的对象会触发构造以外，有很多隐式构造的场景：隐式类型转换（类型转换依赖构造函数来实现；另外，想要在参数传递中禁用参数类型的隐式类型转换可以给形参的构造函数加一个`explict`限定，使其更偏向强类型），函数非引用限定的参数传递与返回，数组、聚合类（`std::pair`，C++中，`class`与`struct`唯一的区别就是`class`中默认的访问控制权限是`private`，而`struct`是`public`，建议用`struct`定义比较简单的聚合类）、可变参数、容器的列表初始化（除了初始化结构体成员外，列表初始化的本质以初始化列表`std:initializer_list<T>`为形参），`new`。

4. 当我们使用`new`这个关键字的时候：首先，类的`operator new`被调用以分配内存空间，然后，进行类的默认构造，最后，返回对象指针。`operator new`有三种 [重载](https://www.cplusplus.com/reference/new/operator%20new/) 形式，以设定是否抛出异常（`std::bad_alloc`，不抛出异常所用的`std::nothrow_t`类型的字面值`std::nothrow`只是用来区分重载版本的。注意：它不是带有一个`bool`类型参数的重载版本的`noexcept`，而是类似Java与早期的C++版本中函数的`throw`声明；以及，有一个同名的运算符来判断一个表达式会否抛出异常，与`sizeof`类似，它也不会计算表达式，而是静态地在编译期就获取到一个布尔类型的右值常量），以及是否真的需要分配这个类大小的空间而不仅仅只是为了调用默认构造函数（不分配大小的以一个既有的指针为额外的入参，不论地址所指向的内存块是分配在堆还是栈上都能完成类对象的构造）。P.S. 这里带`void *`参数的重载版本的功能是C++给定的，但C++允许提供自定义其他类型为参数的`new`操作符用来控制空间分配。

    ```cpp
    /* `new` operator */
    
    // i may throw (but only) std::bad_alloc
    void* operator new (std::size_t size) throw (std::bad_alloc);
    // i throw nothing, and return 0 if fail to alloc
    void* operator new (std::size_t size, const std::nothrow_t& nothrow_value) throw();
    // i alloc nothing
    void* operator new (std::size_t size, void* ptr) throw();
    
    // examples:
    auto p = new Type;
    auto p = new (std::nothrow) Type;
    new (p) Type;
    ```

5. 析构 —— 与构造相反，析构的作用是释放类占用的 **资源**（如在初始化对象或方法调用过程中，申请了动态内存或文件操作符等）；而数据成员所占内存的回收（除非用`new`创建才需要`delete`指针以释放所指向的内存空间，否则）不归开发者管。析构可以被显式调用（以显示地释放这些资源），但一般来说，依赖在该类型的对象被销毁前，析构函数的自动调用就够了（因为是一个自动调用的过程，因而最好不要抛出析构函数自己不能处理掉的异常）。而释放一个对象的场景，也与构造相反：变量离开作用域、成员随其对象被销毁、元素随其容器被销毁、动态内存的`delete`以及临时对象随其所在表达式结束而被销毁。析构、成员对象销毁的顺序是构造过程的逆序。

    ```cpp
    #include <iostream>
    
    /* [test] the order of invocation of constructors and destructors */
    
    class IN1 {
        int i = 1;
    
       public:
        IN1() { std::cout << "IN1() done." << std::endl; }
        IN1(int i) : i(i) { std::cout << "IN1(int) " << i << " done." << std::endl; }
        ~IN1() { std::cout << "~IN1() done." << std::endl; }
    };
    class IN2 {
        int i = 2;
    
       public:
        IN2() { std::cout << "IN2() done." << std::endl; }
        IN2(int i) : i(i) { std::cout << "IN2(int) " << i << " done." << std::endl; }
        ~IN2() { std::cout << "~IN2() done." << std::endl; }
    };
    class IN3 {
       public:
        IN3() { std::cout << "IN3() done." << std::endl; }
        ~IN3() { std::cout << "~IN3() done." << std::endl; }
    };
    class OUT {
        IN3 in3;
        IN2 in2;
        IN1 in1;
    
       public:
        OUT() : in1(1), in2(2) {
            new (&in3) IN3();  // only re-construct in3, which is a data member
            std::cout << "OUT() done." << std::endl;
        }
        ~OUT() {
            in3.~IN3();  // only destruct in3
            std::cout << "~OUT() done." << std::endl;
        }
    };
    
    class father {
       public:
        father() { std::cout << "father() called " << cnt++ << " times, done." << std::endl; }
        // virtual
        ~father() { std::cout << "~father() done." << std::endl; }
    
       public:
        static int cnt;
    };
    
    int father::cnt = 1;
    
    class son : public father {
       public:
        son() { std::cout << "son() done." << std::endl; }
        ~son() { std::cout << "~son() done." << std::endl; }
    };
    
    int main() {
        OUT out;
    
        // the ~son() will not be executed without virtual on ~father().
        std::cout << "1. " << std::endl;
        father *pf = new son();
        pf->~father();
    
        // ~son() will execute the ~father() any way.
        std::cout << "2. " << std::endl;
        son().~son();
    
        std::cout << "main() done." << std::endl;
    }
    
    /* output
    IN3() done.
    IN2(int) 2 done.
    IN1(int) 1 done.
    IN3() done.
    OUT() done.
    1.
    father() called 1 times, done.
    son() done.
    ~father() done.
    2.
    father() called 2 times, done.
    son() done.
    ~son() done.
    ~father() done.
    ~son() done.
    ~father() done.
    main() done.
    ~IN3() done.
    ~OUT() done.
    ~IN1() done.
    ~IN2() done.
    ~IN3() done.
    */
    ```

###### 访问控制、友元、类的作用域

访问控制限定的是外部类型对本类型的可访问性。

1. 友元是一种指定类型或函数的`public`访问控制。友元声明，不是一般意义上的类型声明，它并未使友元本身可见。除非，类及其友元都定义在同名的命名空间中，该操作依赖于这样一个机制，即，命名空间是候选的名字搜索空间。比如在成员中使用友元的情况，即便友元声明带着自己的定义，在成员使用友元之前也需要先做一个声明。
2. 继承：子类是 **外部类型**，所以，子类在其内部可以访问其继承自父类的`protected`成员，但不能通过父类来获取父类对象的该成员。子类继承（拥有）父类的全部数据，但在自己的空间内不一定有访问权限。另外，在多继承中，虚继承（是一种实际需求可能还未出现的预先设计，因为先限定的是父类）的标识符`virtual`也是放在同样的位置。
3. 在继承体系内，子类定义起始行的派生列表限定符的作用也是对继承来的成员在被其他外部类访问的时候进行限定，是派生类在基类自有访问控制之上添加的访问控制，如果原来派生类就没有该继承来的成员的访问权限，即`private`限定，则新限定也就无从谈起。可以使用`using`声明屏蔽访问说明符的作用并重新规划访问控制权限，类似于在`lambda`表达式的隐式捕获的基础上，特别指定个别参数的显式捕获。

除非通过作用域限定符指明是哪一个类的成员；否则，当编译器要找一个名字时，先查看它是否为局部变量，不是局部变量再看类内数据成员，如果不在类里面再从成员函数定义点（成员函数可能定义在类外部）往前看是否存在该名字的声明。派生类的作用域嵌套在其基类的作用域之内（派生类能访问基类成员，除非受访问控制的限制）。因此，在派生类中声明基类成员的名字时，基类成员就被覆盖（函数签名的名字与参数相同）或 **隐藏**（只有函数名相同）了；而重载则发生在相同的作用域空间（函数名相同，参数不同）。对于类类型来说，遇到类名即进入类的作用域，而对于类模板来说，在类的作用域中可以省略类型参数，对于返回值来说，如果不是尾置返回类型，就位于类的作用域之外。PS. 作用域符号默认获取模板参数的非类型成员，要获取其成员类型需要附加`typename`关键字限定。

每个`static`数据成员可以看成是类的一个对象，而不与该类定义的对象有任何关系。因此，静态成员可以是该类型的对象，而对象的成员只能是指向该类型对象的指针（不完全类型的内涵就是意指对象不能持有另外一个自己）。

###### 继承、虚函数、动态绑定

与Java不同，C++需要显式地将基类方法标记为`virtual`以启用动态绑定，并且要求是指针或引用类型（比如，声明要捕获一个在继承体系中的异常类时，就最好用引用），否则，在子类对象赋给基类对象的时候，子类部分将会被截断（异常捕获也一样，P.S. 异常捕获还允许`const`声明类型捕获一个非常量类型，但除了函数和数组自动转换为指针以外的所有隐式类型转换都不被允许）。也因此，`vector<T>`中最好存指针类型的对象。同时，如果想使用特定基类的`virtual`方法，可以用作用域限定符指定。运行态中的任何虚函数调用都将调用实际对象的实现版本，父类的私有方法也可以被覆盖（甚至声明为`public`）。覆盖行为可以用`override`声明出来，让编译器检查。覆盖只涉及函数名字与参数，不涉及返回值类型。此外，只有名字相同的子类方法将隐藏父类的同名方法。C++和Java一样用`final`表示类不可继承，但Java的`final`还具有类似C++`const`关键字的功能，而`const`虽然也是Java关键字，却未被使用。Java使用`abstract`表示纯虚函数，而C++使用`=0`，纯虚函数表示该类不定义此方法，且该类也无法生成实例对象。

```cpp
#include <iostream>

// virtual method

class B {
    virtual void do_f() { std::cout << "B::do_f();" << std::endl; }    // private member
    virtual void do_f2() { std::cout << "B::do_f2();" << std::endl; }  // private member 2
   public:
    void f() {
        do_f();
        do_f2();
    }  // public interface
};
struct D : public B {
    void do_f() override {
        do_f2(2);
        std::cout << "D::do_f();" << std::endl;
    }                                                                // overrides B::do_f
    void do_f2(int i) { std::cout << "D::do_f2(i);" << std::endl; }  // hide B:do_f2
};

int main() {
    D d;
    B* bp = &d;
    bp->f();  // internally calls D::do_f();
}

// D::do_f2(i);
// D::do_f();
// B::do_f2();
```

###### 拷贝控制、移动、右值引用

1. 拷贝构造函数 —— 第一个参数是（一般为非`explict`、`const`限定，可指定默认参数的）**自身类类型的引用**，如果不是引用另一个对象、拷贝其成员来构造自己就存在无限递归。`explict`的构造函数会限制只能进行直接初始化，而不能进行拷贝初始化，因此一般也不会这样设计，因为拷贝构造与拷贝赋值运算符往往是要成对使用的。
2. 拷贝赋值运算符 —— 接受一个与其所属类相同类型的参数（可以不为引用，一般为`const`）并返回指向其左侧类型的引用，它的本质是 **析构（、销毁）与构造的结合**。类似地，因为事先不知道所存储的成员类型，所以，包含类对象的`union`的构造与赋值操作中需要自定义析构操作，可以结合一个内嵌的`enum`类型来判断所存储的类型。
3. 类似默认构造函数，如无定义，编译器默认会合成拷贝构造函数与拷贝赋值运算符。因为，拷贝是隐式调用默认的手段，而隐式调用就是一些经常发生的隐式构造、赋值，因此有必要合成该操作。需要析构（释放动态内存或其他类外资源）的类，一般也需要定义拷贝来避免指针地址（指向资源）的简单拷贝：不管是类值还是类指针的（两者的区别就是如何拷贝指针成员，即额外资源，前者`new`一个，而后者类似共享指针`shared_ptr`）；对于类指针的，如果使用智能指针，析构和拷贝就都不需要自定义了。

很多算法（比如原址排序）要用到`swap`接口（它本质是一次初始化和两次赋值），为了减少默认调用的`std::swap`中的 **三次拷贝**，我们通常自定义`swap`接口以减少内存分配操作（尤其是类值的）；虽然`std::swap`接口其实调用了`std::move`以挖掘潜在的性能提升点（如果被移动的对象定义了移动操作的话）。注：很多时候，我们习惯于用`std::`标明任何来自标准库的名字以降低冲突的可能性，但此处，我们习惯于在使用交换操作的块作用域中的首先声明`using std::swap;`，然后使用不加限定的`swap`以使可能的自定义接口获得优先匹配。赋值操作的本质是释放原对象、持有一个新对象，“拷贝并赋值”（选择普通可修改参数来定义拷贝赋值）使用自定义的`swap`接口可极大地简化其实现，并潜在地具有效率优势，不论是类值的还是类指针的，它的原理是：将`*this`置换到作为自动变量的入参上以实现 **隐式的资源释放**。在以下示例中，可以观测到拷贝、移动等定义在隐式的传参、赋值与返回值处的逻辑是怎样的：

- STEP 1：在定义了赋值运算符的情况下，执行`hp0 = hp1`时，在`operator=`中，会依据实参`hp1`构造形参`tmp`，交换操作之后，返回原`tmp`对象给`hp0`，而（通过debug单步调试可以看到）局部变量`tmp`指向的原`hp0`的对象将在` (hp0 = hp1).seti(5).show();`这行执行完时进行析构。
- STEP 2：而在`hp0 = std::move(hp2);`这一行中，同样是在`operator=`中，会依据实参构造形参`tmp`，只不过这次的实参是`std::move`（什么都不做，只执行了往右值引用的强转）返回的一个右值引用对象。虽然移动操作其实啥也没做，但移动构造函数却将接收到的对`h2`的右值引用（依然是引用类型，而不是右值）底层指针置空了。
- 类指针与类值的过程是类似的，只不过要维护一个计数器，就像Linux系统里文件的硬链接个数一样。

```cpp
// copy_ctrl.hpp

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
        // using std::swap;
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
    Shared(const std::string &s, const int i)
        : res(new std::string(s)), i(i), use_count(new size_t(1)) {}
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

template <typename T>
void bar(T &&t) {
    T tt = t;
    std::cout << tt;
}
```

```cpp
// copy_ctrl.cpp

#include "copy_ctrl.hpp"

#include <iostream>

int main() {
    Exclusive hp1("exclusive-res-1", 1), hp2("exclusive-res-2", 2), hp0("exclusive-res-0");
    std::cout << "\n---STEP1: swap(hp1, hp2); (hp0 = hp1).seti(0);---" << std::endl;
    swap(hp1, hp2);
    (hp0 = hp1).seti(5).show();
    std::cout << "\n---STEP2: hp0 = std::move(hp2);---" << std::endl;
    // [original      ]         hp2{0x1920, 1},                    hp1{0x1970, 2}, hp0{0x1c70, 5}
    // [tmp(&&hp2)    ]  tmp(&&hp2){0x1920, 1}, &&hp2{nullptr, 1}, hp1{0x1970, 2}, hp0{0x1c70, 5}
    // [swap(tmp, hp0)]  hp0       {0x1920, 1}, &&hp2{nullptr, 1}, hp1{0x1970, 2}, tmp{0x1c70, 5}
    // [hp0 =(&&hp2)  ]  hp0       {0x1920, 1}, &&hp2{nullptr, 1}, hp1{0x1970, 2}
    hp0 = std::move(hp2);

    Shared hp3("shared-res-3", 3), hp4("shared-res-4", 4), hp00("shared-res-00");
    std::cout << "\n---STEP3: swap(hp3, hp4); (hp00 = hp3).seti(0);---" << std::endl;
    swap(hp3, hp4);
    (hp00 = hp3).seti(6).show();
    std::cout << "\n---STEP4: hp00 = std::move(hp4);---" << std::endl;
    // [ori ]        hp4{0x1c20[1], 3},                       hp3{0x1cb0[2], 4}, hp00{0x1cb0[2], 6}
    // [tmp ] tmp(&&hp4){0x1c20[1], 3}, &&hp4{nullptr[1], 3}, hp3{0x1cb0[2], 4}, hp00{0x1cb0[2], 6}
    // [swap] hp00      {0x1c20[1], 3}, &&hp4{nullptr[1], 3}, hp3{0x1cb0[2], 4}, tmp {0x1cb0[2], 6}
    // [= &&] hp00      {0x1c20[1], 3}, &&hp4{nullptr[1], 3}, hp3{0x1cb0[2], 4}
    hp00 = std::move(hp4);

    std::cout << "\n---STEP5: self move(release);---" << std::endl;
    (hp3 = std::move(hp3)).show();
    (hp1 = std::move(hp1)).show();

    std::cout << "\n---exit---\n" << std::endl;
}

/* output
---STEP1: swap(hp1, hp2); (hp0 = hp1).seti(0);---
show(): exclusive-res-2 with id 5
show(): exclusive-res-0 with id 0: auto gc

---STEP2: hp0 = std::move(hp2);---
show(): exclusive-res-2 with id 5: auto gc

---STEP3: swap(hp3, hp4); (hp00 = hp3).seti(0);---
show(): shared-res-4 with id 6, cnt 2
show(): shared-res-00 with id 0, cnt 1: auto gc

---STEP4: hp00 = std::move(hp4);---
show(): shared-res-4 with id 6, cnt 2

---exit---

show(): shared-res-3 with id 3, cnt 1: auto gc
show(): shared res moved: auto gc
show(): shared-res-4 with id 4, cnt 1: auto gc
show(): exclusive-res-1 with id 1: auto gc
show(): exclusive res moved: auto gc
show(): exclusive-res-2 with id 2: auto gc
*/
```

`std::move`只做了往右值引用的强转，但 **右值引用**（不是右值，它还是一种引用）暗示（当然，我们也可以随便定义它，就像可以把`operator+`定义去做减法一样）了对象的内容是可被窃取的，就像上面例子中的移动构造函数一样（同时，虽然可以窃取入参的“资源”，但要保证被窃取的参数后续可析构，一般来说，置空指针即可，就像`va_start`要搭配一个`va_end`一样）。右值引用这种暗示，就像拿到的是一个右值一样，而右值意味着 **临时性** 的数据，这是右值引用的对象可以被随意“窃取”并在后续将“资源”已被窃取的临时对象做垃圾回收的原因。因此，`std::move`可用在赋值运算符的右侧对象或非引用的返回值之上，借用或隐式或显式的移动操作，直接窃取被移动对象的内容，就像`std::swap`就是调用了`std::move`来尝试获取移动操作的那样。P.S. 赋值、下标、解引用和前置增减运算符返回左值（用的是对象的身份、内存中的位置），而算术、关系、位及后置增减运算符返回右值（用的是对象的值、内容）。

在模板中，左值引用无法引用一个右值，但一个右值引用可以接受任何类型的实参，如果实参不是右值类型，那么模板形参和函数的形参都将被推断或折叠为左值引用类型。如下示例还可以看出很多语言背后机制的细节，比如，虽然我们不能自定义以不是任何引用限定的类型为入参的拷贝构造，但在下面这个例子中，它似乎是存在的，且掌握在编译器手里，执行的是值的拷贝；实参和局部变量的构造与析构分别发生在栈的内外两边；引用折叠发生时，模板函数内的模板类型的局部变量将采用引用声明。

```cpp
#include <iostream>

// std::move

struct A {
    static int cnt;
    int i;
    A() : i(4) { std::cout << "A(): this.i=" << i << ", done[" << ++cnt << "]." << std::endl; };
    A(const A &a) : i(5) {
        std::cout << "A(const T &a): this.i=" << i << ", a.i=" << a.i << ", done[" << ++cnt << "]."
                  << std::endl;
    }
    A(A &a) : i(6) {
        std::cout << "A(A &a): this.i=" << i << ", a.i=" << a.i << ", done[" << ++cnt << "]."
                  << std::endl;
    }
    A(A &&a) : i(7) {
        std::cout << "A(A &&a): this.i=" << i << ", a.i=" << a.i << ", done[" << ++cnt << "]."
                  << std::endl;
    }
    A &operator=(A a) {
        std::cout << "operator=(A a): this.i=" << i << ", a.i=" << a.i << std::endl;
        std::swap(*this, a);
        std::cout << "operator=(A a): this.i=" << i << ", a.i=" << a.i << ", done." << std::endl;
        return *this;
    }
    ~A() { std::cout << "~A() done[" << cnt-- << "]." << std::endl; }
};

int A::cnt = 0;

std::ostream &operator<<(std::ostream &os, const A &a) {
    os << "operator<<(const A &a): os << this.i=" << a.i << ", done." << std::endl;
    return os;
}
std::ostream &operator<<(std::ostream &os, A &a) {
    os << "operator<<(A &a): os << this.i=" << a.i << ", done." << std::endl;
    return os;
}
std::ostream &operator<<(std::ostream &os, A &&a) {
    os << "operator<<(A &&a): os << this.i=" << a.i << ", done." << std::endl;
    return os;
}

template <typename T>
void print(T &&t) {
    std::cout << "print(T &&t): t.i=" << t.i << ", T tt(t)," << std::endl;
    T tt(t);
    std::cout << "t{" << t << "}, tt{" << tt << "}, done." << std::endl;
}

int main() {
    A a = A();
    const A &ca = a;
    // only reference(point to)
    print(ca);
    print(a);
    // construct from a left-reference
    print(A());
    print(std::move(A()));
    print(std::move(a));

    std::cout << "------- here we test more -------\n"
              << ca << a << A(ca) << A(a)
              << std::endl
              // only construct one time
              << A(A(A())) << std::endl;
}

/*
A(): this.i=4, done[1].
print(T &&t): t.i=4, T tt(t),
t{operator<<(const A &a): os << this.i=4, done.
}, tt{operator<<(const A &a): os << this.i=4, done.
}, done.
print(T &&t): t.i=4, T tt(t),
t{operator<<(A &a): os << this.i=4, done.
}, tt{operator<<(A &a): os << this.i=4, done.
}, done.
A(): this.i=4, done[2].
print(T &&t): t.i=4, T tt(t),
A(A &a): this.i=6, a.i=4, done[3].
t{operator<<(A &a): os << this.i=4, done.
}, tt{operator<<(A &a): os << this.i=6, done.
}, done.
~A() done[3].
~A() done[2].
A(): this.i=4, done[2].
print(T &&t): t.i=4, T tt(t),
A(A &a): this.i=6, a.i=4, done[3].
t{operator<<(A &a): os << this.i=4, done.
}, tt{operator<<(A &a): os << this.i=6, done.
}, done.
~A() done[3].
~A() done[2].
print(T &&t): t.i=4, T tt(t),
A(A &a): this.i=6, a.i=4, done[2].
t{operator<<(A &a): os << this.i=4, done.
}, tt{operator<<(A &a): os << this.i=6, done.
}, done.
~A() done[2].
------- here we test -------
operator<<(const A &a): os << this.i=4, done.
operator<<(A &a): os << this.i=4, done.
A(const T &a): this.i=5, a.i=4, done[2].
operator<<(A &&a): os << this.i=5, done.
A(A &a): this.i=6, a.i=4, done[3].
operator<<(A &&a): os << this.i=6, done.

A(): this.i=4, done[4].
operator<<(A &&a): os << this.i=4, done.

~A() done[4].
~A() done[3].
~A() done[2].
~A() done[1].
*/
```

当自定义类未定义任何拷贝操作的时候，编译器才会合成移动操作，且要求类的数据成员都可移动（成员是内置类型或是定义了移动操作的类）。移动操作不可用时，对应的拷贝操作就会代替移动操作被调用。移动操作可以加一个`noexcept`关键字，否则，比如，标准库的`vector`还是会调用拷贝操作符以将原数据赋值到扩容的新数组上。因为`vector`调用的是`std::move_if_noexcept`，它依条件对元素对象调用`std::move`或返回`const &`类型。

对象的`const`、`&`限定 —— 访问控制限定的是外部的类访问类内成员的权限，而引用与`const`限定符是为了声明类方法对类数据成员的修改与否。常量成员函数（`const`限定）不修改对象的数据成员，要在`const`限定声明的方法内修改一个数据成员，可以给这个数据成员加一个`mutable`声明。`&`和`&&`（左值与右值引用，一旦标记其一，所有同签名方法就都要做标记）限定符声明对象是一个左值或右值。`const`与引用限定都可以区分重载版本，编译器会选出与对象的类型限定最匹配的方法。注：由于早期版本没有引用限定特性，就会出现，比如，`str1+str2="wow"`的情况了，因为`string`没有限定赋值运算符可作用的对象只能是左值引用类型的。



#### III

###### 容器、迭代器、泛型算法库

1. 列表初始化 —— 是一种非常好的构造方式，容器、数组和聚合类（再比如，参数绑定的`bind`接口）都支持它，构造、赋值都支持它，（除了聚合类）它本质上就是以`initializer_list`为入参。
2. 关联容器：关联是指键与值之间的关联。关联容器对象的下标操作在元素不存在的情况下会执行关联键（值）的默认初始化（相当于插入这个键），对应的`at`接口则会在元素不存在的时候抛出`out_of_range`异常，但后者一般没必要用，因为查找接口很方便。删除方法`erase`接受key，返回被删除的元素个数。
3. 迭代器：C++容器基本操作与迭代器设计风格：正向迭代器的范围是从首到尾后节点，反向迭代器是从尾到首前节点。擦除只接收正向迭代器。反向迭代器有一个`base`方法用来转换成对应的正向迭代器。插入操作是往迭代器指向的元素前面插入。单向链表与其他容器的设计是相反的，即，往后插入与删除，并拥有首前迭代器。迭代器一般取前闭后开范围。`vector`（`string`是`char`类型的`vector`）可修改，可随机访问，元素往一个方向追加，存储空间（数组）会重新分配。可变数组`vector`的迭代器就是数组下标的封装。`deque`[STL](https://www.cnblogs.com/ybf-yyj/p/10185711.html) 的实现基于一个主控可变数组（其元素为指针，指向固定大小的数组段），迭代器由主控数组下标、固定数组段的首尾点及当前点构成。首尾操作无需像`vector`那样移动很多元素。`list`(`forward_list`)，迭代器依赖链表节点指针实现。建议：除了频繁在首尾操作，一般操作频繁的，都选择性能更高的链表类。顶多追加的就用可变数组类。以迭代器为入参的删除操作返回下一个迭代（以支持`for range`）。

泛型算法库有很多容易实现的函数，比如`fill` `fill_n` `equal` `accumulate`（`numeric`库）`for_each` `copy` `replace` `replace_copy` `replace_copy_if` `find_if` `unique`等，我们知晓这些库函数的存在后，就不该自己去手写。甚至，有些复杂的函数，还可以用快速实现某个算法，比如算法库中`partation`与可基于它实现的 leetcode 283 或快排算法。其中，有些可能需要对容器进行插入操作，可以使用`back_inserter(container)`、`front_inserter(container)`与`inserter(container, container.itr)`获取可调用`push_back`或`push`的迭代器对象。PS. 链表类有自己的 `merge`, `remove`, `reverse`, `sort` 和 `unique` 等库函数。此外，容器类也普遍支持一些简单的算法，比如，查找方法`find`、`count`、`lower_bound`、`upper_bound`（两个`bound`方法，分别表示查找第一个不小于给定值，和大于给定值的元素，二者合起来表示一个相等元素的范围）和`equal_range`接受key，返回迭代器类型的`pair`。

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

// leetcode 283

class Solution {
   public:
    template <class ForwardIt, class UnaryPredicate>
    ForwardIt partition(ForwardIt first, ForwardIt last, UnaryPredicate p) { // partition
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
    void quicksort(ForwardIt first, ForwardIt last) { // qsort
        if (first == last) return;
        auto pivot = *std::next(first, std::distance(first, last) / 2);
        ForwardIt first_not = partition(first, last, [=](const auto& em) { return em < pivot; });
        quicksort(first, first_not);
        first_not = partition(first_not, last, [=](const auto& em) { return em == pivot; });
        quicksort(first_not, last);
    }
    void moveZeroes(std::vector<int>& nums) {
        // std::partition is not stable, use std::stable_partition
        partition(nums.begin(), nums.end(), [](const auto& em) { return em != 0; });
    }
};

int main() {
    std::vector<int> v{0, 1, 0, 5, 4, 3, 0, 2};
    Solution().quicksort(v.begin(), v.end());
    for_each(v.begin(), v.end(), [](const auto& em) { std::cout << em << " "; });

    v.assign({0, 3, 0, 1, 12});
    Solution().moveZeroes(v);
    for_each(v.begin(), v.end(), [](const auto& em) { std::cout << em << " "; });
}
```

###### 字符串处理、输入输出

字符串类本质是一个`char`类型`vector`，C风格的字符指针可以自动转换成字符串。与Java不同，C++的字符串不是`const`的，其中的每个字符都可以像数组那样被替换。改变字符串的库函数也可以看作在改变`vector`，`append`在`end`处插入，`replace`先删再插入。`r?find`（借用一下正则表达）查找子串，`find_(?:last|first)(?:_not)?_of`查找字符。`to_string`和`sto(:?i|l|ul|ll|ull|f|d|ld)`用于在字符串与数值类型之间转换。

注：`cctype`中有处理字符类型的API，字符串的`c_str`接口可以转C风格的字符指针。

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

我们在写算法题的时候，一般不会涉及文件的处理，都是在内存中。但涉及到大型的软件工程，就需要文件或数据库这些外围数据节点了。同时，为了降低对内存空间的消耗，用流来读取文件。所有编程语言的IO设计模式都是大同小异的，比如，C系语言类似地都会默认打开的三个标准输入输出，对应了操作系统标准文件操作符0、1与2，而像Go这样的现代C语言还提供了类似`>/dev/null`这样的东西：`io.ioutil.Discard`，而一切文件操作又都是通过操作系统的文件描述符或句柄来实现的，它们都基于类似的一套底层的`read`和`write`API，又外加了一层缓存或者流处理接口而已。而对于写算法题来说，常用的是字符串相关的IO库。建议：基于`sstream`，`getline`（就像Go的`Scanner.Scan`一样，C/C++的`getline`会读取并抛弃流中的分隔符`delim`，而`get[s]`方法不会，它会读取分隔符，然后将分隔符`unget`或`putback`，而流操作允许我们回退最多一个字符），`copy`和`stream_iteartor`实现C++的`split`与`join`接口。`getline`比较原始，只能接受一个字符类型作为分隔符；而新标准的泛型接口支持C原生字符串类型作为分隔符，且风格将与迭代器更为统一，譬如可以用这样一条语句来打印容器内容：`std::copy(v.begin(), v.end(), std::ostream_iterator<T>(std::cout, "\n"));`。

```cpp
#include <algorithm>
#include <iostream>
#include <iterator>
#include <sstream>
#include <vector>

// leetcode 1451
    // split, sort and join

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

###### 智能指针、`allocator`

智能指针是基于`use_count`对指针类型的封装。原生的`new`和`delete`、`delete []`能够完成**动态内存**管理，但智能智能类更加安全和方便，它提供了：解引用`*`、解引用并取成员`->`、`swap`、`reset([p[, d]])`、`get()`等接口；`deleter`机制来代替默认的`delete`操作（如果所持有的指针不是动态内存指针会报错）来实现资源的自动释放；同时，智能指针的析构会执行删除操作。可以用原生的`new`创建出的动态内存指针来初始化一个智能指针，但对于共享指针，优先使用`make_shared<T>(args)`方法构造对象。

`deleter`类是`unique_ptr`类的一部分，而`shared_ptr`可以在运行时更改它的删除器，它的实现机制可能是这样的：`del ? del(p) : delete p;`，其中`del`是删除器。`shared_ptr`与`unique_ptr`的删除器都可以做以一个所指类型的指针为入参的调用，但`unique_ptr`要求在定义的时候明确删除器类型。

```cpp
#include <iostream>
#include <memory>

class DebugDelete {
   public:
    DebugDelete(std::ostream &s = std::cerr) : os(s) {}
    // as with any function template, the type of T is deduced by the compiler
    template <typename T>
    void operator()(T *p) const {
        os << "deleting unique_ptr" << std::endl;
        delete p;
    }

   private:
    std::ostream &os;
};

template <typename T>
void deleter(T *p) {
    std::cout << *p << std::endl;
    delete p;
};

void called_locally() {
    int i = 2;
    std::shared_ptr<int> p(
        &i, [](int *ptr) { std::cout << "before killed, I hold a number: " << *ptr << std::endl; });
    std::cout << p.use_count() << std::endl;
}

int main() {
    called_locally();
    int *pi = new int;
    std::unique_ptr<int, DebugDelete> p(pi, DebugDelete());
    std::unique_ptr<int, void (*)(int *)> pp(new int(2), deleter);  // decltype(deleter<int>) *
}
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



#### ...

###### 正则表达式

正则，就是用字符来描述字符。正则的规则在不同编程语言中是通用，而正则的规则又有几种不同的标准。

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
    
3. 不同的正则标准会用到一些不同的自定义字符集，比如有的标准使用`[[::Alpha:]]`这样的字符串表示字母。

:two: 正则表达式中 [不] 捕获子表达式的规则：

1. 在正则表达式中，反斜杠加数字序号，表示被捕获的子表达式（一旦匹配完成了，一般就用美元符号`$`获取该子表达式，比如在IDE的字符串替换、或者编程语言基于正则匹配的子串替换中使用）
2. `?`表示**非捕获**，`?`结合`:`表示不捕获但匹配，`?`结合`= ! <`表示不捕获也不匹配（预查是否匹配，不消耗字符）
    0. `(?:exp)` 不捕获但匹配，比如 `fl(?:y|ies)`匹配单复数的苍蝇的英文
    1. `exp1(?=exp2)`： 正向肯定预查（后面是 `exp2` 吧？）
    2. `exp1(?!exp2)`： 正向否定预查（后面不是 `exp2` 吧？）
    3. `(?<=exp2)exp1`： 反向肯定预查（前面是 `exp2` 吧？）
    4. `(?<!exp2)exp1`： 反向否定预查（前面不是 `exp2`）

