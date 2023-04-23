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
