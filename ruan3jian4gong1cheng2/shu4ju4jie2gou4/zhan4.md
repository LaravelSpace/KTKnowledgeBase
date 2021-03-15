## 栈(stack)

线性表结构

操作受限，只允许在一端插入和删除

后进先出(Last In First Out)，称为LIFO结构(想象一下弹夹)

顺序栈(数组)，链栈(链表)

### 递归

栈有一个很重要的应用：在应用程序中实现递归。我们把一个直接调用自己或通过一系列的调用语句间接调用自己的函数，称做递归函数。

递归函数必须至少有一个条件，满足时递归不再进行，即不在引用自身而是返回值退出。写递归程序，永远要注意，不要让程序陷入无穷的递归中。

关于递归的调用，不应该把递归的入口直接暴露给外部，这样风险很大。建议将递归入口封装，做好递归前的各项校验工作，保证递归可以顺利的执行结束。

### 后缀（逆波兰）表达式

后缀表达式，也称为逆波兰（Reverse Polish Notation，RPN）表达式。所有的符号都是要在运算的数字后面出现。我们通常写的标准的四则运算表达式也叫中缀表达式。

- 中缀表达式：`(1-2)*(4+5)`，后缀表达式：`12-45+*`。
- 中缀表达式：`a+(b-c)*d` ，后缀表达式：`abc-d*+`。

简单地说，中缀表达式是给人看的，后缀表达式是给计算机看的。中缀表达式转后缀表达式的时候，数字是直接输出的，操作符和括号需要入栈并判断顺序。文章末尾提供了一个比较简单的后缀表达式的样例，只考虑了个位数的带括号的四则运算，下面是几点主要思路。

- 括号的优先级是最高的，需要用到一点贪心算法的策略。一旦遇到括号就要一直出栈操作符直到遇见另一半括号。
- 乘号和除号优先级比加号和减号要高，乘号和除号在入栈时如果前面是加号和减号，则直接入栈，如果是乘号和除号则需要先把前面的操作符出栈，自己在入栈。
- 加号和减号入栈时要输出前面所有的符号。

使用后缀表达式计算结果的时候就比较简单了，碰到数字就入栈，碰到操作符就出栈两个数字，计算之后的结果再入栈。最后栈里的那个数字就是表达式计算的结果。

### 八皇后问题



---

| 代码说明   | 代码位置                                        |
| ---------- | ----------------------------------------------- |
| 栈-链式    | CYangLi/shu4ju4jie2gou4/zhan4_lian4biao3.c      |
| 后缀表达式 | CYangLi/shu4ju4jie2gou4/hou4zhui4biao3da2shi4.c |