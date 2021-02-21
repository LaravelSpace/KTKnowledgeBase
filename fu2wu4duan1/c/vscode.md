## 在VisualStudioCode中配置C语言的工作环境

### 安装扩展

菜单栏=>文件=>首选项=>扩展，可以在左侧打开扩展子窗口。

- 在扩展中安装中文语言包（搜索Chinese），可以把操作界面替换成中文环境。
- 在扩展中安装C/C++组件。
- 在扩展中安装CodeRunner组件。
- （可选）在扩展中安装Clang-Format代码格式化组件。

### 安装C/C++编译器

去[sourceforge](https://sourceforge.net/projects/mingw/)下载编译器，在Windows操作系统上可以使用MinGW。比如，MinGW-W64-GCC-8.1.0。网站上可以看到有很多版本，这里使用的是x86_64-8.1.0-posix-seh。下载下来一个压缩包，解压出来得到一个mingw64目录，移动到合适的位置。

然后打开我的电脑（右击）=>属性=>高级系统设置=>环境变量=>系统变量。修改系统变量中的Path变量，添加刚才解压出来的mingw64目录下的bin目录的路径。添加完成之后，打开命令行界面（cmd）输入`gcc -v`命令，如果成功的话可以看到下面的输出：

```
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=E:/C/mingw64/bin/../libexec/gcc/x86_64-w64-mingw32/8.1.0/lto-wrapper.exe
Target: x86_64-w64-mingw32
Configured with: ../../../src/gcc-8.1.0/configure --host=x86_64-w64-mingw32 --build=x86_64-w64-mingw32 --target=x86_64-w64-mingw32 --prefix=/mingw64 --with-sysroot=/c/mingw810/x86_64-810-posix-seh-rt_v6-rev0/mingw64 --enable-shared --enable-static --disable-multilib --enable-languages=c,c++,fortran,lto --enable-libstdcxx-time=yes --enable-threads=posix --enable-libgomp --enable-libatomic --enable-lto --enable-graphite --enable-checking=release --enable-fully-dynamic-string --enable-version-specific-runtime-libs --disable-libstdcxx-pch --disable-libstdcxx-debug --enable-bootstrap --disable-rpath --disable-win32-registry --disable-nls --disable-werror --disable-symvers --with-gnu-as --with-gnu-ld --with-arch=nocona --with-tune=core2 --with-libiconv --with-system-zlib --with-gmp=/c/mingw810/prerequisites/x86_64-w64-mingw32-static --with-mpfr=/c/mingw810/prerequisites/x86_64-w64-mingw32-static --with-mpc=/c/mingw810/prerequisites/x86_64-w64-mingw32-static --with-isl=/c/mingw810/prerequisites/x86_64-w64-mingw32-static --with-pkgversion='x86_64-posix-seh-rev0, Built by MinGW-W64 project' --with-bugurl=https://sourceforge.net/projects/mingw-w64 CFLAGS='-O2 -pipe -fno-ident -I/c/mingw810/x86_64-810-posix-seh-rt_v6-rev0/mingw64/opt/include -I/c/mingw810/prerequisites/x86_64-zlib-static/include -I/c/mingw810/prerequisites/x86_64-w64-mingw32-static/include' CXXFLAGS='-O2 -pipe -fno-ident -I/c/mingw810/x86_64-810-posix-seh-rt_v6-rev0/mingw64/opt/include -I/c/mingw810/prerequisites/x86_64-zlib-static/include -I/c/mingw810/prerequisites/x86_64-w64-mingw32-static/include' CPPFLAGS=' -I/c/mingw810/x86_64-810-posix-seh-rt_v6-rev0/mingw64/opt/include -I/c/mingw810/prerequisites/x86_64-zlib-static/include -I/c/mingw810/prerequisites/x86_64-w64-mingw32-static/include' LDFLAGS='-pipe -fno-ident -L/c/mingw810/x86_64-810-posix-seh-rt_v6-rev0/mingw64/opt/lib -L/c/mingw810/prerequisites/x86_64-zlib-static/lib -L/c/mingw810/prerequisites/x86_64-w64-mingw32-static/lib '
Thread model: posix
gcc version 8.1.0 (x86_64-posix-seh-rev0, Built by MinGW-W64 project)
```

### 修改VSCode调试配置文件

在扩展窗口中选择CodeRunner，点击右侧的设置图标，选择扩展设置。或者点击菜单栏=>首选项=>设置，进去之后，在上面的输入框中输入`@ext:formulahendry.code-runner`。这两个方式都可以找到CodeRunner的设置。找到之后，勾选Code-runner: Ignore Selection和Code-runner: Run In Terminal这两项。

### 配置C/C++任务文件

点击菜单栏=>终端=>配置任务，在出现的对话中框选择`C/C++ gcc.exe build active file`。这个操作会在根目录创建一个.vscode目录，然后创建一个tasks.json文件，文件的内容如下。

```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"type": "cppbuild",
			"label": "C/C++: gcc.exe build active file",
			"command": "E:\\C\\mingw64\\bin\\gcc.exe",
			"args": [
				"-g",
				"${file}",
				"-o",
				"${fileDirname}\\${fileBasenameNoExtension}.exe"
			],
			"options": {
				"cwd": "E:\\C\\mingw64\\bin"
			},
			"problemMatcher": [
				"$gcc"
			],
			"group": "build",
			"detail": "compiler: E:\\C\\mingw64\\bin\\gcc.exe"
		}
	]
}
```

在根目录创建一个hello.c文件，编写测试代码。

```c
#include <stdio.h>

int main()
{
    printf("hello world!");
    
    return 0;
}
```

点击菜单栏=>终端=>运行任务，在出现的对话中框选择C/C++ gcc.exe build active file。之后再窗口下方的终端子窗口会输出以下内容，并生成一个hello.exe可执行文件。使用`./hello`命令就可以在控制台执行这个可执行文件，输出`hello world！`。

```
> Executing task: C/C++: gcc.exe build active file <

Starting build...
Build finished successfully.

终端将被任务重用，按任意键关闭。
```

或者操作界面选中hello.c文件，然后点击右上角的RunCode按钮。这时终端子窗口会输出以下内容，相当于编译了hello.c文件，然后执行了生成的hello.exe可执行文件。

```
PS E:\GongZuoQu\CYangLi> cd "e:\GongZuoQu\CYangLi\" ; if ($?) { gcc hello.c -o hello } ; if ($?) { .\hello }
hello world!
```

结合上面的tasks.json文件，可以知道`"command": "E:\\C\\mingw64\\bin\\gcc.exe"`表示这里的gcc命令。args参数表示gcc命令的参数，其中`${file}`表示当前选中的文件，`${fileDirname}\\${fileBasenameNoExtension}.exe`表示输出的文件的路径以及文件名。`${fileDirname}`表示当前选中的文件的目录路径。

### 多文件编译

上面的方法可以解决单文件编译，但是不会进行多文件编译。当然，可以手动编写gcc命令执行多文件编译，不过这里也可以通过修改tasks.json文件实现多文件编译。

```json
"args": [
	"-g",
	"${fileDirname}\\hello.c",
	"${fileDirname}\\hello2.c",
	"-o",
	"${fileDirname}\\hello.exe"
],
```

这里需要修改args参数，原来的`${file}`只会编译当前选中的文件，所以我们可以在这里指定要编译的多个文件。然后点击菜单栏=>终端=>运行生成任务，在出现的对话中框选择C/C++ gcc.exe build active file。如果没有异常，终端子窗口会输出以下内容，同时生成hello.exe可执行文件。

```
> Executing task: C/C++: gcc.exe build active file <

Starting build...
Build finished successfully.

终端将被任务重用，按任意键关闭。
```

### 断点调试

首先备份launch.json文件的内容，然后删了launch.json文件（如果里面有别的配置，稍后再加进去）。

找一个源码文件，打上断点。点击菜单栏=>运行=>启动调试。在弹出的对话框中选择C++(GDB\LLDB)，然后再选择gcc.exe - 生成和调试活动文件。文件编译完成后，就会自动运行并进入调试状态。

这个操作还会创建一个launch.json文件。

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "gcc.exe - 生成和调试活动文件",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\${fileBasenameNoExtension}.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "E:\\C\\mingw64\\bin",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "E:\\C\\mingw64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "为 gdb 启用整齐打印",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: gcc.exe build active file"
        }
    ]
}
```

### 代码格式化

好像安装C/C++组件的时候会自动安装Clang-Format组件。在扩展窗口中选择C/C++，点击右侧的设置图标，选择扩展设置。或者点击菜单栏=>首选项=>设置，进去之后，在上面的输入框中输入`@ext:ms-vscode.cpptools`。这两个方式都可以找到C/C++的设置。

找到C_Cpp: Formatting配置项检查一下是不是clangFormat，这个是默认配置一般不会变。在检查一下C_Cpp: Clang_format_style配置项是不是file，设置为file表示格式化的时候会先寻找Clang-Format组件的格式化配置文件。

接着在项目根目录新建一个**.clang-format**文件，这个文件就是上面的格式化配置文件，详细的配置可以去Clang-Format的文档里去看，在扩展窗口组件的描述页面就可以点进去。

```
# 每行字符的限制，0表示没有限制
ColumnLimit: 0
# 缩进宽度
IndentWidth: 4
# 连续空行的最大数量
MaxEmptyLinesToKeep: 1
# 缩进case标签
IndentCaseLabels: true
```

配置完成就可以格式化代码了，默认快捷键是`shift+alt+f`，也可以右击选择格式化文档。