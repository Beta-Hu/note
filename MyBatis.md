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
  - _${}的底层实现是字符串拼接，#{}底层实现是占位符赋值。${}可能出现sql注入_
  - _#{}会被替换为?，如果sql中本身就含有sql，则会产生冲突_
  - _#{}会自动将值的左右两侧添加单引号，这是很多问题的根源_
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
- 获取实体类对象(推荐)
  ```xml
  <!-- User insertUser(User user) -->
  <!-- User {int id; String name;} -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{id} and name=#{name}
  </select>
  ```
  - _直接使用实体类的属性进行访问_
  - **_需要提供getter和setter_**
-使用@Param注解为变量赋键(推荐)
  ```xml
  <!-- User checkUser(@Param("userId") int id, @Param("UserName") String name); -->
  <select id="checkUser" resultType="user">
      select * from t_table where id=#{userId} and name=#{userName}
  </select>
  ```
  - _即使使用了@Param注解，依然可以使用param作为键来获取参数_

# 各种查询
- 聚合函数
  - sql中的聚合函数在mybatis中依然是可用的，只是需要将resultType指定为相应的返回值类型
- 将结果返回为Map集合
  ```xml
  <!-- Map<String, Objects> getUserByIdToMap(int id);-->
  <select id="getUserByIdToMap" resultType="Map">
      select * from t_table where id=#{id}
  </select>
  ```
  - _返回值的键为字段名，返回值的值为属性值_
  - _**返回记录有多条时，需要将方法返回值类型改为List<Map<String, Objects>>**_
  - _返回记录有多条时，可以在mapper方法前通过@MapKey注解将某个字段设置为键_
- 模糊查询
  - 由于#{}采用的是占位符赋值方式，在解析时首先被替换为?。在模糊查询中，该?会出现在字符串中，被认为是字符串的一部分，导致占位符无法被赋值
  - 解决方案1: 采用${}替换#{}
    ```xml
    <!-- List<User> getUserByLike(String name);-->
    <select id="getUserByLike" resultType="user">
        select * from t_table where name like '${name}%'
    </select>
    ```
  - 解决方案2: 采用sql的concat函数进行拼接
    ```xml
    <!-- List<User> getUserByLike(String name);-->
    <select id="getUserByLike" resultType="user">
        select * from t_table where name like concat(#{name}, '%')
    </select>
    ```xml
  - 解决方案3: 采用双引号将模糊符号包裹起来
    ```xml
    <!-- List<User> getUserByLike(String name);-->
    <select id="getUserByLike" resultType="user">
        select * from t_table where name like #{name}"%"
    </select>
    ```
- 批量删除
  ```xml
  <!-- int batchDelete(String ids);-->
  <delete id="batchDelete">
      delete from t_table where id in (${ids})
  </delete>
  ```
  - _此时不能使用#{}, 因为它会将多个值拼接为1个字符串，导致sql总是找不到匹配的记录。当条件字段是数值型时，会报数值转换异常_
- 动态表名
  ```xml
  select * from ${t_name}
  ```
  - 动态表名只能使用${}，因为表名不能为字符串，而#{}会自动添加单引号
- 添加时获取自增主键
  - 由于insert的返回值是受影响行数，因此只能需要考虑将主键值通过mapper方法的参数传递出来
  ```xml
  <!-- void insertUserReturnAutoIncrementKey(User user);-->
  <insert id="insertUserReturnAutoIncrementKey" useGeneratedKeys="true" keyProperty="id">
      insert into t_table values(null, #{name})
  </insert>
  ```
  - 利用java传引用的特性，传入的参数不需要设置id，而会通过mybatis自动将其值设置为自增主键值
  - 通过java语句可以查看user被赋予的id

# 映射关系: 解决字段名与属性名不一致问题
- 使用字段别名
  - _当字段名与属性名不一致时，该属性无法被赋值，因此最终结果为null_
  - 通过将sql字段设置与属性名一致的别名可以解决该问题
- 通过核心配置文件中的setting进行配置(官方的下划线转驼峰)
  ```xml
  <settings>
      <setting name="mapUnderscoreToCamelCase" value="true"/>
  </settings>
  - _**要求属性名与字段名必须对应，除了命名规则导致的不一致外**_
- 通过resultMap
  ```xml
  <!-- 配置映射规则 -->
  <!-- type为数据库对应的java类的全类名或alias -->
  <resultMap id="empResultMap" type="employee">
      <!-- id用于配置主键映射 -->
      <!-- propertu为属性名，column为字段名 -->
      <id property="employeeId" column="employee_id"/>
      <!-- result用于配置非主键映射 -->
      <result property="firstName" column="first_name"/>
      <result property="lastName" column="last_name"/>
      <result property="email" column="email"/>
      <result property="phoneNumber" column="phone_number"/>
      <result property="hireDate" column="hire_date"/>
      <result property="jobId" column="job_id"/>
      <result property="salary" column="salary"/>
      <result property="commissionPct" column="commission_pct"/>
      <result property="managerId" column="manager_id"/>
      <result property="departmentId" column="department_id"/>
  </resultMap>
  <!-- 使用resultMap代替resultType进行映射 -->
  <select id="getEmployeeById" resultMap="empResultMap">
      select * from employees where employee_id=#{id}
  </select>
  ```
- 解决多对一的映射关系: 级联赋值
  ```xml
  <select id="getEmployeeWithJobHistoryById" resultMap="simplifiedEmployeeMap">
      select e.employee_id e_id, concat(e.first_name, e.last_name) name, j.employee_id j_e_id, j.start_date j_s_date, j.end_date j_e_date, j.job_id j_j_id, j.department_id j_d_id from employees e right join job_history j on e.employee_id = j.employee_id where e.employee_id=#{id};
  </select>

  <resultMap id="simplifiedEmployeeMap" type="simplifiedEmployee">
      <!-- SimplifiedEmployee包含id name jobHistory属性，其中jobHistory为JobHistory对象 -->
      <id property="id" column="eid"/>
      <result property="name" column="name"/>
      <result property="jobHistory.employeeId" column="j_e_id"/>
      <result property="jobHistory.startDate" column="j_s_date"/>
      <result property="jobHistory.endDate" column="j_e_date"/>
      <result property="jobHistory.jobId" column="j_j_id"/>
      <result property="jobHistory.departmentId" column="j_d_id"/>
  </resultMap>
  ```
- 解决多对一的映射关系: association
  ```xml
<select id="getEmployeeWithJobHistoryById" resultMap="simplifiedEmployeeMap">
      select e.employee_id e_id, concat(e.first_name, e.last_name) name, j.employee_id j_e_id, j.start_date j_s_date, j.end_date j_e_date, j.job_id j_j_id, j.department_id j_d_id from employees e right join job_history j on e.employee_id = j.employee_id where e.employee_id=#{id};
  </select>

  <resultMap id="simplifiedEmployeeMap" type="simplifiedEmployee">
      <id property="id" column="eid"/>
      <result property="name" column="name"/>
      <association property="jobHistory" javaType="indi.beta.bean.JobHistory">
          <result property="employeeId" column="j_e_id"/>
          <result property="startDate" column="j_s_date"/>
          <result property="endDate" column="j_e_date"/>
          <result property="jobId" column="j_j_id"/>
          <result property="departmentId" column="j_d_id"/>
      </association>
  </resultMap>
  ```
