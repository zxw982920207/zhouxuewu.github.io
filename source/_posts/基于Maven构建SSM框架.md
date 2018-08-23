---

title: 基于Maven构建SSM框架教程
date: 2016-08-20
tags: [java,maven,spring,idea]

---

![](http://pccmxww5q.bkt.clouddn.com/spring.png?imageView2/0/w/560/h/380/q/100 "")

#### 1.建立数据库表

```mysql
CREATE TABLE `t_blogger` (
  id int(11) NOT NULL AUTO_INCREMENT,
  username varchar(50) DEFAULT NULL,
  password varchar(100) DEFAULT NULL, 
  PRIMARY KEY (`id`)
) DEFAULT CHARSET=utf8;
```



#### 2.Maven方式搭建Spring+SpringMVC+Mybatis环境

- ##### pom.xml

  ```xml
  <!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-context</artifactId>
              <version>4.3.7.RELEASE</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/javax.servlet/servlet-api -->
          <dependency>
              <groupId>javax.servlet</groupId>
              <artifactId>servlet-api</artifactId>
              <version>2.5</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-webmvc -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-webmvc</artifactId>
              <version>4.3.7.RELEASE</version>
          </dependency>
          <!-- 添加连接池druid支持 -->
          <dependency>
              <groupId>com.alibaba</groupId>
              <artifactId>druid</artifactId>
              <version>1.0.16</version>
          </dependency>
          <!-- 添加mybatis支持 -->
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis</artifactId>
              <version>3.3.0</version>
          </dependency>
          <!-- 添加mybatis-spring支持-->
          <dependency>
              <groupId>org.mybatis</groupId>
              <artifactId>mybatis-spring</artifactId>
              <version>1.2.3</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.springframework/spring-beans -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-beans</artifactId>
              <version>4.3.8.RELEASE</version>
          </dependency>

          <!-- https://mvnrepository.com/artifact/org.springframework/spring-dao -->
          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-dao</artifactId>
              <version>2.0.3</version>
          </dependency>

          <dependency>
              <groupId>org.springframework</groupId>
              <artifactId>spring-jdbc</artifactId>
              <version>4.1.7.RELEASE</version>
          </dependency>

          <!-- jdbc驱动包  -->
          <dependency>
              <groupId>mysql</groupId>
              <artifactId>mysql-connector-java</artifactId>
              <version>5.1.37</version>
          </dependency>
  ```

  ​


- ##### Spring配置文件applicationContext.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns:context="http://www.springframework.org/schema/context"
         xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

      <!-- 启用spring-mvc注解-->
      <context:annotation-config/>

      <!-- 配置数据源 -->
      <bean id="dataSource"
            class="com.alibaba.druid.pool.DruidDataSource">
          <property name="url" value="jdbc:mysql://localhost:3306/db_springdemo"/>
          <property name="username" value="root"/>
          <property name="password" value="123456"/>
      </bean>

      <!-- 配置mybatis的sqlSessionFactory -->
      <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
          <property name="dataSource" ref="dataSource" />
          <!-- 自动扫描mappers.xml文件 -->
          <property name="mapperLocations" value="classpath*:cn.successli.mapper/*.xml"></property>
      </bean>

      <!-- DAO接口所在包名，Spring会自动查找其下的类 -->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
          <property name="basePackage" value="cn.successli.dao" />
          <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
      </bean>

      <!-- 自动扫描 -->
      <context:component-scan base-package="cn.successli.service" />

  </beans>
  ```


- ##### spring-mvc.xml

  ```xml
  <mvc:annotation-driven />

         <!-- 视图解析器 -->
         <bean id="viewResolver"
               class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                <property name="prefix" value="/" />
                <property name="suffix" value=".jsp"></property>
         </bean>

         <!-- 使用注解的包，包括子集 -->
         <context:component-scan base-package="cn.successli.controller" />
  ```

  ​


- ##### web.xml

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee"
           xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
           id="WebApp_ID" version="2.5">
      <display-name>aa</display-name>
      <welcome-file-list>
          <welcome-file>a.jsp</welcome-file>
      </welcome-file-list>

      <!-- 配置web.xml，使其具有springmvc特性，主要配置两处，一个是ContextLoaderListener，一个是DispatcherServlet -->

      <!-- spring mvc 配置 -->
      <!-- 1.配置DispatcherServlet表示，该工程将采用springmvc的方式。 -->
      <servlet>
          <servlet-name>webmvc</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <!-- 启动项目的时候要加载的配置文件 -->
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath*:/spring-mvc.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>webmvc</servlet-name>
          <url-pattern>/</url-pattern>
      </servlet-mapping>

      <!-- 2.spring配置 :配置ContextLoaderListener表示，该工程要以spring的方式启动.启动时会默认在/WEB-INF目录下查找applicationContext.xml
  作为spring容器的配置文件，该文件里可以初始化一些bean-->
      <listener>
          <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
      </listener>

      <!-- 指定Spring Bean的配置文件所在目录。默认配置在WEB-INF目录下 -->
      <context-param>
          <param-name>contextConfigLocation</param-name>
          <param-value>classpath*:applicationContext.xml</param-value>
      </context-param>

      <!-- 字符集过滤器 -->
      <filter>
          <filter-name>encodingFilter</filter-name>
          <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
          <init-param>
              <param-name>encoding</param-name>
              <param-value>UTF-8</param-value>
          </init-param>
      </filter>
      <filter-mapping>
          <filter-name>encodingFilter</filter-name>
          <url-pattern>/*</url-pattern>
      </filter-mapping>

  </web-app>
  ```

  以上，Spring+SpringMVC+Mybatis的开发环境基本搭建好。

  建立实体类，完成对数据库的操作。

#### 3.建立实体类及Mapper对象

- 实体类User

  ```java
  package cn.successli.entity;

  /**
   * Created by liyon on 2017/5/30.
   */
  public class User {

      private Integer id;
      private String username;//用户名
      private String password;//密码

      public Integer getId() {
          return id;
      }

      public void setId(Integer id) {
          this.id = id;
      }

      public String getUsername() {
          return username;
      }

      public void setUsername(String username) {
          this.username = username;
      }

      public String getPassword() {
          return password;
      }

      public void setPassword(String password) {
          this.password = password;
      }
  }
  ```

- 建立DAO层及Mapper对象

  UserDAO.java

  ```java
  package cn.successli.dao;

  import cn.successli.entity.User;
  import org.springframework.stereotype.Component;

  /**
   * Created by liyon on 2017/5/30.
   */

  public interface UserDAO {

      /**
       * 增加一个用户
       * @param user
       * @return
       */
      public Integer add(User user);

      /**
       * 删除一个用户
       * @param user
       * @return
       */
      public Integer delete(User user);

      /**
       * 修改某个用户信息
       * @param user
       * @return
       */
      public Integer modify(User user);

      /**
       * 通过用户名查找用户信息
       * @param username
       * @return
       */
      public User findUserByName(String username);
  }

  ```

  UserMapper.xml

  ```java
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE mapper
          PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

  <mapper namespace="cn.successli.dao.UserDAO">
      <resultMap id="UserResult" type="cn.successli.entity.User">
          <result property="id" column="id"/>
          <result property="username" column="username"/>
          <result property="password" column="password"/>
      </resultMap>


      <insert id="add" useGeneratedKeys="true" keyProperty="id">
          INSERT INTO t_blogger (username, password)
          VALUES (#{username},#{password});
      </insert>

      <delete id="delete">
          DELETE FROM t_blogger WHERE id = #{id};
      </delete>

      <update id="modify">
          UPDATE t_blogger
          SET username = #{username}, password = #{password}
          WHERE id = #{id};
      </update>

      <select id="findUserByName" resultMap="UserResult">
          SELECT * FROM t_blogger WHERE username = #{username};
      </select>
      
  </mapper>
  ```

  ​
#### 4.编写测试代码

- 测试代码需要引用的包

  ```xml
  <!-- https://mvnrepository.com/artifact/junit/junit -->
  <dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
  </dependency>

  <!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>2.5</version>
  </dependency>
  ```

- 测试类基本配置

  ```java
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations = "classpath:applicationContext.xml")
  ```

  ​

- 测试类的编写

  ```java
  package cn.successli.dao;

  import cn.successli.entity.User;
  import junit.framework.Assert;
  import junit.framework.TestCase;
  import org.junit.Test;
  import org.junit.runner.RunWith;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.test.context.ContextConfiguration;
  import org.springframework.test.context.junit4.AbstractJUnit4SpringContextTests;
  import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

  import javax.annotation.Resource;

  import static org.junit.Assert.*;

  /**
   * Created by liyon on 2017/5/30.
   */
  @RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(locations = "classpath:applicationContext.xml")
  public class UserDAOTest extends AbstractJUnit4SpringContextTests {

      @Resource
      private UserDAO userDAO;

      @Test
      public void testAdd() throws Exception {
          User user = new User();
          user.setUsername("hou");
          user.setPassword("123456");
          int result = userDAO.add(user);
          org.junit.Assert.assertEquals(1, result);
      }

      @Test
      public void testDelete() throws Exception {
          User user = userDAO.findUserByName("hou");
          int result = userDAO.delete(user);
          org.junit.Assert.assertEquals(1, result);
      }

      @Test
      public void testModify() throws Exception {
          User user = userDAO.findUserByName("hou");
          user.setUsername("houyr");
          user.setPassword("123");
          int result = userDAO.modify(user);
          org.junit.Assert.assertEquals(1, result);
      }

      @Test
      public void testFindUserByName() throws Exception {
          User user = userDAO.findUserByName("hou");
          org.junit.Assert.assertNotNull(user);
      }
  }
  ```




测试成功，完成Demo。
