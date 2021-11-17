# JDK5, JDK 1.5 java Redis Client 

使用方法, 将lib文件夹下的2个jar包放入到你的项目中即可


## 与Spring集成

在Spring 的 application.xml 配置文件中增加以下配置
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:context="http://www.springframework.org/schema/context"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:tx="http://www.springframework.org/schema/tx"
  xsi:schemaLocation="http://www.springframework.org/schema/beans 
              http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
        http://www.springframework.org/schema/tx 
            http://www.springframework.org/schema/tx/spring-tx-2.5.xsd              
              http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-2.5.xsd">

<!-- other beans ..... -->


	<!-- jdk1.5 java redis  by Tekin -->
	<bean id="tredis" class="cn.tekin.jdk5.redis.Jdk5RedisClient" init-method="init">
		<!-- Redis服务名称 如果下面的propFilePath 未提供全路径,则必须指定使用redis的部署包名 JYLWSP.war -->
		<property name="name">
			<value>TRedis</value>
		</property>
		<!-- Redis服务类型 -->
		<property name="type">
			<value>redis</value>
		</property>
		<!-- redis key的默认前缀, 以_结尾,不设置请留空!-->
		<property name="keyPrefix">
			<value></value>
		</property>
		<!-- 是否启用Redis   false/true 默认false-->
		<property name="useRedis">
			<value>false</value>
		</property>
		<!-- 配置文件名称, 如果在deploy目录中则只需要提供名称即可, 如果在非deploy目录中则提供全路径 如: D:\myApp\redis.properties , 这个配置是为了将配置与WAR包分离,不使用请留空 这里的配置优先级最高  -->
		<property name="propsFile">
			<value></value>
		</property>
		<!-- Redis相关配置 -->
		<property name="config">
			<props>
				<!--Redis服务器地址 -->
				<prop key="host">127.0.0.1</prop>
				
				<!--Redis服务端口 -->
				<prop key="port">6379</prop>
				
				<!--Redis服务密码 不设置请留空 -->
				<prop key="pwd"></prop>
				
				<!--Redis链接的库 默认 0 -->
				<prop key="db">0</prop>
				
				<!-- 超时时间 默认60 -->
				<prop key="timeOut">60</prop>
				<!-- 超时单位 默认SECONDS 可选  NANOSECONDS, MICROSECONDS, MILLISECONDS  -->
				<prop key="timeUnit">SECONDS</prop>

				<prop key="chanel">tredis</prop>
			</props>
		</property>
	</bean>
    <!-- jdk1.5 java redis end -->
</beans>
~~~



### 将redis配置从 应用中拨离部署功能使用说明

 将redis配置信息放到应用外的目录  或者应用的部署目录后将文件或者路径信息配置号 propsFile 中即可实现, 注意这里的配置信息具有最高优先级!

如果不使用,则将  propsFile 置空.



-  将redis.properties文件复制到jboss/tomcat的deploy目录中,  deploy目录为你的jboss/tomcat的应用部署目录, 如  D:\Server\apache-tomcat-8.5.66\webapps   Jboss 的部署目录  jboss-4.x\server\default\deploy

- 第二中方法, 直接将 redis.properties文件的全路径配置到  propsFile  中 ,即可将配置和应用彻底脱离;

  


redis.properties 文件内容

~~~properties
keyPrefix=YNBS_
host=127.0.0.1
pwd=t8888
db=0
~~~


- 参数说明:

keyPrefix   全局的redis key前缀,  命名规则:   区域加部署站名简称首字母加下划线 _   ,   如:    云南保山站的前缀为    YNBS_   

pwd  redis应用的链接密码

useRedis  是否启用redis, 这里 需要设置为   true

db   redis默认链接的库  默认  0  , 多个站点使用同一个 redis服务时修改, 否则默认.


注意, 如果需要使用系统默认的配置, 则将redis.properties 文件中的对应项目删除即可!





## 使用说明



可以使用已经封装的 RRedisUtils 或者直接拿到 RedisConnection 后使用, 一般情况下建议直接使用 RRedisUtils 即可, 如果需要更高级的redis功能,可以直接拿到redisconnection后再使用.


### RRedisUtils操作工具类使用示例

~~~java
// 建议在使用前 先检测一下redis是否可用
    boolean isRedisActive = TRedisUtils.isRedisActive();//检查redis状态
    if(!isRedisActive){
      throw new Exception("Redis不可用");
    }


    TRedisUtil.set("test", "Tekin RedisClient  "+sdf.format(new Date())+" - "+System.nanoTime());
    TRedisUtil.setnx("expired_after_60sec", "key from lwsp"+sdf.format(new Date())+" - "+System.nanoTime());
    TRedisUtil.expire("expired_after_60sec", 60);


    TRedisUtil.setAndExpireDayEnd("curDateEnd", String.format("当天结束时过期  现在时间%s 过期时间:%s", sdf.format(new Date()),TimeUtils.getCurDayEndDateStr()) );
    //TRedisUtil.expireat("curDateEnd", TimeUtils.getCurDayEndLeftSec());

    TRedisUtil.setnxAndExpireNextDayBegin("setnxAndExpiredNextDayBegin", String.format("第二天开始时过期  现在时间%s 过期时间:%s", sdf.format(new Date()),TimeUtils.getNextDayBeginDateStr()));

    TRedisUtil.saddAndExpireAtDayEnd("km001", "aaaa","99999","99999","99998","99997");
    //添加set集合
    TRedisUtil.saddIntAndExpireAtDayEnd("km002", 999,888,22222,6666,6666,888);

    //检查集合中的元素是否存在
    boolean isExist = TRedisUtil.sismember("km002", "99900");


    System.out.println(TRedisUtil.get("curDateEnd"));

    TRedisUtil.smembers("km001");//返回集合中的所有元素


~~~

###  RRedisUtils已实现方法

~~~java
exists(String)
expire(String, long)
expireat(String, Date)
expireat(String, long)
get(String)
getbit(String, long)
getFk(String)
getrange(String, long, long)
getRedis()
getset(String, String)
hdel(String, String...)
hexists(String, String)
hget(String, String)
hgetall(String)
hincrby(String, String, long)
hincrbyfloat(String, String, double)
hkeys(String)
hlen(String)
hmget(String, String...)
hmset(String, Map<String, String>)
hset(String, String, String)
hsetnx(String, String, String)
hvals(String)
incr(String)
incrby(String, long)
incrbyfloat(String, double)
info()
info(String)
isRedisActive()
keys(String)
lastsave()
lindex(String, long)
linsert(String, boolean, String, String)
llen(String)
lpop(String)
lpush(String, String...)
lpushx(String, String)
lrange(String, long, long)
lrem(String, long, String)
lset(String, long, String)
ltrim(String, long, long)
mget(String...)
mset(Map<String, String>)
msetnx(Map<String, String>)
objectEncoding(String)
objectIdletime(String)
objectRefcount(String)
persist(String)
pexpire(String, long)
pexpireat(String, Date)
pexpireat(String, long)
pttl(String)
publish(String, String)
randomkey()
rename(String, String)
renamenx(String, String)
restore(String, long, byte[])
rpop(String)
rpoplpush(String, String)
rpush(String, String...)
rpushx(String, String)
sadd(String, String...)
saddAndExpireAtDayEnd(String, String...)
saddIntAndExpireAtDayEnd(String, int...)
save()
scard(String)
sdiff(String...)
sdiffstore(String, String...)
set(String, String)
setAndExpire(String, String, long)
setAndExpireAfterMin(String, String, long)
setAndExpiredAt(String, String, Date)
setAndExpireDayEnd(String, String)
setbit(String, long, int)
setex(String, long, String)
setnx(String, String)
setnxAndExpireNextDayBegin(String, String)
setrange(String, long, String)
sinter(String...)
sinterstore(String, String...)
sismember(String, String)
slowlogGet()
slowlogGet(int)
slowlogLen()
slowlogReset()
smembers(String)
smove(String, String, String)
sort(String)
sort(String, SortArgs)
sortStore(String, SortArgs, String)
spop(String)
srandmember(String)
srandmember(String, long)
srem(String, String...)
strlen(String)
sync()
ttl(String)
type(String)
unwatch()
watch(String...)
zadd(String, double, String)
zadd(String, Object...)
zcard(String)
zcount(String, double, double)
zcount(String, String, String)
zincrby(String, double, String)
zinterstore(String, String...)
zinterstore(String, ZStoreArgs, String...)
zrange(String, long, long)
zrangebyscore(String, double, double)
zrangebyscore(String, double, double, long, long)
zrangebyscore(String, String, String)
zrangebyscore(String, String, String, long, long)
zrangebyscoreWithScores(String, double, double)
zrangebyscoreWithScores(String, double, double, long, long)
zrangebyscoreWithScores(String, String, String)
zrangebyscoreWithScores(String, String, String, long, long)
zrangeWithScores(String, long, long)
zrank(String, String)
zrem(String, String...)
zremrangebyrank(String, long, long)
zremrangebyscore(String, double, double)
zremrangebyscore(String, String, String)
zrevrange(String, long, long)
zrevrangebyscore(String, double, double)
zrevrangebyscore(String, double, double, long, long)
zrevrangebyscore(String, String, String)
zrevrangebyscore(String, String, String, long, long)
zrevrangebyscoreWithScores(String, double, double)
zrevrangebyscoreWithScores(String, double, double, long, long)
zrevrangebyscoreWithScores(String, String, String)
zrevrangebyscoreWithScores(String, String, String, long, long)
zrevrangeWithScores(String, long, long)
zrevrank(String, String)
zscore(String, String)
zunionstore(String, String...)
zunionstore(String, ZStoreArgs, String...)
~~~




### RedisConnection 使用示例

- 基本用法
    ~~~java
     RedisClient client = new RedisClient("localhost");
      RedisConnection<String, String> connection = client.connect();
      String value = connection.get("key");
    ~~~



- 异步用法

  ~~~java
   RedisAsyncConnection<String, String> async = client.connectAsync()
    Future<String> set = async.set("key", "value")
    Future<String> get = async.get("key")
  
    async.awaitAll(set, get) == true
  
    set.get() == "OK"
    get.get() == "value"
  ~~~

  

- 发布订阅

    ~~~java
    RedisPubSubConnection<String, String> connection = client.connectPubSub()
      connection.addListener(new RedisPubSubListener<String, String>() { ... })
      connection.subscribe("channel")
    ~~~

- 设置自己的Codec

  ~~~java
  RedisConnection<K, V> connect(RedisCodec<K, V> codec)
    RedisAsyncConnection<K, V> connectAsync(RedisCodec<K, V> codec)
    RedisPubSubConnection<K, V> connectPubSub(RedisCodec<K, V> codec)
  ~~~

  




### RedisConnection已实现的方法


~~~java
RedisConnection(RedisAsyncConnection<K, V>)
append(K, V)
auth(String)
await(Future<T>)
await(Future<T>, long, TimeUnit)
bgrewriteaof()
bgsave()
bitcount(K)
bitcount(K, long, long)
bitopAnd(K, K...)
bitopNot(K, K)
bitopOr(K, K...)
bitopXor(K, K...)
blpop(long, K...)
brpop(long, K...)
brpoplpush(long, K, K)
clientGetname()
clientKill(String)
clientList()
clientSetname(K)
close()
configGet(String)
configResetstat()
configSet(String, String)
dbsize()
debugObject(K)
decr(K)
decrby(K, long)
del(K...)
digest(V)
discard()
dump(K)
echo(V)
eval(V, ScriptOutputType, K...)
eval(V, ScriptOutputType, K[], V...)
evalsha(String, ScriptOutputType, K...)
evalsha(String, ScriptOutputType, K[], V...)
exec()
exists(K)
expire(K, long)
expireat(K, Date)
expireat(K, long)
flushall()
flushdb()
get(K)
getbit(K, long)
getrange(K, long, long)
getset(K, V)
hdel(K, K...)
hexists(K, K)
hget(K, K)
hgetall(K)
hincrby(K, K, long)
hincrbyfloat(K, K, double)
hkeys(K)
hlen(K)
hmget(K, K...)
hmset(K, Map<K, V>)
hset(K, K, V)
hsetnx(K, K, V)
hvals(K)
incr(K)
incrby(K, long)
incrbyfloat(K, double)
info()
info(String)
keys(K)
lastsave()
lindex(K, long)
linsert(K, boolean, V, V)
llen(K)
lpop(K)
lpush(K, V...)
lpushx(K, V)
lrange(K, long, long)
lrem(K, long, V)
lset(K, long, V)
ltrim(K, long, long)
mget(K...)
migrate(String, int, K, int, long)
move(K, int)
mset(Map<K, V>)
msetnx(Map<K, V>)
multi()
objectEncoding(K)
objectIdletime(K)
objectRefcount(K)
persist(K)
pexpire(K, long)
pexpireat(K, Date)
pexpireat(K, long)
ping()
pttl(K)
publish(K, V)
quit()
randomkey()
rename(K, K)
renamenx(K, K)
restore(K, long, byte[])
rpop(K)
rpoplpush(K, K)
rpush(K, V...)
rpushx(K, V)
sadd(K, V...)
save()
scard(K)
scriptExists(String...)
scriptFlush()
scriptKill()
scriptLoad(V)
sdiff(K...)
sdiffstore(K, K...)
select(int)
set(K, V)
setbit(K, long, int)
setex(K, long, V)
setnx(K, V)
setrange(K, long, V)
setTimeout(long, TimeUnit)
shutdown()
shutdown(boolean)
sinter(K...)
sinterstore(K, K...)
sismember(K, V)
slaveof(String, int)
slaveofNoOne()
slowlogGet()
slowlogGet(int)
slowlogLen()
slowlogReset()
smembers(K)
smove(K, K, V)
sort(K)
sort(K, SortArgs)
sortStore(K, SortArgs, K)
spop(K)
srandmember(K)
srandmember(K, long)
srem(K, V...)
strlen(K)
sunion(K...)
sunionstore(K, K...)
sync()
ttl(K)
type(K)
unwatch()
watch(K...)
zadd(K, double, V)
zadd(K, Object...)
zcard(K)
zcount(K, double, double)
zcount(K, String, String)
zincrby(K, double, K)
zinterstore(K, K...)
zinterstore(K, ZStoreArgs, K...)
zrange(K, long, long)
zrangebyscore(K, double, double)
zrangebyscore(K, double, double, long, long)
zrangebyscore(K, String, String)
zrangebyscore(K, String, String, long, long)
zrangebyscoreWithScores(K, double, double)
zrangebyscoreWithScores(K, double, double, long, long)
zrangebyscoreWithScores(K, String, String)
zrangebyscoreWithScores(K, String, String, long, long)
zrangeWithScores(K, long, long)
zrank(K, V)
zrem(K, V...)
zremrangebyrank(K, long, long)
zremrangebyscore(K, double, double)
zremrangebyscore(K, String, String)
zrevrange(K, long, long)
zrevrangebyscore(K, double, double)
zrevrangebyscore(K, double, double, long, long)
zrevrangebyscore(K, String, String)
zrevrangebyscore(K, String, String, long, long)
zrevrangebyscoreWithScores(K, double, double)
zrevrangebyscoreWithScores(K, double, double, long, long)
zrevrangebyscoreWithScores(K, String, String)
zrevrangebyscoreWithScores(K, String, String, long, long)
zrevrangeWithScores(K, long, long)
zrevrank(K, V)
zscore(K, V)
zunionstore(K, K...)
zunionstore(K, ZStoreArgs, K...)
~~~


## 已知问题


如果使用了额外的配置文件, 在jboss 4.x版本部署的时候会有一个warn提示  Incompletely deployed packages  不必理会即可,这个是Jboss的问题 https://developer.jboss.org/thread/143576


