#### 使用ElasticSearch期间遇到的问题

##### 1.文件描述符数量太少

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

##### 2.最大线程个数太低

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

##### 3.最大内存太小

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

##### 4.这个我也不知道怎么说

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

