# github 图床搭建

> 图床:一般是指储存图片的服务器
> 要求：访问快、免费、上传和使用方便快捷 



![github_jsdeliver_picgo](https://cdn.jsdelivr.net/gh/wudg/blog@master/images/github_jsdeliver_picgo.png)


## 一 github 存储
> github 用来作为个人图片的存储最好不过，但由于github一直以来由于网络问题，使用效果并不太好，此时，使用加速器解决网络慢的问题，使得github作为图床则十分可靠

## 二 jsDelivr
> 开源的免费CDN
> 快速、可靠、自动化

## 三 PicGo
> 自动把本地图片转换成链接的一款工具
> 开源

## 四 搭建

### 4.1 github
1. 在github上新建一个仓库，如 `picgo`
2. 在仓库中新建 `images` 文件夹(用于存储具体图片的目录)
3. 在github主页依次选择【Settings】-【Developer settings】-【Personal access tokens】-【Generate new token】，填写好描述，勾选【repo】，然后点击【Generate token】生成一个Token，单独保存这个 token 值

`注意`
1. 在【Personal access tokens】页面勾选 `repo` 复选框
2. 生成的 token 值先单独保存
3. 建议github仓库设置为public
   

### 4.2 配置 PicGo

1. 下载并安装 [picgo](https://github.com/Molunerfinn/picgo/releases)
2. 进入PicGo 软件，选择【Github图床】进行配置
![picgo_github_config](https://cdn.jsdelivr.net/gh/wudg/blog@master/images/picgo_github_config.png)
3. 手动将图片拖至 PicGo 上传区即可将图片上传到我们的github仓库中对应的图片文件夹下

`注意`
1. 分支名建议直接使用 `master`
2. 指定存储路径为刚刚创建的 `images` 目录
3. 设置自定义域名，使用 `jsdeliver` 进行加速，域名示例:`https://cdn.jsdelivr.net/gh/wudg/blog@master`，前缀统一使用jsdeliver提供的域名，后面是我们 `用户名/仓库名@分支名`

## 五 总结
> 由于偶尔机会下，在浏览某大佬github仓库时好奇到是如何在github中优雅的使用图片发现，他的图片都是 `https://cdn.jsdelivr.net` 开头，于是更加好奇，接下来就是程序员基本操作了，百度...CSDN...各种搜索文档，最终知道是如何使用，并尝试自己搭建一个供自己日常使用的图床



