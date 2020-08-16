# Mybatis

环境：

-   JDK1.8
-   Mysql5.7
-   maven3.6.1
-   IDEA

回顾

-   JDBC
-   Mysql
-   Java基础
-   Maven
-   Junit



SSM框架：配置文件的。 

最好的学习方式，学习官方文档

# 1.简介

## 1.1 什么是Mybatis

-   MyBatis 是一款优秀的**持久层框架**
-   它支持自定义 SQL、存储过程以及高级映射
-   MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。
-   MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。
-   2013年11月迁移至Github



如何获得Mybatis？

-   Maven仓库

    ```xml
    <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.2</version>
    </dependency>
    ```

-   Github：https://github.com/mybatis/mybatis-3

-   中文文档：https://mybatis.org/mybatis-3/zh/index.html



## 1.2 什么是持久化

数据持久化

-   持久化就是将程序的数据从持久状态和瞬时状态转化的过程
-   内存： **断电即失**
-   数据库（JDBC）， IO文件持久化
-   生活：冷藏、罐头



**为什么需要持久化？**

-   有一些对象，不能让他丢失
-   内存价格昂贵，硬盘便宜



## 1.3 持久化

Dao层、Service层，Controller层…

-   完成持久化工作的代码块
-   层与层之间的界限非常明显



## 1.4 为什么需要Mybatis？

-   帮助程序员将数据存入数据库。

-   方便
-   传统的JDBC代码过于复杂，所以出现了框架对原有的编码方式进行简化------》 自动化
-   不用Mybatis也可以，更容易上手。**技术没有高低之分**
-   优点：
    -   简单易学
    -   灵活
    -   sql和代码分离，提高可维护性
    -   提供映射标签，支持对象与数据库的orm字段关系映射
    -   提供对象关系映射标签，支持对象关系组建维护
    -   提供xml标签，支持编写动态sql



**最重要的一点：使用人数多，生态氛围良好**

Spring SpringMVC SpringBoot



# 2. 第一个Mybatis程序

思路： 搭建环境 --》 导入Mybatis --》 编写代码 ---》 测试！

![image-20200815124114712](Mybatis.assets/image-20200815124114712.png)

## 2.1 搭建环境

搭建数据库

```sql
CREATE DATABASE `mybatis`;

USE `mybatis`;

CREATE TABLE `user`(
	`id` INT(20) PRIMARY KEY NOT NULL;
	`name` VARCHAR(30) DEFAULT NULL;
	`pwd` VARCHAR(30) DEFAULT NULL;
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user` (`id`, `name`, `pwd`) VALUES
	(1, '老八', '123456'),
	(2, '张三', '123456'),
	(3, '李四', '123456'),
```



新建项目

1.  新建一个普通的maven项目

2.  删除src目录（将该文件作为父工程）

3.  导入maven依赖

    ```xml
    <dependencies>
    <!--    mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
    <!--    mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
    <!--    junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    ```



## 2.2 创建一个模块

-   编写mybatis的核心配置文件

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE configuration
            PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-config.dtd">
    <!--核心配置文件-->
    <configuration>
        <environments default="development">
    <!--    环境-->
            <environment id="development">
    <!--        事务管理-->
                <transactionManager type="JDBC"/>
                <dataSource type="POOLED">
                    <property name="driver" value="com.mysql.jdbc.Driver"/>
                    <property name="url" value="jdbc:mysql://localhost:3306/mybatis?userSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                    <property name="username" value="root"/>
                    <property name="password" value="123456"/>
                </dataSource>
            </environment>
        </environments>
        
    <!--    控制作用域，删掉作用与全局-->
    <!--    <mappers>-->
    <!--        <mapper resource="org/mybatis/example/BlogMapper.xml"/>-->
    <!--    </mappers>-->
    </configuration>
    ```

    

-   编写mybatis工具类

    ```java
    //sqlSessionFactory --> sqlSession
    public class MybatisUtils {
        private static SqlSessionFactory sqlSessionFactory;
        static{
            try {// 使用Mybatis第一步：获取sqlSessionFactory对象
                String resource = "mybatis-config.xml";
                InputStream inputStream = Resources.getResourceAsStream(resource);
                sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    
        //既然有了 SqlSessionFactory，顾名思义，我们可以从中获得 SqlSession 的实例。SqlSession 提供了在数据库执行 SQL 命令所需的所有方法。
        // 你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句
        public static SqlSession getSqlSession(){
            return sqlSessionFactory.openSession();
        }
    }
    ```



## 2.3 编写代码

-   实体类

    ```java
    public class User {
        private int id;
        private String name;
        private String pwd;
    
        public User(int id, String name, String pwd) {
            this.id = id;
            this.name = name;
            this.pwd = pwd;
        }
    
        public User() {
        }
    
        public int getId() {
            return id;
        }
    
        public void setId(int id) {
            this.id = id;
        }
    
        public String getName() {
            return name;
        }
    
        public void setName(String name) {
            this.name = name;
        }
    
        public String getPwd() {
            return pwd;
        }
    
        public void setPwd(String pwd) {
            this.pwd = pwd;
        }
    
        @Override
        public String toString() {
            return "User{" +
                    "id=" + id +
                    ", name='" + name + '\'' +
                    ", pwd='" + pwd + '\'' +
                    '}';
        }
    }
    ```

-   Dao接口

    ```java
    public interface UserDao {
        List<User> getUserList();
    }
    ```

-   接口实现类(由原来的UserDaoImpl文件转变为一个Mapper配置文件)

    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <!--nameSpace 绑定一个对应的Dao/Mapper接口-->
    <mapper namespace="com.kuang.dao.UserDao">
        <!--查询语句-->
        <select id="getUserList" resultType="com.kuang.pojo.User">
            select * from `user`
        </select>
    </mapper>
    ```

    



## 2.4 测试

**注意点：**

`org.apache.ibatis.binding.BindingException: Type interface com.kuang.dao.UserDao is not known to the MapperRegistry.`

```xml
<!--每一个Mapper.xml 都需要在Mybatis核心配置文件中注册-->
    <mappers>
        <mapper resource="com/kuang/dao/UserMapper.xml"/>
    </mappers>
```

**xml文件中不要有任何中文注释**

-   junit测试

    ```xml
        @Test
        public void test(){
            //第一步：获得SqlSession对象
            SqlSession sqlSession = MybatisUtils.getSqlSession();
            //方式一：执行SQL
            UserDao userDao = sqlSession.getMapper(UserDao.class);
            List<User> userList = userDao.getUserList();
            for (User user : userList) {
                System.out.println(user);
            }
    
            //关闭sqlSession
            sqlSession.close();
        }
    ```



可能会遇到的问题：

1.  配置文件没有注册
2.  绑定接口不对
3.  方法名不对
4.  返回类型不对
5.  Maven导出资源问题

