## 位操作

#### 1.大小写转换

```c++
('A' | ' ') = 'a';	//大写转换成小写
('a' & ' ') = 'A';	//小写转换成大写
//通过^(异或)大写与小写互相转换
('A' ^ ' ') = 'a';	
('a' ^ ' ') = 'A';	
```

#### <br>2.不用临时变量交换两个数

```c++
int a = 1, b = 2;
a ^= b;
b ^= a;
a ^= b;
```

#### <br>3.简单运算

```c++
int n = 2;
n = -~n;	//等效于n += 1
n = ~-n;	//等效于n -= 1
```

#### <br>4.判断异号

```c++
int x = -1, y = 2;
bool f = ((x ^ y) < 0);	//true
x = 1;
f = ((x ^ y) < 0);	//false
```

#### <br>5. n & (n - 1)的运用

目的是消除数字n的 二进制表示中最后一个1，其核心逻辑就是n-1一定可以消除最后一个1，同时把其后的0都变成1，这样再和n做一次&运算，就可以仅把最后一个1变成0了。

#### <br>6.a ^ a = 0