### 一.fluentd环境准备

##### 1.1 安装td-agent

最好就按官方文档的版本来，版本大于等于3，这样自带elasticsearch插件

~~~shell
curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent3.sh | sh
~~~

启停

~~~shell
/etc/init.d/td-agent start
/etc/init.d/td-agent stop
~~~

发个请求测试下是否成功启动

~~~shell
$ curl -X POST -d 'json={"json":"message"}' http://localhost:8888/debug.test
~~~



### 二.elasticsearch环境准备

##### 2.1下载elasticsearch

据说elasticsearch要和kibana版本保持一致，至于为啥我还没搞明白

wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.2.4.tar.gz

![](http://ww1.sinaimg.cn/large/006tNc79ly1g3lgj5pwypj317w0a00uz.jpg)

##### 2.2 启动elasticsearch

默认情况下elasticsearch是不能用root用户启动的，所以如果我们使用root用户启动elasticsearch的话，是会报错的

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3lh9m1gq5j317g0dc41j.jpg)

这时候需要我们为elasticsearch的目录新建一个用户及用户组，并把elasticsearch的所有者赋给我们新创建的用户及用户组

~~~shell
groupadd esgroup
useradd esuser -g esgroup -p esuser
chown -R esuser:esgroup elasticsearch-6.2.4
su eauser
elasticsearch-6.2.4/bin/elasticsearch
# 也可以通过 -d 参数让 elasticsearch 后台运行
elasticsearch-6.2.4/bin/elasticsearch -d
~~~



完成上述步骤后再进行启动，就可以启动成功了

![](http://ww1.sinaimg.cn/large/006tNc79ly1g3lhi4mchxj317u0ewtdt.jpg)



访问下我们刚启动的elasticsearch

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3lhkkj4qnj30s80cmtaj.jpg)



修改config/elasticsearch.yml，设置监听的IP和端口

~~~shell
network.host: 192.168.17.133
http.port: 9200
~~~

再重新启动elasticsearch，竟然报错了

![](http://ww4.sinaimg.cn/large/006tNc79ly1g3ligpdi6nj311y0i2442.jpg)

~~~shell
[1]: max number of threads [3795] for user [esuser] is too low, increase to at least [4096]
~~~

修改配置文件/etc/security/limits.conf，增加如下配置即可:

~~~shell
* soft nproc 4096
* hard nproc 4096
~~~



~~~shell
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
~~~

修改配置文件/etc/sysctl.conf，增加如下配置即可:

~~~shell
vm.max_map_count=262144
# 修改完成后执行
sysctl -p
~~~

##### 2.3 安装elastic search-head

~~~shell
git clone git://github.com/mobz/elasticsearch-head.git
curl --silent --location https://rpm.nodesource.com/setup_8.x | bash -
yum install -y nodejs
npm install -g grunt-cli
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install
~~~



修改Gruntfile.js

~~~shell
vim Gruntfile.js
~~~

新增如下配置，允许所有IP可以访问：

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3mmbwmuapj30uw082wez.jpg)



修改_site/app.js，将ocalhost修改为192.168.17.133，指定具体IP，否则如果head插件和elasticsearch不在同一台服务器，使用 localhost是无法访问的

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3mmgvb0snj312k0fqwhe.jpg)



修改elasticsearch-6.2.4/config/elasticsearch.yml，设置elasticsearch允许跨域访问

![](http://ww3.sinaimg.cn/large/006tNc79ly1g3mmm8qrvij30to07at9g.jpg)



开放9100端口，head插件默认使用9100端口

![](http://ww4.sinaimg.cn/large/006tNc79ly1g3mmr80liyj30zk03c74o.jpg)



启动elasticsearch-head，这里要注意先启动leasticsearch，再启动elastic search-head

![](http://ww1.sinaimg.cn/large/006tNc79ly1g3mn66xsq2j30vo03wgm5.jpg)



![](http://ww4.sinaimg.cn/large/006tNc79ly1g3mn9gprblj30qq0cqq4s.jpg)



![](http://ww1.sinaimg.cn/large/006tNc79ly1g3mn8daeb0j31z00d8juv.jpg)

### 三.kibana环境准备

##### 3.1下载kibana

wget https://artifacts.elastic.co/downloads/kibana/kibana-6.2.4-linux-x86_64.tar.gz



修改config/kibana.yml

![](http://ww2.sinaimg.cn/large/006tNc79ly1g3mntllnnej30u60iqadi.jpg)

##### 3.2 访问Kibana

~~~shell
ip:5601
~~~

### 四.修改td-agent配置文件

~~~shell
<source>
  @type tail
  path /usr/local/worktools/workprojects/log/server-api.log
  tag tvproxy.vip.error.log
  pos_file /var/log/td-agent/pos/tvproxy.vip.error.log.pos
  <parse>
    @type multiline
    format_firstline /\d{4}-\d{1,2}-\d{1,2}/
    format1 /^(?<time>\d{4}-\d{1,2}-\d{1,2} \d{1,2}:\d{1,2}:\d{1,2}) \[(?<thread>.*)\] (?<level>.*) \[(?<class>.*):(?<line>.*)\] (?<message>.*)/
  </parse>
</source>

<match tvproxy.vip.error.**>
  @type elasticsearch
  host 192.168.160.131
  port 9200
  include_tag_key true
  tag_key @log_name
  logstash_format true
  logstash_prefix ${tag}
  flush_interval 10s
  <buffer>
    @type memory
    total_limit_size 2G
    flush_thread_count 2
  </buffer>
</match>

<source>
  @type tail
  path /usr/local/worktools/workprojects/log/error.log
  tag tvproxy.vip.api.log
  pos_file /var/log/td-agent/pos/tvproxy.vip.api.log.pos
  <parse>
    @type multiline
    format_firstline /\d{4}-\d{1,2}-\d{1,2}/
    format1 /^(?<time>\d{4}-\d{1,2}-\d{1,2} \d{1,2}:\d{1,2}:\d{1,2}) \[(?<thread>.*)\] (?<level>.*) \[(?<class>.*):(?<line>.*)\] (?<message>.*)/
  </parse>
</source>

<match tvproxy.vip.api.**>
  @type elasticsearch
  host 192.168.160.131
  port 9200
  include_tag_key true
  tag_key @log_name
  logstash_format true
  logstash_prefix ${tag}
  flush_interval 10s
  <buffer>
    @type memory
    total_limit_size 2G
    flush_thread_count 2
  </buffer>
</match>
~~~

