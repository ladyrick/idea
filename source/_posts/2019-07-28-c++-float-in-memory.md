---
title: C++ float数据在内存中的表示形式
date: 2019-07-28 17:21:41
tags:
---

# 介绍
参考：https://blog.csdn.net/hemingliang1987/article/details/11630409
简单地说，一个float型实数在内存中占4个字节，即32个二进制bit，从低位到高位依次叫第0位到第31位。
这32位可以分为3个部分：符号位（第31位），阶码（第30位到第23位共8位），尾数（最低23位）。 
1. 符号位。最高位也就是第31位表示这个实数是正数还是负数，为0表示正数或0，为1表示负数。
2. 阶码。第30位到第23位这8个二进制位表示该实数转化为规格化的二进制实数后的指数与127(127即所谓偏移量)之和即所谓阶码。
规格化的二进制实数的指数只能在-127~+127之间，所以，一个float型数的最大值在+2^127即+3.4\*10^38，最小值在-2^127即-3.4\*10^38。 
3. 尾数。其他最低的23位即第22位到第0位表示该实数转化为规格化的二进制实数后小数点以后的其余各位即所谓尾数。

<!-- more -->

---
# 例一
例如，将十进制178.125表示成机器内的32个字节的二进制形式。

## 第一步：将178.125表示成二进制数
(178.125)(十进制数)=(10110010.001)(二进制形式)
## 第二步：将二进制形式的浮点实数转化为规格化的形式
小数点向左移动7个二进制位可以得到：
10110010.001=1.0110010001*2^7
因而产生了以下三项: 
- 符号位：该数为正数，故第31位为0，占一个二进制位。
- 阶码：指数为7，故其阶码为127+7=134=(10000110)(二进制)，占从第30到第23共8个二进制位。
- 尾数为小数点后的部分， 即0110010001。因为尾数共23个二进制位，在后面补13个0，即：01100100010000000000000

所以，178.125在内存中的实际表示方式为: 
```
0 10000110 01100100010000000000000
```

---
# 例二
再如，将-0.15625表示成机器内的32个字节的形式. 
## 第一步：将-0.15625表示成二进制形式
(-0.15625)(十进制数)=(-0.00101)(二进制形式)
## 第二步：将二进制形式的浮点数转化为规格化的形式
小数点向右移动3个二进制位可以得到：
-0.00101=-1.01*2^(-3)
同样，产生了三项: 
- 符号位：该数为负数，故第31位为1，占一个二进制位。
- 阶码：指数为-3，故其阶码为127+(-3)=124=01111100，占从第30到第23共8个二进制位。
- 尾数为小数点后的01，当然后面要补21个0。
所以，-0.15625在内存中的实际表示形式为: 
```
1 01111100 01000000000000000000000
```

# 验证
可以通过以下的C++程序验证之。记得添加编译选项`--std=c++11`
```c++
#include <iostream>
#include <string>
#include <sstream>
#include <bitset>
#include <typeinfo>

using namespace std;

class cast2bits {
private:
    struct bits {
        unsigned char b7 : 1;
        unsigned char b6 : 1;
        unsigned char b5 : 1;
        unsigned char b4 : 1;
        unsigned char b3 : 1;
        unsigned char b2 : 1;
        unsigned char b1 : 1;
        unsigned char b0 : 1;
    };
    int length;
    ostringstream oss;
    ostringstream data;

public:
    template<class T>
    explicit cast2bits(const T &input) {
        data << input << " type: " << typeid(input).name();
        const T *pTInput = &input;
        auto *pBitsinput = (bits *) pTInput;
        int n = sizeof(T) / sizeof(bits);
        length = 9 * n - 1;
        for (int i = n - 1; i >= 0; --i) {
            oss << (int) ((pBitsinput + i)->b0)
                << (int) ((pBitsinput + i)->b1)
                << (int) ((pBitsinput + i)->b2)
                << (int) ((pBitsinput + i)->b3)
                << (int) ((pBitsinput + i)->b4)
                << (int) ((pBitsinput + i)->b5)
                << (int) ((pBitsinput + i)->b6)
                << (int) ((pBitsinput + i)->b7)
                << ' ';
        }
    }

    friend ostream &operator<<(ostream &os, const cast2bits &c) {
        os << c.data.str() << endl;
        os << "M" << string((unsigned long long int) (c.length - 2), ' ') << "L" << endl;
        os << c.oss.str() << endl;
        return os;
    }
};


int main() {
    cout << cast2bits(178.125f) << endl;
    cout << cast2bits(-0.15625f) << endl;
    return 0;
}
```

输出为：
```
178.125 type: f
M L
01000011 00110010 00100000 00000000 

-0.15625 type: f
M L
10111110 00100000 00000000 00000000

```

最后，这个程序其实不仅可以处理浮点数，也可以处理其他各种数据类型在内存中的表示形式。
比如，你可以试试以下代码的输出结果。
```c++
cout << cast2bits(178.125) << endl;
cout << cast2bits(-0.15625) << endl;
cout << cast2bits(123) << endl;
cout << cast2bits(123l) << endl;
cout << cast2bits(123ll) << endl;
```