### CentOS Nginx安装及配置

#### nginx安装

首先替换yum源，我用的centOS系统，自带有国外yum源，这里需要先替换为国内的。具体可以参照这里：[将Centos的yum源更换为国内的阿里云源](http://www.centoscn.com/CentOS/config/2015/0416/5189.html)。

安装完我天真的以为直接yum install nginx就可以，确实之前使用腾讯云这样可以的，但是巨多坑啊。后来找了好久找到这个方法，成功搞定：[Linux（CentOS）下，下载安装Nginx并配置](http://www.cnblogs.com/jtlgb/p/5809808.html)。

具体过程和上述博文差不多，但是这篇博客是16年写的，现在新版都升到1.11.10了，所以安装nginx前大家可以先去http://nginx.org/download/看下最新版本，将命令中的版本号手动改为最新版，其他照该博客一步步安装设置系统服务即可。

#### nginx基础配置

安装完下一步就是配置。我服务器放在[digitalocean](https://m.do.co/c/41a85b4d947a)，所以最开始是在这里找了官方教程[How To Set Up nginx Virtual Hosts (Server Blocks) on CentOS 6](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-virtual-hosts-server-blocks-on-centos-6?utm_source=Customerio&utm_medium=Email_Internal&utm_campaign=Email_CentOSDistroNginxWelcome&mkt_tok=eyJpIjoiWW1aaU5HVTNNamN3TmpreSIsInQiOiJicXkzZ3VNYmtLQXNuaEFvVTBvZWZcLzlock9BYWNOMyttcUR3QVEwdm55MGMrcWxlUnZiaDVqcVZoSWllSVJTaW9Xd3o5UjRKcVgrSjl5T08rTTNYa2FcL0lRdTBFdXc4SlBvdkh1SVVoTk13UnlOQ0MxZ3ZNenZ3M1F5Q3htRDFKIn0%3D) 。但是实际配置中却有不少坑。

上面的安装教程可以直接忽略，亲测失败，毕竟国外的东西，吃不消啊。直接看下方的step，前面简单明了，自己按需配置下路径放个测试用的index.html就可以，不喜欢用vi的可以用xftp来操作文件。

到step four，问题来了，`sudo vi /etc/nginx/conf.d/virtual.conf` 是把编辑器打开了，内容我也照着写好了，但是不让我保存这就很尴尬了。我退出来查了一下，发现根本没这个路径啊T_T。我进到nginx的安装路径里看了下nginx.conf（路径：/usr/local/nginx/conf），发现根本没有include，这就意味着即使你创建了virtual.conf，nginx也是找不到的。对比po主的virtual.conf和原装nginx.conf，发现这里已经对80端口进行监听了，所以干脆直接在nginx.conf上改。

首先把listen 80下面的server_name改成了我的域名，然后把下面location里的root路径改为../../../var/www/example.com/public_html/（原来这里是html目录，可以在nginx里找到html目录和conf目录在同一级，这说明root指向的目录是nginx.conf的上一级目录，而这里找到var路径需要退到根目录去，所以加了三个../）。保存，`service nginx reload` **重启nginx服务**，即可打开浏览器输入自己的域名看到效果了（当然在此之前确保你的域名配置了域名解析）。

tip：linux下找某个文件可以问where is：`# whereis(空格)文件名或者路径名`

#### nginx多端口配置

有时候我们服务器上有一个域名，但是我们有好几个项目，想通过这一个域名访问到所有项目的index，这里就需要用到端口配置。依然是找到nginx.conf，在最下面添加一条 server，如下所示：

```
    server {
        listen       8000;
        server_name  example.com:8000;

        location / {
            root   ../../../var/www/project/;
            index  index.html index.htm;
        }
    }
```

保存退出，**重启nginx服务**即可。此时你访问example.com和example.com:8000会看到不同的页面。

tip：多端口配置不需要做域名解析。

#### nginx二级域名配置

一个域名多个端口，访问的时候必须敲入端口才能访问到正确的网页，这样很不方便也不美观。这时可以用到二级域名来处理。

和之前配置类似，依然监听80端口，server_name为设置过解析的二级域名，location按需修改。

```
    server {
        listen       80;
        server_name  project.example.com;

        location / {
            root   ../../../var/www/project/;
            index  index.html index.htm;
        }
    }
```

保存退出，**重启nginx服务！**

tip:二级域名一定要设置解析。以project.example.com为例，在example.com的域名下添加一条A记录，主机记录为project，记录值为服务器ip，保存等一下就看到结果了。

#### 坑：中文路径问题

在实际使用中，因为业务需要，某个项目中存在中文路径和中文文件名，然后在访问的时候nginx就报了404  T_T  。

回过头来仔细研究了我的nginx配置，感觉没有问题啊。然后尝试把项目中中文路径改为英文，可以访问，看来问题出在编码上。

进一步查看请求发现，浏览器在请求中文路径时虽然地址栏显示中文，但实际发出的请求地址为utf-8的字符集，问题的第一步是先让nginx识别utf-8，这个很简单，在sever_name下添加一条`charset  utf-8;` 即可，保存重启nginx，404，失败。

冷静下来仔细想想，就算nginx拿到了utf-8的地址，但linux中的实际路径字符编码依然为GBK，依然找不到路径，所以现在问题变成了如何让nginx拿着utf-8的地址去找到GBK的路径。

（我觉得我给自己挖了个大坑）

然后开始查各种解决方法，最常见的应该就是使用convmv转一下，把中文的文件夹和文件名转为utf-8。但是这样的话我每次更新一下网站就要重新转一下？这也太麻烦了吧。

然后看到有po主提到了如果使用securecrt上传文件的话，可以把securecrt的编码修改为utf-8。我看了下自己用的xftp，在 ‘属性=> 选项’ 中找到了一个 ‘使用utf-8编码’ 的选框，勾选，服务器的中文文件夹变成了乱码，删除这些文件重新上传，打开浏览器刷新，已经可以正确访问到了。问题圆满解决。



