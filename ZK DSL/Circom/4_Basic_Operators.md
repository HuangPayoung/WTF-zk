# 基本运算符

Circom 提供了布尔运算符、算术运算符和按位运算符。它们具有标准语义，但应用于数值的算术运算符在 p 模数下工作。

运算符的优先级和关联性类似于 Rust（在[这里](https://doc.rust-lang.org/1.22.1/reference/expressions/operator-expr.html#operator-precedence)定义）。

表达式可以使用以下运算符构建，但条件运算符 `?_:_` 只能出现在顶层。

## 域元素

域元素是 Z/pZ 范围内的值，其中 p 是默认设置的素数：

`p = 21888242871839275222246405745257275088548364400416034343698204186575808495617.`

因此，域元素在算术上是模 p 进行操作的。

Circom 语言对这个数字是参数化的，可以在不影响语言其他部分的情况下更改它（使用 `GLOBAL_FIELD_P`）。

## 条件表达式

**Boolean\_condition ? true\_value : false\_value**

```plaintext
var z = x > y ? x : y;
```

这种条件表达式不允许嵌套形式，因此只能在顶层使用。

## 布尔运算符

允许使用以下布尔运算符：

| 运算符 | 示例     | 解释                |
| :----- | :------- | :------------------ |
| &&     | a && b   | 布尔运算符 AND      |
| \|\|   | a \|\| b | 布尔运算符 OR       |
| !      | ! a      | 布尔运算符 NEGATION |

## 关系运算符

关系运算符 **`< , > , <= , >= , == , !=`** 的定义取决于数学函数 `val(x)`，该函数定义如下：

```plaintext
val(z) = z-p  if p/2 +1 <= z < p

val(z) = z,    otherwise.
```

根据这个函数，关系运算符的定义如下：

```plaintext
x < y 定义为 val(x % p) < val(y % p)

x > y 定义为 val(x % p) > val(y % p)

x <= y 定义为 val(x % p) <= val(y % p)

x >= y 定义为 val(x % p) >= val(y % p)
```

其中 `<, >, <=, >=` 是整数比较。

## 算术运算符

所有算术运算在模 p 下工作。我们有以下运算符：

| 运算符 |   示例   |      解释      |
| :----: | :------: | :------------: |
|   +    |  a + b   |  算术加法模 p  |
|   -    |  a - b   |  算术减法模 p  |
|   \*   |  a \* b  |  算术乘法模 p  |
|  \*\*  | a \*\* b |   幂运算模 p   |
|   /    |  a / b   | 乘以逆元素模 p |
|   \    |  a \ b   |  整数除法的商  |
|   %    |  a % b   | 整数除法的余数 |

有些运算符将算术运算与最终赋值相结合。

| 运算符 |   示例   |            解释             |
| :----: | :------: | :-------------------------: |
|   +=   |  a += b  |     算术加法模 p 并赋值     |
|   -=   |  a -= b  |     算术减法模 p 并赋值     |
|  \*=   | a \*= b  |     算术乘法模 p 并赋值     |
| \*\*=  | a \*\* b |      幂运算模 p 并赋值      |
|   /=   |  a /= b  |    乘以逆元素模 p 并赋值    |
|   \=   |  a \= b  |     整数除法的商并赋值      |
|   %=   |  a %= b  |    整数除法的余数并赋值     |
|   ++   |   a++    | 单位增量。语法糖用于 a += 1 |
|   --   |   a--    | 单位减量。语法糖用于 a -= 1 |

## 按位运算符

所有按位运算都是模 p 进行的。

| 运算符   | 示例         | 解释            |
| :------- | :----------- | :-------------- |
| &        | a & b        | 按位 AND        |
| \|       | a \| b       | 按位 OR         |
| ~        | ~a           | 补码 254 位     |
| ^        | a ^ b        | 按位异或 254 位 |
| &gt;&gt; | a &gt;&gt; 4 | 右移运算符      |
| &lt;&lt; | a &lt;&lt; 4 | 左移运算符      |

移位运算也在模 p 下工作，定义如下（假设 p >= 7）。

对于所有 `0=< k <= p/2`（整数除法）我们有：

* `x >> k = x/(2**k)` 
* `x << k = (x*(2**k) & mask) % p` 

其中 b 是 p 的有效位数，mask 是 `2**b - 1`。

对于所有 `p/2 +1 <= k < p` 我们有：

* `x >> k = x << (p-k)` 
* `x << k = x >> (p-k)` 

注意，k 也是负数 `k-p`。

有些运算符将按位运算与最终赋值相结合。

| 运算符    | 示例          | 解释                  |
| :-------- | :------------ | :-------------------- |
| &=        | a &= b        | 按位 AND 并赋值       |
| \|=       | a \|= b       | 按位 OR 并赋值        |
| ~=        | ~=a           | 补码 254 位并赋值     |
| ^=        | a ^= b        | 按位异或 254 位并赋值 |
| &gt;&gt;= | a &gt;&gt;= 4 | 右移运算符并赋值      |
| &lt;&lt;= | a &lt;&lt;= 4 | 左移运算符并赋值      |

## 使用 Circom 库中运算符的示例

以下是一些使用上述运算符组合的示例。

```plaintext
pragma circom 2.0.0;

template IsZero() {
    signal input in;
    signal output out;
    signal inv;
    inv <-- in!=0 ? 1/in : 0;
    out <== -in*inv +1;
    in*out === 0;
}

component main {public [in]}= IsZero();
```

此模板检查输入信号 `in` 是否为 `0`。如果是，输出信号 `out` 的值为 `1`。否则为 `0`。请注意，我们使用中间信号 `inv` 来计算 `in` 的逆元素或不存在时的 `0`。如果 `in` 为 0，那么 `in*inv` 为 0，`out` 的值为 `1`。否则，`in*inv` 始终为 `1`，则 `out` 为 `0`。

```plaintext
pragma circom 2.0.0;

template Num2Bits(n) {
    signal input in;
    signal output out[n];
    var lc1=0;
    var e2=1;
    for (var i = 0; i<n; i++) {
        out[i] <-- (in >> i) & 1;
        out[i] * (out[i] -1 ) === 0;
        lc1 += out[i] * e2;
        e2 = e2+e2;
    }
    lc1 === in;
}

component main {public [in]}= Num2Bits(3);
```

此模板返回一个 n 维数组，包含 `in` 的二进制值。第 7 行使用右移 `>>` 和 `&` 运算符在每次迭代中获取数组的第 `i` 个组件。最后，第 12 行添加约束 `lc1 = in` 以保证转换正确。