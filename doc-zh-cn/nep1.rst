==============================================
Nim 增强提案 #1 - 标准库编码规范
==============================================
:作者: Clay Sweetser, Dominik Picheta
:版本: |nimversion|

.. contents::


介绍
============
尽管 Nim 支持多种代码风格和格式，但在进行某些需要合作完成的工作时，遵循一致的代码风格仍然是有益的。本 NEP 旨在列出一些在进行标准库代码的编写时应遵守的规范。

注意，本文档仅为指导建议，必要时可有例外。Nim 作为一门十分灵活的语言，本文中的某些部分在特定情况下可能会不适用。此外，就像《 `Python 编码规范<http://legacy.python.org/dev/peps/pep-0008/>`_ 》一样，本规范也会随时间而不断更新。

此规范并不会强制执行，仅要求 Nim 和 Nim 官方项目的代码贡献者遵守此规范。官方项目包括 Nim 编译器、标准库、和其他官方工具（比如 C2Nim）。

代码风格指南
================

空白和空格的惯用法
-----------------------------------

- 每行不应超过 80 个字符。通过限制一行中的信息量可以让阅读者单次需要处理的信息变少，从而使代码更加可读。

- 使用两个空格来缩进；不允许使用 tab （由编译器保证）。使用空格可以让代码在不同的编辑器上展示时保持一致性。tab 的宽度由编辑器决定，而且并不是所有编辑器都提供修改 tab 宽度的功能。

- 虽然本指南也允许为了代码美观而使用空格，但这种情况下请务必深思熟虑。并不是所有编辑器都支持自动对齐代码段，而手动对齐一段很长的代码很快就会让你觉得很烦……

  .. code-block:: nim

    # 别这样做，因为当其他人修改这段代码时，他们必须要重新对齐这些赋值语句
    type
      WordBool*    = int16
      CalType*     = int
      ... # 5 lines later
      CalId*       = int
      LongLong*    = int64
      LongLongPtr* = ptr LongLong

命名方式
------------------

注意：下面列出的仅为 *当前* 的命名方式，而这些规范并不是从一开始就存在的。之前，标识符的命名遵循古老的 PascalCase，并带有一个表示其基本类型的前缀。比如 PFoo 是指针或引用类型，TFoo 是类型，EFoo 是异常，诸如此类。尽管现在规范已经改变，但因为历史遗留原因，这种命名方式在标准库的很多地方还依旧存在着，将来它们会被修改成符合当前规范的形式。

- 类型应该使用 PascalCase。所有其他的标识符应使用 camelCase，常量作为例外可以（但不要求）使用 PascalCase。

  .. code-block:: nim

    # 常量使用小些或大写字母开头均可
    const aConstant = 42
    const FooBar = 4.2

    var aVariable = "Meep" # 变量必须由小写字母开头

    # 类型必须由大写字母开头
    type
      FooBar = object

  从 C/C++ 代码生成的封装代码中的常量允许使用 ALL_UPPERCASE 方式，但这很难看。（为什么要责怪常量呢？常量明明很乖巧，危险的是变量！）

  PS：上段原文为 "Why shout CONSTANT"，全大写的表示法在英文语境下有大声喊叫/训斥/责怪的感觉。


- 当某个类型有值类型、指针类型、引用类型这几种变体时，最常使用的那种不加后缀，其他变体分别加上 “Obj”、“Ref” 和 “Ptr” 后缀。如果没有最常使用的，那就只给指针类型加上后缀。对于 C/C++ 的封装代码本规则同样适用。

  .. code-block:: nim

      Handle = object # 最常使用
        fd: int64
      HandleRef = ref Handle # 不那么经常使用


- 异常和错误类型需要加 “Error” 后缀。

  .. code-block:: nim

    type
      UnluckyError = object of Exception

- 对于枚举类型，除非带有 {. pure .} 标注，否则其枚举值必须带有前缀，比如枚举类型名的缩写。

  .. code-block:: nim

    type
      PathComponent = enum
        pcDir
        pcLinkToDir
        pcFile
        pcLinkToFile


- 非纯枚举值使用 camelCase，纯枚举值使用 PascalCase。

  .. code-block:: nim

    type
      PathComponent {.pure.} = enum
        Dir
        LinkToDir
        File
        LinkToFile

- 在当今时代，还把 HTTP, HTML, FTP, TCP, IP, UTF, WWW 这类词当作专有名词而全部大写实在有些过于古板。请他们当作普通单词看待吧。所以是 ``parseUrl`` 而不是 ``parseURL`` ， ``checkHttpHeader`` 而不是 ``checkHTTPHeader`` ，以此类推。

- 像 ``mitems`` 、 ``mpairs`` （和已经废弃的 ``mget`` ）这种用于获取数据结构的内部数据的可变引用的操作需要以 ``m`` 开头。

- 当就地处理和“返回处理结果”两种方式均存在时，后一种方式使用过去分词形式：

  - reverse 和 reversed
  - sort 和 sorted
  - rotate 和 rotated

- 当“返回处理结果”的版本已经存在时，就地处理版本需要加 ``In`` 后缀。举例： ``strutils.replace`` 已存在，这时就地处理版本应该命名为 ``replaceIn`` 。

标准库的 API 的设计原则是**易用**和一致。易用性是通过完成某些高级操作需要的函数调用次数来衡量的。最终目的是开发人员可以根据需要的功能*猜*出要使用的类型或方法名。

标准库使用了一个简单的命名方案，通过使用简称的方式，使名字又短又有意义。


-------------------     ------------   --------------------------------------
英语单词                  使用时          备注
-------------------     ------------   --------------------------------------
initialize              initT          ``init`` 用来创建一个 ``T`` 类型的值
new                     newP           ``new`` 用来创建一个引用类型 ``P``
find                    find           应该返回结果的位置; 当仅仅返回是否找到时，使用 ``contains``
contains                contains       一般来说是 ``find() >= 0`` 的简写形式
append                  add            使用 ``add`` ，不使用 ``append``
compare                 cmp            应该返回 int，并且在 ``< 0`` ``== 0`` 和 ``> 0`` 时具有不同语义。如果是返回布尔值，请使用 ``sameXYZ``
put                     put, ``[]=``   推荐通过重载 ``[]=`` 来提供 put 语义
get                     get, ``[]``    推荐通过重载 ``[]`` 来提供 get 语义。推荐不使用 get 前缀，比如获取长度使用 ``len`` 不用 ``getLen``
length                  len            也有“元素个数”的含义
size                    size, len      size 应该总是指 byte size
capacity                cap
memory                  mem            暗示这是一个底层操作
items                   items          容器类型的默认迭代器
pairs                   pairs          通过 (key, value) 方式迭代
delete                  delete, del    del 应该比 delete 快，因为它不保留顺序，而 delete 保留
remove                  delete, del    当前代码中存在不一致
include                 incl
exclude                 excl
command                 cmd
execute                 exec
environment             env
variable                var
value                   value, val     使用 val, 当前代码中存在不一致
executable              exe
directory               dir
path                    path           举个例子，path 就是 "/usr/bin" 这个字符串本身, 而 dir 应该表示 "/usr/bin" 这个路径所定位到的那个文件夹;  当前代码中存在不一致
extension               ext
separator               sep
column                  col, column    使用 col, 当前代码中存在不一致
application             app
configuration           cfg
message                 msg
argument                arg
object                  obj
parameter               param
operator                opr
procedure               proc
function                func
coordinate              coord
rectangle               rect
point                   point
symbol                  sym
literal                 lit
string                  str
identifier              ident
indentation             indent
-------------------     ------------   --------------------------------------

编码风格
------------------

- 必须使用控制流来完成代码逻辑的情况下才使用 return 语句。在任何可能的情况下都应优先使用隐式定义的 result 变量。 这会提高可读性。

  .. code-block:: nim

    proc repeat(text: string, x: int): string =
      result = ""

      for i in 0 .. x:
        result.add($i)


- 在任何可能的情况下都应优先使用过程（proc）, 只在必要时使用宏，模板，迭代器和转换器。

- 定义在其作用域内值不会改变的变量时使用 ``let`` 。使用 ``let`` 可以保证不可变性，也能让代码阅读者更容易理解代码的含义和意义。


多行语句表达式
-----------------------------------------------------

- 多行元组应该按元素对齐。

  .. code-block:: nim

    type
      LongTupleA = tuple[wordyTupleMemberOne: int, wordyTupleMemberTwo: string,
                         wordyTupleMemberThree: float]


- 多行的过程或过程类型定义也遵循相同的规则。

  .. code-block:: nim

    type
      EventCallback = proc (timeReceived: Time, errorCode: int, event: Event,
                            output: var string)

    proc lotsOfArguments(argOne: string, argTwo: int, argThree: float
                         argFour: proc(), argFive: bool): int
                        {.heyLookALongPragma.} =


- 多行的过程调用需要对齐左括号（和过程定义一样）。

  .. code-block:: nim

    startProcess(nimExecutable, currentDirectory, compilerArguments
                 environment, processOptions)
