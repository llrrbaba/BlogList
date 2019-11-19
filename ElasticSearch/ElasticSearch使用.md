#### 一.安装ElasticSearch

##### 1.1 下载`elasticsearch-7.4.2-linux-x86_64.tar.gz`

~~~shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.4.2-linux-x86_64.tar.gz
~~~



#### 二.使用ElasticSearch期间遇到的问题

![image.png](https://i.loli.net/2019/11/19/toNpMEz7m5YjXFg.png)

![image.png](https://i.loli.net/2019/11/19/qjysShIkoiAnFaL.png)

##### 2.1 不能用root用户启动ElsaticSearch

~~~shell
java.lang.RuntimeException: can not run elasticsearch as root
~~~

解决方案：

~~~shell
# 添加用户组
groupadd elsearch
# 添加用户
useradd elsearch -g elsearch -p elsearch
# 递归将ElasticSearch目录的权限赋予新建的用户
chown -R elsearch:elsearch elasticsearch-7.4.2/
# 切换到新建的用户
su elsearch
# ElasticSearch跑起来
./bin/elasticsearch
~~~



##### 2.2 文件描述符数量太少

~~~shell
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
~~~

解决方案：

~~~shell
vi /etc/security/limits.conf
# 增加以下配置
* soft nofile 65536
* hard nofile 65536
~~~

##### 2.3 最大线程个数太低

~~~shell
[2]: max number of threads [3797] for user [elsearch] is too low, increase to at least [4096]
~~~

解决方案：

~~~shell
vi /etc/security/limits.conf
# 增加以下配置
* soft nproc 4096
* hard nproc 4096
~~~

##### 2.4 最大内存太小

~~~
[3]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
~~~

解决方案：

~~~shell
vim /etc/sysctl.conf
# 增加以下配置
vm.max_map_count=262144
# 执行以下命令，使配置生效
sysctl -p
~~~

##### 2.5 这个我也不知道怎么说

~~~shell
[4]: the default discovery settings are unsuitable for production use; at least one of [discovery.seed_hosts, discovery.seed_providers, cluster.initial_master_nodes] must be configured
~~~

解决方案：

~~~shell
# 修改config/elasticsearch.yml，放开node.name注释
node.name:node-1
#同样修改config/elasticsearch.yml，放开cluster.initial_master_nodes注释
cluster.initial_master_nodes: ["node-1"]
~~~

#### 三.访问ElasticSearch

![image.png](https://i.loli.net/2019/11/19/wAOoCzkf2F1vNxM.png)

可以看到ElasticSearch启动成功，默认监听`9200`端口，我们访问下

![image.png](https://i.loli.net/2019/11/19/cBskNxSu7d28wTF.png)



