## Introduction

![img](https://oi-wiki.org/ds/images/stack.svg)

后进先出

## 数组模拟

```c++
// C++ Version
int st[N];
// 这里使用 st[0] (即 *st) 代表栈中元素数量，同时也是栈顶下标

// 压栈 ：
st[++*st] = var1;

// 取栈顶 ：
int u = st[*st];

// 弹栈 ：注意越界问题, *st == 0 时不能继续弹出
if (*st) --*st;

// 清空栈
*st = 0;
```

```java
// Java Version
        int[] st = new int[100];
// 这里使用 st[0] 代表栈中元素数量，同时也是栈顶下标

// 压栈 ：
        st[++st[0]] = var1;

// 取栈顶 ：
        int u = st[st[0]];

// 弹栈 ：注意越界问题, *st == 0 时不能继续弹出
        if (st[0] > 0){
            --st[0];
        }

// 清空栈
        st[0] = 0;
```

## c++ STL

```c++
	std::stack<int> st1,st2;
    st1.push(1);    //插入到栈顶
    st1.pop();         //弹出栈顶
    st1.empty();       //返回是否为空
    st1.size();        //返回元素数量
    st2 = st1;         //直接赋值，重载了 = 符号
```

