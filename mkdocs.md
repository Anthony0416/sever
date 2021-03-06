## MD文档快速构建私人博客

博主没事儿买了服务器域名打算建个个人网站，写写博客什么的。因为博主平时喜欢用md写东西，所以手头的博客也打算拿md写，方便快捷。但是管理大堆的md文档就很头疼了，所以一直想找一个方便的维护md文档的东西，最好能直接把我的md文档直接生成html，还能省去博客的维护时间。

然后让我找到这个神器MkDocs 顺便附上[中文文档](http://markdown-docs-zh.readthedocs.io/zh_CN/latest/) 。

官方列出的  **任意托管**  **大量主题**   **即时预览**   **易于配置**   **交叉索引**   这些优点让我心动不已，虽然项目还在开发阶段，但是还是忍不住上手试试。

### 安装

首先是安装Python和pip，MkDocs项目基于Python，所以首先要安装Python，我直接去[Python官网](https://www.python.org/)下载了最新windows安装包直接安装。现在新版Python都会附带安装pip，所以这里不需要安装。可以键入`pip -V` 来检查pip版本确认一下。

然后是使用pip安装MkDocs  `pip install mkdocs`  ，提示找不到合适的版本？查了一下有人说是墙的原因，开了vpn，找到四个包，但是连接超时继续报错。无奈继续找方法，最后找到了这个贴 [pip安装python库总是下载超时，有什么解决方法吗？](https://segmentfault.com/q/1010000000162410) 前面的不用看了，之前的源基本都不是https不能使用了，直接跳到后面。我使用的是豆瓣的镜像 `pip install -i https://pypi.doubanio.com/simple/ mkdocs`  很快就装上了。

使用`mkdocs help` 命令可以快速确认是否安装成功。

### 配置

按照官方教程，使用如下简单两个命令就可以快速在当前目录创建出一个项目：

```
$ mkdocs new my-project
$ cd my-project
```

执行  `$ mkdocs serve`  ，在浏览器打开 [http://127.0.0.1:8000/](http://127.0.0.1:8000/) ,就能快速预览当前项目。

然后官方意思是在项目的 mkdocs.yml 中编辑配置，但实际我发现直接在 docs 文件夹中新建文件夹，每一个文件夹都会成为导航栏的一个菜单，而在文件夹中放置的*.md文档则会成为该菜单的二级菜单。MkDocs还十分贴心做了检测，如果某个文件夹中没有md文档，那这个菜单就不会存在于导航栏中，避免了空导航的尴尬。

而在 mkdocs.yml 中只需要修改标题和主题即可，十分方便快捷。

在运行  `$ mkdocs build`  后MkDocs会在当前项目路径中生成一个 site 文件夹，把其中的文件全部拷入服务器即可直接发布上线。

### 注意

1. docs文件夹根目录index.md不能删除，删除之后会报错；
2. 项目中的index.md在创建菜单时（不论等级）都会被重命名为Home；
3. md文档的H5字体在页面中效果等同于普通段落，想要突出H5字体要么手动修改主题base.css，要么换用更大的标题（H1~H4）;
4. 项目中可以出现中文路径和中文文件名，但是linux服务器及需要做对应配置才能正确找到路径。linux下nginx中文路径配置可以参考我的另一篇博文: [Nginx安装及配置](http://blog.chenshiwen.top/%E5%BB%BA%E7%AB%99%E7%9B%B8%E5%85%B3/Nginx%E5%AE%89%E8%A3%85%E4%B8%8E%E9%85%8D%E7%BD%AE/) 。