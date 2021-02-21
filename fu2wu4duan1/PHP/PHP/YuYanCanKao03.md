## PHP手册--语言参考03

### 类与对象

#### 基本概念

当一个方法在类定义内部被调用时，有一个可用的伪变量$this。$this是一个到主叫对象的引用（通常是该方法所从属的对象，但如果是从第二个对象静态调用时也可能是另一个对象）。

要创建一个类的实例，必须使用new关键字。当创建新对象时该对象总是被赋值，除非该对象定义了构造函数并且在出错时抛出了一个异常。类应在被实例化之前定义（某些情况下则必须这样）。

如果在new之后跟着的是一个包含有类名的字符串string，则该类的一个实例被创建。如果该类属于一个命名空间，则必须使用其完整名称。

```php
class SimpleClass
{
    public $var;
    
    public function displayVar() {
        echo $this->var;
    }
}

$instance = new SimpleClass();
// 也可以这样做
$className = 'SimpleClass';
$instance = new $className();
// new SimpleClass()
```

PHP5.3.0引进了两个新方法来创建一个对象的实例。

```php
class Test
{
    static public function getNew()
    {
        return new static;
    }
}

class Child extends Test
{}

$obj1 = new Test();
$obj2 = new $obj1;
var_dump($obj1 !== $obj2);
// bool(true)
$obj3 = Test::getNew();
var_dump($obj3 instanceof Test);
// bool(true)
$obj4 = Child::getNew();
var_dump($obj4 instanceof Child);
// bool(true)
```

PHP5.4.0起，可以通过一个表达式来访问新创建对象的成员。

```php
(new SimpleClass())->displayVar();
```

在类定义内部，可以用new self和new parent创建新对象。

当把一个对象已经创建的实例赋给一个新变量时，新变量会访问同一个实例。可以用克隆给一个已创建的对象建立一个新实例。

```php
$instance = new SimpleClass();
$assigned = $instance;
$reference = &$instance;

$instance->var = '$assigned will have this value';

$instance = null;
// $instance and $reference become null

var_dump($instance,$reference,$assigned);
// NULL
var_dump($reference);
// NULL
var_dump($assigned);
// object(SimpleClass)#1 (1) {
//    ["var"]=>string(30) "$assigned will have this value"
// }
```

这个例子中，$instance和$assigned指向了同一个实例，$reference指向了$instance。所以，当$instance变为null时，$reference因为指向了$instance，所以也是null，但是$assigned依然指向那个实例，所以有值。

可以通过`parent::`来访问被覆盖的方法或属性。当覆盖方法时，参数必须保持一致否则PHP将发出E_STRICT级别的错误信息。但构造函数例外，构造函数可在被覆盖时使用不同的参数。

自PHP5.5起，关键词class也可用于类名的解析。使用`ClassName::class`你可以获取一个字符串，包含了类ClassName的完全限定名称。这对使用了命名空间的类尤其有用。

#### 属性

属性中的变量可以初始化，但是初始化的值必须是常数，这里的常数是指PHP脚本在编译阶段时就可以得到其值，而不依赖于运行时的信息才能求值。

为了向后兼容PHP4，PHP5声明属性依然可以直接使用关键字var来替代（或者附加于）public，protected或private。但是已不再需要var了。如果直接使用var声明属性，而没有用public，protected或private之一，PHP5会将其视为public。

在类的成员方法里面，可以用`->`（对象运算符）：`$this->property`（其中property是该属性名）这种方式来访问非静态属性。静态属性则是用`::`（双冒号）：`self::$property`来访问。

#### 类常量

可以把在类中始终保持不变的值定义为常量。在定义和使用常量的时候不需要使用$符号。常量的值必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用。接口（interface）中也可以定义常量。

自PHP5.3.0起，可以用一个变量来动态调用类。但该变量的值不能为关键字（如self，parent或static）。

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

// 自 PHP 5.3.0 起
$classname = "MyClass";
echo $classname::constant . "\n";

// 自 PHP 5.3.0 起
$class = new MyClass();
$class->showConstant();
echo $class::constant . "\n"; 
```

####  构造函数和析构函数

PHP允许开发者在一个类中定义一个方法作为构造函数。具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。

如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用`parent::__construct()`。如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承（假如没有被定义为private的话）。

自PHP5.3.3起，在命名空间中，与类名同名的方法不再作为构造函数。不使用命名空间中的类则不受影响。构造函数是一个普通的方法，在对应对象实例化时自动被调用。因此可以定义任何数量的参数，可以是必选、可以有类型、可以有默认值。构造器的参数放在类名后的括号里调用。如果一个类没有构造函数，以及构造函数的参数不是必填项时，括号就可以省略。

##### Static创造方法

在PHP中每个class只能有一个构造器。然而有些情况下，需要用不同的输入实现不同的方式构造对象。这种情况下推荐使用static方法包装构造。

可以设置构造器为private或protected，防止自行额外调用。这时只有static方法可以实例化一个类。由于它们位于同一个定义的class因此可以访问私有方法，也不需要在同一个对象实例中。当然构造器不一定要设置为private，是否合理取决于实际情况。在下面三个例子中，static关键词会被翻译成代码所在类的类名Product。

```php
class Product
{
    private $id;
    private $name;

    private function __construct(?int $id = null, ?string $name = null)
    {
        $this->id = $id;
        $this->name = $name;
    }

    public static function fromBasicData(int $id, string $name)
    {
        $new = new static($id, $name);
        return $new;
    }

    public static function fromJson(string $json)
    {
        $data = json_decode($json);
        return new static($data['id'], $data['name']);
    }
}

$p1 = Product::fromBasicData(5, 'Widget');
$p2 = Product::fromJson('{"id":5,"name":Widget}');
```

PHP5引入了析构函数的概念，这类似于其它面向对象的语言，如C++。析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。和构造函数一样，父类的析构函数不会被引擎暗中调用。要执行父类的析构函数，必须在子类的析构函数体中显式调用`parent::__destruct()`。

此外也和构造函数一样，子类如果自己没有定义析构函数则会继承父类的。析构函数即使在使用exit()终止脚本运行时也会被调用。在析构函数中调用exit()将会中止其余关闭操作的运行。试图在析构函数（在脚本终止时被调用）中抛出一个异常会导致致命错误。

#### 对象继承

除非使用了自动加载，否则一个类必须在使用之前被定义。如果一个类扩展了另一个，则父类必须在子类之前被声明。此规则适用于类继承其它类与接口。

#### Static（静态）关键字

声明类属性或方法为静态，就可以不实例化类而直接访问。静态属性只能被初始化为文字或常量，不能使用表达式。可以把静态属性初始化为整数或数组，但不能初始化为另一个变量或函数返回值，也不能指向一个对象。

由于静态方法不需要通过对象即可调用，所以伪变量$this在静态方法中不可用。静态属性不能通过一个类已实例化的对象来访问（但静态方法可以）。静态属性不可以由对象通过`->`操作符来访问。用静态方式调用一个非静态方法会导致一个E_STRICT级别的错误。

自PHP5.3.0起，可以用一个变量来动态调用类。但该变量的值不能为关键字self，parent或static。

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
print $foo->my_static . "\n";
// Undefined "Property" my_static

// 自 PHP 5.3.0 起
$classname = "Foo";
echo $classname::my_static . "\n";
$classname::aStaticMethod(); 
```

#### 抽象类

任何一个类，如果它里面至少有一个方法是被声明为抽象的，那么这个类就必须被声明为抽象的。

继承一个抽象类的时候，子类必须定义父类中的所有抽象方法；另外，这些方法的访问控制必须和父类中一样（或者更为宽松）。此外方法的调用方式必须匹配，即类型和所需参数数量必须一致。

#### 对象接口

接口中定义的所有方法都必须是公有，这是接口的特性。需要注意的是，在接口中定义一个构造方法是被允许的。接口中也可以定义常量。接口常量和类常量的使用完全相同，但是不能被子类或子接口所覆盖。

在PHP5.3.9之前，实现多个接口时，接口中的方法不能有重名，因为这可能会有歧义。在最近的PHP版本中，只要这些重名的方法签名相同，这种行为就是允许的。接口也可以继承，通过使用extends操作符。

#### Trait

我们先来看看软件开发中的两种常用代码复用模式，继承和组合。继承：强调父类与子类的关系，即子类是父类的一个特殊类型。组合：强调整体与局部的关系，侧重的一种需要的关系。

软件开发中有一条原则，叫做组合优于继承。这是因为从耦合度来看，继承要高于组合。底层的代码应当多使用组合，而具体的业务逻辑或顶层代码应当多使用继承，这样能够大大提高的开发效率。继承关系中，子类与父类保持着高度的依赖关系，加上PHP不支持多继承，为了避免重写编写代码，很多功能都被统一封装到父类中。这样做有两个坏处：一是随着继承的层数和子类的增加，代码复杂度不断增加，大量的方法都将面临着重写；二是这些功能对于一些子类来说可能是不必要的，破坏了代码的封装性。

Trait的提出弥补了PHP对组合支持的不足，一个Trait就相当于一个模块，Trait可包含属性、方法与抽象方法，不同的Trait以组合的方式注入到类中。通过逗号分隔，在use声明列出多个Trait，可以都插入到一个类中。其它Trait也能够使用trait。

在Trait定义时通过使用一个或多个Trait，能够组合其它trait中的部分或全部成员。但是无法通过Trait自身来实例化。从基类继承的成员会被Trait插入的成员所覆盖。优先顺序是来自当前类的成员覆盖了Trait的方法，而Trait则覆盖了被继承的方法。为了对使用的类施加强制要求，Trait支持抽象方法的使用。

如果两个Trait都插入了一个同名的方法，如果没有明确解决冲突将会产生一个致命错误。为了解决多个trait在同一个类中的命名冲突，需要使用`insteadof`操作符来明确指定使用冲突方法中的哪一个。以上方式仅允许排除掉其它方法，`as`操作符可以为某个方法引入别名。注意，as操作符不会对方法进行重命名，也不会影响其方法。

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

使用as语法还可以用来调整方法的访问控制。

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

Trait可以定义属性。Trait定义了一个属性后，类就不能定义同样名称的属性，否则会产生fatal error。有种情况例外：属性是兼容的（同样的访问可见度、初始默认值）。在PHP7.0之前，属性是兼容的，则会有E_STRICT的提醒。

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

Trait可以使用静态属性和定义静态方法。

#### 匿名类

PHP7开始支持匿名类。可以传递参数到匿名类的构造器，也可以扩展（extend）其他类、实现接口（implement interface），以及像其他普通的类一样使用trait。声明的同一个匿名类，所创建的对象都是这个类的实例。

#### 重载

PHP所提供的重载（overloading）是指动态地创建类属性和方法。我们是通过魔术方法（magicmethods）来实现的。当调用当前环境下未定义或不可见的类属性或方法时，重载方法会被调用。所有的重载方法都必须被声明为public。

属性重载，在给不可访问属性赋值时，\_\_set()会被调用。读取不可访问属性的值时，\_\_get()会被调用。当对不可访问属性调用isset()或empty()时，\_\_isset()会被调用。当对不可访问属性调用unset()时，\_\_unset()会被调用。

参数$name是指要操作的变量名称。__set()方法的$value参数指定了$name变量的值。

```php
public __set ( string $name , mixed $value ) : void
public __get ( string $name ) : mixed
public __isset ( string $name ) : bool
public __unset ( string $name ) : void
```

方法重载，在对象中调用一个不可访问方法时，\_\_call()会被调用。在静态上下文中调用一个不可访问方法时，\_\_callStatic()会被调用。

$name参数是要调用的方法名称。$arguments参数是一个枚举数组，包含着要传递给方法$name的参数。

```php
public __call ( string $name , array $arguments ) : mixed
public static __callStatic ( string $name , array $arguments ) : mixed
```

#### 对象复制

对象复制可以通过clone关键字来完成（如果可能，这将调用对象的\_\_clone()方法）。对象中的\_\_clone()方法不能被直接调用。`$copy_of_object = clone $object;`。当对象被复制后，PHP5会对对象的所有属性执行一个浅复制（shallowcopy）。所有的引用属性仍然会是一个指向原来的变量的引用。

当复制完成时，如果定义了\_\_clone()方法，则新创建的对象（复制生成的对象）中的\_\_clone()方法会被调用，可用于修改属性的值（如果有必要的话）。克隆以后，两个对象互不干扰。

#### 对象比较

当使用比较运算符（==）比较两个对象变量时，比较的原则是：如果两个对象的属性和属性值都相等，而且两个对象是同一个类的实例，那么这两个对象变量相等。而如果使用全等运算符（===），这两个对象变量一定要指向某个类的同一个实例（即同一个对象）。

#### 类型约束

函数的参数可以指定必须为对象（在函数原型里面指定类的名字），接口，数组（PHP5.1起）或者callable（PHP5.4起）。不过如果使用NULL作为参数的默认值，那么在调用函数的时候依然可以使用NULL作为实参。类型约束允许NULL值。

如果一个类或接口指定了类型约束，则其所有的子类或实现也都如此。类型约束不能用于标量类型如int或string，Traits也不允许。函数调用的参数与定义的参数类型不一致时，会抛出一个可捕获的致命错误。类型约束不只是用在类的成员函数里，也能使用在函数里。

#### 对象和引用

在PHP5的对象编程经常提到的一个关键点是默认情况下对象是通过引用传递的。但其实这不是完全正确的。PHP的引用是别名，就是两个不同的变量名字指向相同的内容。在PHP5，一个对象变量已经不再保存整个对象的值。只是保存一个标识符来访问真正的对象内容。当对象作为参数传递，作为结果返回，或者赋值给另外一个变量，另外一个变量跟原来的不是引用的关系，只是他们都保存着同一个标识符的拷贝，这个标识符指向同一个对象的真正内容。

#### 对象序列化

所有PHP里面的值都可以使用函数serialize()来返回一个包含字节流的字符串来表示。unserialize()函数能够重新把字符串变回PHP原来的值。序列化一个对象将会保存对象的所有变量，但是不会保存对象的方法，只会保存类的名字。

为了能够unserialize()一个对象，这个对象的类必须已经定义过。如果序列化类A的一个对象，将会返回一个跟类A相关，而且包含了对象所有变量值的字符串。如果要想在另外一个文件中解序列化一个对象，这个对象的类必须在解序列化之前定义，可以通过包含一个定义该类的文件或使用函数spl_autoload_register()来实现。

#### 后期静态绑定

自PHP5.3.0起，PHP增加了一个叫做后期静态绑定的功能，用于在继承范围内引用静态调用的类。

准确说，后期静态绑定工作原理是存储了在上一个非转发调用（non-forwarding call）的类名。当进行静态方法调用时，该类名即为明确指定的那个（通常在::运算符左侧部分）；当进行非静态方法调用时，即为该对象所属的类。所谓的“转发调用”（forwarding call）指的是通过以下几种方式进行的静态调用：`self::`，`parent::`，`static::`以及forward_static_call()。可用get_called_class()函数来得到被调用的方法所在的类名，`static::`则指出了其范围。

该功能从语言内部角度考虑被命名为后期静态绑定。后期绑定的意思是说，`static::`不再被解析为定义当前方法所在的类，而是在实际运行时计算的。也可以称之为“静态绑定”，因为它可以用于（但不限于）静态方法的调用。

使用 `self::` 或者 \_\_CLASS\_\_ 对当前类的静态引用，取决于定义当前方法所在的类。

```php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        self::who();
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::test();
// A
```

后期静态绑定本想通过引入一个新的关键字表示运行时最初调用的类来绕过限制。简单地说，这个关键字能够让你在上述例子中调用test()时引用的类是B而不是A。最终决定不引入新的关键字，而是使用已经预留的static关键字。

```php
class A {
    public static function who() {
        echo __CLASS__;
    }
    public static function test() {
        // 后期静态绑定从这里开始
        static::who();
    }
}

class B extends A {
    public static function who() {
        echo __CLASS__;
    }
}

B::test();
// B
```

在非静态环境下，所调用的类即为该对象实例所属的类。由于`$this->`会在同一作用范围内尝试调用私有方法，而`static::`则可能给出不同结果。另一个区别是`static::`只能用于静态属性。

```php
class A {
    private function foo() {
        echo "success!\n";
    }
    public function test() {
        $this->foo();
        static::foo();
    }
}

class B extends A {
   // foo() will be copied to B, hence its scope will still be A and the call be successful
}

class C extends A {
    private function foo() {
        /* original method is replaced; the scope of the new one is C */
    }
}

$b = new B();
$b->test();
// success!
// success!
$c = new C();
$c->test();
// success!
// Fatal error:  Call to private method C::foo() from context 'A' in /tmp/test.php on line 9
```

这里报错的原因是在调用`$c->test();`时，由于方法是继承过来的所以`$this->foo();`调的是A的foo()方法。而`static::foo();`是在C里调的，所以解析为`C::foo()`，这个方法对于A来说是不可访问的。

后期静态绑定的解析会一直到取得一个完全解析了的静态调用为止。另一方面，如果静态调用使用`parent::`或者`self::`将转发调用信息。

```php
class A {
    public static function foo() {
        static::who();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}

class B extends A {
    public static function test() {
        A::foo();
        parent::foo();
        self::foo();
    }

    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}

C::test();
// A
// C
// C
```

