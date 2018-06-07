---
title: C++：explicit关键字
tags:
date: 2017-11-09 16:14:33
---

explicit关键字作用于类的构造函数。一旦类的构造函数声明了explicit关键字，构造函数就必须使用显示的方式调用。这样做可以防止构造函数被不知不觉地，莫名其妙地调用。

下面举例子。

# 在普通构造函数中的explicit：

类的定义如下（未声明explicit）：

```
class num {
    int n;

public:
    num(int n) {
        this->n = n;
        cout << "constructor: " << n << endl;
    }
    ~num() {
        cout << "destructor: " << n << endl;
    }
};
```

此时，由于未声明explicit，因此，可以使用这种方式来初始化：

```
num a = 1;
num b = 2;
num c(3);
```

而且，如果你写了多个构造函数，有的有explicit，有的没有，代码就会自动去找没有声明explicit的构造函数去调用，就像是声明了explicit的构造函数不存在一样。例如：

```
class num {
    int n;

public:
    num(int n) {
        this->n = n;
        cout << "int constructor: " << n << endl;
    }

    explicit num(float n) {
        this->n = int(n);
        cout << "float constructor: " << n << endl;
    }

    num(double n) {
        this->n = int(n);
        cout << "double constructor: " << n << endl;
    }

    ~num() {
        cout << "destructor: " << n << endl;
    }
};
```

调用：

```
num a = 1;
num b = 2.0f;
num c = 3.0;
```

此时，编译器会自动将`2.0f`转换成double，导致double constructor调用两次。但是，假如声明了explicit的不是float constructor，而是int或者double，就会直接导致编译不通过。因为，如果是int constructor声明了explicit，那么编译器无法决定int该转成float还是double。同理，如果是double constructor声明了explicit，那么编译器无法决定double该转成float还是int。这就很坑。凭什么float就可以义无反顾地转成了double，到了int和double就迷茫了？因此，我建议不要采用这种写法。

# 在拷贝构造函数中的explicit：

类的定义如下：

```
class num {
    int n;

public:
    num(int n) {
        this->n = n;
        cout << "constructor: " << n << endl;
    }

    ~num() {
        cout << "destructor: " << n << endl;
    }
    
    num operator++() {
        n = n + 1;
        return *this;
    }
};
```

调用：

```
num n(3);
++n;
++n;
```

需要注意的是，在重载++运算符时，我返回了一个对象。在运行过程中，可以发现，我写的构造函数调用了一次，析构函数却调用了3次。这就是因为，返回的对象，其实是调用了拷贝构造函数，然后返回的一个匿名对象。因此，默认的拷贝构造函数被悄悄调用了两次。在实际中，可以通过返回引用的方法避免生成匿名对象。但是如何让编译器来帮我们检查，以避免这种事情发生呢？方法就是，将拷贝构造函数声明为explicit。如下。

```
class num {
    int n;

public:
    num(int n) {
        this->n = n;
        cout << "constructor: " << n << endl;
    }

    explicit num(const num& nm) {
        this->n = nm.n;
        cout << "copy constructor: " << n << endl;
    }

    ~num() {
        cout << "destructor: " << n << endl;
    }
    
    num operator++() {
        n = n + 1;
        return *this;
    }
};
```

像这样写，就会导致编译错误。因为`++`运算符返回对象需要隐式调用拷贝构造函数，而explicit不让它被隐式调用。由于返回对象的时候，必然会隐式调用拷贝构造函数，所以这么做其实是根本性杜绝了返回对象这种操作（返回引用多好，返回对象吃力不讨好）。如果真的有需求，需要实现返回新的对象，可以在函数中构造一个local的对象，然后返回其引用。像这样：

```
num& operator++() {
    n = n + 1;
    num temp(*this);
    return temp;
}
```

当然，这样会产生一个警告，告诉你返回了局部变量。注意，不要在函数中new一个对象，返回其引用。像这样：

```
num& operator++() {
    n = n + 1;
    num* temp = new num(*this);
    return *temp;
}
```

这种做法会导致new的那个对象没法delete，对象没法析构，造成内存泄漏。如果运行的话，可以发现，析构函数调用次数会比构造函数少。

**题外话：关于直接赋值初始化**

前面说到，可以直接用赋值来进行初始化。例如：

```
num a = 1;
num b = 2.0f;
num c = 3.0;
```

在测试的时候，我发现，当普通构造函数和拷贝构造函数都自己写，并且拷贝构造函数的参数没有const关键字时，这种赋值的方式是会报错的，跟是否声明explicit无关。如下：

```
#include <iostream>
using namespace std;
class num {
    int n;

public:
    num(int n) {
        this->n = n;
        cout << "constructor: " << n << endl;
    }

    num(num& nm) {
        this->n = nm.n;
        cout << "copy constructor: " << n << endl;
    }

    ~num() {
        cout << "destructor: " << n << endl;
    }
};
int main(int argc, char* argv[]) {
    num a = 1;
    return 0;
}
```

报的错是：`error: invalid initialization of non-const reference of type 'num&' from an rvalue of type 'num'`。

OK，你说我不const，还说我是右值是吧？Fine。我改。修改方案1：参数改成const

```
num(const num& nm) {
    this->n = nm.n;
    cout << "const copy constructor: " << n << endl;
}
```

修改方案2：改成右值引用

```
num(num&& nm) {
    this->n = nm.n;
    cout << "right value copy constructor: " << n << endl;
}
```

经过测试，这两种修改方案，都不会报错。更神奇的是，无论哪种修改方案，输出都显示只调用了普通的构造函数  (╯-_-)╯┴**—**┴ 拷贝构造函数内心：你根本不调用我，还管我管这么宽？经过轮子哥指点，终于明白了。**GCC就是会帮你检查一下num(num(1))是否合法，然后当num(1)来生成代码的。**真是无语呢……