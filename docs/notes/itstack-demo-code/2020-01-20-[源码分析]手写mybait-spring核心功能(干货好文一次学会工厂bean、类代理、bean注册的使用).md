## 一、前言介绍

一个知识点的学习过程基本分为；运行helloworld、熟练使用api、源码分析、核心专家。在分析mybaits以及mybatis-spring源码之前，我也只是简单的使用，因为它好用。但是他是怎么做的多半是凭自己的经验去分析，但始终觉得这样的感觉缺少点什么，在几次夙兴夜寐，靡有朝矣之后决定彻底的研究一下，之后在去仿照着写一版核心功能。依次来补全自己的技术栈的空缺。在现在技术知识像爆炸一样迸发，而我们多半又忙于工作业务开发。就像一个不会修车的老司机，只能一脚油门，一脚刹车的奔波。车速很快，但经不起坏，累觉不爱。好！为了解决这样问题，也为了钱程似锦（形容钱多的想家里的棉布一样），努力！

开动之前先庆祝下我的iPhone4s又活了，还是那么好用(嗯！有点卡)；
![](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-code-mybatis-2-1.jpg)

## 二、以往章节

关于mybaits & spring 源码分析以及demo功能的章节汇总，可以通过下列内容进行系统的学习，同时以下章节会有部分内容涉及到demo版本的mybaits；

- [源码分析 - Mybatis接口没有实现类为什么可以执行增删改查](https://bugstack.cn/itstack-demo-any/2019/12/25/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-Mybatis%E6%8E%A5%E5%8F%A3%E6%B2%A1%E6%9C%89%E5%AE%9E%E7%8E%B0%E7%B1%BB%E4%B8%BA%E4%BB%80%E4%B9%88%E5%8F%AF%E4%BB%A5%E6%89%A7%E8%A1%8C%E5%A2%9E%E5%88%A0%E6%94%B9%E6%9F%A5.html)
- [源码分析 - 像盗墓一样分析Spring是怎么初始化xml并注册bean的](https://bugstack.cn/itstack-demo-any/2020/01/08/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%83%8F%E7%9B%97%E5%A2%93%E4%B8%80%E6%A0%B7%E5%88%86%E6%9E%90Spring%E6%98%AF%E6%80%8E%E4%B9%88%E5%88%9D%E5%A7%8B%E5%8C%96xml%E5%B9%B6%E6%B3%A8%E5%86%8Cbean%E7%9A%84.html)
- [源码分析 - 基于jdbc实现一个Demo版的Mybatis](https://bugstack.cn/itstack-demo-any/2020/01/13/%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-%E5%9F%BA%E4%BA%8Ejdbc%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AADemo%E7%89%88%E7%9A%84Mybatis.html)

## 三、一碟小菜类代理

往往从最简单的内容才有抓手。先看一个接口到实现类的使用，在将这部分内容转换为代理类。

### 1. 定义一个 IUserDao 接口并实现这个接口类

```java
public interface IUserDao {

    String queryUserInfo();

}

public class UserDao implements IUserDao {

    @Override
    public String queryUserInfo() {
        return "实现类";
    }

}
```

### 2. new() 方式实例化

```java
IUserDao userDao = new UserDao();
userDao.queryUserInfo();
```

这是最简单的也是最常用的使用方式，new 个对象。

### 3. proxy 方式实例化

```java
ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
Class<?>[] classes = {IUserDao.class};
InvocationHandler handler = (proxy, method, args) -> "你被代理了 " + method.getName();

IUserDao userDao = (IUserDao) Proxy.newProxyInstance(classLoader, classes, handler);

String res = userDao.queryUserInfo();
logger.info("测试结果：{}", res);
```

- Proxy.newProxyInstance 代理类实例化方式，对应传入类的参数即可
- ClassLoader，是这个类加载器，我们可以获取当前线程的类加载器
- InvocationHandler 是代理后实际操作方法执行的内容，在这里可以添加自己业务场景需要的逻辑，在这里我们只返回方法名

**测试结果：**

```java
23:20:18.841 [main] INFO  org.itstack.demo.test.ApiTest - 测试结果：你被代理了 queryUserInfo

Process finished with exit code 0
```

## 四、盛宴来自Bean工厂

在使用Spring的时候，我们会采用注册或配置文件的方式，将我们的类交给Spring管理。例如；

```java
<bean id="userDao" class="org.itstack.demo.UserDao" scope="singleton"/>
```

UserDao是接口IUserDao的实现类，通过上面配置，就可以实例化一个类供我们使用，但如果IUserDao没有实现类或者我们希望去动态改变他的实现类比如挂载到别的地方(像mybaits一样)，并且是由spring bean工厂管理的，该怎么做呢？

### 1. FactoryBean的使用

FactoryBean 在spring起到着二当家的地位，它将近有70多个小弟(实现它的接口定义)，那么它有三个方法；
- T getObject() throws Exception; 返回bean实例对象
- Class<?> getObjectType(); 返回实例类类型
- boolean isSingleton(); 判断是否单例，单例会放到Spring容器中单实例缓存池中

那么我们现在就将上面用到的**代理类**交给spring的FactoryBean进行管理，代码如下；

>ProxyBeanFactory.java & bean工厂实现类

```java
public class ProxyBeanFactory implements FactoryBean<IUserDao> {

    @Override
    public IUserDao getObject() throws Exception {

        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        Class<?>[] classes = {IUserDao.class};
        InvocationHandler handler = (proxy, method, args) -> "你被代理了 " + method.getName();

        return (IUserDao) Proxy.newProxyInstance(classLoader, classes, handler);
    }

    @Override
    public Class<?> getObjectType() {
        return IUserDao.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

}
```

>spring-config.xml & 配置bean类信息

```java
<bean id="userDao" class="org.itstack.demo.bean.ProxyBeanFactory"/>
```

>ApiTest.test_IUserDao() & 单元测试

```java
@Test
public void test_IUserDao() {
    BeanFactory beanFactory = new ClassPathXmlApplicationContext("spring-config.xml");
    IUserDao userDao = beanFactory.getBean("userDao", IUserDao.class);
    String res = userDao.queryUserInfo();
    logger.info("测试结果：{}", res);
}
```

**测试结果：**

```java
一月 20, 2020 23:43:35 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [spring-config.xml]
23:43:35.440 [main] INFO  org.itstack.demo.test.ApiTest - 测试结果：你被代理了 queryUserInfo

Process finished with exit code 0

```

**咋样**，神奇不！你的接口都不需要实现类，就被安排的明明白白的。记住这个方法FactoryBean和动态代理。

### 2. BeanDefinitionRegistryPostProcessor 类注册

你是否有怀疑过你媳妇把你钱没收了之后都存放到哪去了，为啥你每次get都那么费劲，像垃圾回收了一样，不可达。

**好嘞**，媳妇那就别想了，研究下你的bean都被注册到哪了就可以了。在spring的bean管理中，所有的bean最终都会被注册到类DefaultListableBeanFactory中，接下来我们就主动注册一个被我们代理了的bean。

>RegisterBeanFactory.java & 注册bean的实现类

```java
public class RegisterBeanFactory implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {

        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(ProxyBeanFactory.class);

        BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(beanDefinition, "userDao");
        registry.registerBeanDefinition(definitionHolder.getBeanName(), definitionHolder.getBeanDefinition());
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // left intentionally blank
    }

}
```

- 这里包含4块主要内容，分别是；
	- 实现BeanDefinitionRegistryPostProcessor.postProcessBeanDefinitionRegistry方法，获取bean注册对象
	- 定义bean，GenericBeanDefinition，这里主要设置了我们的代理类工厂。我们已经测试过他获取一个代理类
	- 创建bean定义处理类，BeanDefinitionHolder，这里需要的主要参数；定义bean、bean名称
	- 最后将我们自己的bean注册到spring容器中去，registry.registerBeanDefinition()
	
>spring-config.xml & 配置bean类信息

```java
<bean id="userDao" class="org.itstack.demo.bean.RegisterBeanFactory"/>
```

>ApiTest.test_IUserDao() & 单元测试

```java
@Test
public void test_IUserDao() {
    BeanFactory beanFactory = new ClassPathXmlApplicationContext("spring-config.xml");
    IUserDao userDao = beanFactory.getBean("userDao", IUserDao.class);
    String res = userDao.queryUserInfo();
    logger.info("测试结果：{}", res);
}
```

**测试结果：**

```java
信息: Loading XML bean definitions from class path resource [spring-config.xml]
一月 20, 2020 23:42:29 上午 org.springframework.beans.factory.support.DefaultListableBeanFactory registerBeanDefinition
信息: Overriding bean definition for bean 'userDao' with a different definition: replacing [Generic bean: class [org.itstack.demo.bean.RegisterBeanFactory]; scope=; abstract=false; lazyInit=false; autowireMode=1; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [spring-config.xml]] with [Generic bean: class [org.itstack.demo.bean.ProxyBeanFactory]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null]
23:42:29.754 [main] INFO  org.itstack.demo.test.ApiTest - 测试结果：你被代理了 queryUserInfo

Process finished with exit code 0
```

**纳尼**？是不有一种满脑子都是骚操作的感觉，自己注册的bean自己知道在哪了，咋回事了。

## 五、老板郎上主食呀(mybaits-spring)

如果通过上面的知识点；代理类、bean工厂、bean注册，将我们一个没有实现类的接口安排的明明白白，让他执行啥就执行啥，那么你是否可以想到，这个没有实现类的接口，可以通过我们的折腾，去调用到我们的mybaits呢！

如下图，通过mybatis使用的配置，我们可以看到数据源DataSource交给SqlSessionFactoryBean，SqlSessionFactoryBean实例化出的SqlSessionFactory，再交给MapperScannerConfigurer。而我们要实现的就是MapperScannerConfigurer这部分；

![](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-code-mybatis-2-2.png)

### 1. 需要实现哪些核心类

为了更易理解也更易于对照，我们将实现mybatis-spring中的流程核心类，如下；

- MapperFactoryBean ｛给每一个没有实现类的接口都代理一个这样的类，用于操作数据库执行crud｝
- MapperScannerConfigurer ｛扫描包下接口类，免去配置。这样是上图中核心配置类｝
- SimpleMetadataReader ｛这个类完全和mybaits-spring中的类一样，为了解析class文件。如果你对类加载处理很好奇，可以阅读我的[《用java实现jvm虚拟机》](https://bugstack.cn/itstack-demo-jvm/itstack-demo-jvm.html)｝
- SqlSessionFactoryBean {这个类核心内容就一件事，将我们写的demo版的mybaits结合进来}

在分析之前先看下我们实现主食是怎么食用的，如下；

```xml
<bean id="sqlSessionFactory" class="org.itstack.demo.like.spring.SqlSessionFactoryBean">
    <property name="resource" value="spring/mybatis-config-datasource.xml"/>
</bean>

<bean class="org.itstack.demo.like.spring.MapperScannerConfigurer">
    <!-- 注入sqlSessionFactory -->
    <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <!-- 给出需要扫描Dao接口包 -->
    <property name="basePackage" value="org.itstack.demo.dao"/>
</bean>
```

### 2. (类介绍)SqlSessionFactoryBean

这类本身比较简单，主要实现了FactoryBean<SqlSessionFactory>, InitializingBean用于帮我们处理mybaits核心流程类的加载处理。（关于demo版的mybaits已经在上文中提供学习链接）

>SqlSessionFactoryBean.java 

```java
public class SqlSessionFactoryBean implements FactoryBean<SqlSessionFactory>, InitializingBean {

    private String resource;
    private SqlSessionFactory sqlSessionFactory;

    @Override
    public void afterPropertiesSet() throws Exception {
        try (Reader reader = Resources.getResourceAsReader(resource)) {
            this.sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    @Override
    public SqlSessionFactory getObject() throws Exception {
        return sqlSessionFactory;
    }

    @Override
    public Class<?> getObjectType() {
        return sqlSessionFactory.getClass();
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

    public void setResource(String resource) {
        this.resource = resource;
    }

}
```

- 实现InitializingBean主要用于加载mybatis相关内容；解析xml、构造SqlSession、链接数据库等
- FactoryBean，这个类我们介绍过，主要三个方法；getObject()、getObjectType()、isSingleton()

### 3. (类介绍)MapperScannerConfigurer

这类的内容看上去可能有点多，但是核心事情也就是将我们的dao层接口扫描、注册

```java
public class MapperScannerConfigurer implements BeanDefinitionRegistryPostProcessor {

    private String basePackage;
    private SqlSessionFactory sqlSessionFactory;

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        try {
            // classpath*:org/itstack/demo/dao/**/*.class
            String packageSearchPath = "classpath*:" + basePackage.replace('.', '/') + "/**/*.class";

            ResourcePatternResolver resourcePatternResolver = new PathMatchingResourcePatternResolver();
            Resource[] resources = resourcePatternResolver.getResources(packageSearchPath);

            for (Resource resource : resources) {
                MetadataReader metadataReader = new SimpleMetadataReader(resource, ClassUtils.getDefaultClassLoader());

                ScannedGenericBeanDefinition beanDefinition = new ScannedGenericBeanDefinition(metadataReader);
                String beanName = Introspector.decapitalize(ClassUtils.getShortName(beanDefinition.getBeanClassName()));
                
                beanDefinition.setResource(resource);
                beanDefinition.setSource(resource);
                beanDefinition.setScope("singleton");
                beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(beanDefinition.getBeanClassName());
                beanDefinition.getConstructorArgumentValues().addGenericArgumentValue(sqlSessionFactory);
                beanDefinition.setBeanClass(MapperFactoryBean.class);

                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(beanDefinition, beanName);
                registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        // left intentionally blank
    }

    public void setBasePackage(String basePackage) {
        this.basePackage = basePackage;
    }

    public void setSqlSessionFactory(SqlSessionFactory sqlSessionFactory) {
        this.sqlSessionFactory = sqlSessionFactory;
    }
}

```

- 类的扫描注册，classpath*:org/itstack/demo/dao/**/*.class，解析calss文件获取资源信息；Resource[] resources = resourcePatternResolver.getResources(packageSearchPath);
- 遍历Resource，这里就你的class信息，用于注册bean。ScannedGenericBeanDefinition
- 这里有一点，bean的定义设置时候，是把beanDefinition.setBeanClass(MapperFactoryBean.class);设置进去的。同时在前面给他设置了构造参数。**（细细品味）**
- 最后执行注册registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());

### 4. (类介绍)MapperFactoryBean

这个类就非常有意思了，因为你所有的dao接口类，实际就是他。他这里帮你执行你对sql的所有操作的分发处理。为了更加简化清晰，目前这里只实现了查询部分，在mybatis-spring源码中分别对select、update、insert、delete、其他等做了操作。

```java
public class MapperFactoryBean<T> implements FactoryBean<T> {

    private Class<T> mapperInterface;
    private SqlSessionFactory sqlSessionFactory;

    public MapperFactoryBean(Class<T> mapperInterface, SqlSessionFactory sqlSessionFactory) {
        this.mapperInterface = mapperInterface;
        this.sqlSessionFactory = sqlSessionFactory;
    }

    @Override
    public T getObject() throws Exception {
        InvocationHandler handler = (proxy, method, args) -> {
            System.out.println("你被代理了，执行SQL操作！" + method.getName());
            try {
                SqlSession session = sqlSessionFactory.openSession();
                try {
                    return session.selectOne(mapperInterface.getName() + "." + method.getName(), args[0]);
                } finally {
                    session.close();
                }
            } catch (Exception e) {
                e.printStackTrace();
            }

            return method.getReturnType().newInstance();
        };
        return (T) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class[]{mapperInterface}, handler);
    }

    @Override
    public Class<?> getObjectType() {
        return mapperInterface;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

}
```

- T getObject()，中是一个java代理类的实现，这个代理类对象会被挂到你的注入中。真正调用方法内容时会执行到代理类的实现部分，也就是“你被代理了，执行SQL操作！”
- InvocationHandler，代理类的实现部分非常简单，主要开启SqlSession，并通过固定的key；“org.itstack.demo.dao.IUserDao.queryUserInfoById”执行SQL操作；
	
	>session.selectOne(mapperInterface.getName() + "." + method.getName(), args[0]);
	
	```xml
	<mapper namespace="org.itstack.demo.dao.IUserDao">

		<select id="queryUserInfoById" parameterType="java.lang.Long" resultType="org.itstack.demo.po.User">
			SELECT id, name, age, createTime, updateTime
			FROM user
			where id = #{id}
		</select>
		
	</mapper>
	```

- 最终返回了执行结果，关于查询到结果信息会反射操作成对象类，这部分内容可以遇到demo版本的mybatis

## 六、酒倒满走一个

好！到这一切开发内容就完成了，测试走一个。

>mybatis-config-datasource.xml & 数据源配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/itstack_demo_ddd?useUnicode=true"/>
                <property name="username" value="root"/>
                <property name="password" value="123456"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="mapper/User_Mapper.xml"/>
        <mapper resource="mapper/School_Mapper.xml"/>
    </mappers>

</configuration>
```

>test-config.xml & 配置xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd     http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.0.xsd"
       default-autowire="byName">
    <context:component-scan base-package="org.itstack"/>

    <aop:aspectj-autoproxy/>

    <bean id="sqlSessionFactory" class="org.itstack.demo.like.spring.SqlSessionFactoryBean">
        <property name="resource" value="spring/mybatis-config-datasource.xml"/>
    </bean>

    <bean class="org.itstack.demo.like.spring.MapperScannerConfigurer">
        <!-- 注入sqlSessionFactory -->
        <property name="sqlSessionFactory" ref="sqlSessionFactory"/>
        <!-- 给出需要扫描Dao接口包 -->
        <property name="basePackage" value="org.itstack.demo.dao"/>
    </bean>

</beans>
```

>SpringTest.java & 单元测试

```java
public class SpringTest {

    private Logger logger = LoggerFactory.getLogger(SpringTest.class);

    @Test
    public void test_ClassPathXmlApplicationContext() {
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("test-config.xml");
        IUserDao userDao = beanFactory.getBean("IUserDao", IUserDao.class);
        User user = userDao.queryUserInfoById(1L);
        logger.info("测试结果：{}", JSON.toJSONString(user));
    }

}
```

**测试结果；**

```java
一月 20, 2020 23:51:43 上午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@30b8a058: startup date [Mon Jan 20 23:51:43 CST 2020]; root of context hierarchy
一月 20, 2020 23:51:43 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [test-config.xml]
你被代理了，执行SQL操作！queryUserInfoById
2020-01-20 23:51:45.592 [main] INFO  org.itstack.demo.SpringTest[26] - 测试结果：{"age":18,"createTime":1576944000000,"id":1,"name":"水水","updateTime":1576944000000}

Process finished with exit code 0
```

酒干热火笑红尘，春秋几载年轮，不问。回首皆是Spring！**Gun！变心！你被代理了！**

## 七、综上总结

- 通过这些核心关键类的实现；SqlSessionFactoryBean、MapperScannerConfigurer、SqlSessionFactoryBean，我们将spring与mybaits集合起来使用，解决了没有实现类的接口怎么处理数据库CRUD操作
- 那么这个知识点可以用到哪里，不要只想着面试！在我们业务开发中是不会有很多其他数据源操作，比如ES、Hadoop、数据中心等等，包括自建。那么我们就可以做成一套统一数据源处理服务，以优化服务开发效率
- 由于这次工程类是在itstack-demo-code-mybatis中继续开发，如果需要获取源码可以关注公众号：bugstack虫洞栈，回复：源码分析
- 最后祝福大家在新的一年里；万事如意、恭贺新禧、喜气洋洋、福星高照、欢天喜地、吉祥如意、一帆风顺、万事大吉、龙凤呈祥、步步高升，一家瑞气，二气雍和，三星拱户，四季平安，五星高照。六六大顺，七星高照，八方来财，九九同心，十全十美。
