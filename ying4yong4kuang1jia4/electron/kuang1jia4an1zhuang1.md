## 框架安装

为你的新Electron应用创建一个新的空文件夹。 打开你的命令行工具，然后从该文件夹运行`npm init`

npm 会帮助你创建一个基本的 `package.json` 文件。 其中的 `main` 字段所表示的脚本为应用的启动脚本，它将会在主进程中执行。 

如果 `main` 字段没有在 `package.json` 中出现，那么 Electron 将会尝试加载 `index.js` 文件（就像 Node.js 自身那样）。

安装`electron`。 我们推荐的安装方法是把它作为您 app 中的开发依赖项，这使您可以在不同的 app 中使用不同的 Electron 版本。 在您的app所在文件夹中运行下面的命令：

```sh
npm install --save-dev electron
```

正常情况下的输出

```
$ npm install --save-dev electron

> core-js@3.6.5 postinstall E:\Workspace\TuPianGuanLiQi\node_modules\core-js
> node -e "try{require('./postinstall')}catch(e){}"

Thank you for using core-js ( https://github.com/zloirock/core-js ) for polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open Collective or Patreon:
> https://opencollective.com/core-js
> https://www.patreon.com/zloirock

Also, the author of core-js ( https://github.com/zloirock ) is looking for a good job -)


> electron@10.1.0 postinstall E:\Workspace\TuPianGuanLiQi\node_modules\electron
> node install.js

npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN tupianguanliqi@0.1.0 No description
npm WARN tupianguanliqi@0.1.0 No repository field.

+ electron@10.1.0
added 87 packages from 97 contributors in 107.859s

6 packages are looking for funding
  run `npm fund` for details
```

检查下载源

```
$ npm config get registry
https://registry.npmjs.org/
```

修改成阿里云的

```
$ npm config set registry https://registry.npm.taobao.org
```

安装卡住如下

```
$ npm install --save-dev electron

> core-js@3.6.5 postinstall E:\Workspace\TuPianGuanLiQi\node_modules\core-js
> node -e "try{require('./postinstall')}catch(e){}"

Thank you for using core-js ( https://github.com/zloirock/core-js ) for polyfilling JavaScript standard library!

The project needs your help! Please consider supporting of core-js on Open Collective or Patreon:
> https://opencollective.com/core-js
> https://www.patreon.com/zloirock

Also, the author of core-js ( https://github.com/zloirock ) is looking for a good job -)


> electron@10.1.0 postinstall E:\Workspace\TuPianGuanLiQi\node_modules\electron
> node install.js

```

卡在这里不动 $ node install.js

这句命令的install.js是electron这个包里的，里面的下载是依赖于electron-download这个模块。
 在github上面，electron-download这个包里有如下标注：

```
You can set the ELECTRON_MIRROR or NPM_CONFIG_ELECTRON_MIRROR environment variable or mirror opt variable to use a custom base URL for grabbing Electron zips. The same pattern applies to ELECTRON_CUSTOM_DIR and ELECTRON_CUSTOM_FILENAME:

## Electron Mirror of China
ELECTRON_MIRROR="https://npm.taobao.org/mirrors/electron/"
## or for a local mirror
ELECTRON_MIRROR="https://10.1.2.105/"
ELECTRON_CUSTOM_DIR="our/internal/filePath"
You can set ELECTRON_MIRROR in .npmrc as well, using the lowercase name:

electron_mirror=https://10.1.2.105/
```

所以解决的方法就是在~/.npmrc里做如下设置，

```
electron_mirror="https://npm.taobao.org/mirrors/electron/"
```

在终端命令行输入`npm config ls -l`查看.npmrc配置路径（如果没有，则执行`npm config set registry https://registry.npm.taobao.org`就会生成该文件）

直接把上面那个加到这个文件里就行

nodeIntergration保证renderer.js可以使用require语句

enableRemoteModule保证renderer.js可以可以正常require('electron').remote，**此选项默认关闭且网上很多资料没有提到**