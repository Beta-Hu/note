# 搭建MyBatis
- 创建核心配置文件(可从MyBatis官方文档获取)
  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!-- 存在于MyBatis官方文档中 -->
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "https://mybatis.org/dtd/mybatis-3-config.dtd">
  <configuration>
      <!-- 核心配置文件中的标签是有序的，应当按顺序配置 -->
      <!-- 引入properties文件 -->
      <properties resource="jdbc.properties"/>
      <typeAliases>
          <!-- 配置别名，简化mapper.xml中的类名引用 -->
          <!-- alias默认为类名，且不区分大小写 -->
          <typeAlias type="indi.beta.pojo.User" alias="user"/>
          <!-- 将某个软件包下所有类都使用其类名作为别名，用于类过多的情况 -->
          <package name="indi.beta.pojo"/>
      </typeAliases>
      <environments default="development">
          <environment id="development">
              <!-- transactionManager: JDBC(原生的事务管理方式，手动提交) | MANAGED -->
              <transactionManager type="JDBC"/>
              <!-- dataSource: POOLED(使用连接池) | UNPOOLED(不使用连接池) | JNDI(使用上下文数据源) -->
              <dataSource type="POOLED">
                  <property name="driver" value="${jdbc.driver}"/>
                  <property name="url" value="${jdbc.url}"/>
                  <property name="username" value="${jdbc.username}"/>
                  <property name="password" value="${jdbc.password}"/>
              </dataSource>
          </environment>
      </environments>
      <!-- 配置映射文件 -->
      <mappers>
          <mapper resource="org/mybatis/example/BlogMapper.xml"/>
      </mappers>
  </configuration>
  ```
- 创建映射文件
  - 映射文件: java对象与数据库对象之间映射关系。
  - 一般来说，一张表对应了一个映射文件。推荐对每张表建立单独的mapper
  - 两个一致
    - 映射文件namespace与全类名必须一致
    - 映射文件中SQL语句的id与mapper接口中的方法名一致
  - 添加一条映射
    ```xml
    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper
            PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
    <mapper namespace="indi.beta.mapper.UserMapper">
        <insert id="insertUser">
            insert into t_table(id, name) values (3001, 'AlphaDog')
        </insert>
        <delete id="deleteUserById">
            delete from t_table where id=3001
        </delete>
        <!-- 查询语句必须设置resultType或rsultMap属性，从而实现查询结果与java类的映射 -->
        <!-- resultType用于数据表字段与java类属性一致的情况，resultMap用于不一致的情况 -->
        <select id="queryUserById" resultType="indi.beta.pojo.User">
            select * from t_table where id=15
        </select>
    </mapper>
  - 在mybatis配置中导入该映射
  ```xml
  <!-- 配置映射文件 -->
  <mappers>
      <mapper resource="mappers/UserMapper.xml"/>
      <!-- 将目录下所有mapper.xml全部导入，要求被导入的xml包名与对应的接口类包名相同，且文件名(不含后缀)相同 -->
      <package name="indi.beta.mapper"/>
  </mappers>
  ```
- 执行
  ```java
  // 加载核心配置文件
  InputStream resource = Resources.getResourceAsStream("mybatis-config.xml");
  // 获取SqlSessionFactoryBuilder
  SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
  // 获取SqlSessionFactory
  SqlSessionFactory factory = builder.build(resource);
  // 获取SqlSession。可以为openSession提供参数true实现自动提交
  SqlSession sqlSession = factory.openSession();
  // 获取mapper接口对象
  UserMapper mapper = sqlSession.getMapper(UserMapper.class);
  // 执行mapper中的方法
  int result = mapper.insertUser();
  sqlSession.commit();
  System.out.println("Row affected: " + result);
  ```

# MyBatis获取参数
- 通过#{}或${}获取单个参数
  ```xml
  <!-- User queryUserById(int id) -->
  <select id="queryUserById" resultType="user">
      select * from t_table where id=#{id}
  </select>
  ```
  - _#{}与其中包括的变量名无关，但不能缺省。推荐设置为方法的参数名_
  - _字面量为字符串时，应当使用单引号将${}包裹起来_
  - _获取单个字面量时，${}与#{}时等效的_
- 获取多个参数(自动)
  ```xml
  <!-- User queryUserById(int id) -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{arg0} and name=#{arg1}
  </select>
  ```
  - _使用arg作为键时，从0开始。使用param作为键时，从1开始。二者可以混用，但不推荐_
  - _键的创建是自动的_
  - _也可以使用${},依然需要注意字符串拼接问题_
- 获取多个参数(手动)
  ```xml
  <!-- User queryUserById(int id, String name) -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{my_key_1} and name=#{my_key_2}
  </select>
  ```
  - _将方法的参数类型设置为Map，并将键设置为字符串_
  - _在mapper.xml中使用自定义的键即可访问_
- 获取实体类对象
  ```xml
  <!-- User insertUser(User user) -->
  <!-- User {int id; String name;} -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{id} and name=#{name}
  </select>
  ```
  - _直接使用实体类的属性进行访问_
  - **_需要提供getter和setter_**
-使用@Param注解为变量赋键
  ```xml
  <!-- User checkUser(@Param("userId") int id, @Param("UserName") String name); -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{userId} and name=#{userName}
  </select>
  ```
  - _即使使用了@Param注解，依然可以使用param作为键来获取参数_
