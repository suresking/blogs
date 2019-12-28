---
title: Hexo（nexT主题）Github一体化部署方案
date: 2019-12-19 18:02:11
tags:
- hexo
- github
---

## 一、基础安装

1. `Git`安装
2. `NodeJs`安装，二者都可以从淘宝镜像下载。
3. `Hexo`安装
   `$ npm install -g hexo-cli`

<!-- more -->

## 二、建站

1. 相应目录下运行Git Bash，输入

   ```
   $ hexo init <folder>
   $ cd <folder>
   $ npm install
   ```

   进行新建；如果是重装或在另一台电脑，从`Github`克隆回来之后，只在博客目录下运行第三句即可。

2. 安装主题有两种方法：2.1.到`https://github.com/theme-next/hexo-theme-next`下载最新的release包，解压到theme目录下，改名为next；2.2.直接克隆`git clone https://github.com/theme-next/hexo-theme-next themes/next`，克隆完成后到next目录下把隐藏的`.git`文件夹删除。

## 三、覆盖主题配置
通常情况下，`Hexo` 主题是一个独立的项目，并拥有一个独立的` _config.yml` 配置文件。
你可以在站点的` _config.yml `配置文件中配置你的主题，这样你就不需要 fork 一份主题并维护主题独立的配置文件。

以下是一个覆盖主题配置的例子：

```
# _config.yml
theme_config:
  bio: "My awesome bio"
```

```
# themes/my-theme/_config.yml
bio: "Some generic bio"
logo: "a-cool-image.png"
```

最终主题配置的输出是：

```
{
  bio: "My awesome bio",
  logo: "a-cool-image.png"
}
```

这样直接把需要修改的内容拷贝到`Hexo`的`_config.yml`修改就行了，主题内部的配置文件仍在一边不需要管他。

## 四、部署到Github

**摘录官网说明**：（ https://hexo.io/zh-cn/docs/github-pages ）

在本教程中，我们将会使用 `Travis CI `将 `Hexo` 博客部署到 `GitHub Pages` 上。`Travis CI` 对于开源` repository `是免费的，但是这意味着你的站点文件将会是公开的。如果你希望你的站点文件不被公开，请直接前往本文 `Private repository `部分。

1. 新建一个 `repository`。如果你希望你的站点能通过 `<你的 GitHub 用户名>.github.io `域名访问，你的 `repository` 应该直接命名为` <你的 GitHub 用户名>.github.io`。
2. 将你的 `Hexo` 站点文件夹推送到` repository` 中。默认情况下 public 目录将不会被推送到 `repository` 中，你应该检查` .gitignore `文件中是否包含` public` 一行，如果没有请加上。
3. 将 `Travis CI` 添加到你的 `GitHub` 账户中。
4. 前往` GitHub` 的 `Applications settings`，配置 `Travis CI `权限，使其能够访问你的 repository。
5. 你应该会被重定向到 `Travis CI `的页面。如果没有，请 手动前往。
6. 在浏览器新建一个标签页，前往` GitHub `新建 `Personal Access Token`，只勾选` repo `的权限并生成一个新的` Token`。`Token `生成后请复制并保存好。
7. 回到 `Travis CI`，前往你的` repository` 的设置页面，在 `Environment Variables `下新建一个环境变量，`Name` 为 `GH_TOKEN`，`Value `为刚才你在 `GitHub` 生成的 `Token`。确保` DISPLAY VALUE IN BUILD LOG `保持 不被勾选 避免你的 `Token `泄漏。点击` Add `保存。
8. 在你的 `Hexo` 站点文件夹中新建一个` .travis.yml `文件：

```
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
script:
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public
```

9. 将` .travis.yml` 推送到 `repository` 中。`Travis CI `应该会自动开始运行，并将生成的文件推送到同一 `repository` 下的 `gh-pages` 分支下
10. 在 `GitHub` 中前往你的` repository` 的设置页面，修改` GitHub Pages `的部署分支为 `gh-pages`。
11. 前往 `https://<你的 GitHub 用户名>.github.io `查看你的站点是否可以访问。这可能需要一些时间。

**Project page**
如果你更希望你的站点部署在 `<你的 GitHub 用户名>.github.io `的子目录中，你的` repository` 需要直接命名为子目录的名字，这样你的站点可以通过` https://<你的 GitHub 用户名>.github.io/<repository 的名字>` 访问。你需要检查你的` Hexo `配置文件，将` url `修改为` https://<你的 GitHub 用户名>.github.io/<repository 的名字>`、将` root` 的值修改为 `/<repository 的名字>/`

## 五、后续

1. 打开tags标签。`tags: /tags/ || tags`前面的#号去掉。然后执行`$ hexo new page "tags"`，修改`"blogs/source/tags/index.md"`文件内容为：

   ```
   title: 标签云
   date: 2019-12-20 08:48:17
   type: "tags"
   comments: false
   ```

   最后将`tag_icon: false`改成`true`即可。

2. 打开版权信息。找到`creative_commons:`将两个`false`改成`true`即可。