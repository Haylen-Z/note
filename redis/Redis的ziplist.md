# Redis的ziplist

**ziplist**叫压缩表，是一种在内存上连续储存的数据结构，可节省内存。List、hash和zset在数据量较小的情况下会使用ziplist实现，主要由以下几个配置决定：

    hash-max-zipmap-entries 512  //entry的总数不能大于512，否则就采用hashmap/链表/跳跃表结构存储
    hash-max-zipmap-value 64 //每个entry值的长度不能大于64个字节
    list-max-ziplist-entries 512
    list-max-ziplist-value 64
    zset-max-ziplist-entries 128
    zset-max-ziplist-value 64
  
## ziplist结构

ziplist的结构如下：

| zlbytes | zltail | zllen | entry | ... | entry | zlend |
| :----- | :-----| :-----| :-----| :---- | :---- | :---- |
  
- **zlbytes**：ziplist的总字节数
- **zltail**：ziplist中最后一个entry的字节偏移量
- **entry**：ziplist的数据项
- **zlend**：单字节，固定值255，作为ziplist结束的标志

### entry结构

**entry**的结构如下：

| 上一个entry的长度 | 编码 | 数据 |
| :----: | :----: | :----: |
| prevlen | encoding | entry-data |         
    
#### prevlen

prevlen表示上一个entry的长度，这样ziplist就从尾到头进行遍历。当上一个entry的长度小于254时，prevlen占用一个字节，直接储存上一个entry的长度；当上一个entry的长度大于或等于254时，prevlen占用5个字节，第一个字节为固定值254作为标记，后四个字节储存上一个entry的长度。

#### encoding

encoding表示entry-data里数据的类型。

| 11000000 | 16位整数类型 |
| :----: | :----: |
| 11010000 | 32位整数类型 |
| 11100000 | 64位整数类型 |
| 11110000 | 24位整数类型 |
| 11111100 | 8位整数类型 |
| 1111xxxx | 此种类型不存在entry-data，xxxx即为要储存的数据（0001-1101），可表示整数1-12 |
