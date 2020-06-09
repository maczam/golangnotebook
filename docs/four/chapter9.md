# 简介

这是一个Go编程语言的参考手册。更多信息和其他文件，请参见golang.org。

Go是一种基于系统编程设计的通用语言。它是强类型和垃圾收集的，并显式支持并发编程。程序是由包构造的，包的属性允许有效地管理依赖关系。

该语法紧凑且易于解析，允许通过集成开发环境等自动工具进行简单的分析。



# 符号

语法使用扩展的Backus-Naur形式(EBNF)指定:

```
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

结果是由术语和下列运算符组成的表达式，优先级递增:

```
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

小写产品名称用于标识词法标记。非终端机在骆驼箱。词法标记用双引号""或反引号``括起来。

形式a…b表示从a到b作为备选项的一组字符。规范中的其他地方也使用了水平省略号，以非正式地表示没有进一步指定的各种枚举或代码片段。字符(与三个字符相对)不是Go语言的标记。



# 源代码表示

源代码是用UTF-8编码的Unicode文本。文本没有被规范化，因此一个重音编码点有别于由重音和字母组合而成的相同字符;它们被视为两个代码点。为简单起见，本文档将使用非限定术语字符来引用源文本中的Unicode代码点。

每块代码都有不同的含义，例如：大小写字母是不同的字符。

实现限制:为了与其他工具兼容，编译器可能不允许源文本中的NUL字符(U+0000)。

实现限制:为了与其他工具兼容，如果utf -8编码的字节顺序标记(U+FEFF)是源文本中的第一个Unicode编码点，编译器可能会忽略它。在源文件的其他任何地方都可能不允许使用字节顺序标记。



# 字符

以下术语用于表示特定的Unicode字符类:

```go
newline        = /* the Unicode code point U+000A */ .
unicode_char   = /* an arbitrary Unicode code point except newline */ .
unicode_letter = /* a Unicode code point classified as "Letter" */ .
unicode_digit  = /* a Unicode code point classified as "Number, decimal digit" */ .
```

在Unicode标准8.0中，第4.5节“一般类别”定义了一组字符类别。Go将字母类别Lu、Ll、Lt、Lm或Lo中的所有字符作为Unicode字母，而将数字类别Nd中的所有字符作为Unicode数字。



# 字母和数字

下划线字符_ (U+005F)被认为是一个字母。

```go
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```



# 词法元素

## 注释

注释服务源码，主要有两种形式：

1. 行注释从字符序列//开始，止于行尾。
2. 一般注释从字符序列/*开始，到第一个后续字符序列*/结束。

注释不能在符文或字符串文字中开始，也不能在注释中开始。不包含新行的一般注释就像一个空格。任何其他注释都类似于换行符。

## 符号

符号构成了Go语言的词汇表。有四个类:标识符、关键字、操作符和标点符号，以及文本。由空格(U+0020)、水平制表符(U+0009)、回车符(U+000D)和换行符(U+000A)组成的空白将被忽略，除非它将标记分隔开，否则这些标记将合并为单个标记。此外，换行符或文件结束符可能会触发分号的插入。在将输入分解为符号时，下一个符号是形成有效符号的最长字符序列。

## 分号

形式语法使用分号";"作为许多结果中的终止符。Go程序可以使用以下两个规则省略大部分分号:

1. 当输入被分解为记号时，分号会自动插入到记号流中，紧跟在一行的最后记号之后(如果记号是)
	 * 标识符
	 * 一种整数、浮点数、虚数、符文或字符串文字
	 * 关键字break、continue、fallthrough或return中的一个
	 * 操作符和标点符号之一++、——、)、]或}
2. 为了允许复杂语句占据一行，可以在结尾的")"或"}"前面省略分号。

为了反映习惯用法，本文档中的代码示例省略了使用这些规则的分号。

## 标识符

标识符命名程序实体，如变量和类型。标识符是一个或多个字母和数字的序列。标识符中的第一个字符必须是字母。

```
identifier = letter { letter | unicode_digit } .
```

```
a
_x9
ThisVariableIsExported
αβ
```

有些标识符是预先声明的。

## Keywords

以下关键字是保留的，不能用作标识符。

```
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## 运营商和标点符号

下列字符序列表示操作符(包括赋值操作符)和标点符号:

```
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

