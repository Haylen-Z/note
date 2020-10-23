# Redis的String底层数据结构

Redis没有使用C语言的形式储存字符串，而是使用了一个叫**SDS（Simple Dynamic String）**的结构体储存字符串。

        struct sdshdr {
            int len;//字符长度
            int alloc;//数组容量
            unsigned char  flags;//sdshdr类型
            char buf[];//字符数组
        }

SDS使用一个整型变量len保存着当前储存的字符串的长度。在进行字符追加时，如果字符数组容量不够，会对字符数组进行扩容。每次对字符数组长度分配内存时，会分配所存字符所占用的内存两倍，避免频繁的内存分配。

SDS根据可储存的字符串的最大长度分为5种类型：

      struct __attribute__ ((__packed__)) sdshdr5 {
          unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
          char buf[];
      };
      struct __attribute__ ((__packed__)) sdshdr8 {
          uint8_t len; /* used */
          uint8_t alloc; /* excluding the header and null terminator */
          unsigned char flags; /* 3 lsb of type, 5 unused bits */
          char buf[];
      };
      struct __attribute__ ((__packed__)) sdshdr16 {
          uint16_t len; /* used */
          uint16_t alloc; /* excluding the header and null terminator */
          unsigned char flags; /* 3 lsb of type, 5 unused bits */
          char buf[];
      };
      struct __attribute__ ((__packed__)) sdshdr32 {
          uint32_t len; /* used */
          uint32_t alloc; /* excluding the header and null terminator */
          unsigned char flags; /* 3 lsb of type, 5 unused bits */
          char buf[];
      };
      struct __attribute__ ((__packed__)) sdshdr64 {
          uint64_t len; /* used */
          uint64_t alloc; /* excluding the header and null terminator */
          unsigned char flags; /* 3 lsb of type, 5 unused bits */
          char buf[];
      };
      
其中shdhdr5较为特别，没有len字段，主要用来储存固定长度的字符串。

使用SDS有以下优点：
- 避免频繁分配内存。SDS进行分配内存时会进行预分配，分配的内存是所要存的字符串所占的内存的两倍。
- 避免缓存区溢出。SDS在进行字符串拼接时，会检查内部的字符数组的长度是否够用，不够用时会进行扩容。
- 快速的获取字符串的长度。SDS使用了len变量储存字符串的长度，可以以O（1）时间复杂度获取字符串的长度。

