# SpringBoot快速入门
- 导入maven依赖
  ```xml
  <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>3.0.6</version>
  </parent>

  <dependencies>
      <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-web</artifactId>
      </dependency>
  </dependencies>

  <build>
      <plugins>
          <plugin>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-maven-plugin</artifactId>
          </plugin>
      </plugins>
  </build>
  ```
- 编写控制器
  ```java
  @RestController
  public class HelloController {
      @RequestMapping("/hello")
      public String toHello(){
          return "Hello World!";
      }
  }
  ```
- 编写引导类
  ```java
  @SpringBootApplication
  public class HelloApplication {
      public static void main(String[] args) {
          SpringApplication.run(HelloApplication.class, args);
      }
  }
  ```

# 起步依赖原理
- spring-boot-start-parent
  - 定义了各种技术的版本，通过依赖继承获取父项目的依赖

# 配置文件
- 文件名必须是application
- 支持的格式(优先级从高到低，同名属性以高优先级为准)
  - properties
    - 通过xx.xx表示所属关系 
  - yml
    - 通过缩进表示所属关系
  - yaml
    - 通过缩进表示所属关系
- YAML
  - 要求
    - 大小写敏感
    - 缩进只能使用空格，不允许tab
    - 同级缩进相同即可，具体空格数不做要求
    - 冒号后必须要使用空格隔开
  - 数据格式
    - 对象
      ```yml
      person: 
       name: zhangs
      person: (name: zhang)
      ```
    - 数组
      ```yml
      address:
       - bj
       - sh
      addr: [bg, xz]
      ```
    - 纯量
      ```yml
      msg1: 'dadsadada' # 忽略转义字符
      msg2: "dadsada"   # 识别转义字符
      ```
  - 参数引用
    ```yml
    name: dra
    person:
      name: ${name}
    ```
  - 读取配置文件内容
    - @Value注入
      ```java
      @Value("${name}")
      private String name;

      @Value("${addr[0]}")  // 访问数组的元素
      private String addr;
      ```
    - Environment
      ```java
      @Autowired
      private Environment env;
      
      env.getProperty("name")
      ```
    - @Configuration
      ```java
      @Component
      @ConfigurationProperties(prefix = "user")
      // 类名最好不要与系统用户名相同，否则会出现获取属性值始终为系统用户名的问题
      public class User {
        private String name;
        private int age;
        
        // getters
        // setters
        
        // 必须要保留空参构造器
      }
      ```
  - profile: 用于多个环境下的参数切换(dev/test/prod)
    - 通过profile参数
      ```yaml
      spring:
        profiles:
          active: test
      ```
    - 通过命令行参数
      ```
      java -jar xxx.jar --spring.profile.active=test
      ```
    - 通过虚拟机参数
      ```
      -Dspring.profile.active=test
      ```
  - 内部配置加载顺序(优先级由高到低，同名属性以高优先级为准)
    - 当前项目下的config目录(%paren%/config/*.yml)
    - 当前项目的根目录(%parent%/*.yml)
    - classpath下的config(resources/config/*.yml)
    - classpath的根目录(resources/*.yml)
  - 外部配置加载顺序(常用的。优先级由高到低，同名属性以高优先级为准)
    - 命令行参数
    - 命令行指定外部配置文件
    - jar同级目录下的配置文件(config/*.yml或*.yml)

# springboot整合其他框架
- JUnit
  - 搭建springboot工程
  - 引入starter-test起步依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    ```
  - 编写测试类
    ```java
    @RunWith(SpringRunner.class)
    @SpringBootTest(classes = MyApplication.class)
    public class UserTest {
        @Autowired
        private UserService service;

        @Test   // org.junit.Test
        public void test(){
            service.add();
        }
    }
    ```
- Redis
  - ...
- MyBatis
  - 搭建springboot工程
  - 引入mybatis起步依赖，添加mysql驱动
    ```xml
    <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>2.1.3</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.32</version>
    </dependency>
    ```
  - 编写DataSource和MyBatis相关配置
    ```yml
    spring:
      datasource:
        url: jdbc:mysql://localhost:3306/dbtest
        username: root
        password: '0000'  # 密码必须要使用单引号包含
        driver-class-name: com.mysql.cj.jdbc.Driver
    ```
  - 定义表和实体类
  - 编写DAO和mapper
    ```java
    @Mapper
    public interface UserMapper {
        @Select("select * from t_table")
        List<User> getUserById();
    }
    ```
    - _实测引导类加不加@MapperScan("")对mapper的自动注入没有影响_

# SpringBoot高级
- 自定义条件类
  - 用于实现 在一定条件下创建bean。
  - 需要实现Condition接口，重写matches方法。matches返回为true时创建bean
    - 参数context：上下文对象。用于获取属性值、类加载器、BeanFactory等
    - 参数metadata：元数据对象。获取注解属性
- 更换内置Web服务器
  - 通过导入不同的坐标即可实现切换
    - spring-boot-stater-jetty
    - spring-boot-stater-netty
    - spring-boot-stater-tomcat (默认的，自动导入)
    - spring-boot-stater-undertow
  - 切换为其他服务器时，需要在spring-boot-starter-web中排除tomcat依赖
    ```html
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
    ```
- Enable\*注解
- 监听机制
  - 实现ApplicationContextInitializer
    - 需要在META-INF/spring.factories中注册
      - org.springframework.context.ApplicationContextInitializer=XXXXXX(自定义的监听器全路径名)
  - 实现SpringApplicationRunListener(功能更多)
    - 需要声明参数为SpringApplecation 和 String[] 的构造器
    - 需要在META-INF/spring.factories中注册
      - org.springframework.context.SpringApplicationRunListener=XXXXXX(自定义的监听器全路径名)
  - 实现ApplicationRunner
    - 不需要注册，项目启动后自动执行
  - 实现CommandLineRunner
    - 不需要注册，项目启动后自动执行
  
