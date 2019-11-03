---
title: 'Travis: Automatically build and deploy to github pages'
date: 2019-11-03 11:29:19
categories:Tools
tags:Tools
---

# 1. Github仓库branch设置

---

要实现自动将generated public文件夹自动push到相应分支上，我们需要在GitHub的仓库上使用两个分支：

- `src`用于存储Hexo相关文件和我们所写的blog源文件（即md文件）。
- `master/gh-pages`用于存储构建出来的public文件夹里面的文件（即生成的静态网站相关文件）。

这里需要说明的是，如果你使用的是默认的GitHub pages（即<user>.github.io），或者绑定了自己的域名（例如我的hellscript.cc)，想要让blog的地址就是自己的域名地址，而不是自己域名地址下的一个子文件夹，那么必须使用`master`作为GitHub pages的source，详情看[here](https://help.github.com/en/github/working-with-github-pages/about-github-pages#publishing-sources-for-github-pages-sites)。例如，我想要我的博客直接部署到我的域名hellscript.cc下，那么我必须在对应的repo（我的是gzrjzcx.github.io）中使用`master`作为source。但是对于我的gitbook，部署在hellscript.cc/Pointer2SSP/下，即hellscript.cc的一个子文件夹，只需要在gitbook对应的repo中新建一个`gh-pages`分支即可部署，该分支则是部署source。

因此，我们最终需要实现的目标是：

1. 在本地写md文件并push到`src`分支。
2. Travis自动克隆`src`并在服务器上生成静态网站相关文件（即public文件夹），并push到`master`分支。

### Hexo所需文件解释

在这里有必要说明一下Hexo构建静态网站所需要的文件，也即是我们需要在`src`分支上存储什么文件。根据Hexo的[官方文档](https://hexo.io/zh-cn/docs/)。安装好Hexo后，初始化Hexo：

```shell
$ hexo init blog #将初始化的文件目录放到blog的根目录下
```

上述init命令会将`blog`作为Hexo的根目录，并且生成如下目录结构：

```reStructuredText
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes

```

这是Hexo官网列出的目录，然而实际上如果需要成功build，还需要`node_modules`和`themes`两个文件夹。其中`node_modules`主要是各种dependencies, `themes`则主要是所用到的主题文件。其中，`node_modules`文件夹被官方写入了`.gitignore`，也就是说并不会被git记录，所以我们在后面Travis的配置文件中需要install dependencies。至于`themes`，因为大部分使用的主题也都是从别人repo上克隆下来的，所以在此我采用了submodule的方法。以常用的Next主题为例，具体操作方法为：

1. fork [next](https://github.com/theme-next/hexo-theme-next)主题repo到自己的GitHub上。

2. 在`blog`目录下添加submodule：

   ```shel
   $ git submodule add https://github.com/theme-next/hexo-theme-next next
   $ git remote add upstream https://github.com/theme-next/hexo-theme-next
   ```

3. 将fork到我们自己GitHub上的repo添加为submodule后，则可以按照我们自己的想法修改配置并push到fork过来的仓库。同时，将fork过来的仓库设置upstream为原仓库，这样原仓库有更新的话我们也可以获取到更新。具体的更新方法在此不详细说明。将`themes`添加为submodule后，Travis在clone的时候则会自动采用`recursive`的方式将submodule一起克隆，所以`themes`文件夹也会被克隆到`src`下，最终build静态网站的所需文件全部克隆到`src`下，可以保证build的成功。

另外`public`文件夹则是Hexo根据我们写好的md文章构建出来的部署静态网站相关文件夹，只需要push其中内容到`master`分支上就行，并不需要push到`src`上进行存储。

# 2. 配置Travis

---

Travis是Github提供的一个持续集成服务，说白了就是每次当push文件到GitHub仓库的时候，Travis都会被调用。你可以用它来进行测试，例如C++的编译等等。在此，我们利用它来进行自动部署，也就是说让它帮我们执行：

```shell
$ hexo g -d
```

除了Hexo，我的[Gitbook](https://www.hellscript.cc/Pointer2SSP/)也配置了Travi来进行自动部署，教程就在首页。另外，在此还记录下使用评论插件`Gitalk`踩过的坑。

首先，我们需要正确配置Travis才能够使用Travis提供的服务。这里需要注意的是一旦Travis开启服务，`只需要将相关.travis.yml文件push到相应分支即可以实现CI`。具体配置分为以下3个步骤：

1. 给新建的仓库开启`Travis`服务，这篇博客很详细[here](https://www.jianshu.com/p/3b8d86f25ee2)。
2. 因为我们需要push新生成的文件到仓库，所以需要生成GitHub的security token来获取更改仓库的权限[here](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)。
3. 设置好token后则可以给仓库[配置`yml`文件](https://docs.travis-ci.com/user/deployment/pages/)。注意一旦开启了Travis服务，只需要将yml文件上传到仓库则会自动激活Travis的服务。我自己用的yml文件可以在下方代码：

```yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - src # build src branch only
install:
  - npm install -g hexo-cli
script:
  - cd blog
  - npm install # install dependencies, becaseu git ignore node-modules directory
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $TOKEN # generated github secure token
  keep-history: true
  on:
    branch: src
  local-dir: ./blog/public
  target_branch: master
```

Travis配置完毕后，只需要在本地写好md文件，push到`src`分支，Travis则会自己在虚拟机上克隆我们的仓库，包括`themes`的子模块，以及安装build所需的依赖，最后generate静态网站，并push到`master`分支上进行部署。另外，如果想使用评论模块，可以查看[here]()来查看教程。