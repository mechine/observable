# 匹配模式

向量与向量之间进行运算操作时会基于默认的匹配规则：依次找到与左边向量元素匹配（标签完全一致）的右边向量元素进行运算，如果没找到匹配元素，则直接丢弃。

### 一对一匹配 <a href="#yi-dui-yi-pi-pei" id="yi-dui-yi-pi-pei"></a>

使用 `ignoreing` 可以在匹配时忽略某些便签。而 `on` 则用于将匹配行为限定在某些便签之内。

```bash
vector1 <operator> vector2
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

### 多对一和一对多 <a href="#duo-dui-yi-he-yi-dui-duo" id="duo-dui-yi-he-yi-dui-duo"></a>

多对一和一对多两种匹配模式指的是“一”侧的每一个向量元素可以与"多"侧的多个元素匹配的情况。在这种情况下，必须使用 group 修饰符：`group_left` 或者 `group_right` 来确定哪一个向量具有更高的基数（充当“多”的角色）。

多对一和一对多两种模式一定是出现在操作符两侧表达式返回的向量标签不一致的情况。因此需要使用 ignoring 和 on 修饰符来排除或者限定匹配的标签列表。

```bash
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

`group` 修饰符只能在比较和数学运算符中使用。在逻辑运算 `and`，`unless` 和 `or` 操作中默认与右向量中的所有元素进行匹配。

### 聚合操作 <a href="#ju-he-cao-zuo" id="ju-he-cao-zuo"></a>

```bash
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
<aggr-op> [without|by (<label list>)]([parameter,] <vector expression>)
```

