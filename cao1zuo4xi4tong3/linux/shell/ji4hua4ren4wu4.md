## 计划任务

### 每日日志压缩

```shell
#!/bin/bash

dir_date=$(date -d yesterday +%Y-%m-%d)

cd /tmp/logs/$dir_date/

find ./ -name '*.log' | xargs -i gzip {}

exit;
```

这段脚本用于每天压缩前一天的日志文件。

首先获取前一天的**yyyy-mm-dd**格式的日期。然后进入日志目录对应日期的目录，最后使用`gzip`命令压缩所有以**.log**为后缀名的文件

