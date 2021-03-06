# 正则

### 正则的意义

#### 判断

#### 查找或定位

#### 替换

### 正则的分类

### 正则引擎

#### DFA vs. NFA

有穷自动机（Finite Automate）是用来模拟实物系统的数学模型，它包括如下五个部分：有穷状态集States输入字符集Input symbols转移函数Transitions起始状态Start state接受状态Accepting state(s)(终止状态)按照转移函数的不同，又可分为确定型有穷自动机（Determinism Finite Automate, DFA），与非确定型有穷自动机（Non-determinism Finite Automate, NFA）。 非确定有穷自动机容许转移函数不确定，换句话说，对任意状态，输入任意一个字符，可以转移到0个，1个或者多个状态。如果两台自动机能够接受的输入字符串（或者叫做“正则语言”Regular Language）完全相同，则这两台自动机是等价的。可以证明，对于每一个非确定有穷自动机，都存在与之等价的确定型有穷自动机。DFA引擎在任意时刻必定处于某个确定的状态，而NFA引擎可能处于一组状态之中的任何一个，所以，NFA引擎必须记录所有的可能路径（trace multiple possible routes through the NFA），NFA之所以能够提供Backtrack的功能，原因就在这里。NFA功能更多，可回溯。但可因回溯死循环。DFA占用空间更大，匹配更快

#### 不同语言(工具)使用的正则引擎

|    引擎类型   | 程序                                                                                                   |
| :-------: | ---------------------------------------------------------------------------------------------------- |
|    DFA    | awk (大多数版本)、egrep(大多数版本)、flex、lex、MySQL、Procmail                                                     |
|   传统型NFA  | GNU Emacs、Java、grep(大多数版本)、less、more、.NET语言、PCRE library、Perl、PHP(所有三套正则库)、Python、Ruby、sed(大多数版本)、vi |
| POSIX NFA | mawk、Mortice Kern Systems’ utilities、GNU Emacs (明确指定时使用)                                             |
| DFA/NFA混合 | GNU awk、GNU grep/egrep、Tcl                                                                           |

### 正则的三种模式

Greedy（贪婪的）Reluctant（不情愿的）Possessive（独占的）

1. ?: 告诉引擎匹配前导字符0次或一次。事实上是表示前导字符是可选的。
2. \+: 告诉引擎匹配前导字符1次或多次。
3. \*: 告诉引擎匹配前导字符0次或多次。
4. {min, max}: 告诉引擎匹配前导字符min次到max次。min和max都是非负整数。如果有逗号而max被省略了，则表示max没有限制；如果逗号和max都被省略了，则表示重复min次。

|   贪婪   |    懒惰   |    独占   |
| :----: | :-----: | :-----: |
|   X?   |   X??   |   X?+   |
|   X\*  |   X\*?  |   X\*+  |
|   X+   |   X+?   |   X++   |
|  X{n}  |  X{n}?  |  X{n}+  |
|  X{n,} |  X{n,}? |  X{n,}+ |
| X{n,m} | X{n,m}? | X{n,m}+ |

### 分组

#### 捕获组

#### 非捕获组

以 (?) 开头的组是纯的非捕获 组，它不捕获文本 ，也不针对组合计进行计数。就是说，如果小括号中以?号开头，那么这个分组就不会捕获文本，当然也不会有组的编号，因此也不存在Back 引用。它的作用就是匹配Pattern字符，好处就是不捕获文本，不将匹配到的字符存储到内存中，从而节省内存。

**零宽度断言**

环视，在不同的地方又称之为零宽断言，简称断言。环视（lookaround）其实分为两个部分：前瞻（lookahead）和后视（lookbehind）。本质上，lookahead和lookbehind匹配的是位置。与^和$一样的，因此它们都被称为锚。环视强调的是它所在的位置，前面或者后面，必须满足环视表达式中的匹配情况，才能匹配成功。环视可以认为是虚拟加入到它所在位置的附加判断条件，并不消耗正则的匹配字符（一）肯定和否定1、肯定：(?=exp) 和 (?<=exp)2、否定：(?!exp) 和 (?\<!exp)（二）顺序和逆序1、顺序：(?=exp) 和 (?!exp)2、逆序：(?<=exp) 和 (?\<!exp)· 两种类型名称组合1、肯定顺序：(?=exp)2、否定顺序：(?!exp)3、肯定逆序：(?<=exp)4、否定逆序：(?\<!exp)环视与正则的位置组合成8种环视1、肯定顺序常规： \[a-z]+(?=;) 字母序列后面跟着;2、肯定顺序变种： (?=\[a-z]+[![](file:///C:/Users/ADMINI\~1/AppData/Local/Temp/enhtmlclip/YX0dcd2f1e.png)](file:///C:/Users/ADMINI\~1/AppData/Local/Temp/enhtmlclip/\_\_SVG\_\_19153a08b38e00936e6751a07cc7b2d0) 字母序列3、肯定逆序常规： (?<=:)\[0-9]+ :后面的数字4、肯定逆序变种： \b\[0-9]\b(?<=\[13579]) 0-9中的奇数5、否定顺序常规： \[a-z]+\b(?!;) 不以;结尾的字母序列6、否定顺序变种： (?!.\*?\[lo0])\b\[a-z0-9]+\b 不包含l/o/0的字母数字系列7、否定逆序常规： (?\<!age)=(\[0-9]+) 参数名不为age的数据8、否定逆序变种： \b\[a-z]+(?\<!z)\b 不以z结尾的单词

**模式修正符**

正则表达式中常用的模式修正符有i、g、m、s、x、e等。它们之间可以组合搭配使用。(?imnsx-imnsx: ) 应用或禁用子表达式中指定的选项。例如，(?i-s: ) 将打开不区分大小写并禁用单行模式。关闭不区分大小写的开关可以使用(?-i)。有关更多信息，请参阅正则表达式选项。。

#### 反向引用

Back 引用 是说在后面的表达式中我们可以使用组的编号来引用前面的表达式所捕获到的文本序列。注意：反向引用，引用的是前面捕获组中的文本而不是正则，也就是说反向引用处匹配的文本应和前面捕获组中的文本相同，这一点很重要。

### 正则与编码格式

### 正则的性能

1、使用字符组代替分支条件2、优先选择最左端的匹配结果3、标准量词是匹配优先的4、谨慎用点号，尽可能不用\*号和+号5、尽量使用字符串函数处理代替6、合理使用括号7、起始、行描点优化8、拆分大的正则表达式

### 我的正则三板斧

#### 直接测试

#### 测试用例–反例测试–边界条件

#### 逻辑确认

### 案例

### QA

正则表达式与通配符的区别正则表达式与转移字符\* . ? + $ ^ \[ ] ( ) { } | \ /正则表达式优先级
