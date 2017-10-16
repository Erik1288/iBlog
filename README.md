# Erik1288.github.io

``` bash
$ npm -v
5.4.2

$ hexo -v
hexo: 3.3.9
hexo-cli: 1.0.3
os: Darwin 16.0.0 darwin x64
http_parser: 2.7.0
node: 8.5.0
v8: 6.0.287.53
uv: 1.14.1
zlib: 1.2.11
ares: 1.10.1-DEV
modules: 57
nghttp2: 1.25.0
openssl: 1.0.2l
icu: 59.1
unicode: 9.0
cldr: 31.0.1
tz: 2017b
```

### 初始化项目
npm install

### 启动本地预览
hexo s

### 发布
hexo clean && hexo g -d

### 本地高清图片插入

First

1 把主页配置文件_config.yml 里的post_asset_folder:这个选项设置为true

2 在你的hexo目录下执行这样一句话npm install hexo-asset-image --save，这是下载安装一个可以上传本地图片的插件，来自dalao：dalao的git

3 等待一小段时间后，再运行hexo n "xxxx"来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹

Second

4 最后在xxxx.md中想引入图片时，先把图片复制到xxxx这个文件夹中，然后只需要在xxxx.md中按照markdown的格式引入图片：

![你想输入的替代文字](xxxx/图片名.jpg)

注意： xxxx是这个md文件的名字，也是同名文件夹的名字。只需要有文件夹名字即可，不需要有什么绝对路径。你想引入的图片就只需要放入xxxx这个文件夹内就好了，很像引用相对路径。

5 最后检查一下，hexo g生成页面后，进入public\2017\02\26\index.html文件中查看相关字段，可以发现，html标签内的语句是<img src="2017/02/26/xxxx/图片名.jpg">，而不是<img src="xxxx/图片名.jpg>。这很重要，关乎你的网页是否可以真正加载你想插入的图片。