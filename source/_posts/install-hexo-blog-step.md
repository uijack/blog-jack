title: "Hexo 搭建 Blog 新手教程"
date: 2015-12-13 13:07:57
description: "在Mac下怎么用Hexo 搭建属于自己的博客教程分享"
tags:
- Hexo
- "Keng"
---

## 引言

首先，使用Hexo搭建个人Blog是个不错的好方法，使用`Hexo(md)` + `Github(CNAME)` + `域名`组合而成的Blog瞬间感觉高大上。哈哈，请让我卖一个萌。

其次，写Blog可以加深我们对专业知识的理解和吸收，俗话说`好记性不如烂笔头`，这句话确实说的很有道理，同时通过写Blog间接锻炼了语言表达能力，当然这只是个人爱好。

最后，让我们进入正题，分享我在搭建博客的经验，希望对大家有用。

## 前期准备工作

### Github Repository

为了便于维护和操作，我为自己的Blog新建了一个仓库，即通常我们所说的一个项目。此处取名为：`testblog`
接下来找个地方进行一系列终端(`terminal`)初始化操作：

```bash
git clone git@github.com:uijack/testblog.git 
cd testblog
```

添加README.md文件并提交到master分支(不添加可以忽略)
如果没有配置`Git SSH Key`的童鞋, 先配置了在执行以下命令
配置参考[github ssh key config](http://www.cnblogs.com/ayseeing/p/3572582.html)

```bash
echo "# testblog" >> README.md 
git add README.md
git commit -m "first commit and add README file"
git push origin master
```
### 域名指向

由于我使用的是在阿里云平台下购买的uijack.com域名 所有域名商的配置大同小异，我为自己的博客配置了`CNAME`
为blog

想要更详细的配置方案，请自行翻墙咯。详情[Github Pages 绑定阿里云域名](http://quantumman.me/blog/setting-up-a-domain-with-gitHub-pages.html)

## Hexo 配置

### Hexo 安装

想详细了解Hexo使用，可以去官网[Hexo 官网](https://hexo.io/zh-cn/)

首先命令行进入testblog下面，依次运行下面命令：

```bash
npm install hexo-cli -g
hexo init blog
cd blog
sudo npm install
hexo server
```
在浏览地中输入地址:[]， 一个简单的Hexo Blog就展现在了我们的眼前

### 安装 Jacman主题

由于我喜欢使用简洁的Blog，就拿Jacman来做例子，[Jacman Github Site](https://github.com/wuchong/jacman)

如果你想将`Jacaman Themes`源码也上传到自己的项目上面去的话，由于有两种办法可以做到，因为github不允许你自己的项目包含其他仓库的项目，因为.git在作怪的缘故

- 在另外的目录新建一个文件夹执行一下命令：

```bash
git clone https://github.com/wuchong/jacman.git themes/jacman
```
将thems目录下的jackman完整的拷贝到blog下themes下，和默认主题landscape一个目录级别

- 直接在blog目下执行命令:

```bash
git clone https://github.com/wuchong/jacman.git themes/jacman
cd themes/jacman
sudo rm -rf .git
cd ../..    //回到blog目录
```

### 配置`_config.yml`文件

- 修改blog目录下的`_config.yml`文件，将theme修改成jacman，注意冒号`:`后面留一个空格再输入jacman

```plain
theme: jacman
```

- 配置一间将项目打包到github gh-pages分支下面

```plain
deploy:
  type: git
  repo:
    github: git@github.com:uijack/ijack.git,gh-pages
```
具体配置请自行参考相关资料

## 运行并上传Github pages

首先得安装一个插件：在blog目录下

```bash
npm install hexo-deployer-git --save
```

查看jacman看效果
```bash
hexo g
hexo s
```
### 部署到github 上

```bash
hexo g -d
```

### 提交源码到master

```bash
git add -A
git commit -m "change jacman themes"
git push origin master
```

### CNAME映射

在blog/source下添加CNAME文件，没有后缀，添加映射的url

```plain
blog.uijcak.com
```

`uijack.com`是我自己的一级域名
`blog`是在阿里云下设置的二级域名

重新部署：

```bash
hexo g -d
```

别忘了顺带提交源码：
```bash
git add -A
git commit -m "add CNAME"
git push origin master
```

至此打开[我自己的Blog](http://blog.uijack.com/);
还在等什么？赶快去写属于你自己的blog吧~~

## 参考文档

- [Hexo 官网](https://hexo.io/zh-cn/)
- [Gitgub Pages 绑定来自阿里云域名](http://quantumman.me/blog/setting-up-a-domain-with-gitHub-pages.html) 需要翻墙哈
- [Markdown 小书匠](http://markdown.xiaoshujiang.com/)
- [Jacman Themes](https://github.com/wuchong/jacman/)

## 特别鸣谢

> 特别感谢[SHANG殇](http://blog.xinshangshangxin.com/)给予技术指导
> 同时也感谢那个沉着冷静的自我。
> 文章若有纰漏，请大家补充指正，谢谢！
> 如有疑问可留言，我会在第一时间回复。
> [转载请注明出处](http://blog.uijack.com/)
