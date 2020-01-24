https://www.cnblogs.com/vielat/p/10205377.html

GitBook是基于Nodejs，使用Git/Github和Markdown制作电子书的命令行工具。

1、安装Nodejs

　　首先，安装Nodejs，官网地址：https://nodejs.org/en/

　　安装完成后输入命令node -v检测是否安装成功

2、安装全局Gitbook

　　在Nodejs安装目录下打开命令控制台，输入npm install gitbook-cli -g

　　由于安装默认采用国外镜像，所以需要等待一段时间。也可以使用国内镜像

　　在当前用户目录下有一个.cnpmrc文件，加入以下配置信息：

　　　　registry=http://registry.npm.taobao.org

3、初始化

　　在任意文件夹下执行命令gitbook init，最终会生成README.md、SUMMARY.md两个文件。主要目录需要在SUMMARY.md文件中配置。

4、构建

　　执行命令gitbook build，会生成_book文件夹

5、启动

　　执行命令gitbook serve（也会执行构建工作），然后可以通过浏览器输入http://localhost:4000访问电子书目录