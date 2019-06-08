#### 1.问题描述

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3tqmfy2qhj319i0463zm.jpg)

我在获取redis镜像的时候竟然超时了，之前还好好的，不知道为啥



#### 2.解决问题

##### 2.1通过`dig @114.114.114.114 registry-1.docker.io`找到可用IP

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3tqsu9w2cj31980fi0vf.jpg)

没有获取到我想要的IP

##### 2.2安装bind-utils

~~~shell
yum -y install bind-utils
~~~

#####2.3修改/etc/hosts

再次执行上面的dig命令，获取到可用IP，并将获取到的可用IP设置给registry-1.docker.io域名

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3tqq2ttqzj31960m00x5.jpg)



~~~shell
vim /etc/hosts

34.197.189.129 registry-1.docker.io
34.232.31.24   registry-1.docker.io
34.201.196.14  registry-1.docker.io
34.206.236.31  registry-1.docker.io
~~~

##### 2.4保存/etc/hosts，并重试docker pull redis

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3tqlbuw5ej314q0jc0xo.jpg)