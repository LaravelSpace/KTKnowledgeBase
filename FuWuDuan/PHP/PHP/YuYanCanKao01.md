## PHP手册--语言参考01

### 类型

#### Float浮点型

浮点数的精度有限。尽管取决于系统，PHP通常使用IEEE754双精度格式，则由于取整而导致的最大相对误差为1.11e-16。非基本数学运算可能会给出更大误差，并且要考虑到进行复合运算时的误差传递。

此外，以十进制能够精确表示的有理数如0.1或0.7，无论有多少尾数都不能被内部所使用的二进制精确表示，因此不能在不丢失一点点精度的情况下转换为二进制的格式。这就会造成混乱的结果：例如，`floor((0.1+0.7)*10)`通常会返回7而不是预期中的8，因为该结果内部的表示其实是类似7.9999999999999991118...。

所以，PHP程序里对float类型尽量做二次处理（取整数、取2位小数等），防止出现2.23变成2.229999999997这样的情况。而且，永远不要相信浮点数结果精确到了最后一位，也永远不要比较两个浮点数是否相等。如果确实需要更高的精度，应该使用[任意精度数学函数](https://www.php.net/manual/zh/ref.bc.php)或者[gmp函数](https://www.php.net/manual/zh/ref.gmp.php)。

#### [String 字符串](https://www.php.net/manual/zh/language.types.string.php)

一个字符串 string 就是由一系列的字符组成，其中每个字符等同于一个字节。这意味着 PHP 只能支持 256 的字符集，因此不支持 Unicode 。string 最大可以达到 2GB。

PHP 中的 string 的实现方式是一个由字节组成的数组再加上一个整数指明缓冲区长度。并无如何将字节转换成字符的信息，由程序员来决定。字符串由什么值来组成并无限制；特别的，其值为 0（“NUL bytes”）的字节可以处于字符串任何位置（不过有几个函数，在本手册中被称为非“二进制安全”的，也许会把 NUL 字节之后的数据全都忽略）。

字符串类型的此特性解释了为什么 PHP 中没有单独的“byte”类型 - 已经用字符串来代替了。返回非文本值的函数 - 例如从网络套接字读取的任意数据 - 仍会返回字符串。 

由于 PHP 并不特别指明字符串的编码，所以字符串会被按照该脚本文件相同的编码方式来编码。不过这并不适用于激活了 Zend Multibyte 时；此时脚本可以是以任何方式编码的（明确指定或被自动检测）然后被转换为某种内部编码，然后字符串将被用此方式编码。注意脚本的编码有一些约束（如果激活了 Zend Multibyte 则是其内部编码）- 这意味着此编码应该是 ASCII 的兼容超集，例如 UTF-8 或 ISO-8859-1。不过要注意，依赖状态的编码其中相同的字节值可以用于首字母和非首字母而转换状态，这可能会造成问题。 

要做到有用，操作文本的函数必须假定字符串是如何编码的。不幸的是，PHP 关于此的函数有很多变种。

要书写能够正确使用 Unicode 的程序依赖于很小心地避免那些可能会损坏数据的函数。要使用来自于 intl 和 mbstring 扩展的函数。不过使用能处理 Unicode 编码的函数只是个开始。不管用何种语言提供的函数，最基本的还是了解 Unicode 规格。例如一个程序如果假定只有大写和小写，那可是大错特错。 

string 中的字符可以通过一个从 0 开始的下标，用类似 array 结构中的方括号包含对应的数字来访问和修改，比如 `$str[42]`。可以把 string 当成字符组成的 array。函数 substr() 和 substr_replace() 可用于操作多于一个字符的情况。 用超出字符串长度的下标写入将会拉长该字符串并以空格填充。非整数类型下标会被转换成整数。非法下标类型会产生一个 E_NOTICE 级别错误。用负数下标写入字符串时会产生一个 E_NOTICE 级别错误，用负数下标读取字符串时返回空字符串。写入时只用到了赋值字符串的第一个字符。用空字符串赋值则赋给的值是 NULL 字符。 

string 也可用花括号访问，比如 `$str{42}`。PHP 的字符串在内部是字节组成的数组。因此用花括号访问或修改字符串对多字节字符集很不安全。仅应对单字节编码例如 ISO-8859-1 的字符串进行此类操作。 

#### [Array 数组](https://www.php.net/manual/zh/language.types.array.php)

PHP 中的数组实际上是一个 **有序映射**。映射是一种把 values 关联到 keys 的类型。此类型在很多方面做了优化，因此可以把它当成真正的数组，或列表（向量），散列表（是映射的一种实现），字典，集合，栈，队列以及更多可能性。由于数组元素的值也可以是另一个数组，树形结构和多维数组也是允许的。 

数组接受任意数量用逗号分隔的**键（key）=>值（value）对**。PHP 数组可以同时含有 integer 和 string 类型的键名，因为 PHP 实际并不区分索引数组和关联数组。如果对给出的值没有指定键名，则取当前最大的整数索引值，而新的键名将是该值加一（PHP 将自动使用之前用过的最大 integer 键名加上 1 作为新的键名）。如果指定的键名已经有了值，则该值会被覆盖，如果在数组定义中多个单元都使用了同一个键名，则只使用了最后一个，之前的都被覆盖。value 可以是任意类型。

此外 key 会有如下的强制转换：

1、包含有合法整型值的字符串会被转换为整型。例如键名 "8" 实际会被储存为 8。但是 "08" 则不会强制转换，因为其不是一个合法的十进制数值。

2、浮点数也会被转换为整型，意味着其小数部分会被舍去。例如键名 8.7 实际会被储存为 8。

3、布尔值也会被转换成整型。即键名 true 实际会被储存为 1 而键名 false 会被储存为 0。

4、Null 会被转换为空字符串，即键名 null 实际会被储存为 ""。

5、数组和对象*不能*被用为键名。坚持这么做会导致警告：Illegal offset type。

### [运算符](https://www.php.net/manual/zh/language.operators.php)

#### [运算符优先级](https://www.php.net/manual/zh/language.operators.precedence.php)

**++、--** 运算符的优先级位于前面。

```php
$a = 1;
// 输出 3：2+1
echo $a + $a++;
// 输出 4：2+2
echo $a + ++$a;

// array(1){[2] =>int(1)}
$i = 1;
$array = [];
$array[$i] = $i++;
```

#### [比较运算符](https://www.php.net/manual/zh/language.operators.comparison.php)

由于浮点数 float 的内部表达方式，不应比较两个浮点数 float 是否相等。 

三元运算符：“?:”。表达式 (expr1) ? (expr2) : (expr3) 在 expr1 求值为 TRUE 时的值为 expr2，在 expr1 求值为 FALSE 时的值为 expr3。自 PHP 5.3 起，可以省略三元运算符中间那部分。表达式 expr1 ?: expr3 在 expr1 求值为 TRUE 时返回 expr1，否则返回 expr3。 注意三元运算符是个语句，因此其求值不是变量，而是语句的结果。

建议避免将三元运算符堆积在一起使用。当在一条语句中使用多个三元运算符时，将会按照从左往右计算的顺序。如果一定要堆积使用，建议将每一组三元运算符及其参数使用括号分组。

PHP 7 开始存在 "??" （NULL 合并）运算符。当 expr1 为 NULL，表达式 (expr1) ?? (expr2) 等同于 expr2，否则为 expr1。尤其要注意，当不存在左侧的值时，此运算符也和 isset() 一样不会产生警告。 对于 array 键尤其有用。NULL 合并运算符是一个表达式，产生的也是表达式结果，而不是变量。

值得注意的一点。?? 运算符是以判断 $a 变量是否存在于上下文环境中作为条件，而 ?: 不具备这种判断。所以 ?? 运算符可用于判断 $a 变量不存在的情况，而使用 ?: 判断一个未定义的变量，PHP 会抛出异常。所以下面的例子，用 ?: 和 ?? 分别判断一个赋值为 0 的变量的时候，结果是不一样的。

```php
$a = 0; $c = 1; $b = $a ?? $c;
// a:0,b:0,c:1
$a = 0; $c = 1; $b = $a ? $a : $c;
// a:0,b:1,c:1
```

### [流程控制](https://www.php.net/manual/zh/language.control-structures.php)

#### [for](https://www.php.net/manual/zh/control-structures.for.php)

下面这段代码可能执行很慢，因为每次循环时都会重新执行 count() 方法，计算一遍数组的长度。

```php
$people = array(
    array('name' => 'Kalle', 'salt' => 856412),
    array('name' => 'Pierre', 'salt' => 215863)
);

for ($i = 0; $i < count($people); ++$i) {
    $people[$i]['salt'] = rand(000000, 999999);
}
```

由于数组的长度始终不变，可以用一个中间变量来储存数组长度以优化而不是不停调用 count() 方法。

下面这段代码 count() 方法只会被执行一次。

```php
$people = array(
    array('name' => 'Kalle', 'salt' => 856412),
    array('name' => 'Pierre', 'salt' => 215863)
);

for ($i = 0, $size = count($people); $i < $size; ++$i) {
    $people[$i]['salt'] = rand(000000, 999999);
}
```

#### [foreach](https://www.php.net/manual/zh/control-structures.foreach.php)

使用引用赋值，可以修改数组元素。

但需要注意的是，数组最后一个元素的引用在 foreach 循环之后仍会保留。建议使用 unset() 将其销毁。

```php
$arr = [1, 2, 3, 4, 5];
foreach ($arr as &$value) {
    $value = $value * 2;
}
// $arr:[2, 4, 6, 8, 10]
unset($value);
```

引用仅在被遍历的数组可以被引用时才可用（例如是个变量）。以下代码是无法运行的。

```php
foreach ([1, 2, 3, 4, 5] as &$value) {
    $value = $value * 2;
}
```

foreach 循环的功能可以通过 while 循环、list()、each() 一起使用实现，下面两端代码的功能完全相同。

```php
$arr = array("one", "two", "three");
foreach ($arr as $value) {
    echo "Value: $value<br />\n";
}
while (list(, $value) = each($arr)) {
    echo "Value: $value<br>\n";
}

$arr = array("one", "two", "three");
foreach ($arr as $key => $value) {
    echo "Key: $key; Value: $value<br />\n";
}
while (list($key, $value) = each($arr)) {
    echo "Key: $key; Value: $value<br />\n";
}
```

#### [break](https://www.php.net/manual/zh/control-structures.break.php)

break 结束当前  for，foreach，while，do-while  或者 switch 结构的执行。

break 可以接受一个可选的数字参数来决定跳出 **几重** 循环。 

```php
$i = 0;
while (++$i) {
    switch ($i) {
        case 5:
            echo "At 5<br />\n";
            break 1;  /* 只退出 switch. */
        case 10:
            echo "At 10; quitting<br />\n";
            break 2;  /* 退出 switch 和 while 循环 */
        default:
            break;
    }
}
```

#### [continue](https://www.php.net/manual/zh/control-structures.continue.php)

continue 在循环结构用用来跳过本次循环中剩余的代码并在条件求值为真时开始执行下一次循环。 

continue 接受一个可选的数字参数来决定跳过 **几重** 循环到循环结尾。默认值是 1，即跳到当前循环末尾。 

注意在 PHP 中 switch 语句被认为是可以使用 continue 的一种循环结构。

另外在使用 continue 时，不要省略结尾的分号。下面的例子不会获得预期的结果。因为整个 `continue print "$i\n";` 被当做单一的表达式而求值，所以 print() 函数只有在 `$i == 2` 为真时才被调用（print() 的值被当成了上述的可选数字参数而传递给了continue）。

```php
for ($i = 0; $i < 5; ++$i) {
    if ($i == 2)
        continue
        print "$i\n";
}
```

[switch](https://www.php.net/manual/zh/control-structures.switch.php)

case 表达式可以是任何求值为简单类型的表达式，即整型或浮点数以及字符串。不能用数组或对象，除非它们被解除引用成为简单类型。

注意和其它语言不同，continue 语句作用到 switch 上的作用类似于 break。如果在循环中有一个 switch 并希望 continue 到外层循环中的下一轮循环，用 continue 2。

注意 switch/case 作的是[松散比较](https://www.php.net/manual/zh/types.comparisons.php#types.comparisions-loose)。 

switch 语句一行接一行地执行。开始时没有代码被执行。仅当一个 case 语句中的值和 switch 表达式的值匹配时 PHP 才开始执行语句，直到 switch 的程序段结束或者遇到第一个 break 语句为止。如果不在 case 的语句段最后写上 break 的话，PHP 将继续执行下一个 case 中的语句段。

在一个 case 中的语句也可以为空，这样只不过将控制转移到了下一个 case 中的语句。一个 case 的特例是 default。它匹配了任何和其它 case 都不匹配的情况。 

#### [return](https://www.php.net/manual/zh/function.return.php)

如果在一个函数中调用 return 语句，将立即结束此函数的执行并将它的参数作为函数的值返回。return 也会终止 eval() 语句或者脚本文件的执行。

如果在全局范围中调用，则当前脚本文件中止运行。如果当前脚本文件是被 include 的或者 require 的，则控制交回调用文件。此外，如果当前脚本是被 include 的，则 return 的值会被当作 include 调用的返回值。如果在主脚本文件中调用 return，则脚本中止运行。如果当前脚本文件是在 php.ini 中的配置选项 auto_prepend_file 或者 auto_append_file 所指定的，则此脚本文件中止运行。 

注意既然 return 是语言结构而不是函数，因此其参数没有必要用括号将其括起来。通常都不用括号，实际上也应该不用，这样可以降低 PHP 的负担。 

如果没有提供参数，则一定不能用括号，此时返回 NULL。如果调用 return 时加上了括号却又没有参数会导致解析错误。 

当用引用返回值时永远不要使用括号，这样行不通。只能通过引用返回变量，而不是语句的结果。如果使用 return ($a); 时其实不是返回一个变量，而是表达式 ($a) 的值（当然，此时该值也正是 $a 的值）

#### [require](https://www.php.net/manual/zh/function.require.php)，[include](https://www.php.net/manual/zh/function.include.php)

include，语句包含并运行指定文件。 被包含文件先按参数给出的路径寻找，如果没有给出目录（只有文件名）时则按照 include_path 指定的目录寻找。如果在 include_path 下没找到该文件则 include 最后才在调用脚本文件所在的目录和当前工作目录下寻找。如果最后仍未找到文件则 include 结构会发出一条警告；这一点和 require 不同，后者会发出一个致命错误。 

require 和 include 几乎完全一样，除了处理失败的方式不同之外。require 在出错时产生 E_COMPILE_ERROR 级别的错误。换句话说将导致脚本中止而 include 只产生警告（E_WARNING），脚本会继续运行。 

如果定义了路径——不管是绝对路径（在 Windows 下以盘符或者 \ 开头，在 Unix/Linux 下以 / 开头）还是当前目录的相对路径（以 . 或者 .. 开头）——include_path 都会被完全忽略。例如一个文件以 ../ 开头，则解析器会在当前目录的父目录下寻找该文件。 

#### [require_once](https://www.php.net/manual/zh/function.require-once.php)，[include_once](https://www.php.net/manual/zh/function.include-once.php)

include_once 语句在脚本执行期间包含并运行指定文件。此行为和 include 语句类似，唯一区别是如果该文件中已经被包含过，则不会再次包含。如同此语句名字暗示的那样，只会包含一次。include_once 可以用于在脚本执行期间同一个文件有可能被包含超过一次的情况下，想确保它只被包含一次以避免函数重定义，变量重新赋值等问题。 

require_once 语句和 require 语句完全相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。 

#### [goto](https://www.php.net/manual/zh/control-structures.goto.php)

goto 操作符可以用来跳转到程序中的另一位置。该目标位置可以用目标名称加上冒号来标记，而跳转指令是 goto 之后接上目标位置的标记。PHP 中的 goto 有一定限制，目标位置只能位于同一个文件和作用域，也就是说无法跳出一个函数或类方法，也无法跳入到另一个函数。也无法跳入到任何循环或者 switch 结构中。可以跳出循环或者 switch，通常的用法是用 goto 代替多层的 break。 

### [函数](https://www.php.net/manual/zh/language.functions.php)

#### [用户自定义函数](https://www.php.net/manual/zh/functions.user-defined.php)

通常函数无需在调用之前被定义，除非一个函数是有条件被定义的，必须在调用函数之前定义。或者是函数中的函数，也需要在被调用之前定义。

在 PHP 中可以调用递归函数，但是要避免递归函数／方法调用超过 100-200 层，因为可能会使堆栈崩溃从而使当前脚本终止。 无限递归可视为编程错误。

#### [函数的参数](https://www.php.net/manual/zh/functions.arguments.php)

通过参数列表可以传递信息到函数，即以逗号作为分隔符的表达式列表。参数是从左向右求值的。PHP 支持按值传递参数（默认），通过引用传递参数以及默认参数。也支持可变长度参数列表。函数可以定义 C++ 风格的标量参数默认值，PHP 允许使用数组 array 和特殊类型 NULL 作为默认参数。 默认值必须是常量表达式，不能是诸如变量，类成员，或者函数调用等。注意当使用默认参数时，任何默认参数必须放在任何非默认参数的右侧；否则，函数将不会按照预期的情况工作。

默认情况下，函数参数通过值传递（因而即使在函数内部改变参数的值，它并不会改变函数外部的值）。如果希望允许函数修改它的参数值，必须通过引用传递参数。如果想要函数的一个参数总是通过引用传递，可以在函数定义中该参数的前面加上符号 **&**。

在 PHP 5 中，类型声明也被称为类型提示。  类型声明允许函数在调用时要求参数为特定类型。 如果给出的值类型不对，那么将会产生一个错误： 在 PHP 5 中，这将是一个可恢复的致命错误，而在 PHP 7 中将会抛出一个 TypeError 异常。为了指定一个类型声明，类型应该加到参数名前。这个声明可以通过将参数的默认值设为 NULL 来实现允许传递 NULL。 

默认情况下，如果能做到的话，PHP将会强迫错误类型的值转为函数期望的标量类型。 例如，一个函数的一个参数期望是 string，但传入的是 integer，最终函数得到的将会是一个 string 类型的值。

```php
function test(string $param) {
    echo param;
}
```

可以基于每一个文件开启严格模式。在严格模式中，只有一个与类型声明完全相符的变量才会被接受，否则将会抛出一个 TypeError。 唯一的一个例外是可以将 integer 传给一个期望 float 的函数。 严格类型适用于在启用严格模式的文件内的函数调用，而不是在那个文件内声明的函数。 一个没有启用严格模式的文件内调用了一个在启用严格模式的文件中定义的函数，那么将会遵循调用者的偏好（弱类型），而这个值将会被转换。严格类型仅用于标量类型声明，也正是因为如此，这需要 PHP 7.0.0 或更新版本，因为标量类型声明也是在那个版本中添加的。 

PHP 在用户自定义函数中支持可变数量的参数列表。在 PHP 5.6 及以上的版本中，由 **...** 语法实现；在 PHP 5.5 及更早版本中，使用函数 func_num_args()，func_get_arg()，和 func_get_args() 。 

#### [返回值](https://www.php.net/manual/zh/functions.returning-values.php)

值通过使用可选的返回语句返回。可以返回包括数组和对象的任意类型。返回语句会立即中止函数的运行，并且将控制权交回调用该函数的代码行。如果省略了 return，则返回值为 NULL。 函数不能返回多个值，但可以通过返回一个数组来得到类似的效果。 

从函数返回一个引用，必须在函数声明和指派返回值给一个变量时都使用引用运算符 **&**。

```php
function &returns_reference()
{
    return $someref;
}
$newref = &returns_reference();
```

PHP 7 增加了对返回值类型声明的支持。 就如 类型声明一样，返回值类型声明将指定该函数返回值的类型。同样，返回值类型声明也与有效类型中可用的参数类型声明一致。

```php
function sum($a, $b): float {
    return $a + $b;
}
```

严格类型也会影响返回值类型声明。在默认的弱模式中，如果返回值与返回值的类型不一致，则会被强制转换为返回值声明的类型。在强模式中，返回值的类型必须正确，否则将会抛出一个 TypeError 异常。当覆盖一个父类方法时，子类方法的返回值类型声明必须与父类一致。如果父类方法没有定义返回类型，那么子类方法可以定义任意的返回值类型声明。 

#### [可变函数](https://www.php.net/manual/zh/functions.variable-functions.php)

PHP 支持可变函数的概念。这意味着如果一个变量名后有圆括号，PHP 将寻找与变量的值同名的函数，并且尝试执行它。可变函数可以用来实现包括回调函数，函数表在内的一些用途。可变函数不能用于例如 echo，print，unset()，isset()，empty()，include，require 以及类似的语言结构。需要使用自己的包装函数来将这些结构用作可变函数。 

```php
function foo() {
    echo "In foo()<br />\n";
}
$func = 'foo';
// This calls foo()
$func();

// 使用 echo 的包装函数
function echoit($string)
{
    echo $string;
}
$func = 'echoit';
// This calls echoit()
$func('test');
```

也可以用可变函数的语法来调用一个对象的方法。 

```php
class Foo
{
    function Variable()
    {
        $name = 'Bar';
        // This calls the Bar() method
        $this->$name();
    }

    function Bar()
    {
        echo "This is Bar";
    }
}

$foo = new Foo();
$funcname = "Variable";
// This calls $foo->Variable()
$foo->$funcname();
```

当调用静态方法时，函数调用要比静态属性优先。

```php
class Foo
{
    static $variable = 'static property';
    static function Variable()
    {
        echo 'Method Variable called';
    }
}

// This prints 'static property'. It does need a $variable in this scope.
echo Foo::$variable; 
$variable = "Variable";
// This calls $foo->Variable() reading $variable in this scope.
Foo::$variable();  
```

#### [内部（内置）函数](https://www.php.net/manual/zh/functions.internal.php)

PHP 有很多标准的函数和结构。还有一些函数需要和特定地 PHP 扩展模块一起编译，否则在使用它们的时候就会得到一个致命的“未定义函数”错误。

如果传递给函数的参数类型与实际的类型不一致，例如将一个 array 传递给一个 string 类型的变量，那么函数的返回值是不确定的。在这种情况下，通常函数会返回 NULL。但这仅仅是一个惯例，并不一定如此。 

#### [匿名函数](https://www.php.net/manual/zh/functions.anonymous.php)

匿名函数（Anonymous functions），也叫闭包函数（closures），允许临时创建一个没有指定名称的函数。匿名函数目前是通过 Closure 类来实现的。最经常用作回调函数（callback）参数的值。

闭包函数也可以作为变量的值来使用。PHP 会自动把此种表达式转换成内置类 Closure 的对象实例。闭包可以从父作用域中继承变量。 任何此类变量都应该用 use 语言结构传递进去。 PHP 7.1 起，不能传入此类变量： superglobals、 $this 或者和参数重名。这些变量都必须在函数或类的头部声明。 从父作用域中继承变量与使用全局变量是不同的。全局变量存在于一个全局的范围，无论当前在执行的是哪个函数。而闭包的父作用域是定义该闭包的函数（不一定是调用它的函数）
