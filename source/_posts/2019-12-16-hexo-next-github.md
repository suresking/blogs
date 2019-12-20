---
title: 用hexo（next主题）在github上发布自己的博客
date: 2019-12-16 20:49:31
tags:
 - hexo
 - github
---

**这篇是源码/pages分离的，留作参考吧。**

---

基础的东西不写了，网上一堆一堆的。

我本机nodejs、git都安装好了。（ https://hexo.io/ ）

建立`hexo20191215`目录

在目录下启动git bash，输入

```bash
npm install hexo-cli -g
hexo init
```

再输入

```bash
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

直接获取最新的next主题。

<!-- more -->

打开`_config.yml`文件，找theme，找到theme: landscape修改为theme: next

然后进入theme/next文件夹，查找`_config.yml`文件中的Scheme Settings修改scheme: Muse为scheme: Gemini

输入`hexo -g && hexo -s`，运行后就可以用`localhost：4000`先测试看看了。有了内容后用`hexo clean && hexo generate && hexo server`测试。

**主题配置文件**我修改了：footer下的since打开；copyright加上；beian（备案）添加；avatar的url修改为七牛云的avatar.jpg；links修改；social里面的GitHub修改；favicon:下面所有，原来的我给注释了，同时在`themes\next\source\images`下添加favicon.ico文件；

**Hexo主力配置文件**修改：Site项下面的内容；url项；

```
url: https://git.wxzvip.com/blog
root: /blog/
```

因为我是在GitHub新建了一个仓库，叫做blog，建好后里面新建一个readme.md文件，这个不重要，因为发布的时候就自动给删除了。但有至少一个文件，**才能打开blog的pages选项**，选master分支，同时打开https。

再修改最后的deploy: 

```bash
 type: git
 repo: git@github.com:suresking/blog.git
 branch: master
```

**安装发布插件**

`npm install hexo-deployer-git –save`

最后使用`hexo clean && hexo g && hexo d`发布。

**源代码另外在腾讯云开发平台上保存。`git add .`时出错**：

因为themes/next我是用git clone来的，所以它的目录下也有一个.git文件夹，有独立的git控制，这个要用到submodule，很麻烦好像。我找个简单的办法：去next目录查看隐藏文件，找到.git目录后删除；再回到hexo根目录下，运行

```bash
git rm -rf --cached themes/next
git add themes/next
```

好像就可以了。

**[git] warning: LF will be replaced by CRLF | fatal: CRLF would be replaced by LF**，然后就出现这么个提示，搜了搜，是因为Git的换行符检查功能，因为我只在Windows工作，并且这个库是我自己用，我采用第三种办法`git config --global core.autocrlf false`。搞定。

`hexo new filename`时，文件名不要带.md扩展名，它会自动加上的。

------

## 重装或在另一台机器操作时：

若有必要，安装hexo，命令：`npm install hexo-cli -g`

适当位置打开git bash，运行`$ git clone git@git.dev.tencent.com:suresking/blog.git`

然后进入blog目录，运行`npm install`，这条命令会自动安装node_modules目录及相关内容。

运行`hexo g && hexo d`即可。不过又出现上面的CRLF问题，解决方法同上。

## 其他修改

- `motion -> enable:`改为false，关闭动画效果。
- 主题config文件中找到`custom_file_path:`，去掉最后style的注释，然后在`"blog/source/"`目录下新增`"_data/styles.styl"`文件，加入一些自定的css。