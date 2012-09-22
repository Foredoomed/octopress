---
layout: post
title: "iBatis和myBatis源码分析之缓存"
date: 2011-09-16 18:41
comments: true
categories: opensource
---
**一.iBatis**

1.iBatis的cache包结构：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary="">
<tbody>
<tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b>
</th>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.fifo</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.lru</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.memory</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>com.ibatis.sqlmap.engine.cache.oscache</b></td>
<td>&nbsp;</td>
</tr>
</tbody>
</table>

iBatis已实现的四个CacheController类分别是：

* FifoCacheController
* LruCacheController
* MemoryCacheController
* OSCacheController


其中，前三种是最常用的缓存策略，第四种是对[OSCache](http://java.net/projects/oscache "OSCache")的支持。当然，你也可以根据需要实现自己的缓存策略，只需要实现CacheController接口就可以了。 <!--more-->

2.源码分析

{% codeblock FifoCacheController.java lang:java %}
/**
 * FIFO (first in, first out) cache controller implementation
 */
public class FifoCacheController implements CacheController {

  private int cacheSize;
  private Map cache;
  private List keyList;

  /**
   * Default constructor
   */
  public FifoCacheController() {
    this.cacheSize = 100;
    this.cache = Collections.synchronizedMap(new HashMap());
    this.keyList = Collections.synchronizedList(new LinkedList());
  }

  public int getCacheSize() {
    return cacheSize;
  }

  public void setCacheSize(int cacheSize) {
    this.cacheSize = cacheSize;
  }

  /**
   * Configures the cache
   *
   * @param props Optionally can contain properties [reference-type=WEAK|SOFT|STRONG]
   */
  public void setProperties(Properties props) {
    String size = props.getProperty("cache-size");
    if (size == null) {
      size = props.getProperty("size");
    }
    if (size != null) {
      cacheSize = Integer.parseInt(size);
    }
  }

  /**
   * Add an object to the cache
   *
   * @param cacheModel The cacheModel
   * @param key        The key of the object to be cached
   * @param value      The object to be cached
   */
  public void putObject(CacheModel cacheModel, Object key, Object value) {
    cache.put(key, value);
    keyList.add(key);
    if (keyList.size() > cacheSize) {
      try {
        Object oldestKey = keyList.remove(0);
        cache.remove(oldestKey);
      } catch (IndexOutOfBoundsException e) {
        //ignore
      }
    }
  }

  /**
   * Get an object out of the cache.
   *
   * @param cacheModel The cache model
   * @param key        The key of the object to be returned
   * @return The cached object (or null)
   */
  public Object getObject(CacheModel cacheModel, Object key) {
    return cache.get(key);
  }

  public Object removeObject(CacheModel cacheModel, Object key) {
    keyList.remove(key);
    return cache.remove(key);
  }

  /**
   * Flushes the cache.
   *
   * @param cacheModel The cache model
   */
  public void flush(CacheModel cacheModel) {
    cache.clear();
    keyList.clear();
  }

}
{% endcodeblock %}

我们可以看到FifoCacheController是用一个同步的HashMap和一个同步的LinkedList来实现的。其中，HashMap用来保存缓存内容；LinkedList是用来保存缓存中的对象的key。本来缓存只需要一个key-value容器就可以实现了，为什么要多引入一个LinkedList呢？主要原因还是Map的API不够强大，不支持索引操作，为了方便地找到HashMap中的第一个元素就必须加入一个支持索引的操作的集合，比如FifoCacheController类里的LinkedList,这样就可以快速地找到HashMap中的第一个对象的key值，而不是迭代HashMap。

但是，这样的实现存在一个bug。假设有下列代码：

{% codeblock lang:java %}
putObject(cacheModel1, "key1", "value1");
putObject(cacheModel2, "key2", "value2");
putObject(cacheModel3, "key1", "value3");
{% endcodeblock %}

显然，这个时候HashMap中应该有两个值，"key1"->"value3"和"key2"->"value2"；但是HashMap中却有三个key值，这样HashMap和LinkedList无法对应起来，也就是bug产生的原因。这个bug只存在与iBatis中，myBatis已经重写了缓存的实现，所以myBatis没有这个bug。

在putObject方法里，又出现了在catch体里不做任何处理的“坏味道”。而且我们知道在这里捕获的IndexOutOfBoundsException不是已检查异常，完全可以不加try-catch；如果要加上try-catch的话，那么在catch体里可以加上日志，这样对缓存调优是非常有帮助的；又或者可以把异常抛出，在CacheModel类里处理异常。总之，异常处理是一定要做的，绝对不能什么都不做。

从实际情况来看，FIFO策略的缓存实现很少使用，用到最多的还是LRU策略的缓存实现。我们来看一下iBatis的LRU缓存实现：

{% codeblock LruCacheController.java lang:java %}
/**
 * LRU (least recently used) cache controller implementation
 */
public class LruCacheController implements CacheController {

  private int cacheSize;
  private Map cache;
  private List keyList;

  /**
   * Default constructor
   */
  public LruCacheController() {
    this.cacheSize = 100;
    this.cache = Collections.synchronizedMap(new HashMap());
    this.keyList = Collections.synchronizedList(new LinkedList());
  }

  public int getCacheSize() {
    return cacheSize;
  }

  public void setCacheSize(int cacheSize) {
    this.cacheSize = cacheSize;
  }

  /**
   * Configures the cache
   *
   * @param props Optionally can contain properties [reference-type=WEAK|SOFT|STRONG]
   */
  public void setProperties(Properties props) {
    String size = props.getProperty("cache-size");
    if (size == null) {
      size = props.getProperty("size");
    }
    if (size != null) {
      cacheSize = Integer.parseInt(size);
    }
  }

  /**
   * Add an object to the cache
   *
   * @param cacheModel The cacheModel
   * @param key        The key of the object to be cached
   * @param value      The object to be cached
   */
  public void putObject(CacheModel cacheModel, Object key, Object value) {
    cache.put(key, value);
    keyList.add(key);
    if (keyList.size() > cacheSize) {
      try {
        Object oldestKey = keyList.remove(0);
        cache.remove(oldestKey);
      } catch (IndexOutOfBoundsException e) {
        //ignore
      }
    }
  }

  /**
   * Get an object out of the cache.
   *
   * @param cacheModel The cache model
   * @param key        The key of the object to be returned
   * @return The cached object (or null)
   */
  public Object getObject(CacheModel cacheModel, Object key) {
    Object result = cache.get(key);
    keyList.remove(key);
    if (result != null) {
      keyList.add(key);
    }
    return result;
  }

  public Object removeObject(CacheModel cacheModel, Object key) {
    keyList.remove(key);
    return cache.remove(key);
  }

  /**
   * Flushes the cache.
   *
   * @param cacheModel The cache model
   */
  public void flush(CacheModel cacheModel) {
    cache.clear();
    keyList.clear();
  }

}
{% endcodeblock %}

LruCacheController怎么跟FIFOCacheController长得差不多啊，FIFO跟LRU的区别在哪里体现呢？LRU就是当缓存空间不足时，把最近最少使用的对象移除，那怎么说明一个对象是最近最少使用呢？再仔细一看，答案就在getObject方法里。当要在缓存中取对象时，先删除keyList中对应的key值，然后再加入到keyList的末尾，这样keyList中按顺序越是靠前的对象越是最近最少使用的对象。LruCacheController要依靠keyList实现LRU，所以FIFO里的两个集合不同步的bug在LRU里依然存在。

{% codeblock MemoryCacheController.java lang:java %}
/**
 * Memory-based implementation of CacheController
 */
public class MemoryCacheController implements CacheController {

  private MemoryCacheLevel referenceType = MemoryCacheLevel.WEAK;
  private Map cache = Collections.synchronizedMap(new HashMap());

  /**
   * Configures the cache
   *
   * @param props Optionally can contain properties [reference-type=WEAK|SOFT|STRONG]
   */
  public void setProperties(Properties props) {
    String refType = props.getProperty("reference-type");
    if (refType == null) {
      refType = props.getProperty("referenceType");
    }
    if (refType != null) {
      referenceType = MemoryCacheLevel.getByReferenceType(refType);
    }
  }

  public MemoryCacheLevel getReferenceType() {
    return referenceType;
  }

  public void setReferenceType(MemoryCacheLevel referenceType) {
    this.referenceType = referenceType;
  }

  /**
   * Add an object to the cache
   *
   * @param cacheModel The cacheModel
   * @param key        The key of the object to be cached
   * @param value      The object to be cached
   */
  public void putObject(CacheModel cacheModel, Object key, Object value) {
    Object reference = null;
    if (referenceType.equals(MemoryCacheLevel.WEAK)) {
      reference = new WeakReference(value);
    } else if (referenceType.equals(MemoryCacheLevel.SOFT)) {
      reference = new SoftReference(value);
    } else if (referenceType.equals(MemoryCacheLevel.STRONG)) {
      reference = new StrongReference(value);
    }
    cache.put(key, reference);
  }

  /**
   * Get an object out of the cache.
   *
   * @param cacheModel The cache model
   * @param key        The key of the object to be returned
   * @return The cached object (or null)
   */
  public Object getObject(CacheModel cacheModel, Object key) {
    Object value = null;
    Object ref = cache.get(key);
    if (ref != null) {
      if (ref instanceof StrongReference) {
        value = ((StrongReference) ref).get();
      } else if (ref instanceof SoftReference) {
        value = ((SoftReference) ref).get();
      } else if (ref instanceof WeakReference) {
        value = ((WeakReference) ref).get();
      }
    }
    return value;
  }

  public Object removeObject(CacheModel cacheModel, Object key) {
    Object value = null;
    Object ref = cache.remove(key);
    if (ref != null) {
      if (ref instanceof StrongReference) {
        value = ((StrongReference) ref).get();
      } else if (ref instanceof SoftReference) {
        value = ((SoftReference) ref).get();
      } else if (ref instanceof WeakReference) {
        value = ((WeakReference) ref).get();
      }
    }
    return value;
  }

  /**
   * Flushes the cache.
   *
   * @param cacheModel The cache model
   */
  public void flush(CacheModel cacheModel) {
    cache.clear();
  }

  /**
   * Class to implement a strong (permanent) reference.
   */
  private static class StrongReference {
    private Object object;

    /**
     * StrongReference constructor for an object
     * @param object - the Object to store
     */
    public StrongReference(Object object) {
      this.object = object;
    }

    /**
     * Getter to get the object stored in the StrongReference
     * @return - the stored Object
     */
    public Object get() {
      return object;
    }
  }

}
{% endcodeblock %}

基于内存的MemoryCacheController用到了MemoryCacheLevel这个类。在MemoryCacheLevel中定义了三种对象的引用类型：strong，soft和weak。关于Java的引用类型，可以参考我在我以前写的一篇博文[Java的四种引用类型](http://liuxuan.info/2011/03/java-four-types-of-reference/ "Java的四种引用类型")。总的来说，MemoryCacheController的实现还是比较简单的，但要注意到MemoryCacheController里没有keyList，所以不存在FIFO和LRU中都有的那个两个集合不同步的bug。

了解了缓存的实现之后，我们还要要搞清楚iBatis是什么时候把对象放到缓存中去的。其实根据常识，对象肯定是在第一次被从数据库中读取出来的时候存放到缓存中去的，接下去就来验证我们的猜想。

{% codeblock CachingStatement.java lang:java %}
  public CacheKey getCacheKey(StatementScope statementScope, Object parameterObject) {
    CacheKey key = statement.getCacheKey(statementScope, parameterObject);
    if (!cacheModel.isReadOnly() && !cacheModel.isSerialize()) {
      key.update(statementScope.getSession());
    }
    return key;
  }
  
  public Object executeQueryForObject(StatementScope statementScope, Transaction trans, 
  	Object parameterObject, Object resultObject)throws SQLException {
  	
    CacheKey cacheKey = getCacheKey(statementScope, parameterObject);
    cacheKey.update("executeQueryForObject");
    Object object = cacheModel.getObject(cacheKey);
    if (object == CacheModel.NULL_OBJECT){
    	//	This was cached, but null
    	object = null;
    }else if (object == null) {
       object = statement.executeQueryForObject(statementScope, trans, parameterObject, resultObject);
       cacheModel.putObject(cacheKey, object);
    }
    return object;
  }

  public List executeQueryForList(StatementScope statementScope, Transaction trans, Object parameterObject, 
  	int skipResults, int maxResults)throws SQLException {
  	
    CacheKey cacheKey = getCacheKey(statementScope, parameterObject);
    cacheKey.update("executeQueryForList");
    cacheKey.update(skipResults);
    cacheKey.update(maxResults);
    Object listAsObject = cacheModel.getObject(cacheKey);
    List list;
    if(listAsObject == CacheModel.NULL_OBJECT){
      // The cached object was null
      list = null;
    }else if (listAsObject == null) {
      list = statement.executeQueryForList(statementScope, trans, parameterObject, skipResults, maxResults);
      cacheModel.putObject(cacheKey, list);
    }else{
      list = (List) listAsObject;
    }
    return list;
  }
{% endcodeblock %}

我们可以看到，在executeQueryForObject和executeQueryForList方法中，首先根据对象的key值到缓存中获取对象，如果没有找到对应的对象就通过调用下面的方法来把对象存放到缓存中：

{% codeblock lang:java %}
cacheModel.putObject(cacheKey, object);
{% endcodeblock %}

这样我们之前的猜想就得到了验证。我们再来看看缓存中对象对应的key是什么东西，定位到CacheModel类的putObject方法：

{% codeblock lang:java %}
  private CacheController controller;

 /**
   * Add an object to the cache
   *
   * @param key   The key of the object to be cached
   * @param value The object to be cached
   */
  public void putObject(CacheKey key, Object value) {
  	if (null == value) value = NULL_OBJECT;
  	synchronized ( this )  {
      if (serialize && !readOnly && value != NULL_OBJECT) {
        try {
          ByteArrayOutputStream bos = new ByteArrayOutputStream();
          ObjectOutputStream oos = new ObjectOutputStream(bos);
          oos.writeObject(value);
          oos.flush();
          oos.close();
          value = bos.toByteArray();
        } catch (IOException e) {
          throw new RuntimeException("Error caching serializable object.  Cause: " + e, e);
        }
      }
      controller.putObject(this, key, value);
      if ( log.isDebugEnabled() )  {
        log("stored object", true, value);
      }
    }
  }
{% endcodeblock %}

CacheModel类里的putObject方法又调用了CacheController的putObject方法，注意传入的key值是CacheKey对象，也就是说最后作为缓存中对象的key是它的CacheKey对象。不得不说这是一个失败的设计，key值的类型是String类型就已经足够了，完全没有必要用对象类型来做key值的类型。因为内存空间是有限的，要在有限的空间中尽可能地存放更多的内容，就需要key值在保证唯一性的情况下空间占的越小越好。

**二.myBatis**

1.myBatis的cache包结构：

<table border="1" width="100%" cellpadding="3" cellspacing="0" summary="">
<tbody>
<tr>
<th align="left" colspan="2" style="background-color:#CCCCFF">
<b>Packages</b>
</th>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache.decorators</b></td>
<td>&nbsp;</td>
</tr>
<tr bgcolor="white">
<td width="20%"><b>org.apache.ibatis.cache.impl</b></td>
<td>&nbsp;</td>
</tr>
</tbody>
</table>

相比iBatis来说，myBatis的包结构变得更简洁，从五个包简化成了三个包。而且从实现方式上也有了改变，从decorators这样的包名可以猜到在这个包下的类都是装饰类，而实际上也是如此的。myBatis的缓存实现采用了装饰模式，impl包下实现的是缓存的基本功能，不同的缓存策略由decorators包下的类实现。

相比iBatis，myBatis增加了多个缓存实现：

* LoggingCache
* ScheduledCache
* SerializedCache
* SynchronizedCache
* TransactionalCache

2.源码分析

{% codeblock Cache.java lang:java %}
public interface Cache {

  String getId();

  int getSize();

  void putObject(Object key, Object value);

  Object getObject(Object key);

  Object removeObject(Object key);

  void clear();

  ReadWriteLock getReadWriteLock();

}
{% endcodeblock %}

和iBatis不同，Cache接口中定义了concurrent包中的ReadWriteLock接口，而它只有一个实现类ReentrantReadWriteLock。对比iBatis使用同步的Map集合来解决解决并发读写时缓存数据的同步问题，myBatis则使用ReentrantReadWriteLock类的锁机制来防止在并发写过程中的线程安全问题。

{% codeblock LruCache.java lang:java %}
/**
 * Lru (first in, first out) cache decorator
 */
public class LruCache implements Cache {

  private final Cache delegate;
  private Map&lt;object , Object&gt; keyMap;
  private Object eldestKey;

  public LruCache(Cache delegate) {

    this.delegate = delegate;
    setSize(1024);

  }

  public String getId() {
    return delegate.getId();
  }

  public int getSize() {
    return delegate.getSize();
  }

  public void setSize(final int size) {
    keyMap = new LinkedHashMap&lt;object , Object&gt;(size, .75F, true) {
      private static final long serialVersionUID = 4267176411845948333L;
      protected boolean removeEldestEntry(Map.Entry&lt;object , Object&gt; eldest) {
        boolean tooBig = size() > size;
        if (tooBig) {
          eldestKey = eldest.getKey();
        }
        return tooBig;
      }
    };
  }

  public void putObject(Object key, Object value) {
    delegate.putObject(key, value);
    cycleKeyList(key);
  }

  public Object getObject(Object key) {
    keyMap.get(key); //touch
    return delegate.getObject(key);
  }

  public Object removeObject(Object key) {
    return delegate.removeObject(key);
  }

  public void clear() {
    delegate.clear();
    keyMap.clear();
  }

  public ReadWriteLock getReadWriteLock() {
    return delegate.getReadWriteLock();
  }

  private void cycleKeyList(Object key) {
    keyMap.put(key, key);
    if (eldestKey != null) {
      delegate.removeObject(eldestKey);
      eldestKey = null;
    }
  }
}
{% endcodeblock %}

LRU是最常使用的缓存算法，所以就以LruCache为例，其他缓存实现也是差不多的。在iBatis里，key值是用一个同步的LinkedList来维护的，而myBatis则是改用LinkedHashMap来维护。这样做的好处就是解决了在iBatis中的HashMap和LinkedList的不同步问题，而且也不需要自己实现LRU，LinkedHashMap原生就支持LRU。

缓存中的对象是存放在impl包下的PerpetualCache类的HashMap里的，所以各个缓存算法实现类里都需要持有一个PerpetualCache实例。当要从缓存中读取数据时，就委托给PerpetualCache类去执行读取方法，而这对于调用者来说是透明的。

{% codeblock DefaultSqlSession.java lang:java %}
  private Configuration configuration;
  private Executor executor;

  private boolean autoCommit;
  private boolean dirty;

  public DefaultSqlSession(Configuration configuration, Executor executor, boolean autoCommit) {
    this.configuration = configuration;
    this.executor = executor;
    this.autoCommit = autoCommit;
    this.dirty = false;
  }

  public List selectList(String statement, Object parameter, RowBounds rowBounds) {
    try {
      MappedStatement ms = configuration.getMappedStatement(statement);
      return executor.query(ms, wrapCollection(parameter), rowBounds, Executor.NO_RESULT_HANDLER);
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
{% endcodeblock %}

DefaultSqlSession在创建完成后就有了对应的Executor实例，查询就调用Executor实例的query方法。那DefaultSqlSession是在哪里被创建的呢？

{% codeblock DefaultSqlSessionFactory.java lang:java %}
  private final Configuration configuration;

  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level,   boolean autoCommit) {
    Connection connection = null;
    try {
      final Environment environment = configuration.getEnvironment();
      final DataSource dataSource = getDataSourceFromEnvironment(environment);
      TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      connection = dataSource.getConnection();
      if (level != null) {
        connection.setTransactionIsolation(level.getLevel());
      }
      connection = wrapConnection(connection);
      Transaction tx = transactionFactory.newTransaction(connection, autoCommit);
      Executor executor = configuration.newExecutor(tx, execType);
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      closeConnection(connection);
      throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
    } finally {
      ErrorContext.instance().reset();
    }
  }
{% endcodeblock %}

可以看到DefaultSqlSession是在打开session时创建的，而且在创建DefaultSqlSession之前，通过调用了Configuration类的newExecutor方法创建相应的Executor实例。

{% codeblock Configuration.java lang:java %}
  public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
    executorType = executorType == null ? defaultExecutorType : executorType;
    executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
    Executor executor;
    if (ExecutorType.BATCH == executorType) {
      executor = new BatchExecutor(this, transaction);
    } else if (ExecutorType.REUSE == executorType) {
      executor = new ReuseExecutor(this, transaction);
    } else {
      executor = new SimpleExecutor(this, transaction);
    }
    if (cacheEnabled) {
      executor = new CachingExecutor(executor);
    }
    executor = (Executor) interceptorChain.pluginAll(executor);
    return executor;
  }
{% endcodeblock %}

代码还是比较清晰地，通过读取配置文件里的内容来创建Executor实例。到这里为止，在DefaultSqlSession创建完成后就会持有相应的Executor实例，如果是配置了缓存，那么将会创建CachingExecutor。而且CachingExecutor类里将会持有下列Executor的其中一种：BatchExecutor，ReuseExecutor和SimpleExecutor。

{% codeblock CachingExecutor.java lang:java %}
public class CachingExecutor implements Executor {

  private Executor delegate;
  private TransactionalCacheManager tcm = new TransactionalCacheManager();

  public CachingExecutor(Executor delegate) {
    this.delegate = delegate;
  }

  public List query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
    if (ms != null) {
      Cache cache = ms.getCache();
      if (cache != null) {
        flushCacheIfRequired(ms);
        cache.getReadWriteLock().readLock().lock();
        try {
          if (ms.isUseCache() && resultHandler == null) {
            CacheKey key = createCacheKey(ms, parameterObject, rowBounds);
            final List cachedList = (List) cache.getObject(key);
            if (cachedList != null) {
              return cachedList;
            } else {
              List list = delegate.query(ms, parameterObject, rowBounds, resultHandler);
              tcm.putObject(cache, key, list);
              return list;
            }
          } else {
            return delegate.query(ms, parameterObject, rowBounds, resultHandler);
          }
        } finally {
          cache.getReadWriteLock().readLock().unlock();
        }
      }
    }
    return delegate.query(ms, parameterObject, rowBounds, resultHandler);
  }

  public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds) {
    return delegate.createCacheKey(ms, parameterObject, rowBounds);
  }
}
{% endcodeblock %}

与iBatis最大的不同是，在从缓存中读取数据之前，要先获取读锁。读锁可以由多个读线程同时持有，而写线程同时只能有一个线程持有。但是读写锁会有一个饥渴写的问题，具体可以参考[读写锁的问题](http://en.wikipedia.org/wiki/Readers-writers_problem "Readers-writers_problem")。

Executor类也是采用了装饰模式来实现，查询对象委托给持有的Executor类的实例来执行。如果缓存中没有，就交TransactionalCacheManager类来把对象存放到缓存中去。

{% codeblock TransactionalCacheManager.java lang:java %}
public class TransactionalCacheManager {

  private Map&lt;cache , TransactionalCache&gt; transactionalCaches = new HashMap&lt;cache , TransactionalCache&gt;();

  public void clear(Cache cache) {
    getTransactionalCache(cache).clear();
  }

  public void putObject(Cache cache, CacheKey key, Object value) {
    getTransactionalCache(cache).putObject(key, value);
  }

  public void commit() {
    for (TransactionalCache txCache : transactionalCaches.values()) {
      txCache.commit();
    }
  }

  public void rollback() {
    for (TransactionalCache txCache : transactionalCaches.values()) {
      txCache.rollback();
    }
  }

  private TransactionalCache getTransactionalCache(Cache cache) {
    TransactionalCache txCache = transactionalCaches.get(cache);
    if (txCache == null) {
      txCache = new TransactionalCache(cache);
      transactionalCaches.put(cache, txCache);
    }
    return txCache;
  }

}
{% endcodeblock %}

第一次看到这个类的名字感觉有点奇怪，缓存相关的类怎么取了个事务的名字，后来读到TransactionalCache类后才明白原因。继续往下看，在TransactionalCacheManager中，最终还是用TransactionalCache类的put方法。

{% codeblock TransactionalCache.java lang:java %}
public class TransactionalCache implements Cache {

  private Cache delegate;
  private boolean clearOnCommit;
  private Map&lt;object , AddEntry&gt; entriesToAddOnCommit;
  private Map&lt;object , RemoveEntry&gt; entriesToRemoveOnCommit;
  
  public void putObject(Object key, Object object) {
    entriesToRemoveOnCommit.remove(key);
    entriesToAddOnCommit.put(key, new AddEntry(delegate, key, object));
  }

  public Object removeObject(Object key) {
    entriesToAddOnCommit.remove(key);
    entriesToRemoveOnCommit.put(key, new RemoveEntry(delegate, key));
    return delegate.getObject(key);
  }

  public void commit() {
    delegate.getReadWriteLock().writeLock().lock();
    try {
      if (clearOnCommit) {
        delegate.clear();
      } else {
        for (RemoveEntry entry : entriesToRemoveOnCommit.values()) {
          entry.commit();
        }
      }
      for (AddEntry entry : entriesToAddOnCommit.values()) {
        entry.commit();
      }
      reset();
    } finally {
      delegate.getReadWriteLock().writeLock().unlock();
    }
  }
}
{% endcodeblock %}

我们可以看到在TransactionalCache类里也维护着两个HashMap：entriesToAddOnCommit和entriesToRemoveOnCommit。当在TransactionalCacheManager中调用putObject和removeObject方法的时候并不是马上就把对象存放到缓存或者从缓存中删除，而是先把这个对象放到这两个HashMap之中的一个里，然后当执行commit方法时再真正地把对象存放到缓存或者从缓存中删除。现在我们应该可以明白为TransactionalCacheManager和TransactionalCache这两个类要加上事务的前缀了，因为commit方法是一个原子操作，一次会操作多个对象，要么一起成功，要么就一起失败。

**三.总结**

通过这次对iBatis以及myBatis的源码进行分析，发现他们的缓存在设计和实现上都存在一些问题。比如iBatis有两个集合不同步的问题，myBatis的读写锁有写饥渴问题等。这些问题都会给性能造成影响，所以还是不建议在生产环境中使用iBatis或者myBatis自带的二级缓存，只使用他们的ORM功能，而二级缓存还是交给Memcached来实现吧。

