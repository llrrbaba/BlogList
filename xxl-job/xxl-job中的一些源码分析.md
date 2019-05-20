### xxl-job中的一些源码分析

##### 路由策略相关的类

+ 枚举类

~~~java
package com.xxl.job.admin.core.route;

import com.xxl.job.admin.core.route.strategy.*;
import com.xxl.job.admin.core.util.I18nUtil;

/**
 * Created by xuxueli on 17/3/10.
 */
public enum ExecutorRouteStrategyEnum {

  	// 第一个
    FIRST(I18nUtil.getString("jobconf_route_first"), new ExecutorRouteFirst()),
  	// 最后一个
    LAST(I18nUtil.getString("jobconf_route_last"), new ExecutorRouteLast()),
  	// 轮询
    ROUND(I18nUtil.getString("jobconf_route_round"), new ExecutorRouteRound()),
  	// 随机
    RANDOM(I18nUtil.getString("jobconf_route_random"), new ExecutorRouteRandom()),
  	// 一致性HASH
    CONSISTENT_HASH(I18nUtil.getString("jobconf_route_consistenthash"), new ExecutorRouteConsistentHash()),
  	// 最不经常使用
    LEAST_FREQUENTLY_USED(I18nUtil.getString("jobconf_route_lfu"), new ExecutorRouteLFU()),
  	// 最近最久未使用
    LEAST_RECENTLY_USED(I18nUtil.getString("jobconf_route_lru"), new ExecutorRouteLRU()),
  	// 故障转移
    FAILOVER(I18nUtil.getString("jobconf_route_failover"), new ExecutorRouteFailover()),
  	// 忙碌转移
    BUSYOVER(I18nUtil.getString("jobconf_route_busyover"), new ExecutorRouteBusyover()),
    SHARDING_BROADCAST(I18nUtil.getString("jobconf_route_shard"), null);

    ExecutorRouteStrategyEnum(String title, ExecutorRouter router) {
        this.title = title;
        this.router = router;
    }

    private String title;
    private ExecutorRouter router;

    public String getTitle() {
        return title;
    }
    public ExecutorRouter getRouter() {
        return router;
    }

    public static ExecutorRouteStrategyEnum match(String name, ExecutorRouteStrategyEnum defaultItem){
        if (name != null) {
            for (ExecutorRouteStrategyEnum item: ExecutorRouteStrategyEnum.values()) {
                if (item.name().equals(name)) {
                    return item;
                }
            }
        }
        return defaultItem;
    }

}

~~~



+ 策略1:第一个

~~~java
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.util.List;

/**
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteFirst extends ExecutorRouter {

  	// 直接取集群地址列表里面的第一台机器来进行执行
    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList){
        return new ReturnT<String>(addressList.get(0));
    }

}
~~~



+ 策略2:最后一个

~~~java
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.util.List;

/**
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteLast extends ExecutorRouter {

  	// 直接从执行机集群列表的list里面取最后一个
    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList) {
        return new ReturnT<String>(addressList.get(addressList.size()-1));
    }

}
~~~



+ 策略3:轮循

 ~~~java
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.util.List;
import java.util.Random;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteRound extends ExecutorRouter {

    private static ConcurrentHashMap<Integer, Integer> routeCountEachJob = new ConcurrentHashMap<Integer, Integer>();
    private static long CACHE_VALID_TIME = 0;
    private static int count(int jobId) {
        // cache clear
      	// 如果当前的时间，大于缓存的时间，那么说明需要刷新了
        if (System.currentTimeMillis() > CACHE_VALID_TIME) {
            routeCountEachJob.clear();
          	// 设置缓存时间戳，默认缓存一天，一天之后会从新开始
            CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
        }

        // count++
      	// 当第一次执行轮循这个策略的时候，routeCountEachJob这个Map里面肯定是没有这个地址的， count==null ,
        Integer count = routeCountEachJob.get(jobId);
      	// 当 count==null或者count大于100万的时候，系统会默认在100之间随机一个数字 ， 放入hashMap, 然后返回该数字
      	// 当系统第二次进来的时候，count!=null 并且小于100万， 那么把count加1 之后返回出去
        count = (count==null || count>1000000)?(new Random().nextInt(100)):++count;  // 初始化时主动Random一次，缓解首次压力
      	// 为啥首次需要随机一次，而不是指定第一台呢？
      	// 因为如果默认指定第一台的话，那么所有任务的首次加载全部会到第一台执行器上面去，这样会导致第一台机器刚开始的时候压力很大
        routeCountEachJob.put(jobId, count);
        return count;
    }

    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList) {
      	// 在执行器地址列表，获取相应的地址，通过count(jobid) 这个方法来实现，主要逻辑在这个方法
    		// 通过count（jobId）拿到数字之后，通过取模的方式，拿到执行器地址
    		// 例： count=2 , addresslist.size = 3
    		// 2%3 = 2 ,  则拿list中下标为2的地址
      	// 这里这个轮循策略这么搞，我其实没有太看懂，为什么要搞到1000000，再去取模来实现轮循？
        String address = addressList.get(count(triggerParam.getJobId())%addressList.size());
        return new ReturnT<String>(address);
    }

}

 ~~~



+ 策略4:随机

~~~java
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.util.List;
import java.util.Random;

/**
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteRandom extends ExecutorRouter {

    private static Random localRandom = new Random();

  	// 随机这个策略比较简单，通过在集群列表的大小内随机拿出一台机器来执行，比较简单
    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList) {
        String address = addressList.get(localRandom.nextInt(addressList.size()));
        return new ReturnT<String>(address);
    }

}
~~~



+ 策略5:一致性hash

~~~java
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.io.UnsupportedEncodingException;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;
import java.util.List;
import java.util.SortedMap;
import java.util.TreeMap;

/**
 * 分组下机器地址相同，不同JOB均匀散列在不同机器上，保证分组下机器分配JOB平均；且每个JOB固定调度其中一台机器；
 *      a、virtual node：解决不均衡问题
 *      b、hash method replace hashCode：String的hashCode可能重复，需要进一步扩大hashCode的取值范围
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteConsistentHash extends ExecutorRouter {

    private static int VIRTUAL_NODE_NUM = 5;

    /**
     * get hash code on 2^32 ring (md5散列的方式计算hash值)
     * @param key
     * @return
     */
    private static long hash(String key) {

        // md5 byte
        MessageDigest md5;
        try {
            md5 = MessageDigest.getInstance("MD5");
        } catch (NoSuchAlgorithmException e) {
            throw new RuntimeException("MD5 not supported", e);
        }
        md5.reset();
        byte[] keyBytes = null;
        try {
            keyBytes = key.getBytes("UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("Unknown string :" + key, e);
        }

        md5.update(keyBytes);
        byte[] digest = md5.digest();

        // hash code, Truncate to 32-bits
        long hashCode = ((long) (digest[3] & 0xFF) << 24)
                | ((long) (digest[2] & 0xFF) << 16)
                | ((long) (digest[1] & 0xFF) << 8)
                | (digest[0] & 0xFF);

        long truncateHashCode = hashCode & 0xffffffffL;
        return truncateHashCode;
    }

    public String hashJob(int jobId, List<String> addressList) {

        // ------A1------A2-------A3------
        // -----------J1------------------
        TreeMap<Long, String> addressRing = new TreeMap<Long, String>();
        for (String address: addressList) {
            for (int i = 0; i < VIRTUAL_NODE_NUM; i++) {
                long addressHash = hash("SHARD-" + address + "-NODE-" + i);
                addressRing.put(addressHash, address);
            }
        }

        long jobHash = hash(String.valueOf(jobId));
        SortedMap<Long, String> lastRing = addressRing.tailMap(jobHash);
        if (!lastRing.isEmpty()) {
            return lastRing.get(lastRing.firstKey());
        }
        return addressRing.firstEntry().getValue();
    }

    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList) {
        String address = hashJob(triggerParam.getJobId(), addressList);
        return new ReturnT<String>(address);
    }

}

~~~



+ 策略6:最不经常使用

~~~java 
package com.xxl.job.admin.core.route.strategy;

import com.xxl.job.admin.core.route.ExecutorRouter;
import com.xxl.job.core.biz.model.ReturnT;
import com.xxl.job.core.biz.model.TriggerParam;

import java.util.*;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 单个JOB对应的每个执行器，使用频率最低的优先被选举
 *      a(*)、LFU(Least Frequently Used)：最不经常使用，频率/次数
 *      b、LRU(Least Recently Used)：最近最久未使用，时间
 *
 * Created by xuxueli on 17/3/10.
 */
public class ExecutorRouteLFU extends ExecutorRouter {
		// 定义个静态的MAP，用来存储任务ID对应的执行信息
    private static ConcurrentHashMap<Integer, HashMap<String, Integer>> jobLfuMap = new ConcurrentHashMap<Integer, HashMap<String, Integer>>();
  
  	// 定义过期时间戳
    private static long CACHE_VALID_TIME = 0;

    public String route(int jobId, List<String> addressList) {

        // cache clear
      	// 如果当前系统时间大于过期时间
        if (System.currentTimeMillis() > CACHE_VALID_TIME) {
            jobLfuMap.clear();
          	//重新设置过期时间，默认为一天
            CACHE_VALID_TIME = System.currentTimeMillis() + 1000*60*60*24;
        }

        // lfu item init
      	// 从MAP中获取执行信息
   			// lfuItemMap中放的是执行器地址以及执行次数
        HashMap<String, Integer> lfuItemMap = jobLfuMap.get(jobId);     // Key排序可以用TreeMap+构造入参Compare；Value排序暂时只能通过ArrayList；
        if (lfuItemMap == null) {
            lfuItemMap = new HashMap<String, Integer>();
            jobLfuMap.putIfAbsent(jobId, lfuItemMap);   // 避免重复覆盖
        }
      
      // map中不包含，或者值大于一百万的时候，需要重新初始化执行器地址对应的执行次数
      // 初始化的规则是在机器地址列表size里面进行随机
      // 当运行一段时间后，有新机器加入的时候，此时，新机器初始化的执行次数较小，所以一开始，新机器的压力会比较大，后期慢慢趋于平衡
        for (String address: addressList) {
            if (!lfuItemMap.containsKey(address) || lfuItemMap.get(address) >1000000 ) {
                lfuItemMap.put(address, new Random().nextInt(addressList.size()));  // 初始化时主动Random一次，缓解首次压力
            }
        }

        // load least userd count address
      // 将lfuItemMap中的key.value, 取出来，然后使用Comparator进行排序，value小的靠前
        List<Map.Entry<String, Integer>> lfuItemList = new ArrayList<Map.Entry<String, Integer>>(lfuItemMap.entrySet());
        Collections.sort(lfuItemList, new Comparator<Map.Entry<String, Integer>>() {
            @Override
            public int compare(Map.Entry<String, Integer> o1, Map.Entry<String, Integer> o2) {
                return o1.getValue().compareTo(o2.getValue());
            }
        });

      //取第一个，也就是最小的一个，将address返回，同时对该address对应的值加1
        Map.Entry<String, Integer> addressItem = lfuItemList.get(0);
        String minAddress = addressItem.getKey();
        addressItem.setValue(addressItem.getValue() + 1);

        return addressItem.getKey();
    }

    @Override
    public ReturnT<String> route(TriggerParam triggerParam, List<String> addressList) {
        String address = route(triggerParam.getJobId(), addressList);
        return new ReturnT<String>(address);
    }

}

~~~

