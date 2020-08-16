---
title: leetcode224 Basic Calculator
top: false
cover: false
toc: true
mathjax: true
date: 2020-07-24 00:45:21
Password:
summary:
tags: leetcode
categories: 数据结构与算法
typora-copy-images-to: 
---

# [Leetcode 224](https://leetcode.com/problems/basic-calculator/ ).基本计算器

![leetcode224](/images/leetcode224.png)

很明显，这是一个中缀表达式求值的问题，但该题也有其独特性，也诞生出了一些新的解决方法，本人大致尝试了三种方法：

## 常规思路

一开始面对此题，很容易想到的是通常解决此类栈及表达式问题的思路。

前提是只有加减运算，即省去了运算符号优先级的比较。定义两个栈，num_stack用于存储数据，op_stack用于存储操作符。

### 算法：

1. 从左往右扫描字符串，遇到操作数入数据栈;
2. 若遇到操作符 + 或 - 时：

   ​	若操作符栈为空，直接压入该操作符栈；

   ​	若栈不为空，且栈顶不为左括号，则从数据栈弹出两个元素，从操作符栈弹出一个操作符进行计算，将结果并压入数据栈，然后压入该操作符栈；

   ​	若栈顶为左括号，压入该操作符栈；

3. 若是左括号，直接压入操作符栈，继续扫描；
4. 若遇到的是右括号，则从数据栈弹出两个元素，从操作弹出一个操作符进行计算，并将结果加入到数据栈中，直到栈顶为左括号，弹出左括号后，继续扫描；
5. 扫描完成时，若操作符栈为空，则数据栈顶即为计算结果；若操作符不为空，进行一步计算后，结束。

```c++
 class Solution {
public:
    void compute (std::stack<int> &number_stack, std::stack<char> &operator_stack) {
        int num2 = number_stack.top();
        number_stack.pop();
        int num1 = number_stack.top();
        number_stack.pop();
        
        int result = 0;
        if (operator_stack.top() == '+') {
            result = num1 + num2;
        } else {
            result = num1 - num2;
        }
        number_stack.push(result);
        operator_stack.pop();
    }
    
    int calculate(string s) {
        std::stack<int> number_stack;
        std::stack<char> operator_stack;
        int num = 0;
        
        for (int i = 0; i < s.length(); ++i) {
            switch (s[i]) {
                case '+':
                case '-':
                    if (operator_stack.empty() || operator_stack.top() == '(') {
                    } else {
                        compute(number_stack, operator_stack);
                    }
                    operator_stack.push(s[i]);
                    break;
                case '(':
                    operator_stack.push(s[i]);
                    break;
                case ')':
                    while (operator_stack.top() != '(') {
                        compute(number_stack, operator_stack);
                    }
                    operator_stack.pop();
                    break;
                case ' ':
                    break;
                default:
                    if (isdigit(s[i])) {
                        num = s[i] - '0';
                        while (i + 1 < s.length() && isdigit(s[i+1])) {
                            ++i;
                            num = s[i] - '0' + num * 10;
                        }
                        number_stack.push(num);
                        num = 0;
                    }
                    break;
            }
        }
        if (!operator_stack.empty()) {
            compute(number_stack, operator_stack);
        }
        
        return number_stack.top();
    }
};
```

## 变通的解法

[逆波兰表达式](https://zh.wikipedia.org/wiki/%E9%80%86%E6%B3%A2%E5%85%B0%E8%A1%A8%E7%A4%BA%E6%B3%95)，即后序表达式，其求值的问题也在[leetcdoe 150](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/)讨论过；而中序表达式转换为后序表达式可以通过[Shunting yard算法](https://zh.wikipedia.org/wiki/Shunting_yard算法)实现，所以该基本计算器的求解就分为两个相对比较容易的子问题了，其算法过程如下：

```c++
class Solution {
public:
    int calculate(string s) {
        vector<string> vector = shunting_yard_algorithm(s);
        return evalRPN(vector);
    }
    
  	// 后序表达式求值 
    int evalRPN(vector<string>& tokens) {
        stack<int> num_stack;
        for (size_t i = 0; i < tokens.size(); ++i) {
            string &curr = tokens[i];
            if (curr == "+" || curr == "-") {
                int num2 = num_stack.top();
                num_stack.pop();
                int num1 = num_stack.top();
                num_stack.pop();
                int sum;
                if (curr == "-") {
                    sum = num1 - num2;
                } else {
                    sum = num1 + num2;
                }
                num_stack.push(sum);
            } else {
                int num = stoi(curr);
                num_stack.push(num);
            }
        }
        
        return num_stack.top();
    }
    
  	// 中序表达式转为后序表达式
    vector<string> shunting_yard_algorithm(string s) {
        vector<string> str_vec;
        stack<char> operator_stack;
        
        for (int i = 0; i < s.length(); ++i) {
            switch (s[i]) {
                case '+':
                case '-':
                    if (!operator_stack.empty() && (operator_stack.top() == '+' || operator_stack.top() == '-')) 
                    {
                        string op;
                        op += operator_stack.top();
                        operator_stack.pop();
                        
                        str_vec.push_back(op);
                    }
                    operator_stack.push(s[i]);
                    break;
                case '(':
                    operator_stack.push(s[i]);
                    break;
                case ')':
                    while (operator_stack.top() != '(') {
                        string op;
                        op += operator_stack.top();
                        operator_stack.pop();
                        
                        str_vec.push_back(op);
                    }
                    operator_stack.pop();
                    break;
                case ' ':
                    continue;
                    break;
                default:
                    if (isdigit(s[i])) {
                        string num_str;
                        num_str += s[i];
                        while (i + 1 < s.length() && isdigit(s[i + 1])) {
                            ++i;
                            num_str += s[i];
                        }
                        str_vec.push_back(num_str);
                    }
                    break;
            }
        }
        while (!operator_stack.empty()) {
            string op;
            op += operator_stack.top();
            operator_stack.pop();
            
            str_vec.push_back(op);
        }
        return str_vec;
    }
};
```



## 巧妙的解法

leetcode中国官方关于基本计算器也提出了比较巧妙的[解题思路](https://leetcode-cn.com/problems/basic-calculator/solution/ji-ben-ji-suan-qi-by-leetcode/)，选择了其第二种进行说明。

### 算法：

1. 从左到右扫描字符串；

2. 遇到 + 或 - 运算符时，首先将表达式求值到左边，然后将正负符号保存到下一次求值。

3. 如果字符是左括号 (，将迄今为止计算的结果和符号添加到栈上，然后重新开始进行计算，就像计算一个新的表达式一样。

4. 如果字符是右括号 )，则首先计算左侧的表达式。则产生的结果就是刚刚结束的子表达式的结果。如果栈顶部有符号，则将此结果与符号相乘。

   ```c++
   class Solution {
   public:
       int calculate(string s) {
           int num = 0;
           int result = 0;
           int sign = 1;
           stack<int> calculate_stack;
           
           for (auto &c : s) {
               switch (c) {
                   case '+':
                       result += sign * num;
                       num = 0;
                       sign = 1;
                       break;
                   case '-':
                       result += sign * num;
                       num = 0;
                       sign = -1;
                       break;
                   case '(':
                       calculate_stack.push(result);
                       calculate_stack.push(sign);
                       
                       result = 0;
                       sign = 1;
                       break;
                   case ')':
                       result += sign * num; // ')'括号左边的值
                       
                       result *= calculate_stack.top(); // '('括号前的符号
                       calculate_stack.pop();
                       
                       result += calculate_stack.top(); // 之前的计算结果
                       calculate_stack.pop();
                       
                       num = 0;
                       
                       break;
                   case ' ':
                       break;
                   default:
                       if (isdigit(c)) {
                           num = c - '0' + num * 10;
                       }
                       break;
               }
           }
           
           return result + (sign * num); // last number
       }
   };
   ```




