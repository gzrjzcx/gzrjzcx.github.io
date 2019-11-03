---
title: 'Gitalk: A plugin for static website'
date: 2019-11-03 15:59:20
categories: Tools
tags: Tools
---

# Gitalk：符合国情的评论插件

大家都知道`Disqus`使用起来很方便，但是在国内却不能正常使用。后面发现[Gitalk](https://github.com/gitalk/gitalk)这个利用Github上的issue来做评论模块的插件，感觉不错。不过在使用过程中还是碰到了不少坑，记录在此以供参考。

## 1. 基本使用

官方文档上写着使用方法， 看上去也是比较简单，但是对于从没有接触过html的人来说（比如我这个菜鸡），还是会碰到一些问题。

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.css">
<script src="https://cdn.jsdelivr.net/npm/gitalk@1/dist/gitalk.min.js"></script>
<div id="gitalk-container"></div>
<!--Then use the Javascript code below to generate the gitalk plugin:-->
const gitalk = new Gitalk({
  clientID: 'GitHub Application Client ID',
  clientSecret: 'GitHub Application Client Secret',
  repo: 'GitHub repo',
  owner: 'GitHub repo owner',
  admin: ['GitHub repo owner and collaborators, only these guys can initialize github issues'],
  id: location.pathname,      // Ensure uniqueness and length less than 50
  distractionFreeMode: false  // Facebook-like distraction free mode
})

gitalk.render('gitalk-container')
```

这是官网上的介绍，然而我对于`use the Javascript code below to generate the gitalk plugin`并没有什么概念，只是单纯的直接复制以上代码加入到生成的html文件中，结果自然是无论如何都无法出现评论模块。后面通过直接查看官方提供的[demo](https://gitalk.github.io/)的源代码才发现需要将这段代码标记为JavaScript：

```html
<script type="test/javascript">
const gitalk = new Gitalk({
  clientID: 'GitHub Application Client ID',
  clientSecret: 'GitHub Application Client Secret',
  repo: 'GitHub repo',
  owner: 'GitHub repo owner',
  admin: ['GitHub repo owner and collaborators, only these guys can initialize github issues'],
  id: location.pathname,      // Ensure uniqueness and length less than 50
  distractionFreeMode: false  // Facebook-like distraction free mode
})
gitalk.render('gitalk-container')
</script>
```

这样才能正确在Html中载入评论模块。如果实在不明白的话建议先去了解一下html的基本语法。

## 2. Error: Validation Failed

这是因为Github限制了issue的label长度不能超过50引起的问题，具体可以查看[here](https://github.com/gitalk/gitalk/issues/102)。在此只介绍[#115](https://github.com/gitalk/gitalk/issues/115)提出的解决方法，其核心思想就是用将label转为md5值：

1. 自己创建gist文件或者直接下载[此js文件](https://github.com/blueimp/JavaScript-MD5/blob/master/js/md5.min.js)。

2. 上传到自己的Github上，如果按照我之前[gitbook](https://www.hellscript.cc/Pointer2SSP/)上的设置，则应该上传到`src`分支上。

3. 在想要添加`Gitalk`的html文件中引入该js文件：

   ```html
   <script type="text/javascript" src="https://raw.githack.com/gzrjzcx/Pointer2SSP/src/md5.min.js"></script>
   ```

   **一定要注意嵌入js的时候将原来的`githubusercontent.com`改为`raw.githack.com`才能成功嵌入，不然会找不到此js文件。**

4. 将上述代码中的id一项加入md函数：

   ```html
   id: md5(location.pathname), 
   ```

## 3.自动添加Gitalk模块

上述手动在html中添加代码的方法显示并不方便，不可能每写一篇文章就加一段代码。在此，根据[here](https://juejin.im/post/5ca8881951882543e85f155d#heading-17)提出的方法，我们可以利用`tbfed-pagefooter`插件进行自动集成Gitalk模块（以Gitbook为例）。在Gitbook对于仓库目录下的`node_modules`文件夹中，存放着各种plugin的文件。找到其中`tbfed-pagefooter`文件夹下的`index.js`文件，编辑器打开：

```javascript
  hooks: {
    'page:before': function(page) {
      var _label = 'File Modify: ',
          _format = 'YYYY-MM-DD HH:mm:ss',
          _copy = 'powered by Gitbook'
      if(this.options.pluginsConfig['tbfed-pagefooter']) {
        _label = this.options.pluginsConfig['tbfed-pagefooter']['modify_label'] || _label;
        _format = this.options.pluginsConfig['tbfed-pagefooter']['modify_format'] || _format;

        var _c = this.options.pluginsConfig['tbfed-pagefooter']['copyright'];
        _copy = _c ? _c + ' all right reserved，' + _copy : _copy;
      }
      var _copy = '<span class="copyright">'+_copy+'</span>'
      var str = ' \n\n<footer class="page-footer">' + _copy +
        '<span class="footer-modification">' +
        _label +
        '\n{{file.mtime | date("' + _format +
        '")}}\n</span></footer>'
			// 主要修改此处, 在此处引入Gitalk和我们自己存储的gitalk_config文件
      str += '\n\n<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">'+
      '\n\n<script src="https://unpkg.com/gitalk@latest/dist/gitalk.min.js"></script>'+
      '\n\n<script type="text/javascript" src="https://raw.githack.com/gzrjzcx/Pointer2SSP/src/md5.min.js"></script>'+
      '\n\n<div id="gitalk-container"></div>'+
      '\n\n<script src="https://raw.githack.com/gzrjzcx/Pointer2SSP/src/gitalk_config.js"></script>';

      page.content = page.content + str;
      return page;
    }
  },
```

根据代码中的注释，我们只需要让`tbfed-pagefooter`在构建html的时候帮我们一块把Gitalk也给引入集成就好了。其中需要注意的是我们需要自己创建一个`gitalk_config`文件来保存gitalk的配置信息，并push到`src`分支上即可。例如，我的`gitalk_config`内容为：

```javascript
    var gitalk = new Gitalk({
        clientID: '891633df9540ec724578',
        clientSecret: '012103e75c027bbbe2b6687fb84be518d7bb9994',
        repo: 'Pointer2SSP',
        owner: 'gzrjzcx',
        admin: ['gzrjzcx'],
        id: md5(location.pathname),
    });
    gitalk.render('gitalk-container');
```

如果不明白，也可以直接去我的Github上查看源文件: [Pointer2SSP](https://github.com/gzrjzcx/Pointer2SSP)。

## 4. Error: 401(unauthrized)

这个问题主要出现的原因为在Github上的OAuth App上的配置中 Callback URL与Chrome中的redirect URL不匹配，有很多种情况，具体可以查看[Issue #183](https://github.com/gitalk/gitalk/issues/183)。

在此，我只介绍我碰到的情况。我是用了自己的域名，在GitHub上的OAuth App上配置的时候直接复制了自己域名粘贴过去，是
`http://www.hellscript.cc/`
后面在chrome中看发现redirect URL是
`https://www.hellscript.cc/`
二者不一致会验证失败。改一致就行了。

