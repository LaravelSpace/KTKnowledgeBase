## 常用命令

### grep

```shell
# 在fileName文件中查找string，显示string所在行及后5行的数据
grep -A 5 string fileName
# 在fileName文件中查找string，显示string所在行及前5行的数据
grep -B 5 string fileName
# 在fileName文件中查找string，显示string所在行及前后5行的数据
grep -C 5 string fileName
```

