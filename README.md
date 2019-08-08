[事务](https://www.cnblogs.com/yixianyixian/p/8372832.html)
===
1、编程式事务:<br>
>1.1、自己编写事务控制代码<br>
>1.2、OpenSessionInView编程式事务<br>

2、声明式事务：<br>
>2.1、事务控制代码已经由spring写好，程序员需要声明出哪些方法需要进行事务控制<br>

3、声明式事务都是针对ServiceImpl类下的方法的<br>
4、事务管理器基于通知（advice）的<br>
```xml
<context:property-placeholder location="classpath:db.properties"/>

    <!--数据源封装类-->
    <bean id="DataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="${jdbc.driver}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    
    <!--事务管理器-->
    <bean id="dataSourceTransactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="DataSource"/>
    </bean>
    
    <!--配置声明式事务-->
    <tx:advice id="transactionInterceptor" transaction-manager="dataSourceTransactionManager">
        <tx:attributes>
            <tx:method name="insert"/>
        </tx:attributes>
    </tx:advice>

    <!--创建SqlSessionFactory对象-->
    <bean id="factory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--引用数据源-->
        <property name="dataSource" ref="DataSource"/>
        <property name="typeAliasesPackage" value="com.liyan.pojo"/>
        <property name="configLocation" value="classpath:mybatis.xml"/>
    </bean>
    <!--扫描器相当于mybatis中package标签，扫描后会给对应接口创建对象-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <!--扫描包路径-->
        <property name="basePackage" value="com.liyan.mapper"/>
        <!--        <property name="sqlSessionFactory" ref="factory"/>-->
        <property name="sqlSessionFactoryBeanName" value="factory"/>
    </bean>

    <aop:config>
        <aop:pointcut id="mypoin" expression="execution(* com.liyan.service.Impl.*.*(..))"/>
        <aop:advisor advice-ref="transactionInterceptor" pointcut-ref="mypoin"/>
    </aop:config>
```
5、[事务传播行为](https://segmentfault.com/a/1190000013341344)<br>
>5.1、name=''哪些方法需要有事务控制<br>
>>5.1.1、支持*通配符<br>

>5.2、readonly=“boolean”是否是只读事务<br>
>>5.2.1、如果为true，告诉数据库为只读事务，数据化优化会对性能有所提升，所以查询的方法建议使用<br>
>>5.2.2、如果为false（默认），事务需要提交事务，建议新增、删除、更新使用<br>

>5.3、propagation控制事务传播行为<br>
>>5.3.1、当一个具有事务控制的方法被另外一个有事务控制的方法调用后，需要如何管理事务（新建事务？在事务中执行？把事务挂起？报异常）<br>
>>5.3.2、REQUIRED(默认)如果当前有事务，就在事务中执行，如果当前没有事务，新建一个事务<br>
>>5.3.3、SUPPORTS如果当前有事务就在事务中执行，如果没有事务就在非事务状态下执行<br>
>>5.3.4、MANDATORY必须在事务中执行，如果当前有事务，就在事务中执行，如果没有事务报错<br>
>>5.3.5、REQUIRES_NEW如果当前没有事务就新建事务，如果有事务就挂起<br>
>>5.3.6、NOT_SUPPORTED必须在非事务下执行，如果当前没有事务正常执行，如果当前有事务就把事务挂起<br>
>>5.3.7、NEVER必须在非事务下执行，如果当前没有事务正常执行，如果当前有事务报异常<br>
>>5.3.8、NESTED必须在事务状态下执行，如果没有事务新建事务，如果当前没有事务，创建一个嵌套事务<br>

6、[事务隔离级别(isolation="")](https://www.jianshu.com/p/00a468cc5d75)<br>
>6.1、事务并发引起的三种情况<br>
>>6.1.1、在多线程或并发访问下如何保证访问到的数据是合法的<br>
>>6.1.2、脏读：一个事务正在对数据进行更新操作，但是更新还未提交，另一个事务这时也来操作这组数据，并且读取了前一个事务还未提交的数据，而前一个事务如果操作失败进行了回滚，后一个事务读取的就是错误数据，这样就造成了脏读。<br>
>>6.1.3、不可重复读：一个事务多次读取同一数据，在该事务还未结束时，另一个事务也对该数据进行了操作，而且在第一个事务两次次读取之间，第二个事务对数据进行了更新，那么第一个事务前后两次读取到的数据是不同的，这样就造成了不可重复读。<br>
>>6.1.4、幻读：第一个数据正在查询符合某一条件的数据，这时，另一个事务又插入了一条符合条件的数据，第一个事务在第二次查询符合同一条件的数据时，发现多了一条前一次查询时没有的数据，仿佛幻觉一样，这就是幻像读。<br>

>6.2、隔离级别<br>
>>6.2.1、default（默认）:这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与JDBC的隔离级别相对应。<br>
>>6.2.2、READ_UNCOMMITTED（读未提交）：这是事务最低的隔离级别，它允许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻像读。 <br>
>>6.2.3、READ_COMMITTED （读已提交）：保证一个事务修改的数据提交后才能被另外一个事务读取，另外一个事务不能读取该事务未提交的数据。这种事务隔离级别可以避免脏读出现，但是可能会出现不可重复读和幻像读。<br>
>>6.2.4、REPEATABLE_READ （可重复读）： 这种事务隔离级别可以防止脏读、不可重复读，但是可能出现幻像读。它除了保证一个事务不能读取另一个事务未提交的数据外，还保证了不可重复读。<br>
>>6.2.5、SERIALIZABLE（串行化）：这是花费最高代价但是最可靠的事务隔离级别，事务被处理为顺序执行。除了防止脏读、不可重复读外，还避免了幻像读。<br>

7、[spring常用注解](https://mp.weixin.qq.com/s?src=11&timestamp=1565255067&ver=1777&signature=eYtegbVy7CpCXP4OKToeLTnKuO6iWZgA64nzAKH1z3SX9Mrc8-tC1IBl6hMFSjZSZDuZRmgLPO*ESYqnzqHlflotX17LvIN*nflpyl7GWw5feJ7eNwCBrXRF3RVaB*gc&new=1)<br>
>7.1、@Component：@Component是所有受Spring 管理组件的通用形式，@Component注解可以放在类的头上，@Component不推荐使用。创建类对象，相当于配置<br>
>7.2、@Service：：与@Component功能相同，写在serviceImpl类上<br>
>7.3、@Repository:与@Component功能相同，建议写在数据访问层<br>
>7.4、@Controller：与@Component功能相同，建议写在控制器上<br>
>7.5、@Autowired：@Autowired顾名思义，就是自动装配，其作用是为了消除代码Java代码里面的getter/setter与bean属性中的property。当然，getter看个人需求，如果私有属性需要对外提供的话，应当予以保留。@Autowired默认按类型匹配的方式，在容器查找匹配的Bean，当有且仅有一个匹配的Bean时，Spring将其注入@Autowired标注的变量中。<br>
>7.6、@Resource：@Resource注解与@Autowired注解作用非常相似<br>
>7.7、@Value()获取properties文件中的内容<br>
>7.8、@Pointcut()定义切点<br>
>7.9、@Aspect：定义切点<br>
>7.10、@Before前置通知<br>
>7.11、@After后置通知<br>
>7.12、@AfterReturning后置通知，必须切点正确执行<br>
>7.13、@AfterThrowing异常通知<br>
>7.14、@Arround环绕通知<br>