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
| lfu | 最基本的朴素 LFU 策略，命中率由于lru |
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
 
 ![image](https://user-images.githubusercontent.com/93819289/183607211-9f8da2a4-e92c-4916-b598-b50a9c552590.png)
 
1.rdb文件内容

{"key":"mazi","value":"4"}

{"key":"lisi","value":"2"}

{"key":"zhangsan","value":"1"}

{"key":"wanger","value":"3"}

# load初始化
从文件中添加数据到内存
- 测试

        ICache<String, String> cache = CacheBs.<String,String>newInstance()
                .build();
        ICacheLoad load=new CacheLoadDbJson("1.rdb");
        load.load(cache);
        System.out.println(cache.keySet());
        
- 测试结果

![1660035437211](https://user-images.githubusercontent.com/93819289/183608615-4e084fcf-e781-4942-87a6-2dc9ef9d6303.png)

![image](https://user-images.githubusercontent.com/93819289/183608327-72fbe453-0cf9-4574-9172-f63a8544cf95.png)
# 编码
## 1、如何实现固定缓存大小
缓存能根据键快速查找到值，Map是很好的一个实现。但Map的大小可以无限制增长下去，不能实现淘汰策略。
### 1.1 将Map和Queue联合使用，可实现控制缓存的大小和先入先出淘汰策略

private final Queue<K> queue = new LinkedList<>();
        
    @Override
    public CacheEntry<K,V> doEvict(ICacheEvictContext<K, V> context) {
        CacheEntry<K,V> result = null;

        final ICache<K,V> cache = context.cache();
        // 超过限制，执行移除
        if(cache.size() >= context.size()) {
            K evictKey = queue.remove();
            // 移除最开始的元素
            V evictValue = cache.remove(evictKey);
            result = new CacheEntry<>(evictKey, evictValue);
        }

        // 将新加的元素放入队尾
        final K key = context.key();
        queue.add(key);
        return result;
        
### 1.2、 将Map和LinkedList联合使用，可实现控制缓存的大小和LRU淘汰策略
        
     private final List<K> list = new LinkedList<>();

    @Override
    protected ICacheEntry<K, V> doEvict(ICacheEvictContext<K, V> context) {
        ICacheEntry<K, V> result = null;
        final ICache<K,V> cache = context.cache();
        // 超过限制，移除队尾的元素
        if(cache.size() >= context.size()) {
            K evictKey = list.get(list.size()-1);
            V evictValue = cache.remove(evictKey);
            result = new CacheEntry<>(evictKey, evictValue);
        }

        return result;
    }

