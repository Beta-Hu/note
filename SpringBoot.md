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
