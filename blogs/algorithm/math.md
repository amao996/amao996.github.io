## 扩展欧几里得

https://zhuanlan.zhihu.com/p/378728642

```c++
void exgcd(int a, int b, int& x, int& y){
    if(b == 0){
        x = 1, y = 0;
        return;	//此时的a为原始gcd(a,b)
    }
    exgcd(b, a % b, y, x);
    y -= a / b * x;
}
```

其中x,y为引用，传入待解参数；a,b为已知量，求解方程ax+by=gcd(a,b)的一组解(x,y)

## 类欧

```c++
using i128 = __int128;
i128 floor_sum(i128 n, i128 m, i128 a, i128 b) {
    i128 ans = 0;
    if (a < 0) {
        i128 a2 = (a % m + m) % m;
        ans -= n * (n - 1) / 2 * ((a2 - a) / m);
        a = a2;
    }
    if (b < 0) {
        i128 b2 = (b % m + m) % m;
        ans -= n * ((b2 - b) / m);
        b = b2;
    }
    while (1) {
        if (a >= m) {
            ans += n * (n - 1) / 2 * (a / m);
            a %= m;
        }
        if (b >= m) {
            ans += n * (b / m);
            b %= m;
        }
        i128 y_max = a * n + b;
        if (y_max < m) {
            break;
        }
        n = y_max / m;
        b = y_max % m;
        swap(m, a);
    }
    return ans;
}
```

函数返回：



$$
\sum_{i=0}^{n-1}\lfloor \frac{a\times i+b}{m} \rfloor
$$

<br>

