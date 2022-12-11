---
layout: post
emoji: 🔐
title: "Python GIL, Global interpreter Lock"
date: '2022-06-16 14:30:28'
author: Liam
categories: [ Python ]
image: assets/images/gil/GIL1.png
---

<br>
<br>

## 💡 Intro


오늘은 Python의 가장 큰 특징중 하나인 GIL(Global interpreter Lock)에 대해 알아보려고 합니다. Python을 사용하는 개발자라면 누구나 다 한번쯤은 들어봤을 것이고, 저 또한 GIL에 대해 공부해 본적이 있지만 다시 한번 정리해 보려고 합니다.


<br>
<br>


## 🔎 Python 인터프리터란?


우선 GIL이란 Global Interpreter Lock의 약자로 파이썬 인터프리터가 한 쓰레드만이 하나의 바이트코드를 실행 시킬 수 있도록 해주는 Lock입니다. 즉, Python 인터프리터는 한 번에 한 쓰레드만 실행될 수 있다는 것을 의미합니다. 

<br>

그렇다면 파이썬 인터프리터란 무엇일까요? Python 인터프리터란, Python으로 작성된 코드를 한 줄씩 읽으면서 실행하는 프로그램을 뜻 합니다. Python 인터프리터에 대한 자세한 내용은 [파이썬은 인터프리터언어입니까?](https://soooprmx.com/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%80-%EC%9D%B8%ED%84%B0%ED%94%84%EB%A6%AC%ED%84%B0%EC%96%B8%EC%96%B4%EC%9E%85%EB%8B%88%EA%B9%8C/) 블로그에서 조금 더 심도깊게 생각해보실 수 있습니다.


<br>
<br>


## 🔎 GIL(Global Interpreter Lock)


GIL (Global Interpreter Lock)을 🌎[Python 위키](https://wiki.python.org/moin/GlobalInterpreterLock)에서는 다음과 같이 정의하고 있습니다.


> In CPython, the **global interpreter lock**, or **GIL**, is a mutex that protects access to Python objects, preventing multiple threads from executing Python bytecodes at once. The GIL prevents race conditions and ensures thread safety. A nice explanation of 🌎[how the Python GIL helps in these areas can be found here](https://python.land/python-concurrency/the-python-gil). In short, this mutex is necessary mainly because CPython's memory management is not thread-safe.

<br>

해석을 해보면, Python의 객체들에 대한 접근을 보호하는 일종의 Mutex로서 여러 개의 쓰레드가 파이썬 코드를 동시에 실행하지 못하도록 하는 것을 의미합니다. 즉, 한 프로세스 내에서 Python 인터프리터는 한 시점에 하나의 쓰레드에 의해서만 실행될 수 있습니다. 멀티 쓰레딩이 불가능하다는 것이 아니나, 원래 멀티 코어라면 멀티 쓰레딩 시에 여러 개의 쓰레드가 여러 코어 상에서 병렬실행될 수 있는데, Python에서는 그러한 병렬 실행이 불가능하다는 것입니다.

<br>

간단하게 설명을 해보자면, 3개의 쓰레드를 통해 작업을 한다고 가정했을 때 하나의 쓰레드에 모든 자원을 허락하고 그 후에는 Lock을 걸어 다른 스레드는 실행할 수 없게 막아버림으로써, 각각의 쓰레드는 GIL을 얻고 동작하지만 이 때 다른 쓰레드는 모두 동작을 멈추게 됩니다. 이를 그림으로 나타내면 다음과 같습니다.

<br>

![GIL1.png]({{ site.baseurl }}/assets/images/gil/GIL1.png)

<br>
<br>


## 🔎 그럼 GIL이 왜 생긴걸까?


Python에서 모든 것은 객체(Object)입니다. 그리고 각 객체는 Reference Count를 저장하기 위한 필드를 가지고 있습니다. 
> Reference Count란 그 객체를 가리키는 Reference가 몇 개 존재하는지를 나타내는 것으로, Python에서의 GC(Garbage Collection)는 이러한 Reference Count가 0이 되면 해당 객체를 메모리에서 삭제시키는 메커니즘으로 동작하고 있습니다.

<br>

따라서 파이썬의 모든 객체는 Reference count, 즉 해당 변수가 참조된 수를 저장하고 있습니다. 여기서 문제가 발생하게 되는데, 멀티 쓰레드인 경우 여러 쓰레드가 하나의 객체를 사용한다면 Reference count를 관리하기 위해서 모든 객체에 대한 lock이 필요할 것입니다. 

<br>

이러한 비효율을 막기 위해서 Python에서 GIL을 사용하게 되었습니다. 그 당시 GIL을 선택해야 했던 이유는 🌎[What Is the Python Global Interpreter Lock (GIL)?](https://realpython.com/python-gil/) 에서 찾을 수 있었습니다.


> Python has been around since the days when operating systems did not have a concept of threads. Python was designed to be easy-to-use in order to make development quicker and more and more developers started using it. A lot of extensions were being written for the existing C libraries whose features were needed in Python. To prevent inconsistent changes, these C extensions required a thread-safe memory management which the GIL provided. The GIL is simple to implement and was easily added to Python. It provides a performance increase to single-threaded programs as only one lock needs to be managed. C libraries that were not thread-safe became easier to integrate. And these C extensions became one of the reasons why Python was readily adopted by different communities. As you can see, the GIL was a pragmatic solution to a difficult problem that the CPython developers faced early on in Python’s life.


<br>
<br>


## 📚 그렇다면 Python 멀티쓰레딩은 무조건 느릴까?


CPU 연산의 비중이 큰 작업을 할 때, Context Switching으로 인해 괜한 Cost만 잡아먹기 때문에 멀티 쓰레딩은 오히려 성능을 떨어뜨리게 됩니다. 하지만 I/O, Sleep 등의 외부 연산을 하느라 CPU가 아무것도 하지 않고 기다리기만 할 때는 다른 쓰레드로 Context Switching을 하게 됩니다. 

<br>

그러한 이유로 CPU 연산의 비중이 적은 I/O, Sleep 등의 외부 연산 같이 비중이 큰 작업을 할 때는 멀티 쓰레딩이 굉장히 좋은 성능을 보여주게 됩니다. 즉, **Python 멀티쓰레딩은 무조건 느리다라는 말은 맞는 말이 아닙니다.** 다음 예시를 통해 설명이 가능합니다.

<br>

```py

import random
import threading
import time


def working():
    time.sleep(1)
    max([random.random() for i in range(10000000)])
    time.sleep(1)
    max([random.random() for i in range(10000000)])
    time.sleep(1)


# Single Thread
s_time = time.time()
working()
working()
e_time = time.time()
print(f'{e_time - s_time:.5f}')


# Multi Thread
s_time = time.time()
threads = []
for i in range(2):
    threads.append(threading.Thread(target=working))
    threads[-1].start()

for t in threads:
    t.join()

e_time = time.time()
print(f'{e_time - s_time:.5f}')


```  

<br>

아래와 같이 확연한 차이를 볼 수 있습니다.

<br>

```py

Single Thread -> 10.77272
Multi Thread -> 7.17564


```

<br>

Single Thread는 `sleep`으로 인해 *아무 것도 못하고 동작을 대기*하게 되고, Multi Thread는 `sleep`으로 멈춘 상태에서 다른 스레드로 *Context Switching하여 Single Thread의 효율을 개선*하게 됩니다.


<br>
<br>


> 끝맺음
Python으로 작업하는 개발자의 입장으로써 지금까지는 GIL로 인한 불편한점은 느낄 수 없었습니다. 하지만 뜻하지 않은 계기로 오랜만에 다시 GIL(Global Interpreter Lock)을 공부하게 되었는데, GIL에 대한 역사와 Python의 GC에 대해 충분히 이해할 수 있었던것 만으로도 매우 의미있는 시간이었다고 생각합니다.😊


<br>
<br>


**[참고자료]**
- [파이썬은 인터프리터언어입니까?](https://soooprmx.com/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%80-%EC%9D%B8%ED%84%B0%ED%94%84%EB%A6%AC%ED%84%B0%EC%96%B8%EC%96%B4%EC%9E%85%EB%8B%88%EA%B9%8C/)
- [파이썬이 마침내 ‘GIL’을 제거할 수 있을까?](https://www.ciokorea.com/news/210764)
- [Python 위키](https://soooprmx.com/%ED%8C%8C%EC%9D%B4%EC%8D%AC%EC%9D%80-%EC%9D%B8%ED%84%B0%ED%94%84%EB%A6%AC%ED%84%B0%EC%96%B8%EC%96%B4%EC%9E%85%EB%8B%88%EA%B9%8C/)
- [왜 Python에는 GIL이 있는가](https://dgkim5360.tistory.com/entry/understanding-the-global-interpreter-lock-of-cpython)
- [Davie Beazley의 Understanding the Python GIL](http://www.dabeaz.com/python/UnderstandingGIL.pdf)
