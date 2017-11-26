title: Spring boot整合redis
categories: 
  - 后端开发
  - java
tags:
  - Spring boot
  - redis
date: 2017-01-03 10:34:33
author: mzeht
avatar: /images/favicon.png
---

# 基本准备
1. 本地通过brew安装redis
1. 安装一个redis的可视化管理工具
我安装的是[Redis Desktop Manager for Mac](http://mac.softpedia.com/get/Utilities/Redis-Desktop-Manager.shtml)


<!-- more -->

#工程配置
## 新建配置文件 
redis.properties

```
# redis
redis.host=localhost
redis.port=6379
redis.password=
redis.database=1
```
## 注入数据源

```
@Configuration
@PropertySource("classpath:config/redis.properties")
public class RedisConfig {

    @Autowired
    private Environment env;
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
        jedisConnectionFactory.setHostName(env.getProperty("redis.host").trim());
        jedisConnectionFactory.setPort(Integer.parseInt(env.getProperty("redis.port").trim()));
        jedisConnectionFactory.setPassword(env.getProperty("redis.password").trim());
        jedisConnectionFactory.setDatabase(Integer.parseInt(env.getProperty("redis.database").trim()));
        jedisConnectionFactory.setUsePool(true);
        jedisConnectionFactory.setPoolConfig(jedisPoolConfig);
        return jedisConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, String> redisTemplate() {
        RedisTemplate<String, String> redisTemplate = new StringRedisTemplate();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
}
```

## 工具类

```
@Repository
public class RedisBaseDao {

    @Resource(name="redisTemplate")
    protected ValueOperations<String, String> valueOperations;

    public void addValue(String key, String value){
        valueOperations.set(key, value);
    }

    public String getValue(String key){
        return valueOperations.get(key);
    }
}

```



```
@Repository
public class ValueRedisDao {
    @Autowired
    public RedisBaseDao redisBaseDao;

    private String getKey(){
        return "param";
    }

    public void save(String param){
        this.redisBaseDao.addValue(this.getKey(), param);
    }

    public String getParam(){
        return this.redisBaseDao.getValue(this.getKey());
    }
}
```

## 测试类进行测试

```
@RunWith(SpringRunner.class)
@SpringBootTest
public class WpspringbootmybatisApplicationTests {

	@Autowired
	private StringRedisTemplate stringRedisTemplate;
	@Test
	public void test() throws Exception {
		// 保存字符串
		stringRedisTemplate.opsForValue().set("aaa", "111");
		Assert.assertEquals("111", stringRedisTemplate.opsForValue().get("aaa"));
	}

	@Autowired
	private ValueRedisDao valueRedisDao;

	@Test
	public void saveRedisTest() throws Exception {
		this.valueRedisDao.save("LiangGzone");
		System.out.println(this.valueRedisDao.getParam());
	}


	@Test
	public void readRedisTest() throws Exception{
		String value=valueRedisDao.getParam();
		System.out.println(value);
	}
}
```


结合可视化软件，可以确定redis的操作
![](http://7xqtsx.com1.z0.glb.clouddn.com/17-1-3/52959099-file_1483411275568_ad88.png)


不过这样的使用还是比较初级，之后结合公司的项目进行封装

