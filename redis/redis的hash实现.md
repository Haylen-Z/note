# redis的hash实现

**hash**用的是ziplist和hashtable来实现，当hash保存的数据较少时会使用ziplist实现。redis的hashtable使用拉链法来解决哈希冲突，hashtable的结构如下：

         +------------------+
         | dict             |
         +------------------+
         |dictType *type    |
         +------------------+
         | void *privdata   |
         +------------------+
    +----+ dictht ht[2]     |
    |    +------------------+
    |    | long rehashidx   |
    |    +------------------+
    |    | int iterators    |
    |    +------------------+
    |
    |
    |   +------------------------+       +-------------+   +--------------+
    +-->+  dictht                |       | dictEntry * +-->+dictEntry *   +----->
        +------------------------+       +-------------+   +--------------+
        |  dictEntry **table     +------>+ dictEntry * +-->
        +------------------------+       +-------------+
        |  unsigned long size    |       | dictEntry * +-->
        +------------------------+       +-------------+
        |  unsigned long sizemash|       |   .....     +-->
        +------------------------+       +-------------+
        |  unsigned long used    |
        +------------------------+

## 渐进式rehash

由于redis的事件处理是单线程的，hash扩容时如果一次性将旧hashtable的数据移到新的hashtable，整个过程会花费很长时间，阻塞后面命令的执行。因此，redis的hash扩容时会渐进式把旧hashtable中的数据移动到新的hashtable上去。

hash保存有两个指向hashtable的指针，是长度为2、叫ht的数组。其中，ht[0]指向正在储存数据的hashtable。当ht[0]满足扩容条件时，就会分配创建一个容量更大hashtable，并将ht[1]指向新的hashtable。后续对
hash进行访问时，会逐渐地把ht[0]的数据转移到ht[1]上。

rehash过程中，有一个叫rehashidx的索引变量，通过它来获取ht[0]中下一个待转移的节点，将待转移的节点的链表里的数据转移到ht[1]中。转移完成后，对rehashidx加1。其间，对数据进程查找时，会先到ht[0]中查找，找不到再到ht[1]中查找。rehash过程中，新增数据时，会添加到ht[1]中。

rehash完成后，释放旧hashtable的内存，将ht[0]指向新的hashtable。
