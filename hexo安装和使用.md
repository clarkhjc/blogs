# ubuntu下安装和使用hexo

## 安装hexo
```shell
# 更新系统包列表
sudo apt-get update

# 安装Node.js和npm
sudo apt-get install nodejs npm

# 安装Hexo CLI
sudo npm install -g hexo-cli
```

## 使用hexo-cli初始化博客
```shell
hexo init blog
cd blog
npm install
```

## 使用hexo-cli管理博客
```shell
# 如何刚更新代码下来 需要先执行npm install
hexo new post "pageName"
hexo clean
hexo g
hexo s
```

## 使用hexo-cli生成静态页面
```shell
hexo generate
```

## 使用hexo-cli本地预览博客
```shell
hexo server
```

## 安装Hexo主题和插件
下载主题
```shell
git clone https://github.com/yelog/hexo-theme-3-hexo.git themes/3-hexo
```
在_config.yml文件中设置主题：
theme: 3-hexo
