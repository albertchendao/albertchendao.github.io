---
layout: article
title: MyBatis 问题
tags: []
key: 765bf34b-0012-43d9-b6d6-09bd2b363150
---

MyBatis 使用中相关的问题.

<!--more-->

## Spring Boot setTypeHandlersPackage 不起作用

Spring Boot 集成 MyBatis 通过如下 Bean 配置:

```java
@Configuration
public class DbConfig {

    @Bean("druidStatFilter")
    public StatFilter statFilter() {
        StatFilter bean = new StatFilter();
        bean.setLogSlowSql(true);
        bean.setSlowSqlMillis(3000);
        return bean;
    }

    @Bean("druidSlf4j")
    public Slf4jLogFilter slf4jLogFilter() {
        Slf4jLogFilter bean = new Slf4jLogFilter();
        bean.setDataSourceLogEnabled(false);
        bean.setResultSetLogEnabled(false);
        bean.setConnectionLogEnabled(false);
        bean.setStatementExecuteQueryAfterLogEnabled(true);
        bean.setStatementCreateAfterLogEnabled(false);
        bean.setStatementCloseAfterLogEnabled(false);
        bean.setStatementExecuteAfterLogEnabled(false);
        bean.setStatementParameterSetLogEnabled(false);
        return bean;
    }

    public void initDruid(DruidDataSource ds) {
        ds.setMaxActive(30);
        ds.setMinIdle(2);
        ds.setInitialSize(2);
        ds.setMaxWait(60000);
        ds.setTimeBetweenEvictionRunsMillis(60000);
        ds.setMinEvictableIdleTimeMillis(300000);
        ds.setTestWhileIdle(true);
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setValidationQuery("select 1");
        List<Filter> filterList = new ArrayList<>();
        filterList.add(statFilter());
        filterList.add(slf4jLogFilter());
        ds.setProxyFilters(filterList);
    }

    @Bean
    public Interceptor[] interceptors() {
        Interceptor[] interceptors = new Interceptor[2];
        interceptors[0] = new MapResultInterceptor();
        interceptors[1] = new SqlExecuteTimeInterceptor();
        return interceptors;
    }

    @Configuration
    @MapperScan(basePackages = {"cn.jiguang.imarketing.data.dao"}, sqlSessionTemplateRef = "userSqlSessionTemplate")
    @EnableTransactionManagement
    public class UserDb {
        /**
         * 用户数据库
         */
        @Bean("srv")
        @Primary //主数据源
        @ConfigurationProperties(prefix = "spring.datasource.srv")
        public DataSource srv() {
            DruidDataSource bean = DruidDataSourceBuilder.create().build();
            initDruid(bean);
            return bean;
        }

        @Bean("userSqlSessionFactory")
        public SqlSessionFactory userSqlSessionFactory() throws Exception {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(srv());
            bean.setPlugins(interceptors());
            return bean.getObject();
        }

        @Bean("userSqlSessionTemplate")
        public SqlSessionTemplate userSqlSessionTemplate(@Qualifier("userSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
            SqlSessionTemplate template = new SqlSessionTemplate(sqlSessionFactory); // 使用上面配置的Factory
            return template;
        }

        @Bean("userTransactionManager")
        public PlatformTransactionManager transactionManager() {
            final DataSourceTransactionManager transactionManager = new DataSourceTransactionManager(srv());
            return transactionManager;
        }
    }
}
```

希望将 TypeHandler 注册成全局的, 这样在 DAO 里就不用再引入了.

首先在 TypeHandler 上加注解 `@MappedTypes` 指定对应的 Java 类型:

```java
@Slf4j
@MappedTypes({GroupDataType.class})
public class IEnumTypeHandler<T extends IEnum> extends BaseTypeHandler<T> {
    // ...
}
```

然后配置 `SqlSessionFactory`:

```java
        @Bean("userSqlSessionFactory")
        public SqlSessionFactory userSqlSessionFactory() throws Exception {
            SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
            bean.setDataSource(srv());
            bean.setPlugins(interceptors());
            // 这里必须加不然不起作用
            bean.setVfs(SpringBootVFS.class);
            bean.setTypeHandlersPackage("cn.jiguang.imarketing.data.mybatis.typehandler");
            return bean.getObject();
        }
```

这样在 DAO 中的 `type` 字段上就可以这样用了:

```java
    @Select({"select gid, type, JSON_UNQUOTE(data) as data, data_date, consume_time ",
            "from tb_user_group_result ",
            "where gid=#{gid} ",
            "and type=#{type} "})
    @Results(id = "group_result",
            value = {
                    @Result(column = "gid", property = "gid"),
                    @Result(column = "type", property = "type"),
                    @Result(column = "data", property = "data"),
                    @Result(column = "data_date", property = "dataDate"),
                    @Result(column = "consume_time", property = "consumeTime"),
            })
    GroupResultEntity selectGroupResult(@Param("gid") Integer gid, @Param("type") GroupDataType type);
```