# 二元运算

#### 二元运算符 <a href="#suan-shu-er-yuan-yun-suan-fu" id="suan-shu-er-yuan-yun-suan-fu"></a>

`scalar/scalar(标量/标量)`、`vector/scalar(向量/标量)`、和 `vector/vector(向量/向量)`

* `+` 加法
* `-` 减法
* `*` 乘法
* `/` 除法
* `%` 模
* `^` 幂等

#### 布尔运算符 <a href="#bu-er-yun-suan-fu" id="bu-er-yun-suan-fu"></a>

默认情况下布尔运算符只会根据时间序列中样本的值，对时间序列进行**过滤**。我们可以通过在运算符后面使用 `bool` 修饰符来改变布尔运算的默认行为。使用 bool 修改符后，布尔运算不会对时间序列进行过滤，而是直接依次瞬时向量中的各个样本数据与标量的比较结果 `0` 或者 `1`

* `==` (相等)
* `!=` (不相等)
* `>` (大于)
* `<` (小于)
* `>=` (大于等于)
* `<=` (小于等于)



#### 集合运算符 <a href="#ji-he-yun-suan-fu" id="ji-he-yun-suan-fu"></a>

使用瞬时向量表达式能够获取到一个包含多个时间序列的集合，我们称为瞬时向量。 通过集合运算，可以在两个瞬时向量与瞬时向量之间进行相应的集合操作。目前，Prometheus 支持以下集合运算符：

* `and` (并且)
* `or` (或者)
* `unless` (排除)

**vector1 and vector2** 会产生一个由 `vector1` 的元素组成的新的向量。该向量包含 vector1 中完全匹配 `vector2` 中的元素组成。

**vector1 or vector2** 会产生一个新的向量，该向量包含 `vector1` 中所有的样本数据，以及 `vector2` 中没有与 `vector1` 匹配到的样本数据。

**vector1 unless vector2** 会产生一个新的向量，新向量中的元素由 `vector1` 中没有与 `vector2` 匹配的元素组成。





