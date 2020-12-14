## PHP手册--函数参考

### http_build_query()

http_build_query()函数会把true值转换成1，如果一定要true字符串，赋值的时候用true的字符串值不要直接使用布尔值。

```php
$queryData = [
    'true_bool' => true,
    'true_string' => 'true'
];
echo http_build_query($queryData);
// true_bool=1&true_string=true
```

