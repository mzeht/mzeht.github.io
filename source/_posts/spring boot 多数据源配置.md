title: Spring boot配置多数据源
categories: 
  - 后端开发
  - java
tags:
  - Spring boot
date: 2016-12-28 22:20:33
author: mzeht
avatar: /images/favicon.png
---

## 编辑配置文件

可以是默认的application.properties，亦可以是自定义的source.properties


<!-- more -->

```
# mysql
spring.datasource.one.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.one.url=jdbc:mysql://localhost:3306/wpspringboot
spring.datasource.one.username=root
spring.datasource.one.password=root

spring.datasource.two.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.two.url=jdbc:mysql://localhost:3306/wpspringboottwo
spring.datasource.two.username=root
spring.datasource.two.password=root
```
@Configuration


```
@Configuration
//@EnableTransactionManagement
//@PropertySource(value = {"classpath:config/source.properties"})
public class BeanConfig {

//    @Autowired
//    private Environment env;
//
//
//    @Bean(destroyMethod = "close")
//    public DataSource dataSource() {
//        DruidDataSource dataSource = new DruidDataSource();
//        dataSource.setDriverClassName(env.getProperty("source.driverClassName").trim());
//        dataSource.setUrl(env.getProperty("source.url").trim());
//        dataSource.setUsername(env.getProperty("source.username").trim());
//        dataSource.setPassword(env.getProperty("source.password").trim());
//
//        return dataSource;
//    }
//
//    @Bean
//    public JdbcTemplate jdbcTemplate() {
//        JdbcTemplate jdbcTemplate = new JdbcTemplate();
//        jdbcTemplate.setDataSource(dataSource());
//        return jdbcTemplate;
//    }

    @Bean(name = "oneDataSource")
    @Qualifier("oneDataSource")
    @Primary
    @ConfigurationProperties(prefix="spring.datasource.one")
    public DataSource oneDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "twoDataSource")
    @Qualifier("twoDataSource")
    @ConfigurationProperties(prefix="spring.datasource.two")
    public DataSource twoDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "oneJdbcTemplate")
    public JdbcTemplate oneJdbcTemplate(@Qualifier("oneDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }
    @Bean(name = "twoJdbcTemplate")
    public JdbcTemplate twoJdbcTemplate(@Qualifier("twoDataSource") DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

}
```
引用


```
@Repository
public class AuthorDaoImpl implements AuthorDao {
    @Autowired
    @Qualifier("oneJdbcTemplate")
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

试下单元测试

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(WpspringbootApplication.class)
public class JdbcTest {
}
```

idea提示SpringApplicationConfiguration方法已被废弃
那就用自动生成的


```
@RunWith(SpringRunner.class)
@SpringBootTest
public class WpspringbootApplicationTests {

	@Test
	public void contextLoads() {
	}

	@Resource(name="oneJdbcTemplate")
	protected JdbcTemplate jdbcTemplate1;
	@Resource(name="twoJdbcTemplate")
	protected JdbcTemplate jdbcTemplate2;

	@Test
	public void test() throws Exception {
		jdbcTemplate1.update("insert into t_author(id, real_name, nick_name) values(?, ?, ?)", 5, "梁桂钊", "LiangGzone");
		jdbcTemplate2.update("insert into t_author(id, real_name, nick_name) values(?, ?, ?)", 6, "梁桂钊", "LiangGzone");
	}

}
```

运行之后发现数据库中文是问号
数据库配置文件中配置

```
spring.datasource.one.url=jdbc:mysql://localhost:3306/wpspringboot?useUnicode=true&characterEncoding=utf-8&autoReconnect=true
```
再次测试，正常了




