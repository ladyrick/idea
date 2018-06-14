---
title: 使用travis自动部署hexo博客到github pages
date: 2018-06-14 13:08:14
tags:
---

之前的博客一直用的是wordpress，因为hexo的话，每次还得手动运行`hexo generate`，然后把public文件夹复制到腾讯云上。如果是使用github pages的话，也需要每次把public文件夹push上去。就感觉很繁琐，觉得还不如用wordpress，在线写文章。

后来，越来越觉得在线些文章是个很蛋疼的事情。而且，wordpress使用了数据库来存储文章，就感觉很不优雅。于是开始考虑换回hexo。

<!--more-->

关于hexo部署繁琐的问题，最近接触了持续集成(CI, Continuous integration)，感觉可以用于自动部署，于是研究了一下，成功了。

目前使用的持续集成工具是[travis](https://travis-ci.org/)，使用github pages托管静态页面。现在的效果是，只要往github上一推送，就可以自动generate并将public文件夹push到gh-pages分支，实现github pages的自动更新。

# 原理

持续集成的原理是，每次检测到github上的提交，就运行一些预设的命令。
这些命令定义在仓库根目录下的`.travis.yml`文件中。
因此，只要合理配置这些命令，就可以让travis帮你完成generate和deploy的工作。

另一个需要解决的问题是，要让travis推送到你的github仓库，需要它有读写权限。
但是，我们总不能把github的密码写入配置文件中去吧？更不能把ssh密钥写入配置文件中。
好在，travis提供了加密功能，可以加密一个字符串，然后把加密的字符串写入配置文件中。这样就只有travis和你自己知道原始字符串的内容了。

另外，要让travis拥有读写仓库的权限，其实并不需要给它账号密码，或者ssh密钥。只需要给他一个personal access token即可。而且，这样还可以精确地控制travis的权限，而不是给它所有的用户权限。

# 方法

## 1. 在github中添加`personal access token`
打开[github的设置页面](https://github.com/settings/profile)，点击左侧的`Develpoer settings`进入开发者设置页面。
点击`Personal access tokens`，接着点击`Generate new token`。
`Token description`一栏可以随便填，以供日后辨认即可。
在`Select scopes`中，勾选第一个选项，也就是`repo`选项。
最后点击`Generate token`，可以看到生成了一个很长的字符串，比如：
```
274e38cdf16e254da99a190ffbd0740b5ac3dcac
```
记下它，以备后续使用。

## 2. 在仓库根目录创建travis配置文件
打开仓库文件夹，在根目录下新建`.travis.yml`文件，其内容如下：
```yaml
language: node_js

node_js: stable # 要安装的node版本为当前的稳定版。

cache:
  directories:
  - node_modules # 要缓存的文件夹

install:
- npm install # install阶段执行的命令。

script:
- npx hexo generate # script阶段执行的命令

after_script: # 最后执行的命令
- cd public
- echo blog.ladyrick.com > CNAME # github pages服务，自定义域名
- git init
- git config user.name "ladyrick" # 配置git参数
- git config user.email "ladyrick@qq.com" # 配置git参数
- git add .
- git commit -m "travis"
- git push -f "https://${MY_TOKEN}@github.com/ladyrick/hexo-blog.git" master:gh-pages
# 强制push到gh-pages分支。注意这里使用了环境变量 ${MY_TOKEN}
# 另外，注意修改仓库地址。

branches:
  only:
  - master # 触发持续集成的分支
```

这里需要注意我们使用了`${MY_TOKEN}`环境变量。这个环境变量定义了我们前面得到的`personal access token`。
但是，到目前为止，travis还无法识别这个环境变量。

## 3. 下载安装travis工具，定义环境变量。
首先需要安装ruby环境。[Ruby官网](https://www.ruby-lang.org/)
安装Ruby后，安装travis：
```bash
gem install travis
```

接着，切换到仓库目录，执行以下命令：
```bash
travis encrypt MY_TOKEN="274e38cdf16e254da99a190ffbd0740b5ac3dcac" --add
```

这样即可将前面得到的`personal access token`赋给环境变量`MY_TOKEN`，生成加密字符串，并自动添加到`.travis.yml`文件中。
更多关于travis加密的内容请看[travis文档](https://docs.travis-ci.com/user/encryption-keys/)。

## 4. 在travis网站上启动持续集成
最后一个步骤，很简单，打开[https://travis-ci.org/profile](https://travis-ci.org/profile)，用github账号登陆，找到你的仓库，点击按钮即可启动。

这样，以后每次push变更，都会触发travis的持续集成服务，自动完成构建，并推送到gh-pages分支。
