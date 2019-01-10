---
title: 数组前k小元素的各种方法对比
date: 2019-01-10 14:46:25
tags:
---

# 引子

**给定一个数组，寻找出数组中前k小的元素。**

对于这个经典问题，我之前一直只知道可以用**堆**来解决。

构造一个大根堆，然后遍历所有数据，如果数据小于堆顶元素，就替换堆顶元素，调整堆。

这样最后堆中的元素就是原数组中前k小的元素。

这个算法的时间复杂度为O(nlogk)。


但是，我发现，C++ stl 中关于堆的函数有，`make_heap`，`push_heap`，`pop_heap`，`sort_heap`，`is_heap`。

这其中，貌似并没有提供“替换堆顶元素，调整堆”的函数。要实现这个效果，可以通过先`pop_heap`，再`push_heap`来实现。

因此，我对这两种方式进行了一个对比：
- 方法1：先`pop_heap`，再`push_heap`来替换堆顶元素。
- 方法2：自己实现调整堆顶元素的功能。

# 方法1

使用`pop_heap`和`push_heap`来两步调整堆顶。

```c++
int func(vector<int> &data, size_t k) {
    cout << "std::pop_heap and std::push_heap" << endl;
    vector<int> heap;
    heap.reserve(k);
    for (int i : data) {
        if (heap.size() < k) {
            heap.push_back(i);
            push_heap(heap.begin(), heap.end());
        } else {
            if (i < heap.front()) {
                pop_heap(heap.begin(), heap.end());
                heap.back() = i;
                push_heap(heap.begin(), heap.end());
            }
        }
    }
    return heap.front();
}
```

# 方法2

自己实现调整堆顶。

```c++
int func(vector<int> &data, size_t k) {
    cout << "replace heap top and adjust heap" << endl;
    vector<int> heap;
    heap.reserve(k);
    for (int i : data) {
        if (heap.size() < k) {
            heap.push_back(i);
            push_heap(heap.begin(), heap.end());
        } else {
            if (i < heap.front()) {
                heap.front() = i;
                bool adjust = true;
                int st = 0;
                while (adjust) {
                    int largest = st;
                    int left = (st << 1) + 1;
                    int right = (st << 1) + 2;
                    int size = heap.size();
                    if (left < size && heap[left] > heap[largest])
                        largest = left;
                    if (right < size && heap[right] > heap[largest])
                        largest = right;
                    if (largest != st) {
                        std::swap(heap[st], heap[largest]);
                        st = largest;
                    } else {
                        adjust = false;
                    }
                }
            }
        }
    }
    return heap.front();
}
```

方法1 和方法2 的运行结果如下：

```
std::pop_heap and std::push_heap
result = 31415926 CORRECT
copy data time used = 15ms
pure compute time used = 233ms

replace heap top and adjust heap
result = 31415926 CORRECT
copy data time used = 15ms
pure compute time used = 246ms
```

果然，还是我写的调整堆顶太丑了，连stl的两步调整的效率都比不上。

理论上，一步调整堆顶的效率肯定会比两步调整要高吧。

所以结论是，下次遇到堆调整的问题，直接上`pop_heap`和`push_heap`。

------

回到最初的问题。

对于寻找数组前k小元素的问题，在知乎上讨论之后，发现其实还有比建堆更高效的方法。

那就是不断使用partition算法。

partition算法是这样的，随机取出一个元素（比如就取最后的元素）作为枢轴（pivot），然后调整数组，把数组分为两部分，左边的都是不大于枢轴的，右边的都是不小于枢轴的。

通过使用一次partition算法，我们可以将数组分割。但是此时的分割不一定是符合要求的分割（枢轴不是第k个元素）。因此，我们需要不断分割，直到枢轴落到第k个元素。此时就找到了前k小的元素。

这个算法的时间复杂度为O(n)。

# 方法3

使用stl的`std::nth_element`函数。

这个函数内部就是采用不断partition的方法。

```c++
int func(vector<int> &data, size_t k) {
    cout << "std::nth_element" << endl;
    nth_element(data.begin(), data.begin() + k - 1, data.end());
    display(data);
    return data[k - 1];
}
```

运行结果如下：

```
std::nth_element
result = 31415926 CORRECT
copy data time used = 14ms
pure compute time used = 86ms
```

可以看到运行速度一下快了一大截。

# 方法4

自己手痒实现的类似`std::nth_element`的算法。

```c++
int func(vector<int> &data, size_t k) {
    cout << "my partition function" << endl;

    int left = 0, right = data.size() - 1;
    while (true) {
        int l = left, r = right;
        int pivot = data[r];
        while (l < r) {
            while (l < r && data[l] <= pivot)
                ++l;
            data[r] = data[l];
            while (l < r && data[r] >= pivot)
                --r;
            data[l] = data[r];
        }
        data[l] = pivot;
        if (l < k - 1) {
            left = l + 1;
        } else if (l > k - 1) {
            right = l - 1;
        } else {
            return data[l];
        }
    }
}
```

运行结果如下：

```
my partition function
result = 31415926 CORRECT
copy data time used = 14ms
pure compute time used = 84ms
```

自己写的算法效率跟stl差不多。

# 方法5

知乎上还有人说用`std::partial_sort`函数。

这个函数是部分排序函数，内部使用的是堆排序。

```c++
int func(vector<int> &data, size_t k) {
    cout << "std::partial_sort" << endl;
    partial_sort(data.begin(), data.begin() + k, data.end());
    display(data);
    return data[k - 1];
}
```

运行结果如下：

```
std::partial_sort
result = 31415926 CORRECT
copy data time used = 13ms
pure compute time used = 830ms
```

太慢了吧。

# 方法6

stl实现了一个优先队列，其实内部也是堆。

```c++
int func(vector<int> &data, size_t k) {
    cout << "std::priority_queue" << endl;
    priority_queue<int, vector<int>, less<int>> prique;
    for (int i : data) {
        if (prique.size() < k) {
            prique.push(i);
        } else if (prique.top() > i) {
            prique.pop();
            prique.push(i);
        }
    }
    return prique.top();
}
```

运行结果如下：

```
std::priority_queue
result = 31415926 CORRECT
copy data time used = 14ms
pure compute time used = 268ms
```

可以看到跟维护堆的方法效率差不多。

# 结论

对于“数组中前k小元素”的问题，有两种算法。

1. 维护堆数据结构。（O(nlogk)）
2. 不断使用partition算法。（O(n)）

**方法1**的优点在于，可以不需要一次处理所有数据，也就是说可以处理流数据。

而且，其实不需要纠结`pop_heap`+`push_heap`，跟直接手动替换堆顶元素再调整堆，哪个更快。

除非stl官方实现了后者，我们还是用前者比较方便。或者用优先队列更省心。


如果用**方法2**，虽然时间复杂度比较低，但是其实波动很大。

在我的测试中，有时候可以比方法1高效10倍，有时候跟方法1差不多。

而且这种方法必须一次性处理所有数据，必须纵观全局。

至于`partial_sort`，拜拜了吧您。
