title: Spring boot搭建jdbc
categories: 
  - 后端开发
  - java
tags:
  - Spring boot
date: 2016-12-28 22:20:33
author: mzeht
avatar: /images/favicon.png
---

开发环境：idea


# 初步准备
利用idea创建一个基本的Spring boot工程
java sdk 暂时用 1.7
GUI 选择组建时什么都不选
添加为Maven项目
此时pom文件状态为

<!-- more -->

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>wpspringboot</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>jar</packaging>

	<name>wpspringboot</name>
	<description>Demo project for Spring Boot</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.3.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
		<java.version>1.7</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```

使用idea自带的命令行测试
打包
```
mvn clean && mvn package -Dmaven.test.skip 
```
构建成功后打包的jar生成在当前目录下的target/

```
INFO] Building jar: /Users/wpmac/IntellijProject/wpspringboot/target/wpspringboot-0.0.1-SNAPSHOT.jar
```
进入到jar包所在位置

```
java -jar wpspringboot-0.0.1-SNAPSHOT.jar
```
即可运行

```

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v1.4.3.RELEASE)

2016-12-28 19:12:06.392  INFO 3005 --- [           main] com.example.WpspringbootApplication      : Starting WpspringbootApplication v0.0.1-SNAPSHOT with PID 3005 (/Users/wpmac/IntellijProject/wpspringboot/target/wpspringboot-0.0.1-SNAPSHOT.jar started by wpmac in /Users/wpmac/IntellijProject/wpspringboot/target)
2016-12-28 19:12:06.397  INFO 3005 --- [           main] com.example.WpspringbootApplication      : No active profile set, falling back to default profiles: default
2016-12-28 19:12:06.511  INFO 3005 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.springframework.context.annotation.AnnotationConfigApplicationContext@31407cec: startup date [Wed Dec 28 19:12:06 CST 2016]; root of context hierarchy
2016-12-28 19:12:08.388  INFO 3005 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2016-12-28 19:12:08.436  INFO 3005 --- [           main] com.example.WpspringbootApplication      : Started WpspringbootApplication in 12.949 seconds (JVM running for 13.7)
2016-12-28 19:12:08.438  INFO 3005 --- [       Thread-1] s.c.a.AnnotationConfigApplicationContext : Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@31407cec: startup date [Wed Dec 28 19:12:06 CST 2016]; root of context hierarchy
2016-12-28 19:12:08.440  INFO 3005 --- [       Thread-1] o.s.j.e.a.AnnotationMBeanExporter        : Unregistering JMX-exposed beans on shutdown
```

添加web依赖

```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

测试类

```
@RestController
@SpringBootApplication
public class RestfulApiWebDemo {

    @RequestMapping("/")
    String home() {
        return "Hello World!";
    }
    public static void main(String[] args) throws Exception {
        SpringApplication.run(RestfulApiWebDemo.class, args);
    }
}
```

注释创建时生成的Application

```
//@SpringBootApplication
public class WpspringbootApplication {

//	public static void main(String[] args) {
//		SpringApplication.run(WpspringbootApplication.class, args);
//	}
}
```

重新打包运行，测试8080端口(spring boot默认端口)

```
➜  ~ curl 127.0.0.1:8080
Hello World!%
```

# 使用jdbc，mysql
## 添加jdbc依赖

```
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency> 

<!--mysql依赖-->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.35</version>
		</dependency>

		<!--连接池-->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.14</version>
		</dependency>
```
## 文件目录结构
![](http://7xqtsx.com1.z0.glb.clouddn.com/16-12-29/41925907-file_1482978251512_1187b.png)
##配置数据源信息
新建 src/main/resources/config/source.properties

```
# mysql
source.driverClassName = com.mysql.jdbc.Driver
source.url = jdbc:mysql://localhost:3306/wpspringboot
source.username = root
source.password = root
```
新建confgi.BeanConfig

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

## 新建实体

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

## Dao层


```
public interface AuthorDao {

    int add(Author author);
    int update(Author author);
    int delete(Long id);
    Author findAuthor(Long id);
    List<Author> findAuthorList();
}

```


```
@Repository
public class AuthorDaoImpl implements AuthorDao {
    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Override
    public int add(Author author) {
        return jdbcTemplate.update("insert into t_author(real_name, nick_name) values(?, ?)",
                author.getRealName(), author.getNickName());
    }

    @Override
    public int update(Author author) {
        return jdbcTemplate.update("update t_author set real_name = ?, nick_name = ? where id = ?",
                new Object[]{author.getRealName(), author.getNickName(), author.getId()});
    }

    @Override
    public int delete(Long id) {
        return jdbcTemplate.update("delete from t_author where id = ?", id);
    }

    @Override
    public Author findAuthor(Long id) {
        List<Author> list = jdbcTemplate.query("select * from t_author where id = ?", new Object[]{id}, new BeanPropertyRowMapper(Author.class));
        if(null != list && list.size()>0){
            Author auhtor = list.get(0);
            return auhtor;
        }else{
            return null;
        }
    }
    @Override
    public List<Author> findAuthorList() {
        List<Author> list = jdbcTemplate.query("select * from t_author", new Object[]{}, new BeanPropertyRowMapper<Author>(Author.class));
        return list;
    }
}
```

##  service层

```
public interface AuthorService {

    int add(Author author);
    int update(Author author);
    int delete(Long id);
    Author findAuthor(Long id);
    List<Author> findAuthorList();
}
```


```
@Service("authorService")
public class AuthorServiceImpl implements AuthorService {
    @Autowired
    private AuthorDao authorDao;

    @Override
    public int add(Author author) {
        return this.authorDao.add(author);
    }

    @Override
    public int update(Author author) {
        return this.authorDao.update(author);
    }

    @Override
    public int delete(Long id) {
        return this.authorDao.delete(id);
    }

    @Override
    public Author findAuthor(Long id) {
        return this.authorDao.findAuthor(id);
    }

    @Override
    public List<Author> findAuthorList() {
        return this.authorDao.findAuthorList();
    }
}

```

## controller层

```
@RestController
@RequestMapping(value="/jdbc/author")
public class AuthorController {

    @Autowired
    private AuthorService authorService;
    /**
     * 查询用户列表
     */
    @RequestMapping(method = RequestMethod.GET)
    public Map<String,Object> getAuthorList(HttpServletRequest request) {
        List<Author> authorList = this.authorService.findAuthorList();
        Map<String,Object> param = new HashMap<String,Object>();
        param.put("total", authorList.size());
        param.put("rows", authorList);
        return param;
    }
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

    /**
     * 新增方法
     */
    @RequestMapping(method = RequestMethod.POST)
    public void add(@RequestBody JSONObject jsonObject) {
        String userId = jsonObject.getString("user_id");
        String realName = jsonObject.getString("real_name");
        String nickName = jsonObject.getString("nick_name");
        Author author = new Author();
        if (author!=null) {
            author.setId(Long.valueOf(userId));
        }
        author.setRealName(realName);
        author.setNickName(nickName);
        try{
            this.authorService.add(author);
        }catch(Exception e){
            e.printStackTrace();
            throw new RuntimeException("新增错误");
        }
    }
    /**
     * 更新方法
     */
    @RequestMapping(value = "/{userId:\\d+}", method = RequestMethod.PUT)
    public void update(@PathVariable Long userId, @RequestBody JSONObject jsonObject) {
        Author author = this.authorService.findAuthor(userId);
        String realName = jsonObject.getString("real_name");
        String nickName = jsonObject.getString("nick_name");
        author.setRealName(realName);
        author.setNickName(nickName);
        try{
            this.authorService.update(author);
        }catch(Exception e){
            e.printStackTrace();
            throw new RuntimeException("更新错误");
        }
    }
    /**
     * 删除方法
     */
    @RequestMapping(value = "/{userId:\\d+}", method = RequestMethod.DELETE)
    public void delete(@PathVariable Long userId) {
        try{
            this.authorService.delete(userId);
        }catch(Exception e){
            throw new RuntimeException("删除错误");
        }
    }

}
```

打开本地数据库 用postman进行测试即可








