# 搭建HEXO环境
## 1.在centos上搭建hexo
安装Hexo
```
npm install -g hexo-cli
```
创建Hexo博客
```
Hexo init my-blog
cd my-blog
```
安装Hexo依赖
```
npm i
```
生成静态文件
```
hexo generate
```
在GitHub账户上创建一个仓库名为cheshi，
链接为：git@github.com:Haha047-UU/cheshi.git
配置_config.yml
```
cd _config.yml
```
修改如下：
```
url: 'https://hexo.zhzh118.com'

deploy:
  type: git
  repo: git@github.com:Haha047-UU/cheshi.git
  branch: master
```
将生成的静态文件保存到/var/www/hexo目录下
```
cp -r /my-blog/public /var/www/hexo
```
修改/etc/caddy/Caddyfile文档
```
vim /etc/caddy/Caddyfile
```
添加记录
```
hexo.zhzh118.com:443 {
        root * /var/www/hexo/public
        encode gzip
        file_server
}
```
在cloudflare上添加DNS记录
名称hexo  内容38.47.116.50  代理状态：仅DNS
重启caddy
```
systemctl restart caddy
```
部署博客
```
hexo deploy
```
之后即可在hexo.zhzh118.com上看到博客内容



## 2.配置通过ssh登录github
生成密钥对
```
ssh-keygen -t rsa -b 4096 -C "1184464111@qq.com"
```
启动ssh代理
```
$ eval "$(ssh-agent -s)"
```
将ssh私钥添加到ssh-agent
```
ssh-add ~/.ssh/id_ALGORITHM
```
打开主机终端输入命令
```
scp root@38.47.116.50:root/.ssh/id_ALGORITHM.pub D:\test\four
```
将远端服务器中存储的ssh公钥拷贝到本机中，再把ssh公钥添加到github上的账户。具体操作为登录github账号点击Settings，再点击SSH and GPG keys，点击new SSH keys，Title：the matebook，密钥字段粘贴公钥，添加SSH公钥。

测试SSH连接
在远端服务器上克隆一个本账号上的仓库命令为：
```
git clone git@github.com:Haha047-UU/weeklies.git
```
服务器可以正常下载仓库里的代码。

## 3.将mdxeditor demo部署在zhzh118.com上
在/var/www目录上创建mdx-editor文件夹
```
Mkdir /var/www/mdx-editor
```
项目运行代码来构建项目的静态文件
```
Npm run build
```
将out文件夹上传到远端服务器上
```
Scp D:\test\five\mdx-editor\oyt root@38.47.116.50:/var/www/mdx-editor
```
配置caddyfile文件
```
test.zhzh118.com:443 {
        root * /var/www/mdx-editor/out
        encode gzip
        file_server
}
```
在cloudflare上添加DNS记录
名称test  内容38.47.116.50  代理状态：仅DNS
重启caddy
```
systemctl restart caddy
```
即可在test.zhzh118.com看到项目页面