### Spring 连接 redis 单机 集群实例

> 在虚拟机上装的redis，本地java客户端连接时出现超时，这个时候要去检查redis.config 里面的bind ip 是不是虚拟机的ip,第二个也就是检查centos 的防火墙，最好关闭防火墙

操作环境：

 VMware® Workstation 15 Pro:15.0.1 build-10737736

 centos:CentOS-7-x86_64-DVD-1810

 redis:4.0.8   jdk:1.8  Maven:3.5.4 

首先让我们来看看项目引入的pom文件，jedis为我们封装了底层的命令，我们只需要去调用它的API就好了

```x&#39;m&#39;l
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.0.9.RELEASE</version>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
 <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
         <scope>test</scope>
  </dependency>
```

下面通过一段简单的代码就能去操作redis,调用命令

```java
  //连接redis服务器(在这里是连接本地的)
    Jedis jedis = new Jedis("192.168.204.135",6379);
    //权限认证
    System.out.println("连接服务成功"+jedis.ping());

   //添加数据
    jedis.set("name", "chx"); //key为name放入value值为chx
    System.out.println("拼接前:" + jedis.get("name"));//读取key为name的值

    //向key为name的值后面加上数据 ---拼接
    jedis.append("name", " is my name;");
    System.out.println("拼接后:" + jedis.get("name"));

    //删除某个键值对
    jedis.del("name");
    System.out.println("删除后:" + jedis.get("name"));

    //s设置多个键值对
    jedis.mset("name", "chenhaoxiang", "age", "20", "email", "chxpostbox@outlook.com");
    jedis.incr("age");//用于将键的整数值递增1。如果键不存在，则在执行操作之前将其设置为0。 如果键包含错误类型的值或包含无法表示为整数的字符串，则会返回错误。此操作限于64位有符号整数。
    System.out.println(jedis.get("name") + " " + jedis.get("age") + " " + jedis.get("email"));

    //添加数据
    Map<String, String> map = new HashMap<String, String>();
    map.put("name", "chx");
    map.put("age", "100");
    map.put("email", "***@outlook.com");
    jedis.hmset("user", map);
    //取出user中的name，结果是一个泛型的List
    //第一个参数是存入redis中map对象的key，后面跟的是放入map中的对象的key，后面的key是可变参数
    List<String> list = jedis.hmget("user", "name", "age", "email");
    System.out.println(list);

    //删除map中的某个键值
    jedis.hdel("user", "age");
    System.out.println("age:" + jedis.hmget("user", "age")); //因为删除了，所以返回的是null
    System.out.println("user的键中存放的值的个数:" + jedis.hlen("user")); //返回key为user的键中存放的值的个数2
    System.out.println("是否存在key为user的记录:" + jedis.exists("user"));//是否存在key为user的记录 返回true
    System.out.println("user对象中的所有key:" + jedis.hkeys("user"));//返回user对象中的所有key
    System.out.println("user对象中的所有value:" + jedis.hvals("user"));//返回map对象中的所有value

    //拿到key，再通过迭代器得到值
    Iterator<String> iterator = jedis.hkeys("user").iterator();
    while (iterator.hasNext()) {
        String key = iterator.next();
        System.out.println(key + ":" + jedis.hmget("user", key));
    }
    jedis.del("user");
    System.out.println("删除后是否存在key为user的记录:" + jedis.exists("user"));//是否存在key为user的记录
```

上面我们可以看到已经可以正常的操作redis了，但是在正常的项目我们应该将配置与代码分离。下面将配置放在xml里面并配置redis连接池

redis.properties

```properties
redis.host=192.168.204.135
redis.port=6379
redis.maxWait=1000000
redis.maxIdle=300
redis.maxTotal=60000
```

spring-redis.xml

```xml
// 读取配置文件内容
<context:property-placeholder location="classpath:redis.properties" ignore-unresolvable="true"/>
<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
<property name="maxIdle" value="${redis.maxIdle}" /> <!-- 最大能够保持idel状态的对象数  -->
<property name="maxTotal" value="${redis.maxTotal}" /> <!-- 最大分配的对象数 -->
<property name="testOnBorrow" value="true" /> <!-- 当调用borrow Object方法时，是否进行有效性检查 -->
</bean>

<bean id="jedisPool" class="redis.clients.jedis.JedisPool">
    <constructor-arg name="poolConfig" ref="jedisPoolConfig" />
    <constructor-arg name="host" value="${redis.host}" />
    <constructor-arg name="port" value="${redis.port}" type="int" />
    <constructor-arg name="timeout" value="${redis.maxWait}"/>
</bean>
```
下面代码是测试是否连接成功
redisTest.java

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:Spring/spring-redis.xml"})
public class UserTest {
    @Autowired
    private JedisPool jedisPool;

    @Test
    public  void testJedisPool(){
        System.out.println(jedisPool);
        Jedis jedis = jedisPool.getResource();
        System.out.println(jedis.set("love","son"));
        System.out.println(jedis.get("love"));
        System.out.println(jedis.keys("*"));
    }
}
```

接下来就是如何连接集群的redis，其实也比较简单就是配置多个节点，首先是配置文件。

redis-cluster.properties

```properties
address=192.168.204.135
redis.maxIdle=100
redis.maxTotal=60000
```
xml 文件配置多个类
spring-redisCluster.xml

```xml
<!-- 读取配置文件信息  // 集群版 -->
<context:property-placeholder ignore-unresolvable="true" location="classpath:redis-cluster.properties"/>

<!-- jedis cluster config -->
<bean id="genericObjectPoolConfig" class="org.apache.commons.pool2.impl.GenericObjectPoolConfig">
    <property name="maxTotal" value="1024" />
    <property name="minIdle" value="8" />
    <property name="maxIdle" value="100" />
    <property name="maxWaitMillis" value="1000" />
    <property name="testOnBorrow" value="true" />
</bean>

<!-- 配置Cluster -->
<bean id="redisClusterConfiguration"
      class="org.springframework.data.redis.connection.RedisClusterConfiguration">
    <property name="maxRedirects" value="3"></property>
    <!-- 节点配置 -->
    <property name="clusterNodes">
        <set>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7001"></constructor-arg>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7002"></constructor-arg>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7000"></constructor-arg>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7003"></constructor-arg>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7004"></constructor-arg>
            </bean>
            <bean class="org.springframework.data.redis.connection.RedisClusterNode">
                <constructor-arg name="host" value="${address}"></constructor-arg>
                <constructor-arg name="port" value="7005"></constructor-arg>
            </bean>
        </set>
    </property>
</bean>

<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig">
    <property name="maxIdle" value="${redis.maxIdle}" />
    <property name="maxTotal" value="${redis.maxTotal}" />
</bean>
<bean id="jeidsConnectionFactory"
      class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
    <constructor-arg ref="redisClusterConfiguration" />
    <constructor-arg ref="jedisPoolConfig" />
</bean>
<!-- redis 访问的模版 -->
<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
    <property name="connectionFactory" ref="jeidsConnectionFactory" />
</bean>
```

测试集群是否连接成功
TestRedisCluster

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:Spring/spring-redisCluster.xml"})
public class redisClusterTest {

    @Autowired
    private RedisTemplate redisTemplate;

    @Test
    public void testRedisCluster (){
        System.out.println(redisTemplate);

        redisTemplate.opsForSet().add("a","A");
        System.out.println(redisTemplate.keys("*"));


    }
}
```

[spring集群连接参考](https://www.cnblogs.com/EasonJim/p/7805297.html)<br>
[虚拟机cnetos搭建单机redis集群](http://blog.51cto.com/moerjinrong/2089118)