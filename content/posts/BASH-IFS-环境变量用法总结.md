---
title: BASH IFS 环境变量用法总结
date: 2022-12-14 01:10:28
tags:
  - bash
---

之前从网上找了很多关于 IFS 的二手资料，每次用每次都得再看一遍，理解还是不够透彻。最近对照着 `man bash` ，终于把这个 IFS 搞清楚了。

<!-- more -->

# IFS 的三种作用

IFS 其实只在 3 个地方发挥作用：

1. 用于扩展带双引号的 `"$*"`
2. 用于不带双引号的变量扩展 / 子命令扩展 / 算数扩展
3. 用于在内建命令 read 中进行断词

可以看到，其实只有 bash 本身和 read 命令会用到 IFS 这个环境变量。所以，除了

```shell
IFS=xxx bash -c "xxx"
```

和

```shell
IFS=xxx read a b c
```

之外，任何的 `IFS=xxx` 临时环境变量都是不会起到任何作用的。

## 辅助工具

先写一个小的辅助程序，名为 argsecho，用于直观展示所有参数列表。

```python
import sys
for ind, arg in enumerate(sys.argv[1:], 1):
    print("${}: {}".format(ind, arg), flush=True)
```

## 1. 用于扩展带双引号的 `$*`

对于一个数组

```shell
arr=(1 2 3 4 5)
```

`"${arr[@]}"` 会展开为 `"1" "2" "3" "4" "5"` ，是最忠实于原数组的展开方式。通常用于将参数列表原封不动传递给子命令。展开的过程中不涉及 IFS。

```shell
ladyrick $ argsecho "${arr[@]}"
$1: 1
$2: 2
$3: 3
$4: 4
$5: 5
```

`${arr[*]}` 则会读取 IFS 的第一个字符，作为分隔符。设为 c，则会展开为 `"1c2c3c4c5"`。如果 IFS 为空，则直接拼接，不插入分隔符，直接展开为`"12345"`。

```shell
$> IFS=c
$> argsecho "${arr[*]}"
$1: 1c2c3c4c5
```

bash 中凡是需要把数组展开的，都遵循这一规律。比如：

```shell
ladyrick $ set -- 1 2 3 4 5
ladyrick $ argsecho "$@"
$1: 1
$2: 2
$3: 3
$4: 4
$5: 5
ladyrick $ IFS=c
ladyrick $ argsecho "$*"
$1: 1c2c3c4c5
```

再比如，根据前缀查找环境变量名：

```shell
ladyrick $ MYENV1=1 MYENV2=2 MYENV3=3 MYENV4=4
ladyrick $ argsecho "${!MYENV@}"
$1: MYENV1
$2: MYENV2
$3: MYENV3
$4: MYENV4
ladyrick $ IFS=c
ladyrick $ argsecho "${!MYENV*}"
$1: MYENV1cMYENV2cMYENV3cMYENV4
```

再次强调，直接在命令前面设置临时变量 IFS 是没有用的，因为这种写法的 IFS 环境变量只会在 argsecho 命令内部生效，并不会在参数解析阶段生效。

```shell
ladyrick $ set -- 1 2 3 4 5
ladyrick $ IFS=c argsecho "$*"
$1: 1 2 3 4 5
```

## 2. 用于不带双引号的变量扩展 / 子命令扩展 / 算数扩展

当不带双引号的这三种扩展作为命令行参数时，会根据 IFS 环境变量的值对这三种扩展的结果进行拆分，拆分成多个参数。如果 IFS 为空，则不会进行拆分。

### 变量扩展

```shell
ladyrick $ args="1:2,3.4/5"
ladyrick $ IFS=:,./
ladyrick $ argsecho $args
$1: 1
$2: 2
$3: 3
$4: 4
$5: 5
```

### 子命令扩展

```shell
ladyrick $ IFS=abcd
ladyrick $ argsecho $(echo 1a2b3c4d5)
$1: 1
$2: 2
$3: 3
$4: 4
$5: 5
```

### 算术扩展

```shell
ladyrick $ IFS=0
ladyrick $ argsecho $((102030400 + 5))
$1: 1
$2: 2
$3: 3
$4: 4
$5: 5
```

还有一些细节方面的注意点，一般不需要关注，仅供备忘查阅。

> The shell treats each character of **IFS** as a delimiter, and splits the results of the other expansions into words on these characters. If **IFS** is unset, or its value is exactly **\<space\>\<tab\>\<newline\>**, the default, then sequences of **\<space\>**, **\<tab\>**, and **\<newline\>** at the beginning and end of the results of the previous expansions are ignored, and any sequence of **IFS** characters not at the beginning or end serves to delimit words. If **IFS** has a value other than the default, then sequences of the whitespace characters **space** and **tab** are ignored at the beginning and end of the word, as long as the whitespace character is in the value of **IFS** (an **IFS** whitespace character). Any character in **IFS** that is not **IFS** whitespace, along with any adjacent **IFS** whitespace characters, delimits a field. A sequence of **IFS** whitespace characters is also treated as a delimiter. If the value of **IFS** is null, no word splitting occurs.

## 3. 用于在内建命令 read 中进行断词

这是唯一一个在命令内部生效的 IFS 用法，可以使用临时环境变量的写法。

IFS 用于在 read 内部，把一行输入文本断为一个一个的词，然后赋值给传入的变量名。举个例子：

```shell
ladyrick $ IFS=0 read a b c <<< 10203
ladyrick $ argsecho "a=$a" "b=$b" "c=$c"
$1: a=1
$2: b=2
$3: c=3
```

这里的输入数据为`10203`，read 从 stdin 接收输入后，在内部读取到 IFS 为字符 `'0'`，所以将它拆分为了`["1", "2", "3"]`，然后分别赋值给`a b c` 三个变量。

这里还有一点需要注意，假如传入的变量数量少于分词后词的数量，那么会依次赋值，最后一个变量接收剩余所有的内容，不再进行分词。举个例子：

```shell
ladyrick $ IFS=abcde read a b c <<< 1a2b3c4d5e6
ladyrick $ argsecho "a=$a" "b=$b" "c=$c"
$1: a=1
$2: b=2
$3: c=3c4d5e6
```

注意这里 c 的获取，并不是先整体分词然后再组合，可以理解为一边分词一边赋值，最后 c 只剩一个变量，就不再分词。极端情况下，就是我们常见的只有一个变量的情况。这时也不会进行分词。

> PS：这里可能会有疑问，IFS 分词跟 read 的 `-d` 参数有什么区别？`-d` 也是作为分隔符。
>
> 答案是，IFS 是在一行内进行分词，`-d` 是用来分割不同的行的。
>
> 注意观察有没有 `-d` 参数的行为差别：

```shell
ladyrick $ IFS=0 read -a arr <<< '10203
40506'
ladyrick $ argsecho "${arr[@]}"
$1: 1
$2: 2
$3: 3
ladyrick $ IFS=0 read -a arr -d '' <<< '10203
40506'
ladyrick $ argsecho "${arr[@]}"
$1: 1
$2: 2
$3: 3
4
$4: 5
$5: 6

ladyrick $ # 思考题：为啥上面的输出 6 后面会多一个换行符？
```

# 后记

其实 IFS 还有一个作用，是用在 `complete` 命令的 `-W` 参数中，进行分词用的。这个命令是用来进行 bash 自动补全的，比较小众，所以就不介绍了。
