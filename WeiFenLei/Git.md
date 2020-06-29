# Git 配置

## 全局配置用户名和邮箱

全局配置用户名和邮箱：
```
$ git config --global user.name 'xxx'

$ git config --global user.email 'xxx@xxx.com'
```

这条命令会在 `C:\Users\用户名\` 创建一个 `.gitconfig` 文件用于保存配置。

## 记住密码

永久记住密码:
```
$ git config --global credential.helper store
```

临时记住密码:
```
$ git config –global credential.helper cache

$ git config –global credential.helper 'cache –timeout=3600'
```

这两条命令会在 `C:\Users\用户名\` 创建一个 `.gitconfig` 文件用于保存配置。
第一次提交任然需要输入用户名和密码。提交成功后，同一个用户就不需要再输入了。
提交成功后会在 `C:\Users\用户名\` 创建一个 `.git-credentials` 文件用于保存密码。