---
title: C++ Puzzle
date: 2018-06-24 22:07:19
tags:
---

本文搜集了一些有趣的C++的题目，有些题目令人大跌眼镜。来看看！


<!--more-->


<script>
function toggle(that) {
    var dom = that.parentElement.getElementsByClassName("cpuzzle-hint")[0];
    var display = dom.style.display;
    if (display == "none") {dom.style.display = "block"; that.textContent = "关闭提示";}
    else {dom.style.display = "none"; that.textContent = "查看提示";}
}
</script>


## 请问输出结果是什么？如何解释？？
```C++
#include <iostream>

#define TOTAL_ELEMENTS (sizeof(array) / sizeof(array[0]))
int array[] = {23, 34, 12, 17, 204, 99, 16};

int main() {
    for (int d = -1; d <= (TOTAL_ELEMENTS - 2); d++)
        std::cout << array[d + 1] << std::endl;
    return 0;
}
```
<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
测试发现实际输出结果为空。
sizeof运算符得到的是一个size_t类型，而size_t类型是无符号的。
当用有符号数跟无符号数比较时，会将有符号数转成无符号数，也就是将-1转成无符号数的最大值。

另外，size_t类型在不同位宽的机器上定义不同。如下：
```C++
#ifdef _WIN64
__MINGW_EXTENSION typedef unsigned __int64 size_t;
#else
typedef unsigned int size_t;
#endif /* _WIN64 */
```
PS: 更多关于32位机器和64位机器的区别，请看hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
</div></div>

---

## 请问输出结果是什么？如何解释？？

```C++
#include <iostream>
#define f(a,b) a##b
#define g(a) #a
#define h(a) g(a)

int main()
{
    std::cout << h(f(1,2)) << std::endl;
    std::cout << g(f(1,2)) << std::endl;
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
测试发现输出结果为
```
12
f(1,2)
```
这里首先需要知道宏定义里面`#`的作用。
1. `#`： 其后必须跟一个宏参数，它的作用是将其后的参数内容转换为字符串。
2. `##`: 它的作用是拼接两个符号，如`a##1`，得到`a1`这两个符号不必是宏参数。

然后需要了解宏替换的规则。
1. 如果宏定义中没有`#`或者`##`，则先展开参数再进行替换(跟嵌套的函数调用一样)
2. 否则，参数不展开而是直接替换

在本题中，`h(a)`的定义中没有`#`或者`##`，所以宏定义展开流程是：
展开`f(1,2)` -> 得到`12` -> 展开`h(12)` -> 得到`g(12)` -> 展开`g(12)`发现定义中有`#` -> 直接替换，得到`"12"`

而`g(a)`的定义中有`#`或者`##`，因此会直接替换，在参数两边加上引号，不论参数是什么。于是直接得到字符串`"f(1,2)"`
</div></div>

---

## 程序会输出`None`吗？

```C++
#include <iostream>

int main() {
    int a = 3;
    switch (a) {
        case 1:
            std::cout << "ONE";
            break;
        case 2:
            std::cout << "TWO";
            break;
        defalut:
            std::cout << "NONE";
    }
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
测试发现输出结果为空。
这个问题其实在于`default`拼写错误，写成了`defalut`。
但是真正可怕的是，不像是其它类型的拼写错误，本题的拼写错误，编译没有任何问题，没有人提示你`defalut`未定义。
</div></div>

---

## 它们相等吗？
```C++
#include <stdio.h>

int main() {
    double f = 0.0;
    int i;

    for (i = 0; i < 10; i++)
        f = f + 0.1;

    if (f == 1.0)
        printf("f is 1.0\n");
    else
        printf("f is NOT 1.0\n");

    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
测试发现输出结果显示0.1累加10次并不等于1.0。
这是因为计算机中的浮点数，包括`float`和`double`类型，都是不精确的。因此累加的话会产生累积误差。
计算机中浮点数比较，不要直接用`==`比较，应该计算两者的差，当差小于某个阈值时认为相等。
</div></div>

---

## 逗号表达式可以正确赋值吗？
```C++
#include <iostream>

int main() {
    int a = 1, 2;
    std::cout << "a : " << a;
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
编译错误。逗号表达式在C++中的运算优先级是最低的，因此，赋值运算会优先执行，导致编译器认为`2`是一个表达式，编译错误。
正确的写法是
```C++
int a = (1, 2);
```
</div></div>

---

## 输出结果是什么？
```C++
#include <iostream>

#define SIZE 10

void size(int arr[SIZE]) {
    std::cout << "size of array is:" << sizeof(arr);
}

int main() {
    int arr[SIZE];
    size(arr);
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
输出结果首先不是40。事实上，如果你自己编译了的话，就可以看到显示的警告。
```
warning: 'sizeof' on array function parameter 'arr' will return size of 'int*' [-Wsizeof-array-argument]
```
也就是说，虽然形参中定义的是数组，但是在函数中使用`sizeof`，还是会当作指针来计算。
在我的机器上，输出为8，因为我的是64位机器。
</div></div>

---

## 为什么会产生编译错误？
记住，将一个普通指针传递给const类型的指针是允许的。
```C++
void foo(const char **p) {}

int main(int argc, char **argv) {
    foo(argv);
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
因为这是二级指针，必须先强制类型转换才可以赋值。一级指针则不需要强制类型转换。
`const char ** p`的意思是，类似这样的语句是无效的:`**p = 'a'`。
我们看看假如允许`char**`到`const char**`的隐式类型转换的话，会发生什么。
```C++
int main()
{
    // 我们假设编译器存在漏洞X，允许char** 到 const char**的隐式类型转换。
    char const c = 'a'; // c是一个常量。
    char* p_stupid = &c; // 愚蠢的指针，直接赋值是不行的，太过于耿直，编译错误。
    char* p_smart = nullptr; // 这里有一个聪明的指针，学会了利用漏洞X来修改常量c。
    char const** p_jump = &p_smart; // 利用漏洞X作为跳板。

    *p_jump = &c; // 通过跳板，让p_smart强制指向c。
    // 注意这一步是最有意思的一步，因为这一步的两边都是const char*类型的，赋值完全没问题。

    *p_smart = 'b'; // 因为p_smart本身并不是const char*，所以现在p_smart就可以修改c了，把系统搞崩溃。
}
```
注意看上述步骤，可以看到，除了`char const** p_jump = &p_smart;`这一步，其他步骤没有任何漏洞。
因此我们可以看到，**漏洞X**是可以引发严重后果的。
</div></div>

---

## 输出是什么？
```C++
#include <stdio.h>
#include <stdlib.h>

#define SIZEOF(arr) (sizeof(arr)/sizeof(arr[0]))

#define PrintInt(expr) printf("%s:%d\n",#expr,(expr))

int main() {
    /* The powers of 10 */
    int pot[] = {
            0001,
            0010,
            0100,
            1000
    };
    int i;

    for (i = 0; i < SIZEOF(pot); i++)
        PrintInt(pot[i]);
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
有了前面的关于宏定义的知识，你可能会得出，输出结果是
```
pot[i]:1
pot[i]:10
pot[i]:100
pot[i]:1000
```
但是，虽然躲过了宏定义的坑，其实还是错了，因为数组中的前三个数字带有前导0，这意味着是8进制。。。
所以正确的输出是
```
pot[i]:1
pot[i]:8
pot[i]:64
pot[i]:1000
```
</div></div>

---

## 输出是什么？肯定不是10
```C++
#include <stdio.h>
#include <stdlib.h>

#define PrintInt(expr) printf("%s : %d\n",#expr,(expr))

int main() {
    int y = 100;
    int *p;
    p = (int*)malloc(sizeof(int));
    *p = 10;
    y = y/*p; /*dividing y by *p */;
    PrintInt(y);
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
这道题需要注意的是`y = y/*p;`。
如果是手写代码，或者没有代码高亮，可能确实会以为是`y`除以`*p`。
但是一旦有了代码高亮，瞬间就能看出，`/*p`其实是形成了块注释的开头。因此并不会执行除法。
</div></div>

---

## 如何实现乘5？
```C++
#include <stdio.h>

#define PrintInt(expr) printf("%s : %d\n",#expr,(expr))

int FiveTimes(int a) {
    int t;
    t = a << 2 + a;
    return t;
}

int main() {
    int a = 1, b = 2, c = 3;
    PrintInt(FiveTimes(a));
    PrintInt(FiveTimes(b));
    PrintInt(FiveTimes(c));
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
需要注意的是移位运算符的优先级低于加法，因此得到了错误的结果。
</div></div>

---

## 这个程序哪里有问题？
```C++
#include <stdio.h>
#include <stdlib.h>

int main() {
    int *ptr1, ptr2;
    ptr1 = (int *) malloc(sizeof(int));
    ptr2 = ptr1;
    *ptr2 = 10;
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
定义指针类型，必须每个变量前面都加`*`。
修改：
```C++
int *ptr1, *ptr2;
```
</div></div>

---

## 这个程序可以运行吗？
```C++
#include <stdio.h>

int main() {
    int a = 3, b = 5;

    printf(&a["Ya!Hello! how is this? %s\n"], &b["junk/super"]);
    printf(&a["WHAT%c%c%c  %c%c  %c !\n"], 1["this"],
           2["beauty"], 0["tool"], 0["is"], 3["sensitive"], 4["CCCCCC"]);
    return 0;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
是可以运行的。我们知道，数组下标引用`a[3]`，其实相当于是`*(a+3)`。因此，`a[3]`也可以写成`3[a]`。
因此，`a["Ya!Hello! how is this? %s\n"]`相当于是`"Ya!Hello! how is this? %s\n"[3]`。
</div></div>

---

## `offsetof`的原理是什么？
```C++
#include <stdio.h>

#define offsetof(a, b) ((size_t)(&(((a*)(0))->b)))
struct test {
    int a;
    double b;
    float c;
    int d[20];
};

int main() {
    printf("%d\n", offsetof(test, a));
    printf("%d\n", offsetof(test, b));
    printf("%d\n", offsetof(test, c));
    printf("%d\n", offsetof(test, d));
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
工作原理：
```C++
(
  (size_t)(      // 4.
    &( (         // 3.
      (a*)(0)    // 1.
     )->b )      // 2.
  )
)
```
1. Casting the value zero to the struct pointer type `a*`
2. Getting the struct field `b` of this (illegally placed) struct object
3. Getting the address of this `b` field
4. Casting the address to a `size_t`

注意，现在已经不用这个了，用的是stddef.h自带的`offsetof`宏。它直接调用了编译器提供的函数`__builtin_offsetof`。
直接`include <iostream>`就可以使用了。
</div></div>

---

## `SWAP`宏是如何工作的？
```C++
#include <iostream>

#define SWAP(a, b) ((a) ^= (b) ^= (a) ^= (b))

int main() {
    int a = 1;
    int b = 2;
    SWAP(a, b);
    std::cout << a << b;
}
```

<div><strong
onclick="toggle(this);" style="cursor:pointer;">查看提示</strong><div
class="cpuzzle-hint" style="display:none;padding:10px;box-shadow:0 0 0 3px #f7f0f3 inset;">
可以把`(a) ^= (b) ^= (a) ^= (b)`展开来看。这个表达式，相当于以下三条语句依次执行的结果。
```C++
a = a ^ b;
b = b ^ a;
a = a ^ b;
```
进行一些代换，得到：
```C++
int _a = a;
int _b = b;
a = _a ^ _b;
b = _b ^ (_a ^ _b);
a = (_a ^ _b) ^ (_b ^ (_a ^ _b));
```
根据结合律，去掉所有括号。再根据交换律，以及`x^x`恒等于0，可以化简为：
```C++
b = _a;
a = _b;
```
因此该表达式可以实现交换两个变量的功能。
</div></div>