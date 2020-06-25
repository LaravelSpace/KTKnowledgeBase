```json
{
  "title": "PHP 手册 -- 语言参考02",
  "updated_at": "2020-06-20",
  "updated_by": "KelipuTe",
  "tags": "PHP,PHP 手册"
}
```

---

## PHP 手册 —— [语言参考](https://www.php.net/manual/zh/langref.php) 02

### [类与对象](https://www.php.net/manual/zh/language.oop5.php)

#### [基本概念](https://www.php.net/manual/zh/language.oop5.basic.php)

$this 是一个到主叫对象的引用（通常是该方法所从属的对象，但如果是从第二个对象静态调用时也可能是另一个对象）。

要创建一个类的实例，必须使用 new 关键字。当创建新对象时该对象总是被赋值，除非该对象定义了构造函数并且在出错时抛出了一个异常。类应在被实例化之前定义（某些情况下则必须这样）。如果在 new 之后跟着的是一个包含有类名的字符串 string，则该类的一个实例被创建。如果该类属于一个命名空间，则必须使用其完整名称。

在类定义内部，可以用 new self 和 new parent 创建新对象。当把一个对象已经创建的实例赋给一个新变量时，新变量会访问同一个实例，就和用该对象赋值一样。此行为和给函数传递入实例时一样。可以用克隆给一个已创建的对象建立一个新实例。

```php
class SimpleClass
{
    public $var;
}

$instance = new SimpleClass();
$assigned = $instance;
$reference = &$instance;

$instance->var = '$assigned will have this value';
// $instance and $reference become null
$instance = null;

var_dump($instance);
var_dump($reference);
var_dump($assigned);
```

以上例程会输出：

```php
NULL
NULL
object(SimpleClass)#1 (1) {
    ["var"]=>
    string(30) "$assigned will have this value"
}
```

可以通过 parent:: 来访问被覆盖的方法或属性。当覆盖方法时，参数必须保持一致否则 PHP 将发出 E_STRICT 级别的错误信息。但构造函数例外，构造函数可在被覆盖时使用不同的参数。

自 PHP 5.5 起，关键词 class 也可用于类名的解析。使用 `ClassName::class` 你可以获取一个字符串，包含了类 ClassName 的完全限定名称。这对使用了 命名空间 的类尤其有用。

#### [属性](https://www.php.net/manual/zh/language.oop5.properties.php)

属性中的变量可以初始化，但是初始化的值必须是常数，这里的常数是指 PHP 脚本在编译阶段时就可以得到其值，而不依赖于运行时的信息才能求值。

为了向后兼容 PHP 4，PHP 5 声明属性依然可以直接使用关键字 var 来替代（或者附加于）public，protected 或 private。但是已不再需要 var 了。如果直接使用 var 声明属性，而没有用 public，protected 或 private 之一，PHP 5 会将其视为 public。 

在类的成员方法里面，可以用 **->**（对象运算符）：`$this->property`（其中 property 是该属性名）这种方式来访问非静态属性。静态属性则是用 **::**（双冒号）：`self::$property` 来访问。

#### [类常量](https://www.php.net/manual/zh/language.oop5.constants.php)

可以把在类中始终保持不变的值定义为常量。在定义和使用常量的时候不需要使用 $ 符号。常量的值必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用。接口（interface）中也可以定义常量。

自 PHP 5.3.0 起，可以用一个变量来动态调用类。但该变量的值不能为关键字（如 self，parent 或 static）。

```php
class MyClass
{
    const constant = 'constant value';

    function showConstant()
    {
        echo self::constant . "\n";
    }
}

echo MyClass::constant . "\n";

$classname = "MyClass";
// 自 PHP 5.3.0 起
echo $classname::constant . "\n";

$class = new MyClass();
$class->showConstant();
// 自 PHP 5.3.0 起
echo $class::constant . "\n"; 
```

####  [构造函数和析构函数](https://www.php.net/manual/zh/language.oop5.decon.php)

如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 parent::__construct()。如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承（假如没有被定义为 private 的话）。

PHP 5 引入了析构函数的概念，这类似于其它面向对象的语言，如 C++。析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。和构造函数一样，父类的析构函数不会被引擎暗中调用。要执行父类的析构函数，必须在子类的析构函数体中显式调用 parent::__destruct()。此外也和构造函数一样，子类如果自己没有定义析构函数则会继承父类的。析构函数即使在使用 exit() 终止脚本运行时也会被调用。在析构函数中调用 exit() 将会中止其余关闭操作的运行。试图在析构函数（在脚本终止时被调用）中抛出一个异常会导致致命错误。

#### [对象继承](https://www.php.net/manual/zh/language.oop5.inheritance.php)

除非使用了自动加载，否则一个类必须在使用之前被定义。如果一个类扩展了另一个，则父类必须在子类之前被声明。此规则适用于类继承其它类与接口。

#### [Static（静态）关键字](https://www.php.net/manual/zh/language.oop5.static.php)

声明类属性或方法为静态，就可以不实例化类而直接访问。静态属性只能被初始化为文字或常量，不能使用表达式。可以把静态属性初始化为整数或数组，但不能初始化为另一个变量或函数返回值，也不能指向一个对象。

由于静态方法不需要通过对象即可调用，所以伪变量 $this 在静态方法中不可用。静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）。静态属性不可以由对象通过 -> 操作符来访问。用静态方式调用一个非静态方法会导致一个 E_STRICT 级别的错误。

自 PHP 5.3.0 起，可以用一个变量来动态调用类。但该变量的值不能为关键字 self，parent 或 static。 

```php
class Foo
{
    public static $my_static = 'foo';

    public function staticValue()
    {
        return self::$my_static;
    }

    public static function aStaticMethod()
    {
        // ...
    }
}

echo Foo::my_static . "\n";
Foo::aStaticMethod();

$foo = new Foo();
print $foo::$my_static . "\n";
// Undefined "Property" my_static
print $foo->my_static . "\n";

$classname = "Foo";
// 自 PHP 5.3.0 起
echo $classname::my_static . "\n";
$classname::aStaticMethod(); 
```

#### [抽象类](https://www.php.net/manual/zh/language.oop5.abstract.php)

任何一个类，如果它里面至少有一个方法是被声明为抽象的，那么这个类就必须被声明为抽象的。

继承一个抽象类的时候，子类必须定义父类中的所有抽象方法；另外，这些方法的访问控制必须和父类中一样（或者更为宽松）。此外方法的调用方式必须匹配，即类型和所需参数数量必须一致。

#### [对象接口](https://www.php.net/manual/zh/language.oop5.interfaces.php)

接口中定义的所有方法都必须是公有，这是接口的特性。需要注意的是，在接口中定义一个构造方法是被允许的。

在 PHP 5.3.9 之前，实现多个接口时，接口中的方法不能有重名，因为这可能会有歧义。在最近的 PHP 版本中，只要这些重名的方法签名相同，这种行为就是允许的。接口也可以继承，通过使用 extends 操作符。

接口中也可以定义常量。接口常量和类常量的使用完全相同，但是不能被子类或子接口所覆盖。

#### Trait

无法通过 trait 自身来实例化。从基类继承的成员会被 trait 插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了 trait 的方法，而 trait 则覆盖了被继承的方法。为了对使用的类施加强制要求，trait 支持抽象方法的使用。

通过逗号分隔，在 use 声明列出多个 trait，可以都插入到一个类中。其它 trait 也能够使用 trait。在 trait 定义时通过使用一个或多个 trait，能够组合其它 trait 中的部分或全部成员。

如果两个 trait 都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。为了解决多个 trait 在同一个类中的命名冲突，需要使用 insteadof 操作符来明确指定使用冲突方法中的哪一个。以上方式仅允许排除掉其它方法，as 操作符可以为某个方法引入别名。注意，as 操作符不会对方法进行重命名，也不会影响其方法。

```php
trait A
{
    public function smallTalk()
    {
        echo 'a';
    }

    public function bigTalk()
    {
        echo 'A';
    }
}

trait B
{
    public function smallTalk()
    {
        echo 'b';
    }

    public function bigTalk()
    {
        echo 'B';
    }
}

class Talker
{
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
    }
}

class Aliased_Talker
{
    use A, B {
        B::smallTalk insteadof A;
        A::bigTalk insteadof B;
        B::bigTalk as talk;
    }
}
```

使用 as 语法还可以用来调整方法的访问控制。

```php
trait HelloWorld
{
    public function sayHello()
    {
        echo 'Hello World!';
    }
}
// 修改 sayHello 的访问控制
class MyClass1
{
    use HelloWorld {
        sayHello as protected;
    }
}
// 给方法一个改变了访问控制的别名
// 原版 sayHello 的访问控制则没有发生变化
class MyClass2
{
    use HelloWorld {
        sayHello as private myPrivateHello;
    }
}
```

Trait 可以定义属性。 Trait 定义了一个属性后，类就不能定义同样名称的属性，否则会产生 fatal error。 有种情况例外：属性是兼容的（同样的访问可见度、初始默认值）。在 PHP 7.0 之前，属性是兼容的，则会有 E_STRICT 的提醒。

```php
trait PropertiesTrait
{
    public $same = true;
    public $different = false;
}

class PropertiesExample
{
    use PropertiesTrait;

    // PHP 7.0.0 后没问题，之前版本是 E_STRICT 提醒
    public $same = true;
    // 致命错误
    public $different = true;
}
```

Trait 可以使用静态属性和定义静态方法。

#### [匿名类](https://www.php.net/manual/zh/language.oop5.anonymous.php)

PHP 7 开始支持匿名类。可以传递参数到匿名类的构造器，也可以扩展（extend）其他类、实现接口（implement interface），以及像其他普通的类一样使用 trait。声明的同一个匿名类，所创建的对象都是这个类的实例。

#### [重载](https://www.php.net/manual/zh/language.oop5.overloading.php)

PHP所提供的"重载"（overloading）是指动态地"创建"类属性和方法。我们是通过魔术方法（magic methods）来实现的。当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用。所有的重载方法都必须被声明为 public。

属性重载，在给不可访问属性赋值时，__set() 会被调用。读取不可访问属性的值时，__get() 会被调用。当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用。当对不可访问属性调用 unset() 时，__unset() 会被调用。

参数 `$name` 是指要操作的变量名称。__set() 方法的 `$value` 参数指定了 `$name` 变量的值。

```php
public __set ( string $name , mixed $value ) : void
public __get ( string $name ) : mixed
public __isset ( string $name ) : bool
public __unset ( string $name ) : void
```

方法重载， 在对象中调用一个不可访问方法时，__call() 会被调用。在静态上下文中调用一个不可访问方法时，__callStatic() 会被调用。

$name 参数是要调用的方法名称。$arguments 参数是一个枚举数组，包含着要传递给方法 $name 的参数。

```php
public __call ( string $name , array $arguments ) : mixed
public static __callStatic ( string $name , array $arguments ) : mixed
```

#### [对象复制](https://www.php.net/manual/zh/language.oop5.cloning.php)

对象复制可以通过 clone 关键字来完成（如果可能，这将调用对象的 __clone() 方法）。对象中的 __clone() 方法不能被直接调用。`$copy_of_object = clone $object;`。当对象被复制后，PHP 5 会对对象的所有属性执行一个浅复制（shallow copy）。所有的引用属性仍然会是一个指向原来的变量的引用。

当复制完成时，如果定义了 __clone() 方法，则新创建的对象（复制生成的对象）中的 __clone() 方法会被调用，可用于修改属性的值（如果有必要的话）。克隆以后，两个对象互不干扰。

#### [对象比较](https://www.php.net/manual/zh/language.oop5.object-comparison.php)

当使用比较运算符（==）比较两个对象变量时，比较的原则是：如果两个对象的属性和属性值 都相等，而且两个对象是同一个类的实例，那么这两个对象变量相等。而如果使用全等运算符（===），这两个对象变量一定要指向某个类的同一个实例（即同一个对象）。

#### [类型约束](https://www.php.net/manual/zh/language.oop5.typehinting.php)

函数的参数可以指定必须为对象（在函数原型里面指定类的名字），接口，数组（PHP 5.1 起）或者 callable（PHP 5.4 起）。不过如果使用 NULL 作为参数的默认值，那么在调用函数的时候依然可以使用 NULL 作为实参。类型约束允许 NULL 值。

如果一个类或接口指定了类型约束，则其所有的子类或实现也都如此。类型约束不能用于标量类型如 int 或 string，Traits 也不允许。函数调用的参数与定义的参数类型不一致时，会抛出一个可捕获的致命错误。类型约束不只是用在类的成员函数里，也能使用在函数里。

#### [对象和引用](https://www.php.net/manual/zh/language.oop5.references.php)

在 PHP 5 的对象编程经常提到的一个关键点是“默认情况下对象是通过引用传递的”。但其实这不是完全正确的。PHP 的引用是别名，就是两个不同的变量名字指向相同的内容。在 PHP 5，一个对象变量已经不再保存整个对象的值。只是保存一个标识符来访问真正的对象内容。当对象作为参数传递，作为结果返回，或者赋值给另外一个变量，另外一个变量跟原来的不是引用的关系，只是他们都保存着同一个标识符的拷贝，这个标识符指向同一个对象的真正内容。

#### [对象序列化](https://www.php.net/manual/zh/language.oop5.serialization.php)

所有 PHP 里面的值都可以使用函数 serialize() 来返回一个包含字节流的字符串来表示。unserialize() 函数能够重新把字符串变回 PHP 原来的值。序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。

为了能够 unserialize() 一个对象，这个对象的类必须已经定义过。如果序列化类 A 的一个对象，将会返回一个跟类A相关，而且包含了对象所有变量值的字符串。如果要想在另外一个文件中解序列化一个对象，这个对象的类必须在解序列化之前定义，可以通过包含一个定义该类的文件或使用函数 spl_autoload_register() 来实现。