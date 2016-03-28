# 那些年使用npm进行依赖管理所踩的坑

## 废话

公司的项目用上了node来做前后端分离，相应的自然离不开使用npm来对依赖的第三方包进行管理。

npm使用的语义化版本号管理想法很好，使用 `npm install --save` 安装依赖包时，自动加上的 `^x.x.x` 的版本号，看上去似乎也很有道理。

然而现实总是那么残酷，在经历了多次：在开发环境还跑的好好的项目，提测和上线时就挂了的情况后（此处省略一万字），
我终于意识到，**版本号还是固定的好啊！！！**

当然，依赖包的版本号应该怎么固定，还是有讲究的。
去掉 `^x.x.x` 前面的 `^` 使用一个固定的版本固然可以，但通过 `npm shrinkwrap` 来固定，我觉得更加合适。

## 使用 npm-shrinkwrap

关于 npm-shrinkwrap 的介绍就不多说了，网上一搜一大堆。
反正就是生成一个版本信息固定的 npm-shrinkwrap.json 文件，然后 npm 在安装依赖包的时候会首先参考 npm-shrinkwrap.json 文件的设置。

本以为万事ok，没想到却在生成 npm-shrinkwrap.json 文件的时候踩了坑……

## 问题

项目里面有 dependencies 包 T，T 又 dependencies 包 C。然而在服务器上安装好，运行的时候却偏偏提示少了 C。
于是打开 npm-shrinkwrap.json 一看，里面的 T 依赖的 C 不见了！WTF！

经过一番排查，真想终于水落石出。原来项目有 devDependencies C。
开发的时候 npm insatll，T 里面的 C 就被省略掉了。
原本贴心的处理，却在 npm-shrinkwrap 成了一个大坑，**因为 npm-shrinkwrap 默认是根据当前已安装的 dependencies 的目录结构来生成的。**

## 解决

知道了问题的原因，解决起来自然就很轻松了。
我们在需要 npm-shrinkwrap 的时候：

**方法1：清空 node_modules ，然后 `npm install --production` ，然后再 `npm shrinkwrap`**

方法2：`npm prune --production` ，然后再 `npm shrinkwrap --dev`

## 后话

**npm 使用的版本为 v2。** v3 似乎因为会去计算依赖，没有此问题了。
另外，一个心得是，对依赖的管理，要使用 npm 提供的方法来管理。
比如，我想去掉已安装的 devDependencies 包，应该使用 `npm prune --production` 而不是自己手动的使用 `npm uninstall` 。