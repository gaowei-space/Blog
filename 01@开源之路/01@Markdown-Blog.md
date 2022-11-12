# 🍭 Markdown-Blog
[![GitHub branch checks state](https://img.shields.io/github/checks-status/gaowei-space/meituan-pub-union/main)](https://github.com/gaowei-space/markdown-blog/tree/main)
[![GitHub issues](https://img.shields.io/github/issues/gaowei-space/markdown-blog?color=blueviolet)](https://github.com/gaowei-space/markdown-blog/issues)
[![Latest Release](https://img.shields.io/github/v/release/gaowei-space/markdown-blog)](https://github.com/gaowei-space/markdown-blog/releases)
[![GitHub license](https://img.shields.io/github/license/gaowei-space/markdown-blog)](https://github.com/gaowei-space/markdown-blog/blob/main/LICENSE)

[Markdown-Blog](https://github.com/gaowei-space/markdown-blog) 是一款小而美的**Markdown静态博客**程序

> 如果你和我一样，平时喜欢使用`markdown`文件来记录自己的工作与生活中的点滴，又希望把这些记录生成个人博客，那[Markdown-Blog](https://github.com/gaowei-space/markdown-blog)再适合不过了。它简洁、轻快，部署简单，可以把markdown文件快速变为个人博客，它不需要管理后台，无需进行文章的二次发布。

## 案例
> [TechMan'Blog](https://blog.gaowei.tech)

### Web端
Style | Preveiw
--------|------
Dark | <img max-width="600" alt="pc dark" src="https://user-images.githubusercontent.com/10205742/200173152-ca9fa52c-3590-4528-910a-ad42cb278f06.png">
Light | <img max-width="600" alt="pc white" src="https://user-images.githubusercontent.com/10205742/200173231-90f02b72-9e12-4a8b-8dd2-91e1ec4b4ff8.png">

### 移动端
Dark    | Light
--------|------
<img max-width="400" alt="mobile dark" src="https://user-images.githubusercontent.com/10205742/201472561-7cd1222e-da0a-4d8c-be11-9f7e9d0851e0.png"> | <img max-width="400" alt="mobile white" src="https://user-images.githubusercontent.com/10205742/201472579-458b902a-dcae-4340-a305-3b54f1679aba.png">

## 支持平台
> Windows、Linux、Mac OS

## 更新
* `[v0.1.1]` 2022-11-12 **✨推荐升级**
  - 新增第三方分析统计配置，包括：百度、谷歌
  - 支持配置页面缓存时间
  - 优化移动端样式
  - 修复 **Markdown** 部分语法展示异常问题

* `[v0.0.5]` 2022-11-06
  - 支持 TOC 语法，当文件内容首行使用 `[toc]` 会自动解析
  - 新增明亮🔆主题，支持明暗切换
  - 其他已知问题修复

## 安装
1. 下载 [release](https://github.com/gaowei-space/markdown-blog/releases/)

2. 解压
```
tar zxf markdown-blog-v0.0.5-linux-amd64.tar.gz
```

3. 创建 markdown 文件目录
```
cd markdown-blog-linux-amd64
mkdir md
echo "### Hello World" > ./md/主页.md
```

4. 运行
```
./markdown-blog web
```

5. 访问 http://127.0.0.1:5006，查看效果

## 使用

### 命令
- markdown-blog
    - -h 查看版本
    - web 运行博客服务
- markdown-blog web
   - --dir value, -d value     指定markdown文件夹，默认：./md/
   - --title value, -t value   web服务标题，默认："Blog"
   - --port value, -p value    web服务端口，默认：5006
   - --env value, -e value     运行环境, 可选：dev,test,prod，默认："prod"
   - --index value, -i value   设置默认首页的文件名称, 默认为空
   - --cache value, -c value   设置页面缓存时间，单位分钟，默认3分钟
   - --analyzer-baidu value    设置百度分析统计器
   - --analyzer-google value   设置谷歌分析统计器
   - -h                        查看版本


### 关于默认首页
> 如果启动是未指定`index`，程序默认以导航中的第一个文件作为首页

### 关于分析统计器
#### 百度
##### 1. 访问 https://tongji.baidu.com 创建站点，获取官方代码中的参数 `0952befd5b7da358ad12fae3437515b1`
```html
<script>
	var _hmt = _hmt || [];
	(function() {
	  var hm = document.createElement("script");
	  hm.src = "https://hm.baidu.com/hm.js?0952befd5b7da358ad12fae3437515b1";
	  var s = document.getElementsByTagName("script")[0];
	  s.parentNode.insertBefore(hm, s);
	})();
</script>
```
##### 2. 配置
```
./markdown-blog web --analyzer-baidu 0952befd5b7da358ad12fae3437515b1
```

#### 谷歌
##### 1. 访问 https://analytics.google.com 创建站点，获取官方代码中的参数 `G-MYSMYSMYS`
```html
<script async="" src="https://www.googletagmanager.com/gtag/js?id=G-MYSMYSMYS"></script>
<script>
    window.dataLayer = window.dataLayer || [];
    function gtag(){dataLayer.push(arguments);}
    gtag('js', new Date());

    gtag('config', 'G-MYSMYSMYS');
</script>
```
##### 2. 配置
```
./markdown-blog web --analyzer-google G-MYSMYSMYS
```

### 导航排序
> 博客导航默认按照`字典`排序，可以通过 `@` 前面的数字来自定义顺序

#### 个人博客目录如下图
<img width="390" alt="image" src="https://user-images.githubusercontent.com/10205742/176992908-affe01b6-0a50-488b-bb67-216a75f2a02c.png">

#### 博客导航展示如下图
<img width="407" alt="image" src="https://user-images.githubusercontent.com/10205742/176992913-148a5ba5-bce0-42ed-b09a-9f914556723a.png">

### 部署
> Nginx 反向代理配置文件参考

#### HTTP协议
```
server {
    listen      80;
    listen [::]:80;
    server_name yourhost.com;

    location / {
         proxy_pass  http://127.0.0.1:5006;
         proxy_set_header   Host             $host;
         proxy_set_header   X-Real-IP        $remote_addr;
         proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
     }
}
```
#### HTTPS 协议（80端口自动跳转至443）
```
server {
    listen      80;
    listen [::]:80;
    server_name yourhost.com;

    location / {
        rewrite ^ https://$host$request_uri? permanent;
    }
}

server {
    listen          443 ssl;
    server_name     yourhost.com;
    access_log      /var/log/nginx/markdown-blog.access.log main;


    #证书文件名称
    ssl_certificate /etc/nginx/certs/yourhost.com_bundle.crt;
    #私钥文件名称
    ssl_certificate_key /etc/nginx/certs/yourhost.com.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;

    location / {
         proxy_pass  http://127.0.0.1:5006;
         proxy_set_header   Host             $host;
         proxy_set_header   X-Real-IP        $remote_addr;
         proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
     }
 }
 ```

## 升级
1. 下载最新版 [release](https://github.com/gaowei-space/markdown-blog/releases/)

2. 停止程序，解压覆盖 `markdown-blog` 文件 和 `web` 文件夹

3. 重新启动程序

## 开发
1. 安装 `Golang` 开发环境

2. Fork [源码](https://github.com/gaowei-space/gocron)

3. 启动 web服务

    >运行之后访问地址 http://localhost:5006，API请求会转发给 markdown-blog
    ```
    make run
    ```

4. 编译
    ```
    make
    ```

5. 打包

    >在 markdown-blog-package 生成当前系统的压缩包 markdown-blog-v0.0.5-darwin-amd64.tar
    ```
    make package
    ```

1. 生成 Windows、Linux、Mac 的压缩包

    >在 markdown-blog-package 生成压缩包 markdown-blog-v0.0.5-darwin-amd64.tar markdown-blog-v0.0.5-linux-amd64.tar.gz markdown-blog-v0.0.5-windows-amd64.zip*
    ```
    make package-all
    ```

## 授权许可
本项目采用 MIT 开源授权许可证，完整的授权说明已放置在 [LICENSE](https://github.com/gaowei-space/markdown-blog/blob/main/LICENSE) 文件中。
