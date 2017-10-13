## 拉取aegis分支代码

git clone -b aegis git@github.com:aegis-fed/aegis-fed.github.io.git
cd aegis-fed.github.io/ 
npm i
git checkout -b mybranch

## 全局安装hexo

npm i -g hexo  

## 新建新文章

hexo new "新文章"

## 本地调试

hexo g
hexo s

## 合并分支

master分支存放静态文件，自己的分支进行本地开发

## 提交代码

正常走流程提交到自己的分支

## 请求合并

request merge

## aegis（或者后面再定。。）分支负责合并并发布新文章

hexo g -d