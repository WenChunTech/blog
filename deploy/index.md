# Deploy


<!--more-->

# Hugo博客从零到发布

## 1. 安装

安装hugo:

- 下载地址：`https://github.com/gohugoio/hugo/releases/tag/v0.104.3`

> 注意： 有些主题需要下载extended版本

安装git：

- 下载地址：`https://git-scm.com/`

## 2. 配置远程仓库，并新建一个空项目(不需要README.md文件)，名字一般和站点名相同


## 3. 本地部署

- 新建站点并配置git
```bash
hugo new site your_site_name

# 下载你需要的主题的压缩包放到theme目录下或者使用`git submodule`拉取，例：
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt

# 根据主题文档配置config.toml文件，如果只有一个主题可直接在项目根目录下的config.toml文件配置
# 0. 当本地可以正常预览站点时运行，hugo命令打包，生成静态文件
# 1. 使用git初始化站点
# 2. 添加远程仓库
git remote add origin remtoe_url
# 3. 拉取远程
# 4. 提交当前代码
# 4. 新建分支(例如：release)，命令如下：
git switch -c release 
# 或者
git checkout -b release
# 5. 合并主分支分支public目录到当前分支，命令如下：
git checkout master public/**
# 6. 移动public下所有内容到项目根目录下，例如：
mv public/* .
# 7. 再提交当前分支内容
```


## 3. 推送到远程

```bash
# 使用git推送到远程仓库

git push -u origin master

git push -u origin release

# 将远程仓库默认分支设置为release

```

## 4. 使用cloudflare发布

- 注册一个cloudflare账号：`https://dash.cloudflare.com/`

- 点击Pages，选择创建项目下连接到Git，然后根据需要配置相应信息

![创建项目](../../resources/_gen/images/create.png)

---

> 作者:   
> URL: https://release.blog-12x.pages.dev/deploy/  

