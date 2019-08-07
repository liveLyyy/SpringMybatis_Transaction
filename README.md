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