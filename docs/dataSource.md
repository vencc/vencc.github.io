**当前环境**  
springboot3.2.0
jdk17.0.9

# 数据源为不同数据库  
application.yml配置  
```
mybatis:
  type-aliases-package: com.oa.**.po
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
spring:
  datasource:
    qs:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/test?serverTimezone=Hongkong&useUnicode=true&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true
      username: root
      password: 123
      driver-class-name: com.mysql.cj.jdbc.Driver
    hjldb:
      jdbc-url: jdbc:sqlserver://127.0.0.1:3433;Database=test
      username: root
      password: 123
      driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
```
分别为两个数据库创建两个configuration  
```
@Configuration
@MapperScan(
        basePackages = {"com.oa.sys.mapper", "com.oa.quartz.mapper", "com.oa.persistence.mapper"},
        sqlSessionFactoryRef = "qsSqlSessionFactory",
        sqlSessionTemplateRef = "qsSqlSessionTemplate"
)
public class DataSourceConfig_qs {
    @Bean
    @ConfigurationProperties(prefix = "mybatis.configuration")
    public org.apache.ibatis.session.Configuration qsConfiguration() {
        return new org.apache.ibatis.session.Configuration();
    }

    /**
     * 加载数据源
     *
     * @return
     */
    @Bean(name = "qs")
    @ConfigurationProperties(prefix = "spring.datasource.qs")
    @Primary
    public DataSource qsDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "qsSqlSessionFactory")
    @Primary
    public SqlSessionFactory qsSqlSessionFactory(@Qualifier("dynamicDataSource") DataSource dataSource, org.apache.ibatis.session.Configuration qsConfiguration) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        //驼峰映射注册到SqlSessionFactory中
        bean.setConfiguration(qsConfiguration);
        // 分页插件
        Interceptor interceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "mysql");
        properties.setProperty("reasonable", "true");
        interceptor.setProperties(properties);
        bean.setPlugins(new Interceptor[] {interceptor});
        return bean.getObject();
    }

    /**
     * 添加事务
     *
     * @param dataSource
     * @return
     */
    @Bean(name = "qsTransactionManager")
    @Primary
    public DataSourceTransactionManager qsTransactionManager(@Qualifier("qs") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "qsSqlSessionTemplate")
    @Primary
    public SqlSessionTemplate qsSqlSessionTemplate(@Qualifier("qsSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
```
@Configuration
@MapperScan(
        basePackages = {"com.oa.persistence.hjldbMapper"},
        sqlSessionFactoryRef = "hjldbSqlSessionFactory",
        sqlSessionTemplateRef = "hjldbSqlSessionTemplate"
)
public class DataSourceConfig_hjldb {
    /**
     * 加载数据源
     *
     * @return
     */
    @Bean(name = "hjldb")
    @ConfigurationProperties(prefix = "spring.datasource.hjldb")
    public DataSource hjldbDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "hjldbSqlSessionFactory")
    public SqlSessionFactory hjldbSqlSessionFactory(@Qualifier("hjldb") DataSource dataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        // 分页插件
        Interceptor interceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "sqlserver");
        properties.setProperty("reasonable", "true");
        interceptor.setProperties(properties);
        bean.setPlugins(new Interceptor[] {interceptor});
        return bean.getObject();
    }

    /**
     * 添加事务
     *
     * @param dataSource
     * @return
     */
    @Bean(name = "hjldbTransactionManager")
    public DataSourceTransactionManager hjldbTransactionManager(@Qualifier("hjldb") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "hjldbSqlSessionTemplate")
    public SqlSessionTemplate hjldbSqlSessionTemplate(@Qualifier("hjldbSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
# 数据源为主从库  
application.yml配置  
```
mybatis:
  type-aliases-package: com.oa.**.po
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    map-underscore-to-camel-case: true
spring:
  datasource:
    qs:
      jdbc-url: jdbc:mysql://127.0.0.1:3306/qs?serverTimezone=Hongkong&useUnicode=true&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true
      username: root
      password: 123
      driver-class-name: com.mysql.cj.jdbc.Driver
    slave:
      jdbc-url: jdbc:mysql://192.168.16.12:3306/qs?serverTimezone=Hongkong&useUnicode=true&characterEncoding=utf8&useSSL=false&allowPublicKeyRetrieval=true&nullCatalogMeansCurrent=true
      username: root
      password: 123
      driver-class-name: com.mysql.cj.jdbc.Driver
```
创建configuration  
```
@Configuration
@MapperScan(
        basePackages = {"com.oa.sys.mapper", "com.oa.quartz.mapper", "com.oa.persistence.mapper"},
        sqlSessionFactoryRef = "qsSqlSessionFactory",
        sqlSessionTemplateRef = "qsSqlSessionTemplate"
)
public class DataSourceConfig_qs {
    @Bean
    @ConfigurationProperties(prefix = "mybatis.configuration")
    public org.apache.ibatis.session.Configuration qsConfiguration() {
        return new org.apache.ibatis.session.Configuration();
    }

    /**
     * 加载数据源
     *
     * @return
     */
    @Bean(name = "qs")
    @ConfigurationProperties(prefix = "spring.datasource.qs")
    public DataSource qsDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "slave")
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slaveDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = "dynamicDataSource")
    public DynamicDataSource dynamicDataSource(@Qualifier("qs")DataSource qsDataSource,@Qualifier("slave")DataSource slaveDataSource){
        Map<Object, Object> targetDataSources = new HashMap<>(2);
        targetDataSources.put("qs", qsDataSource);
        targetDataSources.put("slave", slaveDataSource);
        //DynamicDataSource（默认数据源,所有数据源） 第一个指定默认数据库
        return new DynamicDataSource(qsDataSource, targetDataSources);
    }

    @Bean(name = "qsSqlSessionFactory")
    public SqlSessionFactory qsSqlSessionFactory(@Qualifier("dynamicDataSource") DataSource dataSource, org.apache.ibatis.session.Configuration qsConfiguration) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dataSource);
        //驼峰映射注册到SqlSessionFactory中
        bean.setConfiguration(qsConfiguration);
        // 分页插件
        Interceptor interceptor = new PageInterceptor();
        Properties properties = new Properties();
        properties.setProperty("helperDialect", "mysql");
        properties.setProperty("reasonable", "true");
        interceptor.setProperties(properties);
        bean.setPlugins(new Interceptor[] {interceptor});
        return bean.getObject();
    }

    /**
     * 添加事务
     *
     * @param dataSource
     * @return
     */
    @Bean(name = "qsTransactionManager")
    public DataSourceTransactionManager qsTransactionManager(@Qualifier("dynamicDataSource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "qsSqlSessionTemplate")
    public SqlSessionTemplate qsSqlSessionTemplate(@Qualifier("qsSqlSessionFactory") SqlSessionFactory sqlSessionFactory) {
        return new SqlSessionTemplate(sqlSessionFactory);
    }
}
```
动态数据源DynamicDataSource.java  
```
public class DynamicDataSource extends AbstractRoutingDataSource {
    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    /**
     * 配置DataSource, defaultTargetDataSource为主数据库
     */
    public DynamicDataSource(DataSource defaultTargetDataSource, Map<Object, Object> targetDataSources) {
        super.setDefaultTargetDataSource(defaultTargetDataSource);
        super.setTargetDataSources(targetDataSources);
        super.afterPropertiesSet();
    }

    @Override
    protected Object determineCurrentLookupKey() {
        return getDataSource();
    }

    public static void setDataSource(String dataSource) {
        contextHolder.set(dataSource);
    }

    public static String getDataSource() {
        return contextHolder.get();
    }

    public static void clearDataSource() {
        contextHolder.remove();
    }
}
```
