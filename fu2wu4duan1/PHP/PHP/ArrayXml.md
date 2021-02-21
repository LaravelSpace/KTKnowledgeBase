### 数组和XML的相互转换

这个需求有现成的扩展可以使用，这里介绍三个使用过的扩展：openlss/lib-array2xml、verdant/xml2array和spatie/array-to-xml。这三个扩展都可以进行数组和XML的相互转换。

### 安装依赖

```shell
#openlss/lib-array2xml直接安装
$ composer require openlss/lib-array2xml
#spatie/array-to-xml需要根据PHP版本指定扩展的版本
$ composer require spatie/array-to-xml ^2.15.0
#verdant/xml2array只有开发版本
$ composer require verdant/xml2array:dev-master
```

### 使用方式

```php
$array = [
    'Info' => [
        'Telephone' => 13151515151,
        'Email'     => '',
    ]
];
```

verdant/xml2array扩展对openlss/lib-array2xml扩展做了适配，所以从用法上来说openlss/lib-array2xml和verdant/xml2array这两个扩展用法一样，直接把类名换了剩下的代码可以兼容。区别是openlss/lib-array2xml转换出来的XML会带格式（换行和空格）。verdant/xml2array转换出来的XML就是连起来的一大串字符串。

openlss/lib-array2xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Info>
  <Telephone>13151515151</Telephone>
  <Email></Email>
</Info>
```

verdant/xml2array

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Info><Telephone>13151515151</Telephone><Email></Email></Info>
```

spatie/array-to-xml的用法和上面两个不一样，上面两个扩展给节点添加属性的时候用的是`@attribute`，这个扩展是`_attributes`。而且spatie/array-to-xml可以控制输出的XML是不是带格式。

