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

### Nginx安装及配置

首先替换yum源，我用的centOS系统，自带有国外yum源，这里需要先替换为国内的。具体可以参照这里：[将Centos的yum源更换为国内的阿里云源](http://www.centoscn.com/CentOS/config/2015/0416/5189.html)。

安装完我天真的以为直接yum install nginx就可以，确实之前使用腾讯云这样可以的，但是巨多坑啊。后来找了好久找到这个方法，成功搞定：[Linux（CentOS）下，下载安装Nginx并配置](http://www.cnblogs.com/jtlgb/p/5809808.html)。

具体过程和上述博文差不多，但是这篇博客是16年写的，现在新版都升到1.11.10了，所以安装nginx前大家可以先去http://nginx.org/download/看下最新版本，将命令中的版本号手动改为最新版。

安装完添加系统服务下一步就是配置。我服务器放在[digitalocean](https://m.do.co/c/41a85b4d947a)，所以最开始是在这里找了官方教程[How To Set Up nginx Virtual Hosts (Server Blocks) on CentOS 6](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-virtual-hosts-server-blocks-on-centos-6?utm_source=Customerio&utm_medium=Email_Internal&utm_campaign=Email_CentOSDistroNginxWelcome&mkt_tok=eyJpIjoiWW1aaU5HVTNNamN3TmpreSIsInQiOiJicXkzZ3VNYmtLQXNuaEFvVTBvZWZcLzlock9BYWNOMyttcUR3QVEwdm55MGMrcWxlUnZiaDVqcVZoSWllSVJTaW9Xd3o5UjRKcVgrSjl5T08rTTNYa2FcL0lRdTBFdXc4SlBvdkh1SVVoTk13UnlOQ0MxZ3ZNenZ3M1F5Q3htRDFKIn0%3D) 。但是实际配置中却有不少坑。

上面的安装教程可以直接忽略，亲测失败，毕竟国外的东西，吃不消啊。直接看下方的step，前面简单明了，自己按需配置下路径放个测试用的index.html就可以，不喜欢用vi的可以用xftp来操作文件。

到step four，问题来了，`sudo vi /etc/nginx/conf.d/virtual.conf` 是把编辑器打开了，内容我也照着写好了，但是不让我保存这就很尴尬了。我退出来查了一下，发现根本没这个路径啊T_T。我进到nginx的安装路径里看了下nginx.conf（路径：/usr/local/nginx/conf），发现根本没有include，这就意味着即使你创建了virtual.conf，nginx也是找不到的。对比po主的virtual.conf和原装nginx.conf，发现这里已经对80端口进行监听了，所以干脆直接在nginx.conf上改。

首先把listen 80下面的server_name改成了我的域名，然后把下面location里的root路径改为../../../var/www/example.com/public_html/（原来这里是html目录，可以在nginx里找到html目录和conf目录在同一级，这说明root指向的目录是nginx.conf的上一级目录，而这里找到var路径需要退到根目录去，所以加了三个../）。保存，`service nginx reload` 重启nginx，即可打开浏览器输入自己的域名看到效果了。