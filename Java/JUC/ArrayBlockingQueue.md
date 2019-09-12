#### ArrayBlockingQueue

###### Java1.8.0_201

- **底层数据结构**： 循环数组

  通常，队列的实现方式有数组和链表两种方式。对于数组这种实现方式来说，我们可以通过维护一个队尾指针，使得在入队的时候可以在O(1)的时间内完成；但是对于出队操作，在删除队头元素之后，必须将数组中的所有元素都往前移动一个位置，这个操作的复杂度达到了O(n)，效果并不是很好。如下图所示：

  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wrm2d23zj30u009njrm.jpg)

  为了解决这个问题，我们可以使用另外一种逻辑结构来处理数组中各个位置之间的关系。假设现在我们有一个数组A[1…n]，我们可以把它想象成一个环型结构，即A[n]之后是A[1]，如下图所示：那么我们便可以使用两个指针，分别维护队头和队尾两个位置，使**入队**和**出队**操作都可以在O(1)的时间内完成，如下图所示。当然，这个环形结构只是逻辑上的结构，实际的物理结构还是一个普通的数据。

  ![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6wrn9a5boj30yo0cy75e.jpg)

- **容量：** 可以看到ArrayBlockingQueue提供的三个构造函数都是要传入capacity的，也就决定了这是个**有界**(Bounded)的**阻塞队列**(BlockingQueue)
- **顺序：** 底层基于循环数组，元素的进出顺序是FIFO(First-In First-Out)
- **复杂度：** 入队和出队都是O(1)，出队是移除队首的元素，由于循环数组的存在，使得出队复杂度也是O(1)，但是移除队列中间的某个元素的话，时间复杂度就是O(n)了

- **公平性：** 如果有多个**获取**或者**插入**线程被阻塞的话，如果在构造ArrayBlockingQueue时指定fair=true的话，那么signal时等待时间最长的线程将最先获得锁
- **锁：** ArrayBlockingQueue只是用了一个lock，插入数据或者删除数据都是用的这一个锁，并发行的话，可能不如LinkedBlockingQueue好，因为后者是用了两个锁putLock和takeLock





###### 源码注释地址：

###### https://github.com/llrrbaba/javase-learning/blob/master/javase-learning-juc/src/main/java/cn/rocker/javaselearningjuc/blockingqueue/ArrayBlockingQueueNote.md



###### 参考：

###### https://www.codejava.net/java-core/concurrency/java-arrayblockingqueue-examples

###### https://blog.csdn.net/u013124587/article/details/53232140

