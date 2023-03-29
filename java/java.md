# 1、java中如果判断一个字符串包含另位一个字符串
> 使用contains()方法
```java
 public static void main(String[] args) {
        String a = "测试";
        String b = "哈哈测试哈";
        System.out.println(b.contains(a));
    }
```
> 使用indexOf()方法用途是在一个字符串中寻找一个字的位置，同时也可以判断一个字符串中是否包含某个字符
```java
public static void main(String[] args) {
        String a = "测试";
        String b = "哈哈测试哈";
        // 返回不是-1表示b包含a字符
        System.out.println(b.indexOf(a));
    }
```
# 2、如果统计一个字符串中每个字符出现的次数
```java
public static void main(String[] args) {
        String str = "abcdetdadaxdaadfaswqAAd";
        Map<String, Long> collect = Stream.of(str.split("")).collect(Collectors.groupingBy(String::toString, Collectors.counting()));
        System.out.println(collect.toString());
    }
```

# 3、maven中的scope有哪些取值和作用？
> Maven中使用 scope 来指定当前包的依赖范围和依赖的传递性
> scope的取值只要有compile、provided、runtime、test、system

| scope取值 | 有效范围（compile, runtime, test)| 依赖传递 |
|:--------:| -------------:|-------------:|
| compile | all | 是 |
| provided| compile, test |否 |
| runtime| runtime,test |是 |
| test| test |否 |
| system| compile, test |是 |
