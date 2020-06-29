## PHPStrom

#### PHPStrom 注释格式设置

注释格式设置在代码格式设置的地方，菜单栏 **File => Settings => Editor => Code Style => PHP**。

![PHPStrom_01](.\PHPStrom_01.png)

这里设置的是，使用代码格式化的时候，单行注释会和代码缩进格式对齐。

但这个操作并不影响使用快捷键注释的时候生成注释符号的位置。

如果想在使用注释快捷键的时候，不让注释符号出现在最前面，要去下面图里的地方设置。

![PHPStrom_02](.\PHPStrom_02.png)

#### PHPStrom 代码警告

在使用 `json_encode()` 函数时，提示以下警告：`ext-json is missing in composer.json`。

解决方式：在 composer.json 的 `require{}` 中增加一行代码 `"ext-json": "*"`。

## 参考

[ext-json is missing in composer.json. 解决方案](https://note.lxtac.com/index.php/archives/6344/)