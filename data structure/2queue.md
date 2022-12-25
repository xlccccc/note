## Introduction

![img](https://oi-wiki.org/ds/images/queue.svg)

先进先出

## 数组模拟

```c++
int q[SIZE], ql, qr;

q[++qr] = x;		//push
ql++;				//pop
q[ql];				//访问队首
q[qr];				//访问队尾
```

## c++ STL

```c++
std::queue<int> q1, q2;

// 向 q1 的队尾插入 1
q1.push(1);

// 将 q1 赋值给 q2
q2 = q1;

// 输出 q2 的队首元素
std::cout << q2.front() << std::endl;
// 输出: 1
```

