# 排序算法分析器

[TOC]

本案例取自《数据结构（Python 语言描述）》- 3.7 案列学习：算法探查器。
作者提供的源代码中存在些许错误，本文修正了这些错误。

## 1. profiler 模块

Profiler 对象会跟踪指定算法的执行过程，统计比较次数和交换次数，并记录运行时间。另外，当算法交换某两个值时，分析器还可输出修改后的列表，以便观察列表的变化过程。

调用者可以向 Profiler 对象提供自定义的数值列表，或者让 Profiler 对象自动生成指定大小的随机排序数值列表。调用者还可以要求列表中是否允许包含重复元素。

```c#
"""
File: profiler.py

Defines a class for profiling sort algorithms.
A Profiler object tracks the list, the number of comparisons
and exchanges, and the running time. The Profiler can also
print a trace and can create a list of unique or duplicate
numbers.

Example use:

from profiler import Profiler
from algorithms import selectionSort

p = Profiler()
p.test(selectionSort, size = 15, comp = True,
             exch = True, trace = True)
"""

import time
import random


class Profiler(object):

    def test(self, function, lyst=None, size=10,
             unique=True, comp=True, exch=True,
             trace=False):
        """
        function: the algorithm being profiled
        target: the search target if profiling a search
        lyst: allows the caller to use her list
        size: the size of the list, 10 by default
        unique: if True, list contains unique integers
        comp: if True, count comparisons
        exch: if True, count exchanges
        trace: if True, print the list after each exchange

        Run the function with the given attributes and print
        its profile results.
        """
        self._comp = comp
        self._exch = exch
        self._trace = trace
        if lyst is not None:
            self._lyst = lyst
        elif unique:
            self._lyst = list(range(1, size + 1))
            random.shuffle(self._lyst)
        else:
            self._lyst = []
            for count in range(size):
                self._lyst.append(random.randint(1, size))
        self._exchCount = 0
        self._cmpCount = 0
        self._startClock()
        function(self._lyst, self)
        self._stopClock()
        print(self)

    def exchange(self):
        """Counts exchanges if on."""
        if self._exch:
            self._exchCount += 1
        if self._trace:
            print(self._lyst)

    def comparison(self):
        """Counts comparisons if on."""
        if self._comp:
            self._cmpCount += 1

    def _startClock(self):
        """Record the starting time."""
        self._start = time.time()

    def _stopClock(self):
        """Stops the clock and computes the elapsed time
        in seconds, to the nearest millisecond."""
        self._elapsedTime = round(time.time() - self._start, 3)

    def __str__(self):
        """Returns the results as a string."""
        result = "Problem size: "
        result += str(len(self._lyst)) + "\n"
        result += "Elapsed time: "
        result += str(self._elapsedTime) + "\n"
        if self._comp:
            result += "Comparisons:  "
            result += str(self._cmpCount) + "\n"
        if self._exch:
            result += "Exchanges:    "
            result += str(self._exchCount) + "\n"
        return result
```

## 2. algorithms 模块

该模块定义了几种排序函数。排序函数需要接受一个 Profiler 类型的参数，并在算法中涉及比较和交换的地方，调用 `profiler.comparison()` 和 `profiler.exchange()` 。实际上任何处理列表的算法都可以利用类似的方法进行分析。

```python
"""
File: algorithms.py
Algorithms configured for profiling.

"""

from profiler import Profiler


def selectionSort(lyst: list, profiler: Profiler):
    i = 0
    while i < len(lyst) - 1:         # Do n - 1 searches
        minIndex = i                 # for the largest
        j = i + 1
        while j < len(lyst):         # Start a search
            profiler.comparison()
            if lyst[j] < lyst[minIndex]:
                minIndex = j
            j += 1
        if minIndex != i:            # Exchange if needed
            swap(lyst, minIndex, i, profiler)
        i += 1


def bubbleSort(lyst, profiler):
    n = len(lyst)
    while n > 1:                       # Do n - 1 bubbles
        i = 1                          # Start each bubble
        while i < n:
            profiler.comparison()
            if lyst[i] < lyst[i - 1]:  # Exchange if needed
                swap(lyst, i, i - 1, profiler)
            i += 1
        n -= 1


def bubbleSort2(lyst, profiler):
    n = len(lyst)
    while n > 1:
        swapped = False
        i = 1
        while i < n:
            if lyst[i] < lyst[i - 1]:  # Exchange if needed
                swap(lyst, i, i - 1, profiler)
                swapped = True
            i += 1
            profiler.comparison()
        if not swapped:
            return
        n -= 1


def insertionSort(lyst, profiler):
    i = 1
    while i < len(lyst):
        itemToInsert = lyst[i]
        j = i - 1
        while j >= 0:
            profiler.comparison()
            if itemToInsert < lyst[j]:
                lyst[j + 1] = lyst[j]
                profiler.exchange()
                j -= 1
            else:
                break
        if j != i-1:
            lyst[j + 1] = itemToInsert
            profiler.exchange()
        i += 1


def swap(lyst, i, j, profiler):
    """Exchanges the elements at positions i and j."""
    profiler.exchange()
    temp = lyst[i]
    lyst[i] = lyst[j]
    lyst[j] = temp

```

## 3. testprofiler 模块

该模块用于执行测试：

```python
from profiler import Profiler
from algorithms import selectionSort
import algorithms

p = Profiler()
p.test(selectionSort, size=15, comp=True,
       exch=True, trace=True)

p.test(algorithms.insertionSort, lyst=[1, 2, 3], trace=True)
```

输出展示：

```
[15, 1, 3, 5, 14, 9, 7, 11, 10, 13, 12, 4, 2, 6, 8]
[1, 15, 3, 5, 14, 9, 7, 11, 10, 13, 12, 4, 2, 6, 8]
[1, 2, 3, 5, 14, 9, 7, 11, 10, 13, 12, 4, 15, 6, 8]
[1, 2, 3, 4, 14, 9, 7, 11, 10, 13, 12, 5, 15, 6, 8]
[1, 2, 3, 4, 5, 9, 7, 11, 10, 13, 12, 14, 15, 6, 8]
[1, 2, 3, 4, 5, 6, 7, 11, 10, 13, 12, 14, 15, 9, 8]
[1, 2, 3, 4, 5, 6, 7, 8, 10, 13, 12, 14, 15, 9, 11]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 13, 12, 14, 15, 10, 11]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12, 14, 15, 13, 11]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 14, 15, 13, 12]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 15, 13, 14]
[1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 14]
Problem size: 15
Elapsed time: 0.001
Comparisons:  105
Exchanges:    12

Problem size: 3
Elapsed time: 0.0
Comparisons:  2
Exchanges:    0
```

