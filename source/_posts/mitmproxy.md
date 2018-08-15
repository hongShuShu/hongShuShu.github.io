---
title: 抓包神器：mitmproxy安装及使用
---
今天给大家安利一个强大的装逼神器，mitmproxy，做过移动开发的应该都会有过抓包需求吧，数据展示不对的时候看看是否是后台的锅。[mitmproxy](https://github.com/mitmproxy/mitmproxy)是一个基于python的终端扩展包，因此需要python环境才能使用，不过不用担心，Mac已经自带了python环境，所以安装也就十分简单了。
{% img [class names] /images/mitmproxy_zb.jpg [225] [225] [title text [alt text]] %}

<!--more-->
## 如何安装
### 1、利用homebrew
1.1 先安装brew
{% codeblock %}
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
{% endcodeblock %}
1.2 安装mitmproxy
{% codeblock %}
brew install mitmproxy
{% endcodeblock %}

### 2、利用pip
2.1 先安装pip
Mac默认是没有安装pip的，所以需要先安装pip
{% codeblock %}
sudo easy_install pip
{% endcodeblock %}
2.2 安装mitmproxy
{% codeblock %}
pip install mitmproxy
{% endcodeblock %}

## 如何使用
1、确保手机和电脑处于同一局域网下，打开手机WIFI->手动代理->输入IP、端口->保存。
代理地址为Mac的局域网ip地址，mitmproxy默认端口为8080，也可改为自定义端口。
2、第一次抓包的时候需要在iPhone上安装CA证书。打开iPhone Safari，输入地址：mitm.it，按提示操作。
3、打开终端输入 <font color=red>mitmproxy</font> 即可进入抓包页面，如果是自定义端口，则加上-p及端口号 ：<font color=red>mitmproxy -p 端口号</font>
### 常用命令介绍
上：k
下:   j
左：h
右：l
翻页：space
进入接口详情：enter
退出到上一级：q
清屏：z
过滤器：f  输入要过滤的网址即可
附一张结果图：
{% img [class names] /images/mitmproxy_detail.jpg [650] [350] [title text [alt text]] %}
1：请求时间
2：请求方式
3：请求url
4：请求状态
5：响应数据格式
6：响应数据大小
7：请求耗时
左侧为请求参数，response为服务器返回数据，detail为请求的额外信息

### 拦截请求，修改响应
输入 i，再输入 ~s ，按回车键，就会进入了 response 拦截模式；
如果输入 ~q 则进入 request 的拦截模式。
{% img [class names] /images/mitmproxy_edit.jpg [650] [350] [title text [alt text]] %}
橘红色表示请求正被拦截，这时 enter 进入后 再按 e 就可以修改 request 或者 response。修改时是用 vim 进行编辑的，修改完成后按 a 将请求放行，如果要放行所有请求输入 A 即可。
<font color=red>tips：</font>
如果命令记不住，记得常用<font color=red>?</font>
在哪个页面使用"？",就会提示当前页面可用的命令。
快来试试吧。






