## 一、前言介绍
在前面一篇分析了 mybatis 源码，从它为什么之后接口但是没有实现类就能执行数据库操作为入口，整个源码核心流程完全解释了一遍。对于一个3年以上的程序员来说，新知识的学习过程应该是从最开始 helloworld 到熟练使用 api 完成业务功能。下一步为了深入了解就需要阅读部分核心源码，从而在出问题后可以快速定位，迅速排查。从而减少线上事故的持续时长，提升个人影响力。但！这不是学习终点，因为无论是任何一个框架的源码，如果只是看那么就很难学习到它的实用技术。纸上得来终觉浅，唯有实战和操练。

那么，本章节我们去简单实现一个基于jdbc的demo版本Mybatis，从而更加清楚这样框架的设计。与此同时这份思想会让你可以在其他场景使用，比如给ES查询写一个EsBatis。实现了心情也好了；

![微信公众号：bugstack虫洞栈 & DemoMybatis](https://fuzhengwei.github.io/assets/images/pic-content/2019/11/itstack-demo-mybatis-07.png)

## 二、案例工程

扩展上一篇源码分析工程；itstack-demo-mybatis，增加 like 包，模仿 Mybatis 工程。完整规程下载，关注公众号：bugstack虫洞栈 | 回复：源码分析

```java
itstack-demo-mybatis
└── src
    ├── main
    │   ├── java
    │   │   └── org.itstack.demo
    │   │       ├── dao
    │   │       │	├── ISchool.java		
    │   │       │	└── IUserDao.java	
    │   │       ├── like
    │   │       │	├── Configuration.java
    │   │       │	├── DefaultSqlSession.java
    │   │       │	├── DefaultSqlSessionFactory.java
    │   │       │	├── Resources.java
    │   │       │	├── SqlSession.java
    │   │       │	├── SqlSessionFactory.java
    │   │       │	├── SqlSessionFactoryBuilder.java	
    │   │       │	└── SqlSessionFactoryBuilder.java	
    │   │       └── interfaces     
    │   │         	├── School.java	
    │   │        	└── User.java
    │   ├── resources	
    │   │   ├── mapper
    │   │   │   ├── School_Mapper.xml
    │   │   │   └── User_Mapper.xml
    │   │   ├── props	
    │   │   │   └── jdbc.properties
    │   │   ├── spring
    │   │   │   ├── mybatis-config-datasource.xml
    │   │   │   └── spring-config-datasource.xml
    │   │   ├── logback.xml
    │   │   ├── mybatis-config.xml
    │   │   └── spring-config.xml
    │   └── webapp
    │       └── WEB-INF
    └── test
         └── java
             └── org.itstack.demo.test
                 ├── ApiLikeTest.java
                 ├── MybatisApiTest.java
                 └── SpringApiTest.java
```

## 三、环境配置
1. JDK1.8
2. IDEA 2019.3.1
3. dom4j 1.6.1

## 四、代码讲述

关于整个 Demo 版本，并不是把所有 Mybatis 全部实现一遍，而是拨丝抽茧将最核心的内容展示给你，从使用上你会感受一模一样，但是实现类已经全部被替换，核心类包括；
- Configuration 
- DefaultSqlSession
- DefaultSqlSessionFactory
- Resources
- SqlSession
- SqlSessionFactory
- SqlSessionFactoryBuilder
- XNode

### 1. 先测试下整个DemoJdbc框架

>ApiLikeTest.test_queryUserInfoById()

```java
@Test
public void test_queryUserInfoById() {
    String resource = "spring/mybatis-config-datasource.xml";
    Reader reader;
    try {
        reader = Resources.getResourceAsReader(resource);
        SqlSessionFactory sqlMapper = new SqlSessionFactoryBuilder().build(reader);
        SqlSession session = sqlMapper.openSession();
		
        try {
            User user = session.selectOne("org.itstack.demo.dao.IUserDao.queryUserInfoById", 1L);
            System.out.println(JSON.toJSONString(user));
        } finally {
            session.close();
            reader.close();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

**一切顺利结果如下(新人往往会遇到各种问题)；**

```java
{"age":18,"createTime":1576944000000,"id":1,"name":"水水","updateTime":1576944000000}

Process finished with exit code 0
```

可能乍一看这测试类完全和 MybatisApiTest.java 测试的代码一模一样呀，也看不出区别。其实他们的引入的包是不一样；

>MybatisApiTest.java 里面引入的包

```java
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
```

>ApiLikeTest.java 里面引入的包

```java
import org.itstack.demo.like.Resources;
import org.itstack.demo.like.SqlSession;
import org.itstack.demo.like.SqlSessionFactory;
import org.itstack.demo.like.SqlSessionFactoryBuilder;
```

好！接下来我们开始分析这部分核心代码。

### 2. 加载XML配置文件

这里我们采用 mybatis 的配置文件结构进行解析，在不破坏原有结构的情况下，最大可能的贴近源码。mybatis 单独使用的使用的时候使用了两个配置文件；数据源配置、Mapper 映射配置，如下；

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
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/itstack?useUnicode=true"/>
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

>User_Mapper.xml & Mapper 映射配置

```java
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.itstack.demo.dao.IUserDao">

    <select id="queryUserInfoById" parameterType="java.lang.Long" resultType="org.itstack.demo.po.User">
        SELECT id, name, age, createTime, updateTime
        FROM user
        where id = #{id}
    </select>

    <select id="queryUserList" parameterType="org.itstack.demo.po.User" resultType="org.itstack.demo.po.User">
        SELECT id, name, age, createTime, updateTime
        FROM user
        where age = #{age}
    </select>

</mapper>
```

这里的加载过程与 mybaits 不同，我们采用 dom4j 方式。在案例中会看到最开始获取资源，如下；


>ApiLikeTest.test_queryUserInfoById() & 部分截取

```java
String resource = "spring/mybatis-config-datasource.xml";
	Reader reader;
	try {
		reader = Resources.getResourceAsReader(resource);
	...
```

从上可以看到这是通过配置文件地址获取到了读取流的过程，从而为后面解析做基础。首先我们先看 Resources 类，整个是我们的资源类。

>Resources.java & 资源类

```java
/**
 * 公众号 | bugstack虫洞栈
 * 博 客 | https://bugstack.cn
 * Create by 小傅哥 @2020
 */
public class Resources {

    public static Reader getResourceAsReader(String resource) throws IOException {
        return new InputStreamReader(getResourceAsStream(resource));
    }

    private static InputStream getResourceAsStream(String resource) throws IOException {
        ClassLoader[] classLoaders = getClassLoaders();
        for (ClassLoader classLoader : classLoaders) {
            InputStream inputStream = classLoader.getResourceAsStream(resource);
            if (null != inputStream) {
                return inputStream;
            }
        }
        throw new IOException("Could not find resource " + resource);
    }

    private static ClassLoader[] getClassLoaders() {
        return new ClassLoader[]{
                ClassLoader.getSystemClassLoader(),
                Thread.currentThread().getContextClassLoader()};
    }

}
```

这段代码方法的入口是getResourceAsReader，直到往下以此做了；
1. 获取 ClassLoader 集合，最大限度搜索配置文件
2. 通过 classLoader.getResourceAsStream 读取配置资源，找到后立即返回，否则抛出异常

### 3. 解析XML配置文件

配置文件加载后开始进行解析操作，这里我们也仿照 mybatis 但进行简化，如下；

```java
SqlSessionFactory sqlMapper = new SqlSessionFactoryBuilder().build(reader);
```

>SqlSessionFactoryBuilder.build() & 入口构建类

```java
public DefaultSqlSessionFactory build(Reader reader) {
    SAXReader saxReader = new SAXReader();
    try {
        Document document = saxReader.read(new InputSource(reader));
        Configuration configuration = parseConfiguration(document.getRootElement());
        return new DefaultSqlSessionFactory(configuration);
    } catch (DocumentException e) {
        e.printStackTrace();
    }
    return null;
}
```

- 通过读取流创建 xml 解析的 Document 类
- parseConfiguration 进行解析 xml 文件，并将结果设置到配置类中，包括；连接池、数据源、mapper关系

>SqlSessionFactoryBuilder.parseConfiguration() & 解析过程

```java
private Configuration parseConfiguration(Element root) {
    Configuration configuration = new Configuration();
    configuration.setDataSource(dataSource(root.selectNodes("//dataSource")));
    configuration.setConnection(connection(configuration.dataSource));
    configuration.setMapperElement(mapperElement(root.selectNodes("mappers")));
    return configuration;
}
```

- 在前面的 xml 内容中可以看到，我们需要解析出数据库连接池信息 datasource，还有数据库语句映射关系 mappers

>SqlSessionFactoryBuilder.dataSource() & 解析出数据源

```java
private Map<String, String> dataSource(List<Element> list) {
    Map<String, String> dataSource = new HashMap<>(4);
    Element element = list.get(0);
    List content = element.content();
    for (Object o : content) {
        Element e = (Element) o;
        String name = e.attributeValue("name");
        String value = e.attributeValue("value");
        dataSource.put(name, value);
    }
    return dataSource;
}
```

- 这个过程比较简单，只需要将数据源信息获取即可

>SqlSessionFactoryBuilder.connection() & 获取数据库连接

```java
private Connection connection(Map<String, String> dataSource) {
    try {
        Class.forName(dataSource.get("driver"));
        return DriverManager.getConnection(dataSource.get("url"), dataSource.get("username"), dataSource.get("password"));
    } catch (ClassNotFoundException | SQLException e) {
        e.printStackTrace();
    }
    return null;
}
```

- 这个就是jdbc最原始的代码，获取了数据库连接池

>SqlSessionFactoryBuilder.mapperElement() & 解析SQL语句

```java
private Map<String, XNode> mapperElement(List<Element> list) {
    Map<String, XNode> map = new HashMap<>();
    Element element = list.get(0);
    List content = element.content();
    for (Object o : content) {
        Element e = (Element) o;
        String resource = e.attributeValue("resource");
        try {
            Reader reader = Resources.getResourceAsReader(resource);
            SAXReader saxReader = new SAXReader();
            Document document = saxReader.read(new InputSource(reader));
            Element root = document.getRootElement();
            //命名空间
            String namespace = root.attributeValue("namespace");
            // SELECT
            List<Element> selectNodes = root.selectNodes("select");
            for (Element node : selectNodes) {
                String id = node.attributeValue("id");
                String parameterType = node.attributeValue("parameterType");
                String resultType = node.attributeValue("resultType");
                String sql = node.getText();
                // ? 匹配
                Map<Integer, String> parameter = new HashMap<>();
                Pattern pattern = Pattern.compile("(#\\{(.*?)})");
                Matcher matcher = pattern.matcher(sql);
                for (int i = 1; matcher.find(); i++) {
                    String g1 = matcher.group(1);
                    String g2 = matcher.group(2);
                    parameter.put(i, g2);
                    sql = sql.replace(g1, "?");
                }
                XNode xNode = new XNode();
                xNode.setNamespace(namespace);
                xNode.setId(id);
                xNode.setParameterType(parameterType);
                xNode.setResultType(resultType);
                xNode.setSql(sql);
                xNode.setParameter(parameter);
                
                map.put(namespace + "." + id, xNode);
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    return map;
}
```

- 这个过程首先包括是解析所有的sql语句，目前为了测试只解析 select 相关
- 所有的 sql 语句为了确认唯一，都是使用；namespace + select中的id进行拼接，作为 key，之后与sql一起存放到 map 中。
- 在 mybaits 的 sql 语句配置中，都有占位符，用于传参。where id = #{id} 所以我们需要将占位符设置为问号，另外需要将占位符的顺序信息与名称存放到 map 结构，方便后续设置查询时候的入参。

### 4. 创建DefaultSqlSessionFactory

最后将初始化后的配置类 Configuration，作为参数进行创建 DefaultSqlSessionFactory，如下；

```java
public DefaultSqlSessionFactory build(Reader reader) {
    SAXReader saxReader = new SAXReader();
    try {
        Document document = saxReader.read(new InputSource(reader));
        Configuration configuration = parseConfiguration(document.getRootElement());
        return new DefaultSqlSessionFactory(configuration);
    } catch (DocumentException e) {
        e.printStackTrace();
    }
    return null;
}
```
>DefaultSqlSessionFactory.java & SqlSessionFactory的实现类

```java
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    
	private final Configuration configuration;
    
	public DefaultSqlSessionFactory(Configuration configuration) {
        this.configuration = configuration;
    }
	
    @Override
    public SqlSession openSession() {
        return new DefaultSqlSession(configuration.connection, configuration.mapperElement);
    }
	
}
```

- 这个过程比较简单，构造函数只提供了配置类入参
- 实现 SqlSessionFactory 的 openSession()，用于创建 DefaultSqlSession，也就可以执行 sql 操作

### 5. 开启SqlSession

```java
SqlSession session = sqlMapper.openSession();
```

上面这一步就是创建了DefaultSqlSession，比较简单。如下；

```java
@Override
public SqlSession openSession() {
    return new DefaultSqlSession(configuration.connection, configuration.mapperElement);
}
```

### 6. 执行SQL语句

```java
User user = session.selectOne("org.itstack.demo.dao.IUserDao.queryUserInfoById", 1L);
```

在 DefaultSqlSession 中通过实现 SqlSession，提供数据库语句查询和关闭连接池，如下；

>SqlSession.java & 定义

```java
public interface SqlSession {

    <T> T selectOne(String statement);

    <T> T selectOne(String statement, Object parameter);

    <T> List<T> selectList(String statement);

    <T> List<T> selectList(String statement, Object parameter);

    void close();
}
```

接下来看具体的执行过程，session.selectOne

>DefaultSqlSession.selectOne() & 执行查询

```java
public <T> T selectOne(String statement, Object parameter) {
    XNode xNode = mapperElement.get(statement);
    Map<Integer, String> parameterMap = xNode.getParameter();
    try {
        PreparedStatement preparedStatement = connection.prepareStatement(xNode.getSql());
        buildParameter(preparedStatement, parameter, parameterMap);
        ResultSet resultSet = preparedStatement.executeQuery();
        List<T> objects = resultSet2Obj(resultSet, Class.forName(xNode.getResultType()));
        return objects.get(0);
    } catch (Exception e) {
        e.printStackTrace();
    }
    return null;
}
```

- selectOne 就objects.get(0);，selectList 就全部返回 
- 通过 statement 获取最初解析 xml 时候的存储的 select 标签信息；
	
	```xml
	<select id="queryUserInfoById" parameterType="java.lang.Long" resultType="org.itstack.demo.po.User">
		SELECT id, name, age, createTime, updateTime
		FROM user
		where id = #{id}
	</select>
	```

- 获取 sql 语句后交给 jdbc 的 PreparedStatement 类进行执行
- 这里还需要设置入参，我们将入参设置进行抽取，如下；
	
	```java
	private void buildParameter(PreparedStatement preparedStatement, Object parameter, Map<Integer, String> parameterMap) throws SQLException, IllegalAccessException {

        int size = parameterMap.size();
        // 单个参数
        if (parameter instanceof Long) {
            for (int i = 1; i <= size; i++) {
                preparedStatement.setLong(i, Long.parseLong(parameter.toString()));
            }
            return;
        }

        if (parameter instanceof Integer) {
            for (int i = 1; i <= size; i++) {
                preparedStatement.setInt(i, Integer.parseInt(parameter.toString()));
            }
            return;
        }

        if (parameter instanceof String) {
            for (int i = 1; i <= size; i++) {
                preparedStatement.setString(i, parameter.toString());
            }
            return;
        }

        Map<String, Object> fieldMap = new HashMap<>();
        // 对象参数
        Field[] declaredFields = parameter.getClass().getDeclaredFields();
        for (Field field : declaredFields) {
            String name = field.getName();
            field.setAccessible(true);
            Object obj = field.get(parameter);
            field.setAccessible(false);
            fieldMap.put(name, obj);
        }

        for (int i = 1; i <= size; i++) {
            String parameterDefine = parameterMap.get(i);
            Object obj = fieldMap.get(parameterDefine);

            if (obj instanceof Short) {
                preparedStatement.setShort(i, Short.parseShort(obj.toString()));
                continue;
            }

            if (obj instanceof Integer) {
                preparedStatement.setInt(i, Integer.parseInt(obj.toString()));
                continue;
            }

            if (obj instanceof Long) {
                preparedStatement.setLong(i, Long.parseLong(obj.toString()));
                continue;
            }

            if (obj instanceof String) {
                preparedStatement.setString(i, obj.toString());
                continue;
            }

            if (obj instanceof Date) {
                preparedStatement.setDate(i, (java.sql.Date) obj);
            }

        }

    }
	```

	- 单个参数比较简单直接设置值即可，Long、Integer、String ...
	- 如果是一个类对象，需要通过获取 Field 属性，与参数 Map 进行匹配设置

- 设置参数后执行查询 preparedStatement.executeQuery()
- 接下来需要将查询结果转换为我们的类（主要是反射类的操作），resultSet2Obj(resultSet, Class.forName(xNode.getResultType()));
	
	```java
	private <T> List<T> resultSet2Obj(ResultSet resultSet, Class<?> clazz) {
		List<T> list = new ArrayList<>();
		try {
			ResultSetMetaData metaData = resultSet.getMetaData();
			int columnCount = metaData.getColumnCount();
			// 每次遍历行值
			while (resultSet.next()) {
				T obj = (T) clazz.newInstance();
				for (int i = 1; i <= columnCount; i++) {
					Object value = resultSet.getObject(i);
					String columnName = metaData.getColumnName(i);
					String setMethod = "set" + columnName.substring(0, 1).toUpperCase() + columnName.substring(1);
					Method method;
					if (value instanceof Timestamp) {
						method = clazz.getMethod(setMethod, Date.class);
					} else {
						method = clazz.getMethod(setMethod, value.getClass());
					}
					method.invoke(obj, value);
				}
				list.add(obj);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return list;
	}
	```
	
	- 主要通过反射生成我们的类对象，这个类的类型定义在 sql 标签上
	- 时间类型需要判断后处理，Timestamp，与 java 不是一个类型

### 7. Sql查询补充说明

sql 查询有入参、有不需要入参、有查询一个、有查询集合，只需要合理包装即可，例如下面的查询集合，入参是对象类型；

>ApiLikeTest.test_queryUserList()

```java
@Test
public void test_queryUserList() {
    String resource = "spring/mybatis-config-datasource.xml";
    Reader reader;
    try {
        reader = Resources.getResourceAsReader(resource);
        SqlSessionFactory sqlMapper = new SqlSessionFactoryBuilder().build(reader);
        SqlSession session = sqlMapper.openSession();
        
		try {
            User req = new User();
            req.setAge(18);
            List<User> userList = session.selectList("org.itstack.demo.dao.IUserDao.queryUserList", req);
            System.out.println(JSON.toJSONString(userList));
        } finally {
            session.close();
            reader.close();
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
	
}
```

***测试结果：*

```java
[{"age":18,"createTime":1576944000000,"id":1,"name":"水水","updateTime":1576944000000},{"age":18,"createTime":1576944000000,"id":2,"name":"豆豆","updateTime":1576944000000}]

Process finished with exit code 0
```

## 五、综上总结

- 学习完 Mybaits 核心源码，再实现一下核心过程，那么就会很清晰这个过程是怎么个流程，也就不会觉得自己知识栈有漏洞
- 只有深入的学习才能将这样的技术赋能于其他开发上，例如给ES增加这样查询包，让ES更加容易操作。其实还可以有很多创造
- 知识往往是综合的使用，将各个知识点综合起来使用，才能更加熟练。不要总看不做，否则全套的流程不能在自己脑子流程下什么印象


