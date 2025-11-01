---
title: Review Python fundmentals after 3 years
category: Development
date: 2025-09-11 11:47:24
tags:
---

Hi, after many years I still find Python an attractive programming language even after I have worked as a Java engineer for pass three years. 
It has plenty of modern features compared to Java language, and most importantly, many startups use Python or Go for their systems. So I've decided that it's time for me to learn Python again.

### Data Structure
#### int&string
- Immutable data type.
```python
i = 0
s = "123"
```
#### list
- Mutable data type. Use index to search element 
```python
party = [1,2,3]
```
#### tuple
- Immutable data type. often used in *args
- unchange list
```python
party = (1,2,3)
```
#### set
- distinct element.
```python
set([1,2,3,4,5,1,2,3,4,5])
```
#### dict
- map structured data type.
```python
member_score = {"John":98,"Helen":78}
```

### using Function 
```python
def standard_arg(arg):
    print(arg)

def pos_only_arg(arg, /): # only allow position
    print(arg)
pos_only_arg(1)

def kwd_only_arg(*, arg): # only allow argument
    print(arg)
kwd_only_arg(arg=1)

def combined_example(pos_only, /, standard, *, kwd_only):
    print(pos_only, standard, kwd_only)

def error_foo(name, **kwds): # kwds will collect argument into dict. may conflict with name&kwds 
    return 'name' in kwds
def foo(name, /, **kwds): # best practice. Avoid conflict between name&&kwds # kwds will collect argument into dict
    return 'name' in kwds
foo(1, **{'name': 2})

# import argument as tuple
def boo(*args): 
    print(args)
boo(1, 2, 3)

def f(ham: str, eggs: str = 'eggs') -> str: # Comment in Function
    print("Annotations:", f.__annotations__)
    print("Arguments:", ham, eggs)
    return ham + ' and ' + eggs
```

### Python lambda
```python
def make_incre(n):
    return lambda x: x + n
f = make_incre(1)
g = make_incre(2)
h = make_incre(3)

# using Function Composition
result = f(g(h(0)))  # 0 + 3 + 2 + 1

comb = [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
combs = []
for x in [1,2,3]:
    for y in [3,1,4]:
        if x != y:
            combs.append((x, y))

```

### Deque
```python
from collections import deque
queue = deque(["Eric", "John", "Michael"])
queue.append("Terry")           # (["Eric", "John", "Michael","Terry"]
queue.append("Graham")          # (["Eric", "John", "Michael","Terry","Graham"]
queue.popleft() # (["John", "Michael","Terry","Graham"]
queue.pop() # (["John", "Michael","Terry"]
```

### Matrix transpose in Python 
```python
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
]
# 1.
print([[row[i] for row in matrix] for i in range(4)])
# 2.
print(list(zip(*matrix))) 
```

### Techique of for loop
```python
knights = {'gallahad': 'the pure', 'robin': 'the brave'}
for k, v in knights.items(): # key & value
    print(k, v)
    
for i, v in enumerate(['tic', 'tac', 'toe']): # value with index
    print(i, v)
    
questions = ['name', 'quest', 'favorite color']
answers = ['lancelot', 'the holy grail', 'blue']
for q, a in zip(questions, answers): # composition
    print('What is your {0}?  It is {1}.'.format(q, a))
```

### Other concept
- Use `json.dumps(x)` to serialize the object into json string
- Use `json.loads(x)` to deserialize the jsonstring into specific object
- When python interpreter boots, namespace will be created an never be deleted
- `namespace` is a space that python can store the variable, so even variable which has same name may stored in many space based on their scope.
- python is no `private`, convently we use `__argument` as a private attribute and avoid accidently override by subclass or something else.
`_age` use int protect attribute.
- To increase the loading speed of the python module, python store their compile version into `＿_pycache__` and named `module.version.pyc`

### Use inerator
```python
s = 'abc'
it = iter(s)
print(next(it)) #a
print(next(it)) #b
print(next(it)) #c
```

### Use generator
```python
def reverse(data):
    for index in range(len(data)-1, -1, -1):
        yield data[index] # return part of element, to aviod resource over-occupied
# usage1
g = reverse('abc')
next(g)

# usage2
for char in reverse('golf'):
    print(char)
```

### Unit Test
```python
def average(numbers):
    if not isinstance(numbers, list):
        raise TypeError("Input must be a list")
    return sum(numbers) / len(numbers)

import unittest

class TestStatisticalFunctions(unittest.TestCase):

    def test_average(self):
        self.assertEqual(average([20, 30, 70]), 40.0)
        self.assertEqual(round(average([1, 5, 7]), 1), 4.3)
        with self.assertRaises(ZeroDivisionError):
            average([])
        with self.assertRaises(TypeError):
            average(20, 30, 70)
```

### Multi-thread
```python
import threading
import queue
import time

# Build up a shared task queue
task_queue = queue.Queue()

# Define worker thread of what they're doing
def worker(name):
    while True:
        task = task_queue.get() # Thread will be blocked in task_queue.get()，until queue exists value。
        if task is None:
            print(f"{name} 收到結束訊號，停止工作")
            break
        print(f"{name} 正在處理任務：{task}")
        time.sleep(1)  # simulate the execution
        task_queue.task_done()

# Create the worker thread
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(f"工作者-{i}",))
    t.start() # Let it execute worker function. These thread will waiting for the task in the background
    threads.append(t)

# Put task into main thread
for task_num in range(10):
    task_queue.put(f"任務-{task_num}")

# wait until all task complete 
task_queue.join()

# Notify end message to the worker
for _ in threads:
    task_queue.put(None)

# Wait all worker complete
for t in threads:
    t.join()

print("All Task Complete!!!")
```

### Python GIL

//TODO