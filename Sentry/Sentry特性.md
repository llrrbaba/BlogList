### Sentry接入使用

#### 一.项目整合Sentry

 https://github.com/getsentry/examples/tree/master/java 

或者

 https://docs.sentry.io/clients/java/#install 

可以查看官方Demo或者Wiki，这里有关于Sentry与各种日志框架的整合，包括log4j，logback...，可以查看里面的Sentry依赖和log配置文件  



#### 二.新建team||project||member

![image.png](https://i.loli.net/2019/11/06/a9DdxnSOjtU8YGC.png)

可以新建团队，项目，邀请成员；成员和项目都可以隶属于某个团队   



#### 三.相同错误(Exception)合并

![image.png](https://i.loli.net/2019/11/06/XguL3qKREQ5bZWG.png)

异常堆栈信息完全相同的异常信息会合并，这里有五条**UndeclaredThrowableException**，是因为其里面的message不一样 



#### 四.可以将某个问题指定给相关负责人

![image.png](https://i.loli.net/2019/11/06/okw1dunX58v7eJZ.png)



#### 五.可以针对频率较高的异常，做一些忽略规则，避免被异常邮件轰炸

![image.png](https://i.loli.net/2019/11/06/XBqc9fGnlNKECWY.png)



#### 六.可以解决Sentry爆出的相关异常，并通知到团队成员

![image.png](https://i.loli.net/2019/11/06/2iUy4BoAYQh17wD.png)

解决完一个问题，可以将其标记为**Resolve**，Sentry会通知团队其他成员

![image.png](https://i.loli.net/2019/11/06/RUidkuT6reJM7oz.png)



#### 七.生成周报，统计每周的爆出的异常及解决情况等

![image.png](https://i.loli.net/2019/11/06/nogGpP2LIr6D4Os.png)



![image.png](https://i.loli.net/2019/11/06/b6vswG3RAeOTP2a.png)