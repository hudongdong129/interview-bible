# 1、redis都有哪些数据结构？
![img.png](img.png)
> 常见的五大基本数据结构：String,List,Map,Set,Sort Set;
- String：底层采用简单动态字符串（SDS）
- List:底层是（双向链表和压缩链表）
- Hash:底层是（压缩链表和哈希表）
- Set:底层是（整数数组和哈希表）
- Sorted Set:底层（压缩链表和跳表）
> 其他结构：bitmap, GEO,HyperLogLog, Stream