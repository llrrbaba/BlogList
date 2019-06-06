#### Docker

####1.一些简单准备

安装完docker会默认创建一个用户组docker，可以新建一个用户到docker组来操作docker，但是发现还是root用户用着爽

~~~shell
useradd dockeruser -g docker -p dockeruser
~~~

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3rg04hwsrj30wg0fuwgw.jpg)



####2.写一个简单的docker image

###### 2.1准备一个c语言文件hello.c

~~~c
#include<stdio.h>

int main()
{
 printf("hello docker\n");
}
~~~

###### 2.2编译成可执行文件hello

~~~shell
# 前置条件，准备gcc环境
yum install gcc
yum install glibc-static

gcc -static hello.c -o hello
~~~

###### 2.3编写Dockerfile

~~~shell
FROM scratch
ADD hello /
CMD ["/hello"]
~~~

###### 2.4构建docker镜像

~~~shell
docker build -t rockerpg/hello-world .
~~~

###### 2.6运行这个镜像

~~~shell
[root@rocker hello-world]# docker run rockerpg/hello-world
hello docker
~~~

####3.一些命令

###### 3.1快速删除所有容器

~~~shell
docker rm $(docker ps -aq)
~~~

###### 3.2快速删除所有停止的容器

~~~shell
docker rm $(docker ps -f "status=exited" -q)
~~~

###### 3.3查看docker的命令

~~~shell
docker
~~~

###### 3.4查看docker image|container相关的命令

~~~shell
docker image
docker container
~~~

#### 4.基于container创建image

~~~#
# 进入容器，并安装vim
docker run -it centos
yum install -y vim

# commit Create a new image from a container's changes
# 基于我上面的安装了vim的容器commit一个image
docker container commit
# 也可以简写为
docker commit
~~~

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3rin1g9iij31my098tbg.jpg)

我在这里是基于官方的最新的centos镜像运行的容器，然后在容器中执行yum install vim安装vim，然后在这个安装了vim的容器基础上创建镜像，可以看出来，我创建的镜像和官方镜像有一些共有的层(layer)，然后我的镜像在共有层的基础上新建了一个层

![](http://ww4.sinaimg.cn/large/006tNc79ly1g3rirfpim8j31bm0e042o.jpg)

当然通过这种方式创建的image，不提倡，因为这种创建方式不是很透明，后来使用镜像的人不知道你在原有容器上增加了什么东西，可能有不稳定因素存在

#### 5.基于Dockerfile创建image

~~~shell
# 创建一个Dockerfile
FROM centos
RUN yum install -y vim

# 构建image
docker build -t rocker/centos-vim-new .
~~~

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3rjb26n9wj31am0e2q79.jpg)

这种方式更靠谱，知道创建这个镜像都干了啥，然后我们可以把我们编写的Dockerfile分享给别人，别人也能build出和我们一样的image

