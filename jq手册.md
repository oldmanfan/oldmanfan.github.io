# 目录  

[TOC]

---

# jq手册
jq程序就是一个"过滤器": 它接受一个输入, 产生一个输出. 有许多内建的过滤器用来抽取对象中特定的字段, 或者是将一个数字转化为字符串, 或者其它一些标准任务.  

过滤器可以用不同的方式结合起来 - 可以用pipe将一个过滤器的输出导入到另一个过滤器, 或者是将过滤器的输出收集到一个数组里面.  

一些过滤器能够产生多个结果, 比如说对输入的数组中元素依次产生输出. 然后再将输出pipe到第二个过滤器. 在其它编程语言, 这样的任务一般是通过循环和迭代器来完成的, 但是在jq里面把两个过滤器连接起来就可以了.  
需要注意的是, 每一个过滤器都会有输入和输出. 甚至是字面量像"hello"和42也是过滤器--它们的输入和输入是同样的字面量而己, 像加法这样结合两个过滤器的操作, 一般都是将输入同时输入给这两个过滤器, 然后再组合成结果. 你可以实现一个求平均数的过滤器, 它使用`add / length`这两个过滤器 -- 将输入数组分别输入到`add`过滤器和`length`过滤器, 然后再将它们产生的结果做除法.  

这些内容有点太超前了, 让我们从最简单的地方开始吧!!  

## 调用jq
jq过滤器基于JSON格式的数据流运行的. 输入数据将按照以空格为分隔符的JSON值解析, 然后依次传递给过滤器. 过滤器的输出也会被组合成以空格为分隔符的JSON数据, 输出到标准输出上面.  

注意: 请特别留意Shell的引号规则. 作为一个通用规则, 在unix类型的机器上, 最好将jq的参数用单引号('')而不是双引号("")包括起来, 因为很多jq的关键字同时也是Shell的元字符. 举个例子, `jq "foo"`在大多数类unix的机器上执行都会失败, 因为这个命令会被解释为`jq foo`, 而错误原因在于`foo is not defined`. 如果你在Windows的cmd.exe中使用的话, 那么最好使用双引号(也可以使用`-f program-file`选项替代), 但是这时候jq里面的双引号就要通过`\`转义了.  

可以通过命令行选项, 来改变jq读写输入和输出的行为:  

* `--version`  
输出jq的版本号并 exit 0  
* `--seq`
使用`application/json-seq`的MIME类型规范分隔jq的输入和输入JSON文本. 这意味着, 每一个输出的前面都会有一个ASCII RS(记录分隔符), 后面都会有一个ASCII LF(回车符). 输入的JSON文本如果没有遵循这个规范的话, 解析错误会被忽略(但是产生Warning), 同时会丢弃直到下一个RS之前的所有数据. jq默认使用这个模式来产生**输出**, 哪怕并没有使用`--seq`选项.  
* `--stream`  
用数据流的方式解析输入, 同时用数组的方式输出路径和叶结点的值(值和空数组或空对象). 比如, `"a"`变成`[[], "a"]`, `[[], "a", ["b"]]`变成`[[0], []], [[1], "a"]`和`[[1, 0], "b"]`.  
这个模式对处理超大输入很有用. 通过和过滤器联合, 同时通过`reduce`和`foreach`表达式来减少大输入的快速增长.  
* `--slurp/-s`  
 不要对每一个输入的JSON对象单独运行过滤器, 而是将所有输入数据流读到一个大数组中, 然后运行一次过滤器.  


* `--raw-input/-R`  
不要以JSON的方式解析输入, 而是将每一行输入文本以string的方式传递给过滤器. 如果同时使用了`--slurp`, 那所有的输入都会当做一个长string传递给过滤器.  


* `--null-input/-n`  
不读取任何输入! 过滤器使用`null`作为输入运行一次. 在将jq作为简单计算器或是通过scratch来构建JSON数据时, 这非常有用.  


* `--compact-output/-c`  
默认条件下, jq会将输出JSON格式化易读格式. 使用这个选项, 输出JSON将会以紧凑模式输出到一行上.  


* `--tab`  
用tab代替2个空格作为缩进.  


* `--indent n`  
使用指定数量的空格(不超过8)作为缩进.  


* `--color-output/-C` 和 `monochrome-output/-M`  
默认条件下, 如果输出到终端上, jq输出的JSON会有色彩. 你可以使用`-C`选项强制jq在输出到pipe或一个文里面时也生成颜色, 也可以使用`-M`禁止颜色信息.


* `--ascii-output / -a`  
jq通过会把非ASCII的Unicode编码作为UTF-8输出, 哪怕在输入中明确的使用转义序列指明了是unicode编码(比如"\u03bc"). 使用这个选项, 你可以强制jq将每一个非ASCII字符用对应的转义序列代替, 产生纯ASCII输出.  


* `--unbuffered`  
在输出JSON打印之后, 立即清除缓存.(如果你正在将一个慢速的数据源pipe到js, 然后将输出再pipe到其它地方时, 这个选项很有用)  


* `--sort-keys`  
将输出对象中的字段按keys排序输出.  


* `--raw-output / -R`  
使用这个选项, 如果过滤器的结果是一个string, 那它会被直接输出到标准输出, 不会被用引号的方式格式化成JSON格式的string. 这样jq就可以和其它非基于JSON的系统交互.  


* `--join-output / -j`  
类似`-r`选项, 但是jq不会给每一行输出后面添加换行符.  


* `-f filename` / `--from-file filename`  
从文件而不是命令行中读过滤器设置, 类似awk的-f选项. 你也可以在文件中使用`#`注释代码.  


* `-Ldirectory` / `-L directory`  
预设`directory`为模块的搜索路径. 如果使用了这个选项, 那么jq将不会使用内置的搜索路径. 详细信息参考下面关于模块的章节.  


* `-e / --exit-status`  
设置jq的退出码. 如果最后一个输出即不是`false`也不是`null`, 退出码是0; 如果最后一个输出是`false`或`null`, 退出码是1; 如果没有产生合法的输出, 退出码是4. 没有使用这个选项的情况下, 如果有任何使用方面的问题或是系统错误, 退出码是2; 如果jq遇到了编译错误, 退出码是3; 如果jq运行正常, 退出码是0.  
另一个设置退出码的方法是使用内置的`halt_error`方法. 


* `--arg name value`  
这个选项用来把一个变量的值传递给jq程序. 比如运行`jq --arg foo bar`, 那么在程序中就可以使用`$foo`变量, 它的值是`"bar"`. 需要注意的是`value`总是会被解析为string格式, 所有`--arg foo 123`会将`$foo`的值设置为`"123"`.  
具名参数也可以通过jq程序中的`$ARGS.named`来获取.  


* `--argjson name JSON-text`  
这个选项用来传递一个JSON编码的预设置变量给jq程序.  


* `--slurpfile variable-name filename`  
这个选项将指定文件中所有的JSON数据读到一个数组中, 将它和给定的全局变量关联. 比如你使用选项`--slurpfile foo bar`运行jq, 那么在jq程序中访问`$foo`, 并且它是一个数组, 数组中的元素来自于`bar`指定的文件.  


* `--rawfile variable-name filename`  
类似于`--slurpfile`选项, 只是它读取的数据不是解析为数组, 而是一个string.  


* `--argfile variable-name filename`  
Deprecated. 用`--slurpfile`代替.  


* `--args`  
其它的string参数. 在jq程序中使用`$ARGS.positional[]`访问.  


* `--jsonargs`  
其它的JSON格式的参数. 在jq程序中使用`$ARGS.positional[]`访问.  


* `--run-tests [filename]`
运行指定文件或标准输入里的测试. 这个必须是最后一个选项.  
需要注意的是, 这个选项可能会改变并且不后向兼容.

## 基础过滤器
###  标志符/Identity: `.`  
`.`是最简单的过滤器, 它将输入内容不加任何改变的传递给输出, 这大概也是它为什么被叫做标志符吧.   
因为在默认情况下, jq会用JSON格式来打印输出, 所以如果你想把其它程序比如`curl`的结果用JSON格式打印出来, 那么这个特性就非常有用了.   
```bash
	jq '.'
Input	"Hello, world!"
Output	"Hello, world!"
```

###  对象字段过滤器: `.foo`, `.foo.bar`  
最简单但是用处最大的过滤器就是`.foo`这种格式. 如果输入一个JSON对象(有时也叫字典, hash), 这个过滤器将会输出关键字`foo`的值, 如果对象中不存在关键字`foo`, 那么会输出`null`, 而`.foo.bar`格式的过滤器等价于`.foo|.bar`.  

这种过滤器要求关键字是简单识别符, 即关键字必须是数字, 字母和下划线组成, 并且不能以数字开头.  如果关键字中确实包含特殊字符, 那么要用双引号引起来, 像`."foo$"`或者`.["foo$"]`这样.  

比如说, `.["foo::bar"]`和`.["foo.bar"]`能够正常工作, 但是`.foo::bar`是不能正常工作的. 同时`.foo.bar`和`.["foo"].["bar"]`也是等价的.  

```bash
	jq '.foo'
Input	{"foo": 42, "bar": "less interesting data"}
Output	42
```

```bash
	jq '.foo'
Input	{"notfoo": true, "alsonotfoo": false}
Output	null
```

```bash
	jq '.["foo"]'
Input	{"foo": 42}
Output	42
```

###  可选的对象字段过滤器: `.foo?`  
和`.foo`功能一样, 区别在于即使`.`不是一个数组或对象, 这个提取器也不会扔出错误.  


###  通用对象下标过滤器: `.[<string>]`  
可以使用`.["foo"]`格式的提取器来查询一个对象中相关的字段值. (`.foo`是这种表达式的简写方式, 并且只支持简单识别符)  


###  数组下标过滤器: `.[2]`  
如果下标值是整数, `.[<value>]`指示数组下标. 数组下标起始于0, 所以`.[2]`会返回第三个元素.  
下标允许使用负数, `-1`指向最后一个元素, `-2`指向倒数第二个元素, 依次类推.  
```bash
	jq '.[0]'
Input	[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output	{"name":"JSON", "good":true}
```

```bash
jq '.[2]'
Input	[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output	null
```

```bash
jq '.[-2]'
Input	[1,2,3]
Output	2
```

###  数组/String分片: `.[10:15]`  
分片过滤器用来过滤数组的子数组/String的subString. `.[10:15]`过滤结果的长度是5, 包括第10个, 但是不包括第15个元素(半闭半开区间). 分片过滤器的下标可以是负数(这样元素将倒着数), 也可以忽略其中一个或所有(意味着起始/结束的所有元素).  
```bash
	jq '.[2:4]'
Input	["a","b","c","d","e"]
Output	["c", "d"]
```

```bash
	jq '.[2:4]'
Input	"abcdefghi"
Output	"cd"
```

```bash
jq '.[:3]'
Input	["a","b","c","d","e"]
Output	["a", "b", "c"]
```

```bash
jq '.[-2:]'
Input	["a","b","c","d","e"]
Output	["d", "e"]
```


###  数组/对象迭代器: `.[]`  
如果使用`.[index1:index2]`这种表达式但是忽略掉所有的index, 那么将返回数组中的所有元素. 如果输入是`[1, 2, 3]`, 过滤器`.[]`将产生三个独立的结果, 而不是单一的一个数组.  

这个过滤器也可以用于对象, 它将返回对象的**values**.

```bash
    jq '.[]'
Input	[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output	{"name":"JSON", "good":true}
{"name":"XML", "good":false}
```

```bash
	jq '.[]'
Input	[]
Output	none
```

```bash
	jq '.[]'
Input	{"a": 1, "b": 1}
Output	1
        1
```

###  `.[]?`
和`.[]`一样, 区别在于输入如果不是数组或对象, 也不会有错误抛出.  


###  逗号分隔符: `,`  
如果两个过滤器用`,`分隔, 那么同一个输入将会分别传递给这两个过滤器, 并将产生的结果依次序连接起来: 首先是左边过滤器产生的所有输出, 然后是右边过滤器产生的输出.  

```bash
	jq '.foo, .bar'
Input	{"foo": 42, "bar": "something else", "baz": true}
Output	42
"something else"
```

```bash
jq '.user, .projects[]'
Input	{"user":"stedolan", "projects": ["jq", "wikiflow"]}
Output	"stedolan"
"jq"
"wikiflow"
```

```bash
	jq '.[4,2]'
Input	["a","b","c","d","e"]
Output	"e"
        "c"
```


###  管道(Pipe): `|`  
`|`操作符用来串联多个过滤器, 它将左边过滤器产生的输出, 作为输入传递给右边的过滤器. 它和Unix shell里面的管道是一样的.  

如果左边的过滤器产生了多个结果, 那么右边的过滤器将会迭代这些结果. 所以`.[] | .foo`会迭代输入数组中的每一个元素, 过滤出`foo`的值.  

* 注意: `.a.b.c`和`.a | .b | .c`是一样的.  

* 注意2: `.`代表"管道"中每一个特定阶段的输入, 具体来说: 特定阶段指的是`.`出现的位置. 所以, `.a | . | .b`和`.a.b`是一样的, 因为中间的`.`代表的是`.a`产生的值.  

```bash
	jq '.[] | .name'
Input	[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]
Output	"JSON"
"XML"
```


###  括号: `()`  
像大多数编程语言一样, 括号是用来分组的.  

```bash
	jq '(. + 2) * 5'
Input	1
Output	15
```

## 类型和值
jq支持和JSON一样的数据类型 -- number, string, boolean, array, object和"null".  

Boolean, null, string和number的写法和javascript中一样. 和jq中的其它过滤器一样, 这些简单对象的值也可以接受输入, 并产生一个输出 -- `42`是一个合法的jq表达式 -- 它授受任意的输入, 丢弃它, 然后返回42.  

###  数组构造: `[]` 
和JSON中一样, `[]`用来构造一个数组, 比如`[1, 2, 3]`. 数组中的元素可以是任意合法的jq表达式, 包括pipeline.  所有这些表达式产生的结果都会被收集到起来, 产生一个大数组. 你可以使用它来产生一个数量有限的数组, 比如`[.foo, .bar, .baz]`, 也可以用于"收集"过滤器产生的所有输出, 比如`[.items[].name]`.   

如果你理解了`,`操作符, 那么你可以从另一个角度来理解数组表达式: `[1, 2, 3]`并不是一个内置的数组构造器, 而是`[]`过滤器依次操作1, 2, 3产生的输出.  

如果你的过滤器`X`产生了4个结果, 那么表达式`[X]`也会产生一个4个元素的数组.

```bash
	jq '[.user, .projects[]]'
Input	{"user":"stedolan", "projects": ["jq", "wikiflow"]}
Output	["stedolan", "jq", "wikiflow"]
```

```bash
	jq '[ .[] | . * 2]'
Input	[1, 2, 3]
Output	[2, 4, 6]
```

###  对象构造:  `{}`  
和JSON一样, `{}`用来构造对象(aka字典或hash), 如`{"a": 42, "b": 17}`, 如果keys是简单识别符的话, 那么keys的双引号可以省略. 但是通过表达式产生的Key需要用双此号引起来, `{("a" + "b"): 59}`.  

Value可以是任意表达式(当然如果值是包含复杂字符的话, 那么要使用双引号), {}表达式也适应(记住, 所有的过滤器都有输入和输出).

对于过滤器: `{foo: .bar}` :

如果输入是`{"bar":42, "baz":43}`会产生输出一个JSON对象`{"foo": 42}`. 你可以通过这样的表达式从特定的对象中过滤你想要的字段: 如果输入对象包含user, title, id和content字段, 但是你只在间user和title, 那你可以这样写:   

`{user: .user, title: .title}` 
因为这种需求实在太常见了, 所以可以简写成: `{user, title}`.  

如果过滤器的表达式会产生多个结果, 那么整个表达式也会产生多个输出. 如果输入是`{"user":"stedolan","titles":["JQ Primer", "More JQ"]}`, 过滤器是`{user, title: .titles[]}`的话, 那么会产生两个输出:  

```text
{"user":"stedolan", "title": "JQ Primer"}
{"user":"stedolan", "title": "More JQ"}
```

如果用括号`()`将key括起来, 意味着key需要作为表达式求值. 在同样的输入下, 表达式`{(.user): .titles}`将产生`{"stedolan": ["JQ Primer", "More JQ"]}`的输出.  

```bash
jq '{user, title: .titles[]}'
Input	{"user":"stedolan","titles":["JQ Primer", "More JQ"]}
Output	{"user":"stedolan", "title": "JQ Primer"}
{"user":"stedolan", "title": "More JQ"}
```

```bash
	jq '{(.user): .titles}'
Input	{"user":"stedolan","titles":["JQ Primer", "More JQ"]}
Output	{"stedolan": ["JQ Primer", "More JQ"]}
```


###  递归下降: `..`  
// TODO  


## 内置操作符和函数
jq的一些操作符(比如`+`)对不同类型的参数会做不同的操作(array, number等等). 虽然如此, 但是jq不会做任意的隐式类型转换. 如果你想把一个string和一个object相加, 那么将产生一条错误信息, 并且不返回任何值.   


### 加法: `+`  
`+`操作符接受两个过滤器, 把同一个输入分别传递到两个过滤器中, 然后把结果相加. 这里的"相加"操作, 和类型有关:   
1. Number: 普通的算术运算.  
2. Array: 将两个数组拼接成一个大数组.  
3. String: 将两个string拼接成一个大string.  
4. Object: 取对象key-value的并集. 将两个对象中都有的key-value提取出来, 组合成一个新的对象. 如果两个对象都有的同一个key, 但是value不同, 那么+号右边对象的value将胜出.  

`null`可以和任意对象相加, 不加改变的返回另一个对象.  

```bash
jq '.a + 1'
Input	{"a": 7}
Output	8
```

```bash
	jq '.a + .b'
Input	{"a": [1,2], "b": [3,4]}
Output	[1,2,3,4]
```

```bash
	jq '.a + null'
Input	{"a": 1}
Output	1
```

```bash
jq '.a + 1'
Input	{}
Output	1
```

```bash
jq '{a: 1} + {b: 2} + {c: 3} + {a: 42}'
Input	null
Output	{"a": 42, "b": 2, "c": 3}
```


###  减法: `-`  
对于数字, `-`操作符和通常的数学上减法一样. 同时, `-`也可以用在数组上, 作用是从第一个数组中减去第二个数组中出现的所有元素.  
```bash
jq '4 - .a'
Input	{"a":3}
Output	1
```

```bash
jq '. - ["xml", "yaml"]'
Input	["xml", "yaml", "json"]
Output	["json"]
```
###  乘法, 除法, 求余: `*`, `/`,`%`  
这几个中缀操作符对数字的操作和其它编程语言一样, 除以0会抛出一个错误, `x % y`计算x除以y的余数.  

string乘以一个数字n表示重复string n次. `"x" * 0`的结果是`null`.  

string除以一个string, 表示用第二个string作为分隔符, 拆分第一个string.  

object乘以object会把这两个对象递归合并: 和加法操作类似, 区别在于如果这两个对象包含同一个key, 并且这个key的value也都是object, 那这么这两个value的object也会按照同样的策略进行递归合并.  

```bash
    jq '10 / . * 3'
Input	5
Output	6
```

```bash
	jq '. / ", "'
Input	"a, b,c,d, e"
Output	["a","b,c,d","e"]
```

```bash
    jq '{"k": {"a": 1, "b": 2}} * {"k": {"a": 0,"c": 3}}'
Input	null
Output	{"k": {"a": 0, "b": 2, "c": 3}}
```

```bash
    jq '.[] | (1 / .)?'
Input	[1,0,-1]
Output	1
       -1
```

###  `length`方法  
内建方法`length`返回不同数据类型的长度值.  
1. string: 返回unicode编码的长度. (如果string是纯粹ASCII字符串, 那么长度和JSON格式编码的长度一样)  
2. array: 长度是数组中元素的数量.   
3. object: 长度是对象中key-value对的数量.  
4. null: 长度为0.  

```bash
jq '.[] | length'
Input	[[1,2], "string", {"a":2}, null]
Output	2
6
1
0
```


###  `utf8bytelength`方法  
这个内建方法返回用utf-8编码的string的字节数.   

```bash
jq 'utf8bytelength'
Input	"\u03bc"
Output	2
```


###  `keys`, `keys_unsorted`方法  
如果输入是一个object, `keys`方法将会以数组的方式, 返回对象的所有keys. 返回的keys将以"字母表"顺序排序, 但这并不是指对所有的语言都会用同样的方式排序, 而是说对于具有相同keys的两个对象, `keys`方法都会返回同样的排列顺序, 不论本地语言设置是什么.  

如果输入是一个array, `keys`方法将返回下标数组: 从0到length - 1.  

`keys_unsorted` 方法的功能和`keys`方法一样, 只是不会对结果排序, 而是按照对象中key的出现顺序.  

```bash
    jq 'keys'
Input	{"abc": 1, "abcd": 2, "Foo": 3}
Output	["Foo", "abc", "abcd"]
```

```bash
    jq 'keys'
Input	[42,3,35]
Output	[0,1,2]
```


###  `has(key)`方法  
`has`方法返回输入对象中是否有指定的key, 或者是输入数组中是否在指定下标有元素.  

用`has($key)`方法检查的效果, 与先通过`keys`方法获取到所有的keys数组, 然后再检查`$key`是否在数组中, 这两种方法的效果是一样的, 但是`has`的效率更高.  

```bash
	jq 'map(has("foo"))'
Input	[{"foo": 42}, {}]
Output	[true, false]
```

```bash
	jq 'map(has(2))'
Input	[[0,1], ["a","b","c"]]
Output	[false, true]
```


###  `in`方法  
`in`方法返回指定对象中是否存在输入的key, 或者是指定数组中是否存在输入的元素下标. 事实上, `in`是`has`的反向操作.  

```bash
	jq '.[] | in({"foo": 42})'
Input	["foo", "bar"]
Output	true
false
```

```bash
jq 'map(in([0,1]))'
Input	[2, 0]
Output	[false, true]
```

###  `map(x)`, `map_value(x)`  
对于任意一个过滤器`x`, `map(x)`会把输入数组中的每一个元素, 依次执行一遍`x`, 然后将每次执行的输出, 组合成一个新的数组. `map( . + 1)`操作将由数字组成的输入数组中每一个元素加1.  

类似的, 如果输入是一个object, `map_value(x)`将对object中的 每一个value执行`x`操作, 然后把结果作为value的新值, 然后返回这个object.   

`map(x)` 等价于`[.[] | x]`, 事实上这也就是map(x)的定义, `map_value(x)`的定义是`.[] |= x`.  

```bash
    jq 'map(.+1)'
Input	[1,2,3]
Output	[2,3,4]
```

```bash
    jq 'map_values(.+1)'
Input	{"a": 1, "b": 2, "c": 3}
Output	{"a": 2, "b": 3, "c": 4}
```

###  `path(path_expression)`方法
对当前值`.`, 执行path_expression表达式, 返回一个数组. 返回数组中的string元素, 表示object的key, number元素表示数组的下标.  

路径表达式是和`.a`, `.[]`一样的jq表达式. 路径表达式有两种: 精确匹配, 非精确匹配. 例如, `.a.b.c`是精确匹配, `.a[].b`是非精确匹配.  

`path(exact_path_expression)` 都会产生一个数组形式的输出, 无论当前操作对象`.`是否存在对应的路径, 包括`.`是`null`, 或者数组, 或者object.  

`path(pattern)` 会产生一个能够匹配`pattern`路径的数组, 只是`.`中存在对应的路径.  

需要注意的是, 路径表达式和普通的表达式并没有什么不一样. `path(..|select(type=="boolean")`会输出`.`里面所有boolean类型的值的路径, 而且也只会包括boolean类型.  

```bash
    jq 'path(.a[0].b)'
Input	null
Output	["a",0,"b"]
```

```bash
    jq '[path(..)]'
Input	{"a":[{"b":1}]}
Output	[[],["a"],["a",0],["a",0,"b"]]
```

###  `del(path_expression)`方法
`del`方法会删除object里对应路径上的key, 或者是array中对应下标的元素.
```bash
    jq 'del(.foo)'
Input	{"foo": 42, "bar": 9001, "baz": 42}
Output	{"bar": 9001, "baz": 42}
```

```bash
    jq 'del(.[1, 2])'
Input	["foo", "bar", "baz"]
Output	["foo"]
```

###  `getpath(PATHS)`方法
`getpath`方法在`.`中查找PATHS数组中每一个路径, 返回object中对应路径下的value, 组合成array输出.  

```bash
    jq 'getpath(["a","b"])'
Input	null
Output	null
```

```bash
    jq '[getpath(["a","b"], ["a","c"])]'
Input	{"a":{"b":0, "c":1}}
Output	[0, 1]
```

###  `setpath(PATHS; VALUE)`方法
`setpath`方法将`.`中PATHS路径下的value设置为VALUE.(如果路径存在, 则修改; 如果路径不存在, 则添加)  

```bash
    jq 'setpath(["a","b"]; 1)'
Input	null
Output	{"a": {"b": 1}}
```

```bash
    jq 'setpath(["a","b"]; 1)'
Input	{"a":{"b":0}}
Output	{"a": {"b": 1}}
```

```bash
    jq 'setpath([0,"a"]; 1)'
Input	null
Output	[{"a":1}]
```

###  `delpaths(PATHs)`方法
`delpaths`方法删除`.`中的PATHS. PATHS需要是路径表达式数组.  

```bash
    jq 'delpaths([["a","b"]])'
Input	{"a":{"b":1},"x":{"y":2}}
Output	{"a":{},"x":{"y":2}}
```
###  `to_entries`, `from_entries`, `with_entries`方法
这三个方法用于在object和由key-value对组成的array之间互转. 如果输入一个object到`to_entries`方法, 那么对于每一个键-值对`k: v`, 输出数组会由以下格式的对象`{"key": k, "value": v}`组成.  

`from_entries` 是to_entries的逆操作.  

`with_entries` 是`to_entries | map(foo) | from_entries`的简写, 对于想对object的keys或是values都执行同样的操作时很有用. 

`from_entries` 接受key, Key, name, Name, value, Value作为关键字.  

```bash
    jq 'to_entries'
Input	{"a": 1, "b": 2}
Output	[{"key":"a", "value":1}, {"key":"b", "value":2}]
```

```bash
    jq 'from_entries'
Input	[{"key":"a", "value":1}, {"key":"b", "value":2}]
Output	{"a": 1, "b": 2}
```

```bash
    jq 'with_entries(.key |= "KEY_" + .)'
Input	{"a": 1, "b": 2}
Output	{"KEY_a": 1, "KEY_b": 2}
```

###  `select(boolean_expression)`方法
`select(foo)`方法对输入执行`foo`表达式, 如果返回值是true, 那么将不加改变的把输入当作输出; 如果返回值是flase, 那么不会产生任何输出.  

这个方法用做过滤特定条件的数据非常有用: `[1, 2, 3] | map(select(. >= 2))`将会输出`[2, 3]`.  

```bash
	jq 'map(select(. >= 2))'
Input	[1,5,3,0,7]
Output	[5,3,7]
```

```bash
	jq '.[] | select(.id == "second")'
Input	[{"id": "first", "val": 1}, {"id": "second", "val": 2}]
Output	{"id": "second", "val": 2}
```

###  `arrays`,`objects`,`iterables`,`boolean`,`numbers`,`normals`,`finites`, `strings`, `nulls`, `values`, `scalars` 方法
这些内置的选择器, 用于从输入中选择对应的arrays, objects, iterables(arrays和objects), booleans, numbers, normal numbers, finite numbers, strings , null, non-null, non-iterables.  

```bash
    jq '.[]|numbers'
Input	[[],{},1,"foo",null,true,false]
Output	1
```

###  `empty`方法
`empty`方法不返回任何结果, 什么都不产生, 包括`null`.  

在极特殊的场合下, 这个还是很有用的, 当你需要的时候就你知道了. :)  

```bash
	jq '1, empty, 2'
Input	null
Output	1
        2
```

```bash
	jq '[1,2,empty,3]'
Input	null
Output	[1,2,3]
```

###  `error(message)`方法
产生一个错误信息, 类似于将`.a`用于values而不是null和objects时一样, 只是错误信息是由message指定了. 错误可以通过try/catch捕获. 见下述.  

###  `halt`方法
停止jq程序, 并且不产生其它的输出. jq将返回退出状态码`0`.  

###  `halt_error`, `halt_error(exit_code)`方法
停止jq程序, 不再产生其它输出. 输入值将以raw方式作为输出打印到`stderr`(string不会被引号引起来, 不会有其它装饰符号, 包括换行符).  

指定的exit_code(默认是5)将作为jq的退出状态码.  

比如, `"Error: somthing went wrong\n"|halt_error(1)`  

###  `$__loc__`方法
这个方法将产生一个具有`file`和`line`两个key的object, `file`的值是文件名, `line`的值是`$__loc__`出现的位置.  

```bash
	jq 'try error("\($__loc__)") catch .'
Input	null
Output	"{\"file\":\"<top-level>\",\"line\":1}"
```

###  `paths`, `paths(node_filter)`, `leaf_paths`方法
`paths`所印出所有输入元素的路径.  

`paths(node_filter)` 打印出所有`node_filter`是true的元素路径. 例如`paths(numbers)`输出所有number元素的路径.  

`leaf_paths` 已经Deprecated, 后续版本将移除.  

```bash
	jq '[paths]'
Input	[1,[[],{"a":2}]]
Output	[[0],[1],[1,0],[1,1],[1,1,"a"]]
```

```bash
	jq '[paths(scalars)]'
Input	[1,[[],{"a":2}]]
Output	[[0],[1,1,"a"]]
```

###  `add`
过滤器`add`需要一个array输入, 然后将array中所有的元素相加, 再将结果输出. "相加"意味着可能是求和, 连接, 合并等操作, 这取决于输入数据的类型 -- `add`的规则和`+`操作符的规则是一致的.  

如果输入是一个空数组, `add`将返回`null`.  

```bash
	jq 'add'
Input	["a","b","c"]
Output	"abc"
```

```bash
	jq 'add'
Input	[1, 2, 3]
Output	6
```

```bash
    jq 'add'
Input	[]
Output	null
```

###  `any`, `any(condition)`, `any(generator; condition)`
过滤器`any`接受一个boolean类型的输入数组, 只要输入数组中有一个元素是true, 那么它将输出true.  

如果输入数组是空的, 那么它输出false.  

过滤器`any(condition)` 将对输入数组中元素依次执行condition操作, 再按`any`的规则产生输出.  

过滤器`any(generator; condition)`首先对输入元素依次执行generator, 再把generator的输出依次执行condition, 最后对condition的输出执行`any`的规则, 产生最终输出.  

```bash
	jq 'any'
Input	[true, false]
Output	true
```

```bash
	jq 'any'
Input	[false, false]
Output	false
```

```bash
	jq 'any'
Input	[]
Output	false
```

###  `all`, `all(condition)`, `all(generator; condition)`
过滤器`all`接受一个boolean类型的输入数组, 只有当输入数组中每一个元素都是true, 那么它将输出true.  

如果输入数组是空的, 那么它输出true.  

过滤器`all(condition)` 将对输入数组中元素依次执行condition操作, 再按`all`的规则产生输出.  

过滤器`all(generator; condition)`首先对输入元素依次执行generator, 再把generator的输出依次执行condition, 最后对condition的输出执行`all`的规则, 产生最终输出.

```bash
	jq 'all'
Input	[true, false]
Output	false
```

```bash
	jq 'all'
Input	[true, true]
Output	true
```

```bash
	jq 'all'
Input	[]
Output	true
```

###  `flatten`, `flatten(depth)`
过滤器`flatten`接受一个嵌套的数组作为输入, 然后递归访问输入数组中每一个元素, 将所有元素的值产生一个新的数组, 作为输出.  

你也可以设置一个参数来指明平滑多少级的嵌套.  

`flatten(2)` 和`flatten`是一样的, 都是平滑到2级嵌套.  

```bash
    jq 'flatten'
Input	[1, [2], [[3]]]
Output	[1, 2, 3]
```

```bash
	jq 'flatten(1)'
Input	[1, [2], [[3]]]
Output	[1, 2, [3]]
```

```bash
    jq 'flatten'
Input	[{"foo": "bar"}, [{"foo": "baz"}]]
Output	[{"foo": "bar"}, {"foo": "baz"}]
```

###  `range(upto)`, `range(from; upto)`, `range(from; upto; by)`方法
`range`方法用于产生一个数值范围. `range(4; 10)`产生6个数值范围[4, 10). 产生的数值将每一个都会作为独立的输出. 使用`[range(4; 10)]`可以产生一个数组.  

`range(upto)`产生输出[0, upto), 步长为1. 

`range(from; upto)`产生输出[from, upto), 步长为1. 

`range(from; upto; by)`产生输出[from, upto), 步长为by.  

```bash
	jq 'range(2;4)'
Input	null
Output	2
        3
```

```bash
	jq '[range(2;4)]'
Input	null
Output	[2,3]
```

```bash
    jq '[range(4)]'
Input	null
Output	[0,1,2,3]
```

```bash
    jq '[range(0;10;3)]'
Input	null
Output	[0,3,6,9]
```

```bash
	jq '[range(0;10;-1)]'
Input	null
Output	[]
```

```bash
	jq '[range(0;-5;-1)]'
Input	null
Output	[0,-1,-2,-3,-4]
```

###  `floor`方法
对输入的数值取floor.  

```bash
    jq 'floor'
Input	3.14159
Output	3
```

###  `sqrt`方法
对输入的数值取平方根.  

```bash
    jq 'sqrt'
Input	9
Output	3
```

###  `tonumber`方法
将输入转数字. 如果输入是string并且是由纯数字组成, 那么会生成对应的数值; 如果输入是数字, 那么仍然保持是数字. 其它任何输入, 都会抛出错误.  

```bash
	jq '.[] | tonumber'
Input	[1, "1"]
Output	1
        1
```
###  `tostring`方法
将输入作为string输出. 如果输入本身就是string, 那么不作改变. 如果是其它类型的数据, 将使用JSON对应类型的格式编码.  

```bash
    jq '.[] | tostring'
Input	[1, "1", [1]]
Output	"1"
        "1"
        "[1]"
```
###  `type`方法
`type`方法以string的方式返回参数的类型, 可能是null, boolean, number, string, array, object其中之一.  

```bash
    jq 'map(type)'
Input	[0, false, [], {}, null, "hello"]
Output	["number", "boolean", "array", "object", "null", "string"]
```

###  `infinite`, `nan`, `isinfinite`, `isnan`, `isfinite`, `isnormal`方法
有一些数学运算会产生无穷大, 无穷小, 或是NaN.  

`isinfinite`判断输入是否无穷大/小的值. 

`isnan`判断输入是否是NaN.  

`isnormal`判断输入是否是正常数值.

`infinite`产生一个正无穷大值.  

`nan`产生一个NaN.  

```bash
	jq '.[] | (infinite * .) < 0'
Input	[-1, 1]
Output	true
        false
```

```bash
	jq 'infinite, nan | type'
Input	null
Output	"number"
        "number"
```

###  `sort, sort_by(path_expression)`方法
对输入数组进行排序. 排序规则按以下顺序进行:
* null
* false
* true
* numbers
* strings(按unicode编码的字母顺序)
* arrays(按字面量顺序)
* objects
对object进行排序的规则有点复杂, 首先将所有的keys进行排序, 如果keys相同, 再按照value排序.  

`sort`可以针对object的某个字段排序, 也可以通过一个jq过滤器排序.  

`sort_by(foo)`方法首先对输入数组中的每一个元素执行`foo`, 然后再根据`foo`的输出两两比较进行排序.  

```bash
	jq 'sort'
Input	[8,3,null,6]
Output	[null,3,6,8]
```

```bash
	jq 'sort_by(.foo)'
Input	[{"foo":4, "bar":10}, {"foo":3, "bar":100}, {"foo":2, "bar":1}]
Output	[{"foo":2, "bar":1}, {"foo":3, "bar":100}, {"foo":4, "bar":10}]
```

###  `group_by(path_expression)`方法
`group_by(.foo)`接受一个输入数组, 将具有相同`.foo`字段的元素按照`.foo`的值排序后归入一个array, 不含`.foo`字段的元素归入另一个array, 然后把这两个array组合成一个大的数组, 作为输出.  

任何jq表达式都可以取代`.foo`的位置, 用作过滤器. 输出结果的排序规则和`sort`方法一样.  

```bash
	jq 'group_by(.foo)'
Input	[{"foo":1, "bar":10}, {"foo":3, "bar":100}, {"foo":1, "bar":1}]
Output	[[{"foo":1, "bar":10}, {"foo":1, "bar":1}], [{"foo":3, "bar":100}]]
```

###  `min, max, min_by(path_exp), max_by(path_exp)`方法
找出输入数组中最小/最大的元素.  

`min_by(path_exp)`和`max_by(path_exp)`允许检查一个指定的字段或属性, 或过滤器来取得对应的value. 比如`min_by(.foo)`将找到`foo`的值最小的对象.  

```bash
    jq 'min'
Input	[5,4,2,7]
Output	2
```

```bash
    jq 'max_by(.foo)'
Input	[{"foo":1, "bar":14}, {"foo":2, "bar":3}]
Output	{"foo":2, "bar":3}
```

###  `unique, unique_by(path_exp)`方法
接受一个输入数组, 排序, 然后将数组中重复的元素只保留一个, 其它重复的删掉.  

`unique_by(path_exp)`将输入数组中元素依次执行path_exp, 对于相同的结果, 只保存一份.  

```bash
	jq 'unique'
Input	[1,2,5,3,5,3,1,3]
Output	[1,2,3,5]
```

```bash
    jq 'unique_by(.foo)'
Input	[{"foo": 1, "bar": 2}, {"foo": 1, "bar": 3}, {"foo": 4, "bar": 5}]
Output	[{"foo": 1, "bar": 2}, {"foo": 4, "bar": 5}]
```

```bash
    jq 'unique_by(length)'
Input	["chunky", "bacon", "kitten", "cicada", "asparagus"]
Output	["bacon", "chunky", "asparagus"]
```

###  `reverse`方法
将数组反转.  

```bash
	jq 'reverse'
Input	[1,2,3,4]
Output	[4,3,2,1]
```

###  `contains(element)`
对于过滤器`contains(b)`, 如果输入参数完全包含`b`, 将输出true.  

所谓的"完全包含", 对不同类型的数据定义如下:   

* string  如果string A是string B的子串, 那么B包含A.  
* array  如果array A中每一个元素都存在于array B中, 那么B包含A.  
* object 如果object A的每一个key-value对都存在于object B中, 那么B包含A. 
* 其它类型如果他们是相等的, 那么它们互相包含.  

```bash
	jq 'contains("bar")'
Input	"foobar"
Output	true
```

```bash
	jq 'contains(["baz", "bar"])'
Input	["foobar", "foobaz", "blarp"]
Output	true
```

```bash
	jq 'contains(["bazzzzz", "bar"])'
Input	["foobar", "foobaz", "blarp"]
Output	false
```

```bash
    jq 'contains({foo: 12, bar: [{barp: 12}]})'
Input	{"foo": 12, "bar":[1,2,{"barp":12, "blip":13}]}
Output	true
```

```bash
    jq 'contains({foo: 12, bar: [{barp: 15}]})'
Input	{"foo": 12, "bar":[1,2,{"barp":12, "blip":13}]}
Output	false
```

###  `indices(s)`方法
返回`.`中所有`s`出现位置的下标数组. 如果输入参数是数组, 同是s也是一个数组, 输出的下标数组是`.`中所有元素都能够匹配s的位置.  

```bash
	jq 'indices(", ")'
Input	"a,b, cd, efg, hijk"
Output	[3,7,12]
```

```bash
	jq 'indices(1)'
Input	[0,1,2,1,3,1,4]
Output	[1,3,5]
```

```bash
	jq 'indices([1,2])'
Input	[0,1,2,3,1,4,2,5,1,2,6,7]
Output	[1,8]
```

###  `index(s), rindex(s)`方法
`index(s)`返回输入参数中第一个出现s的位置下标.  

`rindex(s)`返回输入参数中最后一个出现s的位置下标.  

```bash
	jq 'index(", ")'
Input	"a,b, cd, efg, hijk"
Output	3
```

```bash
	jq 'rindex(", ")'
Input	"a,b, cd, efg, hijk"
Output	12
```

###  `inside`
对于过滤器`inside(b)`, 如果b完全包含输入参数, 那么返回true.  

它是contains的逆操作.  

```bash
	jq 'inside(["foobar", "foobaz", "blarp"])'
Input	["baz", "bar"]
Output	true
```

###  `startswith(str)`方法
如果`.`是str开头的string, 返回true.  

###  `endswith(str)`方法
如果`.`是str结尾的string, 返回true.  

###  `combinations, combinations(n)`方法
将输入数组数组的元素进行组合. 如果指定了n, 那么将输入数组重复n次之后作为输入, 再进行组合.   

```bash
	jq 'combinations'
Input	[[1,2], [3, 4]]
Output	[1, 3]
[1, 4]
[2, 3]
[2, 4]
```

```bash
	jq 'combinations(2)'
Input	[0, 1]
Output	[0, 0]
[0, 1]
[1, 0]
[1, 1]
```

###  `ltrimstr(str)`方法
如果`.`是以str开头的, 那么将`.`中的str去除, 剩下部分作为输出.  

如果`.`不是以str开头, 那么直接将`.`输出.  

```bash
	jq '[.[]|ltrimstr("foo")]'
Input	["fo", "foo", "barfoo", "foobar", "afoo"]
Output	["fo","","barfoo","bar","afoo"]
```

###  `rtrimstr(str)`方法
类似于`ltrimstr(str)`, 只是操作位置改为`.`的结尾.  

###  `explode`方法
把输入string转为string的编码组成的数组.  

```bash
	jq 'explode'
Input	"foobar"
Output	[102,111,111,98,97,114]
```

###  `implode`方法
`explode`的反向操作.  

###  `split(str)`方法
把`.`以str作为分隔符拆分.  

```bash
	jq 'split(", ")'
Input	"a, b,c,d, e, "
Output	["a","b,c,d","e",""]
```

###  `join(str)`方法
将输入数组中的元素以str连接起来. 它是`split`的逆向操作.  

对于输入数组中元素的类型:  
* numbers和booleans将先转为string.
* null作为空string对象.
* 不能是array和object.  

```bash
	jq 'join(", ")'
Input	["a","b,c,d","e"]
Output	"a, b,c,d, e"
```

```bash
	jq 'join(" ")'
Input	["a",1,2.3,true,null,false]
Output	"a 1 2.3 true  false"
```

###  `ascii_downcase, ascii_upcase`方法
对输入string进行一次copy, 将其中对应的大/小写字母转为对应的小/大写字母.  

###  `while(cond; update)`方法
`while(cond; update)`允许你对`.`重复执行update若干次, 直到cond变为false.  

注意: `while(cond; update)`在内容被定义为递归的jq函数. 如果update对每一个输入至多只产生一个输出的话, 那么在while循环中进行递归调用不会占用额外的内存. 参考高级话题部分.  

```bash
    jq '[while(.<100; .*2)]'
Input	1
Output	[1,2,4,8,16,32,64]
```
###  `until(cond; next)`方法
`until(cond; next)`可以执行next多次, 直到`cond`变成true.  

最初的`.`来源于输入, 然后每次执行next时, 产生的输出会被设置为`.`, 然后进行下一次循环.  

```bash
	jq '[.,1]|until(.[0] < 1; [.[0] - 1, .[1] * .[0]])|.[1]'
Input	4
Output	24
```

###  `recurse(f), recurese, recurse(f; condition), recurse_down`方法
遍历一个递归的数据结构.  
// TODO  

###  `walk(f)`方法
// TODO  

###  `$ENV, env`方法
`$ENV`是当jq程序启动时设置的环境变量对象.  

`env`是jq当前的环境变量. (启动jq时可能添加了其它环境变量)  

```bash
	jq '$ENV.PAGER'
Input	null
Output	"less"
```

###  `transpose`方法
// TODO  

###  `bsearch(x)`方法
`bsearch(x)`对输入数组中使用二分查找法查找x.  

如果输入数组是有序的并且包含x, 那么返回x在数组中的下标;  

如果输入数组是有序但是不包含x, 那么返回(-1-ix), 其中ix是x在输入数组中的插入位置, 以保持输入数组仍然有序.  

如果输入数组是无序的, bsearch(x)将返回一个无意义的整数.  

```bash
	jq 'bsearch(0)'
Input	[0,1]
Output	0
```

```bash
	jq 'bsearch(0)'
Input	[1,2,3]
Output	-1
```

```bash
    jq 'bsearch(4) as $ix | if $ix < 0 then .[-(1+$ix)] = 4 else . end'
Input	[1,2,3]
Output	[1,2,3,4]
```
###  字符串插值 - `\(foo)`
在string中, 如果使用`\()`的方式插入变量.  

```bash
	jq '"The input was \(.), which is one less than \(.+1)"'
Input	42
Output	"The input was 42, which is one less than 43"
```

### JSON格式互转: `tojson`, `fromjson`
`tojson`将变量转化为JSON文本.  

`fromjson`将JSON文本转化为变量.  

tojson和tostring方法的区别在于, tostring返回的string没有改变. tojson会将string用JSON编码.  

```bash
    jq '[.[]|tostring]'
Input	[1, "foo", ["foo"]]
Output	["1","foo","[\"foo\"]"]
```

```bash
    jq '[.[]|tojson]'
Input	[1, "foo", ["foo"]]
Output	["1","\"foo\"","[\"foo\"]"]
```

```bash
	jq '[.[]|tojson|fromjson]'
Input	[1, "foo", ["foo"]]
Output	[1,"foo",["foo"]]
```

### string格式化和转义
将输入的string以不同类型的内容解析.  

// TODO  

### 日期
// TODO   

### SQL类操作
// TODO 

###  `builtins`方法
以`name/arity`的格式返回所有内建的方法. 如果方法有相同的名字但是参数, 那么`all/0`, `all/1`, `all/2`都会出现在列表中.  

## 条件分支和比较操作

### `==, !=`
如果a和b是相等的, 那么`a == b`返回true, 否则返回false.  string永远不会和number相等, 如果你了解javascript的话, jq的`==`和javascript里的`===`一样, 只有在类型和值都相等时, `==`才会返回true.  

```bash
	jq '.[] == 1'
Input	[1, 1.0, "1", "banana"]
Output	true
true
false
false
```

###  `if-then-else`
`if A then B else C end`表达式在A的值不是false或null时, 执行B, 否则执行C.   

相对于javascript和Python, 检查false和null是对"真值"的一种简化表示, 但是这有时候也意味着你要更多的关注一些细节, 比如说你要检查一个string是否为空, 那么`if .name then A else B end`这种方式是不行的(因为.name是空的话, 那么会返回null), 应该明确的检查长度`if (.name | length ) > 0 then A else B end`.  

如果条件A会产生多个输出, 那么B会针对每一个非false或null的输出都执行一遍, 而C会针对每一个false和null执行一遍.  

如果还有更多的条件分支的话, 可以使用`elif A then B`这种语法.  

```bash
    jq 'if . == 0 then
      "zero"
    elif . == 1 then
      "one"
    else
      "many"
    end'
Input	2
Output	"many"
```

### `>, >=, <=, <`
这些比较操作符分别表示大于, 大于或等于, 小于或等于, 小于.  

```bash
	jq '. < 5'
Input	2
Output	true
```

### `and/or/not`
jq也支持布尔操作 -- `and/or/not`. 它们和jq表达式对真值的判断规则是一样的:  false或null被视为"假值", 其它的都被视为"真值".  

如果`and/or/not`的操作数会产生多个结果, 那布尔操作符会对每一个输入都执行一遍, 从而产生多个输出.  

`not`其实是一个内建方法而不是操作符, 所以它一般作为一个过滤器通过结合pipe使用, 没有特别的语法, 比如`.foo and .bar | not`.  

这三个操作符只能产生"true"或"false", 所以它们只对布尔运算有用. 如果你想使用`or`从两个值中挑选一个, 而且并不需要判断某个条件是否满足, 那么请参考下面的`//`操作符.  

```bash
    jq '42 and "a string"'
Input	null
Output	true
```

```bash
    jq '(true, false) or false'
Input	null
Output	true
        false
```

```bash
	jq '(true, true) and (true, false)'
Input	null
Output	true
false
true
false
```

```bash
	jq '[true, false | not]'
Input	null
Output	[false, true]
```

### 二选一操作: `//`
`a//b`形式的过滤器, 如果a产生的结果不是false或null, 那么最终输出是a的结果; 否则最终结果是b产生的结果.  

这种形式的表达式对于提供默认设置相当有用: `.foo//1`这种表达式, 如果输入对象中不存在foo的话, 那么结果就是1. 这种功能在其它很多编程语言中都有, 比如Python中的`or`. (因为jq中的or被严格限制为只能做布尔运算)  

```bash
	jq '.foo // 42'
Input	{"foo": 19}
Output	19
```

```bash
	jq '.foo // 42'
Input	{}
Output	42
```

### `try-catch`
使用`try EXP1 catch HANDLER`这种语法来捕获错误. 首先执行EXP1, 那么再用出错消息(error message)作为参数执行HANDLER. HANDLER的输出, 如果有的话, 将被作为try的输出.  

如果省略了HANDLER, `try EXP1`默认使用`empty`作为handler.  

```bash
	jq 'try .a catch ". is not an object"'
Input	true
Output	". is not an object"
```

```bash
	jq '[.[]|try .a]'
Input	[{}, true, {"a":1}]
Output	[null, 1]
```

```bash
	jq 'try error("some exception") catch .'
Input	true
Output	"some exception"
```

### 从控制结构中跳出: `break`
除了try/catch之外, 另一种从`reduce`, `foreach`, `while`这些控制结构中跳出来的方法. (类似goto????)

// TODO


### 错误抑制/可选操作: `?`
`?`操作符, 像`EXP?`这样, 其实是`try EXP`的简写.  

```bash
	jq '[.[]|(.a)?]'
Input	[{}, true, {"a":1}]
Output	[null, 1]
```

## 正则表达式(PCRE)
jq和php, ruby, TextMate, Sublime Text等一样, 都是使用的Oniguruma正则表达式库, 所以下面我们只关注于jq的例外的地方.  

jq正则过滤器使用以下模式之一:  

* `STRING | FILTER( REGEX )`
* `STRING | FILTER( REGEX; FLAGS )`
* `STRING | FILTER( [REGEX] )`
* `STRING | FILTER( [REGEX, FLAGS] )`

其中:  
* STRING, REGEX和FLAGS都是接受插值方式的jq string. 
* REGEX, 经过string插值之后, 必须是一个合法的PCRE正则表达式.
* FILTER是`test`, `match`, `capture`其中之一. 

FLAGS是由以下一个或多个选项组成的string:  
* `g` -- 全局搜索(找出所有的匹配项, 而不仅仅是第一项)
* `i` -- 大小写敏感.
* `m` -- 多行模式(`.`能够匹配换行符)
* `n` -- 忽略空的匹配项
* `p` -- 同时支持`m`和`s`选项.
* `s` -- 单行模式
* `l` -- 找到最长的匹配项(贪婪模式?)
* `x` -- 扩展的正则表达式格式(忽略空白符和注释) 
在`x`模式下, 如果要匹配空白符, 需要用`\s`进行转义, 例如:  
* test("a\sb", "x")

注意: 这些标志符也可以在REGEX中使用, 例如:   
* jq -n '("test", "TEst", "teST", "TEST") | test( “(?i)te(?-i)st” )'  
产生结果: true, true, false, false  

### `test(val), test(regex; flags)`
类似`match`, 不同之处在于它只返回`true`和`false`表示输入是否和模式匹配, 而不返回匹配到的结果.  

```bash
	jq 'test("foo")'
Input	"foo"
Output	true
```

```bash
    jq '.[] | test("a b c # spaces are ignored"; "ix")'
Input	["xabcd", "ABC"]
Output	true
true
```

### `match(val), match(regex; flags)`
`match`输出所有找到的匹配项的object. 匹配项object包含以下字段:  
* `offset` - 以UTF-8编码的从输入string开头算起的偏移量.
* `length` - 匹配项的UTF-8编码长度.
* `string` - 匹配到的string.
* `captures` - 捕获到的对象数组.

捕获到的对象包含以下字段:  
* `offset` - 以UTF-8编码的从输入string开头算起的偏移量.
* `length` - 捕获项的UTF-8编码长度.
* `string` - 捕获到的string.
* `name` - 捕获项的名字(如果没有命名, 默认为null)
如果捕获项没有匹配到任何内容, `offset`被设置为-1.  

```bash
	jq 'match("(abc)+"; "g")'
Input	"abc abc"
Output	{"offset": 0, "length": 3, "string": "abc", "captures": [{"offset": 0, "length": 3, "string": "abc", "name": null}]}
{"offset": 4, "length": 3, "string": "abc", "captures": [{"offset": 4, "length": 3, "string": "abc", "name": null}]}
```

```bash
	jq 'match("foo")'
Input	"foo bar foo"
Output	{"offset": 0, "length": 3, "string": "foo", "captures": []}
```

```bash
	jq 'match(["foo", "ig"])'
Input	"foo bar FOO"
Output	{"offset": 0, "length": 3, "string": "foo", "captures": []}
{"offset": 8, "length": 3, "string": "FOO", "captures": []}
```

```bash
	jq 'match("foo (?<bar123>bar)? foo"; "ig")'
Input	"foo bar foo foo  foo"
Output	{"offset": 0, "length": 11, "string": "foo bar foo", "captures": [{"offset": 4, "length": 3, "string": "bar", "name": "bar123"}]}
{"offset": 12, "length": 8, "string": "foo  foo", "captures": [{"offset": -1, "length": 0, "string": null, "name": "bar123"}]}
```

```bash
	jq '[ match("."; "g")] | length'
Input	"abc"
Output	3
```

### `capture(val), capture(regex; flags)`
用命名变量捕获对应的值, 然后用命名变量作为key, 捕获项作为value, 返回一个JSON的object.  

```bash
    jq 'capture("(?<a>[a-z]+)-(?<n>[0-9]+)")'
Input	"xyzzy-14"
Output	{ "a": "xyzzy", "n": "14" }
```

### `scan(regex), scan(regex; flags)`
从输入string中捕获一个能够匹配到regex和flags(如果设置了的话)的非重叠的子string. 如果没有匹配, 那么输出是empty. 如果要捕获每一个输入string中的匹配项, 使用`[ expr ]`语法, 如`[ scan(regex)]`.  

### `split(regex; flags)`
[似乎不清不楚]  
为了后向兼容, `split`以string而不是正则的方式.  

### `splits(regex), splits(regex; flags)`
[没看懂]  

### `sub(regex; tostring), sub(regex; string; flags)`
对输入string进行regex匹配, 然后用`tostring`代替第一个匹配项. `tostring`可以是jq的string, 可以包含命名的捕获项.   
string都可以插值.  

### `gsub(regex; string), gsub(regex; string; flags)`
和`sub`功能一样, 区别在于`gsub`会对所有的匹配项都替换为`string`.

## 高级特性

## 数学计算

## I/O

## 流(streaming)

## 赋值(assignment)

## 模块化

## 颜色显示====