# Git与Hugo学习笔记


<!-- ![mywife](https://pic.imgdb.cn/item/66c9cb24d9c307b7e9591480.jpg) -->

本文介绍了个人博客的搭建过程，从安装hugo、git、go到主题下载、文章发布、github设置个人页的过程。

<!--more-->

这里说一下如何如何借助git与hugo制作自己的博客

## 1、三件套下载

下载[hugo](https://github.com/gohugoio/hugo/releases),然后放到某个目录，把目录假如系统环境变量，在cmd输入`hugo version`，~~如果没有报错就说明没有错。~~

下载[git](https://git-scm.com/downloads),安装，一路默认即可，使用`git version`检验安装是否成功。

下载[go](https://go.p2hp.com/doc/install),虽然hugo的运行用到了go，但是hugo的使用不需要理解go语言的语法。顺带一提之后我应该会看看go语言，也许会写点什么东西。同样使用`go version`检验安装是否成功。

## 2、文件夹组织

新建项目文件夹，然后在文件夹里面输入命令,可以用`tree /F`查看当前目录所有文件，hugo的new似乎不需要再content文件夹下运行。
```
  hugo new site hugo-demo
  cd hugo-demo
  hugo new posts/my-first-post.md
```

然后就是从github克隆主题,在themes文件夹里面，使用命令，不需要初始化，git会自己建文件夹
```
  git clone https://gitee.com/archiguru/LoveIt.git
```

在my-first-post.md里面写入
```
  # halo Hugo

  it's my first hugo post...
```

然后打开config.toml,写入
```
  baseURL = "http://example.org/"

  # 更改使用 Hugo 构建网站时使用的默认主题
  theme = "LoveIt"

  # 网站标题
  title = "Dengyunzhe's Site"

  # 网站语言, 仅在这里 CN 大写 ["en", "zh-CN", "fr", "pl", ...]
  languageCode = "zh-CN"
  # 语言名称 ["English", "简体中文", "Français", "Polski", ...]
  languageName = "简体中文"
  # 是否包括中日韩文字
  hasCJKLanguage = true

  # 作者配置
  [author]
    name = "Dengyunzhe"
    email = "dengyunzhe233@outlook.com"


  # 菜单配置
  [menu]
    [[menu.main]]
      weight = 1
      identifier = "posts"
      # 你可以在名称 (允许 HTML 格式) 之前添加其他信息, 例如图标
      pre = ""
      # 你可以在名称 (允许 HTML 格式) 之后添加其他信息, 例如图标
      post = ""
      name = "文章"
      url = "/posts/"
      # 当你将鼠标悬停在此菜单链接上时, 将显示的标题
      title = ""

  # Hugo 解析文档的配置
  [markup]
    # 语法高亮设置 (https://gohugo.io/content-management/syntax-highlighting)
    [markup.highlight]
      # false 是必要的设置 (https://github.com/dillonzq/LoveIt/issues/158)
      noClasses = false

```
## 3、发布及后续推送

使用本地预览代码`hugo server -D`,后面的参数表示加载草稿推文。编辑完成后，退出服务器，在文件夹里输入`hugo`,然后在public文件夹里使用git。

git的配置建议使用ssh，https和http的链接一直不太好，如果是克隆不好的话，可以手动删除git的代理，然后这里讲一下git的大致思路

当在某个文件夹打开终端后，使用命令对其进行初始化，
```
  git init
  git remote add origin git@github.com:username/repository.git
  git add .
  git commit -m "hugo"
  git push -u origin master
```

第一个命令会建立一个`.git`的配置文件夹，第二个会链接线上与本地的仓库，add . 会把本地所有文件移入更改的空间（记录改动），commit后面是记录更改的备注，然后就是push

在第一次信息设置完成之后，下一次更新网站可以这么做，
```
  git add -A
  git commit -m "update-hugo"
  git push -f origin master
```

下一篇文章应该会讨论一下我之前做ViT的心得。
