# 项目简介
用于实现一个可拓展的本地缓存。
# 创作目的
- 为日常开发提供一套简单易用的缓存框架

- 便于后期多级缓存开发

- 学以致用，开发一个类似于 redis 的渐进式缓存框架
# 特性
- MVP 开发策略

- fluent 流式编程体验，纵享丝滑

- 支持 cache 固定大小

- 支持自定义 map 实现策略

- 支持 expire 过期特性

- 支持自定义 evict 驱除策略

内置 FIFO 和 LRU 驱除策略

- 支持自定义删除监听器

- 日志整合框架，自适应常见日志

- 支持 load 初始化和 persist 持久化

RDB 和 AOF 两种模式

# 入门测试
        //默认先入先出
        ICache<String, String> cache = CacheBs.<String,String>newInstance()
                .size(2).build();


        cache.put("zhangsan", "1");
        cache.put("lisi", "2");
        cache.put("wanger", "3");
        cache.put("mazi", "4");

        Assert.assertEquals(2, cache.size());//true
        System.out.println(cache.keySet());//mazi, wanger
# 内存淘汰机制
目前内置了几种内存淘汰策略，可以直接通过CacheEvicts工具类创建
| 策略 | 说明 |
|:---|:---|
| none | 没有任何淘汰策略 |
| fifo | 先进先出（默认策略） |
| lru | 最基本的朴素 LRU 策略，性能一般 |
| lruDoubleListMap | 基于双向链表+MAP 实现的朴素 LRU，性能优于 lru |
| lruLinkedHashMap | 基于 LinkedHashMap 实现的朴素 LRU，与 lruDoubleListMap 差不多 |
# 过期支持
        ICache<String, String> cache = CacheBs.<String,String>newInstance()
                .size(4)
                .build();
        cache.put("zhangsan", "1");
        cache.put("lisi", "2");
        cache.put("wanger", "3");
        cache.put("mazi", "4");

        cache.expire("mazi", 40);
        Assert.assertEquals(4, cache.size());

        TimeUnit.MILLISECONDS.sleep(50);//当前线程睡眠50ms
        Assert.assertEquals(3, cache.size());
        System.out.println(cache.keySet());
 cache.expire("mazi", 40);指定对应key在40ms后过期
 # 内存数据持久化
 目前内置了3中数据持久化机制
 | 持久化机制 | 说明 |
|:---|:---|
| none | 没有任何持久化机制 |
| RDB | 快照 |
| AOF | 文件追加写操作 |
- 测试
-       ICache<String, String> cache = CacheBs.<String,String>newInstance()
                .build();
        cache.put("zhangsan", "1");
        cache.put("lisi", "2");
        cache.put("wanger", "3");
        cache.put("mazi", "4");
        Assert.assertEquals(4, cache.size());
        ICachePersist persist=new CachePersistDbJson("1.rdb");
        persist.persist(cache);
        TimeUnit.SECONDS.sleep(5);
 - 测试结果
 1.rdb文件内容
 ![image](https://user-images.githubusercontent.com/93819289/183604173-0fd0ec97-e70b-463f-acf9-da24f66907c2.png)

 - {"key":"mazi","value":"4"}
{"key":"lisi","value":"2"}
{"key":"zhangsan","value":"1"}
{"key":"wanger","value":"3"}

