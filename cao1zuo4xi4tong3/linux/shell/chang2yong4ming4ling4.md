## 常用shell命令

### clear

clear命令可以清空命令行界面，通常和别的命令连用，达到清空界面并执行命令的目的。比如`$ clear && ps -auwx`可以清空界面然后显示当前进程的状态。`&&`表示逻辑与，和大多数编程语言中的用法一样，如果前面一个命令执行完成并返回真值就接着执行后面的命令。

### grep

```shell
#在fileName文件中查找string，显示string所在行及后5行的数据
grep -A 5 string fileName
#在fileName文件中查找string，显示string所在行及前5行的数据
grep -B 5 string fileName
#在fileName文件中查找string，显示string所在行及前后5行的数据
grep -C 5 string fileName
```

