## 自定义的header参数后端程序获取不到

### 出现此问题的场景

- 1、使用了nginx作为反向代理。
- 2、自定义的header参数的key键名中使用了`_`，而不是`-`。

### 问题出现的原因

ngx_http_parse_header_line()函数

```c
if (ch == '_')
{
    if (allow_underscores)
        Unknown macro:
        {
            hash = ngx_hash(hash, ch);
            r->lowcase_header[i++] = ch;
            i &= (NGX_THHP_LC_HEADER_LEN-- 1);
        }
    else
        Unknown macro:
        {
            r->invalid_header = 1;
        }
}
```

以上代码说明：nginx对header的name的字符串做了限制，默认underscores_in_headers参数为off，表示如果header的name中包含下划线，则忽略。

### 问题的解决方案

- 1、将程序中的`_`都改为`-`。
- 2、在nginx的http配置中，设置underscores_in_headers参数为on。

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [nginx中HTTP自定义header有时候接收不到，无内容分析](https://blog.csdn.net/qq_32684617/article/details/100689110) |

