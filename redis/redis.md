# 1、redis都有哪些数据结构？
![img.png](img.png)
> 常见的五大基本数据结构：String,List,Map,Set,Sort Set;
- String：底层采用简单动态字符串（SDS）
- List:底层是（双向链表和压缩链表）
- Hash:底层是（压缩链表和哈希表）
- Set:底层是（整数数组和哈希表）
- Sorted Set:底层（压缩链表和跳表）
> 其他结构：bitmap, GEO,HyperLogLog, Stream
# 2、什么是SDS？redis为什么不直接使用char*？
```C
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
}
// __attribute__ ((__packed__))的作用
// 编译器，在编译 sdshdr16 结构时，不要使用字节对齐的方式，而是采用紧凑的方式分配内存
```
- len：字符数组现有长度
- alloc：字符数组分配的空间长度
- flags：SDS类型(sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64)
- buf[]：字符数组
```c
struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
};
```
- type: redisObject 的数据类型，是应用程序在 Redis 中保存的数据类型，包括 String、List、Hash 等
- encoding: redisObject 的编码类型，是 Redis 内部实现各种数据类型所用的数据结构。如SDS、dict 等
- lru: redisObject 的 LRU 时间 24个bit
- refcount: redisObject 的引用计数。4字节
- *ptr: 指向值的指针。8字节
```cassandraql
#define OBJ_ENCODING_EMBSTR_SIZE_LIMIT 44
robj *createStringObject(const char *ptr, size_t len) {
    if (len <= OBJ_ENCODING_EMBSTR_SIZE_LIMIT)
        return createEmbeddedStringObject(ptr,len);
    else
        return createRawStringObject(ptr,len);
}
// 从源码中知道
// 当字符长度小于44字节时，采用的emb、超过44采用的时raw
```
> 总结：C语言底层的字符数组，结尾必须为"\0"来表示结尾，这样就不能保存
> 任意二进制的数据，所以redis使用SDS来进行String的底层数据结构。
> 在redis中，所有对象都是通过redisObject来进行封装的，针对不同长度
> redisObject的*ptr有两种分配方式
- 当字符长度小于44时，redisObject和SDS的内存是一次分配在同一个内存区域，只需分配一次内存
- 及embstr模式
- 当字符长度大于44字节时，redisObject和SDS是分开分配，需要分配两次内存空间。及raw模式
- 如果string中存放的是long类型的数据，直接使用redisObject的*ptr而不需要在分配SDS的内存
![img_1.png](img_1.png)

# 3、为什么是根据字符长度44来区分的呢？
> 目前在x86的体系下，一个缓存行的大小是64字节
> redisObject本身需要16字节，
> 在SDS中sdshdr8（3 个字节）+ SDS 字符数组（N 字节 + \0 结束符 1 个字节）
> 64 - 16 - 3- 1 = 44字节

