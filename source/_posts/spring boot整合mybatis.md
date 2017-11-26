title: Spring boot整合mybatis
categories: 
  - 后端开发
  - java
tags:
  - Spring boot
date: 2016-12-28 22:20:33
author: mzeht
avatar: /images/favicon.png
---
# 基本配置
## 配置数据库

```
@Configuration
@EnableTransactionManagement
@PropertySource(value = {"classpath:config/source.properties"})
public class BeanConfig {

    @Autowired
    private Environment env;

    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        DruidDataSource dataSource = new DruidDataSource();
        dataSource.setDriverClassName(env.getProperty("source.driverClassName").trim());
        dataSource.setUrl(env.getProperty("source.url").trim());
        dataSource.setUsername(env.getProperty("source.username").trim());
        dataSource.setPassword(env.getProperty("source.password").trim());
        return dataSource;
    }

    @Bean
    public JdbcTemplate jdbcTemplate() {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource());
        return jdbcTemplate;
    }
}
```

默认配置文件配置端口和Mapper文件位置


<!-- more -->

```
# mysql
spring.datasource.one.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.one.url=jdbc:mysql://localhost:3306/wpspringboot?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&zeroDateTimeBehavior=convertToNull
spring.datasource.one.username=root
spring.datasource.one.password=root

spring.datasource.two.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.two.url=jdbc:mysql://localhost:3306/wpspringboottwo?useUnicode=true&characterEncoding=utf-8&autoReconnect=true&zeroDateTimeBehavior=convertToNull
spring.datasource.two.username=root
spring.datasource.two.password=root

spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/wpspringboot?useUnicode = true&characterEncoding=UTF-8&&autoReconnect=true&zeroDateTimeBehavior=convertToNull
spring.datasource.username=root
spring.datasource.password=root

server.port=9688
#mybatis配置文件位置
mybatis.mapper-locations=classpath*:mybatis/*Mapper.xml

```

自定义配置文件配置数据库地址

```
# mysql
source.driverClassName = com.mysql.jdbc.Driver
source.url = jdbc:mysql://localhost:3306/wpspringboot?useUnicode = true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
source.username = root
source.password = root
```

## 配置实体类

```
public class Author {
    private Long id;
    private String realName;
    private String nickName;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getRealName() {
        return realName;
    }

    public void setRealName(String realName) {
        this.realName = realName;
    }

    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

}
```

## 配置Mapper对应接口

使用@Mapper注解

```
@Mapper
public interface AuthorMapper {


    public Author findAuthor(Long id);
}

```

## 配置Mapper文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.AuthorMapper">


	<resultMap id="authorResultMap" type="com.example.po.Author">
		<id property="id" column="id" />
		<result property="realName" column="real_name" />
		<result property="nickName" column="nick_name" />
	</resultMap>

	<select id="findAuthor" parameterType="com.example.po.Author" resultMap="authorResultMap">
		select id, real_name, nick_name from t_author where id = #{id}
	</select>
</mapper> 
```

## 准备service

```
@Service
public class AuthorService {

    @Autowired
    private AuthorMapper authorMapper;



    public Author findAuthor(Long userId) {
        return authorMapper.findAuthor(userId);
    }
}
```

## Controller层

```
@RestController
@RequestMapping(value="/mybatis/author")
public class AuthorController {

    @Autowired
    private AuthorService authorService;
//    /**
//     * 查询用户列表
//     */
//    @RequestMapping(method = RequestMethod.GET)
//    public Map<String,Object> getAuthorList(HttpServletRequest request) {
//        List<Author> authorList = this.authorService.findAuthorList();
//        Map<String,Object> param = new HashMap<String,Object>();
//        param.put("total", authorList.size());
//        param.put("rows", authorList);
//        return param;
//    }
    /**
     * 查询用户信息
     */
    @RequestMapping(value = "/{userId:\\d+}", method = RequestMethod.GET)
    public Author getAuthor(@PathVariable Long userId, HttpServletRequest request) {
        Author author = this.authorService.findAuthor(userId);
        if(author == null){
            throw new RuntimeException("查询错误");
        }
        return author;
    }
//
//    /**
//     * 新增方法
//     */
//    @RequestMapping(method = RequestMethod.POST)
//    public void add(@RequestBody JSONObject jsonObject) {
//        String userId = jsonObject.getString("user_id");
//        String realName = jsonObject.getString("real_name");
//        String nickName = jsonObject.getString("nick_name");
//
//        try{
//            this.authorService.add(realName, nickName);
//        }catch(Exception e){
//            e.printStackTrace();
//            throw new RuntimeException("新增错误");
//        }
//    }
//    /**
//     * 更新方法
//     */
//    @RequestMapping(value = "/{userId:\\d+}", method = RequestMethod.PUT)
//    public void update(@PathVariable Long userId, @RequestBody JSONObject jsonObject) {
//        Author author = this.authorService.findAuthor(userId);
//        String realName = jsonObject.getString("real_name");
//        String nickName = jsonObject.getString("nick_name");
//
//        try{
//            this.authorService.update(realName, nickName, author.getId());
//        }catch(Exception e){
//            e.printStackTrace();
//            throw new RuntimeException("更新错误");
//        }
//    }
//    /**
//     * 删除方法
//     */
//    @RequestMapping(value = "/{userId:\\d+}", method = RequestMethod.DELETE)
//    public void delete(@PathVariable Long userId) {
//        try{
//            this.authorService.delete(userId);
//        }catch(Exception e){
//            throw new RuntimeException("删除错误");
//        }
//    }
}
```

测试通过

# 配置日志
mybatis配置日志打印
resources文件夹下新建logback.xml

```

<?xml version="1.0" encoding="UTF-8"?>

<configuration scan="true">
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <charset>utf-8</charset>
            <Pattern>%d %-5level [%thread] %logger{0}: %msg%n</Pattern>
        </encoder>
    </appender>

<!-- The FILE and ASYNC appenders are here as examples for a production configuration -->

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>log/wpspringbootmybatis.%d{yyyy-MM-dd}.log.zip</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        <encoder>
            <charset>utf-8</charset>
            <Pattern>%d %-5level [%thread] %logger{0}: %msg%n</Pattern>
        </encoder>
    </appender>

    <appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>512</queueSize>
        <appender-ref ref="FILE"/>
    </appender>

    <logger name="cn.upg" level="${logback.loglevel}"/>

    <logger name="javax.activation" level="WARN"/>
    <logger name="javax.mail" level="WARN"/>
    <logger name="javax.xml.bind" level="WARN"/>
    <logger name="ch.qos.logback" level="WARN"/>
    <logger name="com.codahale.metrics" level="WARN"/>
    <logger name="com.ryantenney" level="WARN"/>
    <logger name="com.sun.xml.internal.bind" level="WARN"/>
    <logger name="com.zaxxer" level="WARN"/>
    <logger name="io.undertow" level="WARN"/>
    <logger name="org.apache" level="WARN"/>
    <logger name="org.apache.catalina.startup.DigesterFactory" level="OFF"/>
    <logger name="org.bson" level="WARN"/>
    <logger name="org.hibernate.validator" level="WARN"/>
    <logger name="org.hibernate" level="WARN"/>
    <logger name="org.hibernate.ejb.HibernatePersistence" level="OFF"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.springframework.web" level="WARN"/>
    <logger name="org.springframework.security" level="WARN"/>
    <logger name="org.springframework.cache" level="WARN"/>
    <logger name="org.thymeleaf" level="WARN"/>
    <logger name="org.xnio" level="WARN"/>
    <logger name="com.mangofactory.swagger" level="INFO"/>
    <logger name="com.wordnik.swagger" level="WARN"/>
    <logger name="sun.rmi.transport" level="WARN"/>

    <logger name="redis.clients.jedis" level="WARN" />
    <logger name="org.springframework.data.redis" level="WARN" />
    <logger name="org.springframework.data.mongodb" level="INFO"/>

    <contextListener class="ch.qos.logback.classic.jul.LevelChangePropagator">
        <resetJUL>true</resetJUL>
    </contextListener>

    <root level="${logback.loglevel}">
        <appender-ref ref="ASYNC"/>
        <appender-ref ref="CONSOLE"/>
    </root>

</configuration>

```

效果


```
2016-12-30 16:16:54,216 DEBUG [http-nio-9688-exec-1] SqlSessionUtils: Creating a new SqlSession
2016-12-30 16:16:54,225 DEBUG [http-nio-9688-exec-1] SqlSessionUtils: SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7c3e793d] was not registered for synchronization because synchronization is not active
2016-12-30 16:16:54,356 INFO  [http-nio-9688-exec-1] DruidDataSource: {dataSource-1} inited
2016-12-30 16:16:54,621 DEBUG [http-nio-9688-exec-1] SpringManagedTransaction: JDBC Connection [com.mysql.jdbc.JDBC4Connection@20dc4786] will not be managed by Spring
2016-12-30 16:16:54,629 DEBUG [http-nio-9688-exec-1] findAuthor: ==>  Preparing: select id, real_name, nick_name from t_author where id = ? 
2016-12-30 16:16:54,672 DEBUG [http-nio-9688-exec-1] findAuthor: ==> Parameters: 1(Long)
2016-12-30 16:16:54,708 DEBUG [http-nio-9688-exec-1] findAuthor: <==      Total: 1
2016-12-30 16:16:54,710 DEBUG [http-nio-9688-exec-1] SqlSessionUtils: Closing non transactional SqlSession [org.apache.ibatis.session.defaults.DefaultSqlSession@7c3e793d]

```


-------
相关链接 
1. [spring boot日志管理](http://blog.didispace.com/springbootlog/)





