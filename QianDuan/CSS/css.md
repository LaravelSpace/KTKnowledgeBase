### div并排显示

让多个div并排显示有3种方法，float:left，display:inline，display:inline-block。

display:inline-block的时候多个div中间会有间隙，这是因为html代码换行导致的。一种解决方案是把代码放到一行，但是那样可读性就太差了，另一种方法可以看情况margin-left一个负数。还需要注意display:inline-block的时候元素有可能发生错位。这是基准线的问题，需要用vertical-align指定他们使用同一个基准线。