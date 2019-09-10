```java
package com.netflix.loadbalancer;

import com.netflix.client.config.IClientConfig;

/**
 * Given that
 * {@link IRule} can be cascaded, this {@link RetryRule} class allows adding a retry logic to an existing Rule.
 * 
 * @author stonse
 * 
 */
public class RetryRule extends AbstractLoadBalancerRule {
   IRule subRule = new RoundRobinRule();
   long maxRetryMillis = 500;

   public RetryRule() {
   }

   public RetryRule(IRule subRule) {
      this.subRule = (subRule != null) ? subRule : new RoundRobinRule();
   }

   public RetryRule(IRule subRule, long maxRetryMillis) {
      this.subRule = (subRule != null) ? subRule : new RoundRobinRule();
      this.maxRetryMillis = (maxRetryMillis > 0) ? maxRetryMillis : 500;
   }

   public void setRule(IRule subRule) {
      this.subRule = (subRule != null) ? subRule : new RoundRobinRule();
   }

   public IRule getRule() {
      return subRule;
   }

   public void setMaxRetryMillis(long maxRetryMillis) {
      if (maxRetryMillis > 0) {
         this.maxRetryMillis = maxRetryMillis;
      } else {
         this.maxRetryMillis = 500;
      }
   }

   public long getMaxRetryMillis() {
      return maxRetryMillis;
   }

   
   
   @Override
   public void setLoadBalancer(ILoadBalancer lb) {       
      super.setLoadBalancer(lb);
      subRule.setLoadBalancer(lb);
   }

   /*
    * Loop if necessary. Note that the time CAN be exceeded depending on the
    * subRule, because we're not spawning additional threads and returning
    * early.
    */
   public Server choose(ILoadBalancer lb, Object key) {
      long requestTime = System.currentTimeMillis();
      long deadline = requestTime + maxRetryMillis;

      Server answer = null;

      answer = subRule.choose(key);

      if (((answer == null) || (!answer.isAlive()))
            && (System.currentTimeMillis() < deadline)) {

         // 这里定义了一个InterruptTask，继承TimerTask；
         // 并且实现了抽象的run方法，run方法的实现很简单，就是判断，
	 // 如果InterruptTask里面构建的当前线程不为null并且还活着，就中断该线程；
         // 这个InterruptTask里面实现的run方法什么时候执行呢，就是在当前时间+maxRetryMillis时执行中断当前线程，
	 // 这个run方法是在Timer里的mainloop里task.run()调用的
         InterruptTask task = new InterruptTask(deadline
               - System.currentTimeMillis());

         // 这里判断Thread.interrupted()，就是和上面InterruptTask的run方法呼应上了，
	 // 在当前时间+maxRetryMillis之前，这个while为true，
	 // 从而在当前时间到当前时间+maxRetryMillis这个时间段内，反复重试获取server
         while (!Thread.interrupted()) {
            answer = subRule.choose(key);

            if (((answer == null) || (!answer.isAlive()))
                  && (System.currentTimeMillis() < deadline)) {
               /* pause and retry hoping it's transient */
               Thread.yield();
            } else {
               break;
            }
         }
         // 最终要把这个InterruptTask取消掉，这样就能把Timer.TaskQueue中对于该task的引用移除
         task.cancel();
      }

      if ((answer == null) || (!answer.isAlive())) {
         return null;
      } else {
         return answer;
      }
   }

   @Override
   public Server choose(Object key) {
      return choose(getLoadBalancer(), key);
   }

   @Override
   public void initWithNiwsConfig(IClientConfig clientConfig) {
   }
}
```

