### linux node环境安装配置

首先定位到需要存放node文件的目录，使用wget下载node

```
wget https://nodejs.org/dist/v6.10.0/node-v6.10.0-linux-x64.tar.xz
```

最先版本下载链接可以在官网找。

下载完使用ls命令查看当前文件会多一个node-v6.10.0-linux-x64.tar.xz的文件，使用如下两个命令解压缩

```
xz -d node-v6.10.0-linux-x64.tar.xz
tar -xvf node-v6.10.0-linux-x64.tar
```

此时使用ls查看该目录下有一个node-v6.10.0-linux-x64.tar的文件和一个node-v6.10.0-linux-x64的目录。cd进node-v6.10.0-linux-x64/bin，ls查看当前目录内有一个node和一个npm，此时键入./node -v ，如正常显示版本号即可。

不过此时node是局部作用，我们需要配置成全局，执行以下两个命令：

```
ln -s /software/node-v6.10.0-linux-x64/bin/node /usr/local/bin/node
ln -s /software/node-v6.10.0-linux-x64/bin/npm /usr/local/bin/npm
```

> /software/node-v6.10.0-linux-x64/bin/npm  为我的node存放路径，以实际为准

完成后此时进入任意目录键入node -v，控制台打印出node版本号及为配置成功。