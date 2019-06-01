#### 1.fluentd环境准备

#### 2.elasticsearch环境准备

###### 2.1下载elasticsearch

据说elasticsearch要和kibana版本保持一致，至于为啥我还没搞明白

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.tar.gz

![](http://ww1.sinaimg.cn/large/006tNc79ly1g3lgj5pwypj317w0a00uz.jpg)

###### 2.2启动elasticsearch

默认情况下elasticsearch是不能用root用户启动的，所以如果我们使用root用户启动elasticsearch的话，是会报错的

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3lh9m1gq5j317g0dc41j.jpg)

这时候需要我们为elasticsearch的目录新建一个用户及用户组，并把elasticsearch的所有者赋给我们新创建的用户及用户组

~~~shell
groupadd esgroup
useradd esuser -g esgroup -p esuser
chown -R esuser:esgroup elasticsearch-6.2.4
~~~

完成上述步骤后再进行启动，就可以启动成功了

#### 3.kibana环境准备

###### 3.1下载kibana

wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.4-linux-x86_64.tar.gz