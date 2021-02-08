## Git配置

### 全局配置用户名和邮箱

全局配置用户名和邮箱：
```
$ git config --global user.name 'xxx'
$ git config --global user.email 'xxx@xxx.com'
```

这条命令会在`C:\Users\用户名\`创建一个**.gitconfig**文件用于保存配置。

### 记住密码

永久记住密码：
```
$ git config --global credential.helper store
```

临时记住密码：
```
$ git config –global credential.helper cache
$ git config –global credential.helper 'cache –timeout=3600'
```

这两条命令会在`C:\Users\用户名\`创建一个**.gitconfig**文件用于保存配置。第一次提交任然需要输入用户名和密码。提交成功后，同一个用户就不需要再输入了。提交成功后会在`C:\Users\用户名\`创建一个**.git-credentials**文件用于保存密码。

### 修改提交地址

```
$ git remote set-url origin 'xxx.git'
```

### 无法拉取代码

git拉取代码卡住，并输出如下信息。

```
$ git pull
Auto packing the repository in background for optimum performance.
See "git help gc" for manual housekeeping.
fatal: failed to run repack
```

这是因为git本地仓库，如果长时间不进行清理，会导致本地dangling commit太多，从而造成拉取代码失败。可以使用`git fsck`命令查看本地的dangling commit。

```
$ git fsck --lost-found
Checking object directories: 100% (256/256), done.
Checking objects: 100% (110393/110393), done.
dangling commit 01039c0c1efcc232be342aacc928af72df82503b
dangling blob 5b0bd062917d784486756ef941a7446b1aa5e340
dangling commit 4a1c18fa0803db6d64e9cffded496beb686aeaa7
dangling commit f528f06bd9739511bf962e32636697f807764cf8
dangling commit ea5fb0909b71b997c3feb5c436a150f745e40886
dangling commit c76448818640c1b0a3c00546abf16e5b4dff94e4
dangling commit d27fd8f73e071d3a629348f40d079888e821b10f
dangling commit 9a84703a10972d28e784d26417d6f53193b7ef78
dangling blob e7841cedb8d8a5b2cb02814cd9a2af06c5f46e00
dangling commit 6e8cf4714bc2ebb4d866a0aa046766598a1bcaf6
dangling blob fa990caae09e8d23923aed1d303e56272f6c8587
dangling commit 269e6c37f2208841d904577cd6e70d36f8ab0283
dangling commit 1ca1f0e7aafa762a675076fbc25ff6ff7048e41b
...
```

使用`git gc`命令清理后，就可以正常拉取代码了。

```
$ git gc --prune=now
Enumerating objects: 93146, done.
Counting objects: 100% (93146/93146), done.
Delta compression using up to 12 threads
Compressing objects: 100% (28200/28200), done.
Writing objects: 100% (93146/93146), done.
Total 93146 (delta 67637), reused 88951 (delta 64013), pack-reused 0
Removing duplicate objects: 100% (256/256), done.
```

---

| 参考来源                                                     |
| ------------------------------------------------------------ |
| [git长时间未清理无法拉取代码](https://blog.csdn.net/fenfeidexiatian/article/details/95308119) |