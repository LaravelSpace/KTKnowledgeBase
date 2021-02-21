## PHP的安装和配置

### 在Windows10操作系统中安装PHP

#### 安装和配置

这里使用的是php-7.1.3版本。首先去官网下一个7.1.3版本的线程安全的压缩包php-7.1.3-Win32-VC14-x64.rar。解压压缩包后放到合适的位置。解压之后在系统环境变量PATH中添加PHP所在的路径：`D:\php;`和`D:\php\ext;`。前者是PHP的根目录，后者是PHP扩展的位置。配置完PATH系统变量之后，我们需要配置php.ini文件。

打开PHP根目录，复制一个php.ini-development，修改为php.ini。打开php.ini，找到extension_dir这一行，修改为：`extension_dir = "你的php根目录\ext"`。这个操作是指定PHP扩展的路径，必须指定扩展路径，否则php7启动不了。

配置完ext扩展目录之后，就可以在Windows10的CMD命令行输入`php -v`命令查看PHP版本信息了，也可以校验这个步骤是否成功。如果仍然不成功，说明你的PHP路径没有添加到环境变量中或者你的环境变量有旧的PHP版本正在使用。

#### 配置PHP扩展

这里我用MySQL扩展举例。在php.ini中找到相应的扩展。比如，MySQL的扩展在php.ini文件中对应`extension=php_mysqli.dll`这一行，如果前面有分号说明这行代码是被注释的状态，去掉最前面的分号，启用扩展。
