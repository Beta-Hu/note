# IoC容器
- Spring通过IoC容器来管理
- 功能
	- 所有Java对象的实例化和初始化
	- 控制对象之间的依赖关系

- Spring Bean: IoC容器管理的Java对象没有区别
- IoC使用map集合来储存bean对象

- 工作机制
	- 在xml中配置Bean的定义信息(BeanDefinition)
	- 通过BeanDefinition的实现类将Bean定义信息加载到IoC
		- 例如ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
	- IoC依据Bean定义信息使用BeanFactory和反射创建对象，并初始化得到最终对象
		- context.getBean("xxx");


# 基于xml管理Bean
- 定义Bean 
	在bean.xml下添加</br>
	```xml
	<bean id="user" class="indi.beta.spring6.iocxml.User"/>
	```
- 获取Bean
	<pre>
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");

        User user1 = (User) context.getBean("user");
        User user2 = context.getBean(User.class);
        User user3 = context.getBean("user", User.class);
	</pre>
- 注意
	- 在bean.xml中创建相同class的多个对象时，不得使用getBean(User.class)来获取Bean(获取是无歧义的)
	- 如果bean.xml定义的bean的class是接口，且当该接口只有一个实现类的时候，可以通过getBean("接口bean名")来获取对象
	- 如果bean.xml定义的bean的class是接口，且当该接口有多个实现类的时候，不能使用class来获取

- 依赖注入(Dependency Injection)
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- **### 注意 ### 傻逼自动注入的对象只能是抽象类型的**
	- 基于setter
		- 通过bean.xml下的bean下的property注入</br>
			```xml
			<bean id="user1" class="indi.beta.spring6.iocxml.User">
		        <property name="age" value="15"/>
		        <property name="name" value="BBB"/>
		    </bean>
			```
	- 基于构造器
		- 通过有参构造器注入</br>
			```xml
			<bean id="user2" class="indi.beta.spring6.iocxml.User">
		        <constructor-arg name="age" value="64"/>
		        <constructor-arg name="name" value="GGG"/>
		    </bean>
			```
		- 会依据给定的属性名自动选择相应的构造器
	- 使用util注入集合
		```xml
	    <bean id="cat" class="indi.beta.spring6.iocxml.Pet">
	        <property name="name" value="catAAA"/>
	    </bean>
	    <bean id="dog" class="indi.beta.spring6.iocxml.Pet">
	        <property name="name" value="dogBBB"/>
	    </bean>
	
	    <bean id="user" class="indi.beta.spring6.iocxml.User">
	        <property name="pet" ref="map1"/>
	    </bean>
	
	    <util:map id="map1">
	        <entry key="cat" value-ref="cat"/>
	        <entry key="dog" value-ref="dog" />
	    </util:map>
		```
	- 使用p命名空间注入
		```xml
	    <bean id="cat" class="indi.beta.spring6.iocxml.Pet">
	        <property name="name" value="catAAA"/>
	    </bean>
	    <bean id="dog" class="indi.beta.spring6.iocxml.Pet">
	        <property name="name" value="dogBBB"/>
	    </bean>
	
	    <bean id="user" class="indi.beta.spring6.iocxml.User" p:pet-ref="map1"/>
	
	    <util:map id="map1">
	        <entry key="cat" value-ref="cat"/>
	        <entry key="dog" value-ref="dog" />
	    </util:map>
			```

	- 特殊值处理
		- 空</br>
			```xml
			<bean id="user1" class="iocxml.User">
		        <null/>
		    </bean>
			```
		- CDATA节(用于处理xml冲突的字符)</br>
			```xml
			<bean id="user1" class="iocxml.User">
		        <property name="age">
					<value><![CDATA[xx]]></value>
				</property>
		    </bean>
			```
		- 对象
			- 引用外部Bean</br>
			```xml
		    <bean id="pet" class="indi.beta.spring6.iocxml.Pet">
		        <property name="name" value="catAA"/>
		    </bean>
		
		    <bean id="user" class="indi.beta.spring6.iocxml.User">
		        <property name="pet" ref="pet"/>
		    </bean>
			```
				- 需要先创建需要被引用的bean，然后是哟个**ref**而非value来引用
			- 内部Bean
			```xml
	        <bean id="user" class="indi.beta.spring6.iocxml.User">
		        <property name="pet">
		            <bean id="pet" class="indi.beta.spring6.iocxml.Pet">
		                <property name="name" value="dogVV"/>
		            </bean>
		        </property>
		    </bean>
			```
			- 级联赋值</br>
			```xml
            		<bean id="pet" class="indi.beta.spring6.iocxml.Pet">
		    </bean>
		
		    <bean id="user" class="indi.beta.spring6.iocxml.User">
		        <property name="pet" ref="pet"/>
		        <property name="pet.name" value="birdDDD"/>
		 	</bean>
			```
				- 需要在外部bean的基础上实现。实际就是对外部bean的属性的赋值

		- 数组</br>
			```xml
		    <bean id="user" class="indi.beta.spring6.iocxml.User">
		        <property name="hobbit">
		            <array>
		                <value>scdad</value>
		                <value>bcbc</value>
		                <value>tfj</value>
		            </array>
		        </property>
		    </bean>
			```
		- 集合</br>
		```xml
		    <bean id="user" class="indi.beta.spring6.iocxml.User">
			<property name="hobbit">
			    <list>
				<value>dsadsa</value>
				<value>daewq</value>
				<value>yiukj</value>
			    </list>
			</property>
		    </bean>
			```
			```
		    <bean id="user" class="indi.beta.spring6.iocxml.User">
			<property name="debt">
			    <map>
				<entry>
				    <key>
					<value>key 1</value>
				    </key>
				    <value>value 1</value>
				</entry>
			    </map>
			</property>
		    </bean>
		```
- 引入外部属性文件
	- 引入数据库依赖
	```xml
      <dependency>
	  <groupId>com.mysql</groupId>
	  <artifactId>mysql-connector-j</artifactId>
	  <version>8.0.32</version>
      </dependency>
      <!-- https://mvnrepository.com/artifact/com.alibaba/druid -->
      <dependency>
	  <groupId>com.alibaba</groupId>
	  <artifactId>druid</artifactId>
	  <version>1.0.31</version>
      </dependency>
	```
	- 创建外部属性文件
	```xml
	<context:property-placeholder location="jdbc.properties"/>
	```
	- 创建spring配置文件，引入context命名空间
	```xml
	    <bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
		<property name="url" value="${url}"/>
		<property name="driverClassName" value="${driverClassName}"/>
		<property name="username" value="${username}"/>
		<property name="password" value="${password}"/>
	    </bean>
	```
	- 引入属性文件，使用表达式完成注入
	- *测试*
	```java
	public void createDruidDataSourceByXml() throws Exception{
		ApplicationContext context = new ClassPathXmlApplicationContext("bean-jdbc.xml");
		DruidDataSource dataSource = (DruidDataSource) context.getBean(DruidDataSource.class);
		System.out.println("get connection " + dataSource.getConnection());
	    }
	```
- xml Bean的作用域: scope
	- 单实例(scope="singleton", 默认的)
	```xml
	 <bean id="user-singleton" class="indi.beta.spring6.iocxml.User" scope="singleton"/>
	 ```
		- 在IoC初始化时创建
	 - 多实例(scope="prototype")
	 ```xml
	 <bean id="user-prototype" class="indi.beta.spring6.iocxml.User" scope="prototype"/>
	```
		- 在获取Bean时创建

- xml Bean的生命周期
	- Bean对象创建(调用无参构造器)
	- Bean属性注入
	- Bean后置处理器(初始化前的。随context一起，并非对单个bean执行)
	- Bean对象初始化
		- 通过bean的init-method指定，属性值不含括号
	- Bean后置处理器(初始化后的。随context一起，并非对单个bean执行)
	- Bean创建完成
	- Bean对象销毁
		- 通过bean的destroy-method指定，属性值不含括号
		- 通过context.close()执行，context销毁前将会执行对象的destroyMethod
	- IoC关闭

- FactoryBean
	- FactoryBean是一个泛型接口
	- 通过实现类的getObject()来确定实例的类

- 基于xml自动装配
	- 意义
		- 当不同类按层次引用时，可以使用自动装配来简化对属性的赋值
	- 操作
		- 在controller层定义service的属性和setter
		- 在service层定义dao的属性和setter
		- 配置自动装配
			- 创建bean并配置autowire
				```xml
				<bean id="controller" class="indi.beta.spring6.autocontroller.controller.UserController" autowire="byType"/>
				<bean id="service" class="indi.beta.spring6.autocontroller.service.UserService" autowire="byType"/>
				<bean id="dao" class="indi.beta.spring6.autocontroller.dao.UserDAO"/>
				```

# 基于注解管理Bean
- 开启注解扫描
	```xml
	<context:component-scan base-package="indi.beta.spring6"/>
	```
	- 开启后会扫描包组件中的注解
- 通过注解创建对象
	```java
	@Component(value="user")
	public class User {}
	```
	- 括号内可以省略，此时对象名为首字母小写的类名
	- 可用的注解类型
		- Component: 用于任意层的对象
		- Respository: 用于DAO层的对象
		- Service: 用于服务层的对象
		- Controlled: 用于控制层的对象
- @Autowired注入(默认依据类型)
	- 属性注入(自动依据类型注入)
		```java
	    	@Autowired
		private UserService userService;
		```
	- 方法注入(使用setter依据类型注入)
		```java
	    	@Autowired
		public void setUserDAO(UserDAO userDAO) {
			this.userDAO = userDAO;
	    	}
		```
	- 构造器注入
		```java
	    	@Autowired
    		public UserController(UserService userService){
        		this.userService = userService;
    		}	
		```
		- 也可以在形参注入
			```java
			public UserController(@Autowired UserService userService){
				this.userService = userService;
			}
			```
		- 仅有一个有参构造器且没有无参构造器时，注解可以省略
	- Autowired与Qualifier注解联合
		- 由于autowired是基于类型的，当有多个实现类时注入会失败，此时需要qualifier指定具体的实现类
		```java
		@Autowired
		@Qualifier(value = "userService")
		private UserServiceApi service; // UserServiceApi存在两个实现类UserService和UserService2
		```
- @Resource注入(依赖jakarta.annotation)
	- 匹配顺序
		- 优先byName(未指明name时以属性名作为name)
		- 无法匹配name时，自动byType来装配
	- **只能作用在属性和setter上**
	- 注入
		- 依据指定名称注入
			```java
			// 将被装配类添加注解指定名称
			@Service("myUserService")
			public class UserService{}
			```
			```java
			// 指定名称进行装配
			@Resource(name="myUserService")
			private UserServiceApi service;
			```
		- 依据属性名称注入(需要保证对象名与被装配类指定名称相同)
			```java
			@Service("myUserService")
			public class UserService{}
			```
			```java
			@Resource
			private UserServiceApi myUserService;
			```
		- 以上两种情况都无法满足时，依据类型进行注入

	- 全注解开发(完全脱离spring的xml配置文件)
		- 创建配置类
			```java
			@Configuration // 声明是配置类
			@ComponentScan("indi.beta") // 开启组件扫描
			@Scope("singleton") // 声明是单例
			public class SpringConfig {}
			```
		- 加载配置类
			```java
			ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
			// 替换了 ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
        		UserController controller = (UserController) context.getBean(UserController.class);
        		controller.getUserService().add();
			```
# 原理探究: 手写IoC

# AOP: 面向切片编程
- 代理模式: 通过代理类来间接调用方法,将业务逻辑代码与非业务逻辑代码解耦
	- 静态代理 
	```java
	public class CalculatorStaticProxy implements CalculatorApi{
		private Calculator calculator;  //被代理的类

		@Override
		public int add(int a, int b) {
			int result = calculator.add(a, b);  // 被代理类业务逻辑的方法
			System.out.println("[log]...");	// 非业务逻辑
			return result;
		}
   	 }
	```
	- 动态代理(实际就是使用反射提高了扩展性)
	```java
	// 创建代理类工厂
	public class ProxyFactory {
	    private Object target;

	    public ProxyFactory(Object target) {
		this.target = target;
	    }

	    public Object getProxy() {
		ClassLoader loader = target.getClass().getClassLoader();
		Class<?>[] interfaces = target.getClass().getInterfaces();
		InvocationHandler handler = (proxy, method, args) -> {
		    System.out.println("[log]...");	// 非业务逻辑
		    return (Object) method.invoke(target, args);	// 业务逻辑
		};
		return Proxy.newProxyInstance(loader, interfaces, handler);
	    }
	}
	```
	```java
	// 测试动态代理
	ProxyFactory proxyFactory = new ProxyFactory(new Calculator());
        CalculatorApi proxy = (CalculatorApi) proxyFactory.getProxy();
        proxy.add(1, 5);
	```
- AOP: 通过**预编译方式和运行时动态代理**实现，在不修改源代码的前提下给程序动态添加功能
	- 术语
		- 横切关注点: 不同模块中解决相同问题的非核心业务(例如日志，用户验证，数据缓存等)
		- 通知(增强): 想要增强的功能。每个横切关注点上都需要写一个通知方法来实现
			- 前置通知
			- 返回通知
			- 异常通知
			- 后置通知
			- 环绕通知
		- 切面: 封装通知方法的类
		- 目标(被代理对象)与代理(代理对象)
		- 连接点: spring允许使用通知的地方
		- 切入点: 定位连接点的方法
	- 作用
		- 简化代码: 将重复代码抽取出来，提升内聚
		- 代码增强: 将特定功能封装到切面类，被套用切面逻辑的方法就被切面增强了(例如动态代理为计算器的方法增加了日志功能)
	- 步骤分析
		- 导入依赖
			- rog.springframework.spring-aop 
			- rog.springframework.spring-aspects
		- 建立切片类
		```java
		@Aspect // 声明为切片类
		@Component  // 声明为ioc容器
		public class LogAspect{
		    // 通知类型
		    @Before(value = "execution(public int indi.beta.aop.AOPCalculator.add(..))")
		    public void beforeMethod(){
			System.out.println("[log] before add...");	// 增强的操作
		    }
		}
		```
		```java
		public class AOPTest {
	    	@Test
	    	public void test(){
			ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
			AOPCalculatorApi bean = context.getBean(AOPCalculatorApi.class);	// 一定要用那傻逼的接口类，而非实现类
			bean.add(15, 3);
	   		}
		}
		```
			- 通知类型
			```java
		    	@Before(value = "execution(public int indi.beta.aop.AOPCalculator.*(..))")
				public void beforeMethod(JoinPoint joinPoint){
				System.out.println("[log] before method @" + joinPoint.getSignature().getName());
		 	}

		    	@After(value = "execution(public int indi.beta.aop.AOPCalculator.*(..))")
		    	public void afterMethod(JoinPoint joinPoint){
				System.out.println("[log] after method @" + joinPoint.getSignature().getName());
		    	}

		    	@AfterReturning(value = "execution(public int indi.beta.aop.AOPCalculator.*(..))", returning = "returnValue")
		    	public void returnMethod(JoinPoint joinPoint, Object returnValue){
				System.out.println("[log] return method @" + joinPoint.getSignature().getName());
				System.out.println("[return] result " + returnValue);
		    	}

		    	@AfterThrowing(value = "execution(public int indi.beta.aop.AOPCalculator.*(..))", throwing = "exceptionName")
		    	public void throwMethod(JoinPoint joinPoint, Throwable exceptionName){
				System.out.println("[log] throw method @" + joinPoint.getSignature().getName());
				System.out.println("[exception] " + exceptionName);
		    	}

		    	@Around(value = "execution(public int indi.beta.aop.AOPCalculator.*(..))")
		    	public Object aroundMethod(ProceedingJoinPoint joinPoint) throws Throwable {
				System.out.println("[log] around method @" + joinPoint.getSignature().getName());
				return joinPoint.proceed();
		    	}
		    	```
			- 切入表达式
				- **execution(权限 返回类型 方法全类名.方法名(参数))**
					- 权限和返回类型可以使用*代替，表示为任意类型任意权限
					- 方法名可以使用*表示任意方法，也可以用作通配符进行筛选
					- 全类名可以使用*号表示任意类，*...表明任意包深度的任意类
		- 重用切入点
		```java
		// 声明切入点
	    	@Pointcut("execution(public int indi.beta.aop.AOPCalculator.*(..))")
    		public void putMethod(){}
		// 利用切入点方法代替。如果不在一个包下，则需要提供切入点方法全类名
    		@Before(value = "putMethod()")
    		public void beforeMethod(JoinPoint joinPoint){
        		System.out.println("[log] before method @" + joinPoint.getSignature().getName());
    		}
		```
	- 基于xml的aop
	```xml
	    <aop:config>
		<aop:aspect ref="logAspect">
		    <aop:pointcut id="pointCut" expression="execution(public int indi.beta.aop.AOPCalculator.*(..))"/>
		    <aop:before method="beforeMethod" pointcut-ref="pointCut"/>
		    <aop:after method="afterMethod" pointcut-ref="pointCut"/>
		    <aop:after-returning method="returnMethod" pointcut-ref="pointCut" returning="returnValue"/>
		    <aop:after-throwing method="throwMethod" pointcut-ref="pointCut" throwing="exceptionName"/>
		    <aop:around method="aroundMethod" pointcut-ref="pointCut"/>
		</aop:aspect>
	    </aop:config>
	```

# Spring整合JUnit
```xml
<!--开启自动扫描-->
<context:component-scan base-package="indi.beta.jdbc"/>
```
``java
// 测试类只能由空参构造器
import org.junit.jupiter.api.Test;  // 使用JUnit5而非JUnit4 (import org.junit.Test;)
@SpringJUnitConfig(locations = "classpath:bean.xml")
public class UserTest {
    @Autowired
    private User user;

    @Test
    public void test(){
        user.run();
    }
}
```

# 使用Spring完成CRUD
- 定义bean
```xml
<context:property-placeholder location="classpath:spring-jdbc.properties" system-properties-mode="NEVER"/>

<bean id="druidDataSource" class="com.alibaba.druid.pool.DruidDataSource">
<property name="url" value="${url}"/>
<property name="driverClassName" value="${driverClassName}"/>
<property name="username" value="${username}"/>
<property name="password" value="${password}"/>
</bean>

<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
<property name="dataSource" ref="druidDataSource"/>
</bean>
```
- 装配并实现CRUD
	- 增、删、改(差别仅在sql语句不同)
		```java
		@SpringJUnitConfig(locations = "classpath:bean-jdbc.xml")
		public class JDBCTest {
		    @Autowired
		    private JdbcTemplate jdbcTemplate;

		    @Test
		    public void update(){
			String sql = "INSERT INTO t_table (id, name) VALUES(?, ?)";
			int row = jdbcTemplate.update(sql, 15, "Nidd");
			System.out.println("finish!");
		    }
		}
		```
	- 查
		```java
		@SpringJUnitConfig(locations = "classpath:bean-jdbc.xml")
		public class JDBCTest {
		    @Autowired
		    private JdbcTemplate jdbcTemplate;

		    // 定义一个内部类
		    static class MyUser{
			public int id;
			public String name;

			// setter必不可少，否则查询结果由于无法赋值而全为null
			public void setId(int id) {
			    this.id = id;
			}

			public void setName(String name) {
			    this.name = name;
			}
		    }

		    @Test
		    public void select(){
			String sql = "SELECT * FROM t_table WHERE id=?";
			// 自行封装
		//        User userResult = jdbcTemplate.queryForObject(sql, (resultSet, rowIndex)->{
		//            User user = new User();
		//            user.id = resultSet.getInt("id");
		//            user.name = resultSet.getString("name");
		//            return user;
		//        }, 14);
			// 使用预定义接口实现类
			MyUser userResult = jdbcTemplate.queryForObject(sql, new BeanPropertyRowMapper<>(MyUser.class), 24);

			System.out.println(userResult.id + userResult.name);
		    }

		    @Test
		    public void selectList(){
			String sql = "SELECT * FROM t_table";
			List<MyUser> myUsers = jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(MyUser.class));

			for (MyUser user : myUsers) {
			    System.out.println("id: " + user.id + "  name: " + user.name);
			}
		    }
		}
		```
- 事务: Transactional
	- 全注解实现事务(可以使用普通测试类测试，无需添加@SpringJUnitConfig注解)
	```java
	@Configuration	// 声明为配置类
	@ComponentScan("indi.beta.bank")	// 开启自动扫描
	@EnableTransactionManagement	// 开启事务控制
	public class SpringConfig {
	    @Bean(name="dataSource")
	    public DataSource getDataSource() throws Exception {
		Properties prop = new Properties();
		prop.load(SpringConfig.class.getClassLoader().getResourceAsStream("spring-jdbc.properties"));
		return DruidDataSourceFactory.createDataSource(prop);
	    }

	    @Bean(name="jdbcTemplate")
	    public JdbcTemplate getTemplate(DataSource dataSource){
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		jdbcTemplate.setDataSource(dataSource);
		return jdbcTemplate;
	    }

	    @Bean(name="transactionManager")
	    public TransactionManager getTransactionManager(DataSource dataSource){
		DataSourceTransactionManager manager = new DataSourceTransactionManager();
		manager.setDataSource(dataSource);
		return manager;
	    }
	}
	```
	```java
	@Service
	public class BankService implements BankServiceApi{
	    @Autowired
	    private BankDAOApi bankDAO;
		
	    @Transactional	// 将服务声明为事务，满足其ACID特性
	    public void transfer(int targetId, int sourceId, int balance){
		bankDAO.updateBalance(sourceId, balance);
		bankDAO.updateBalance(targetId, -balance);
	    }
	}
	```
# 国际化
- Java国际化
	- 依据规范命名的配置文件可以被Locale读取
		```java
		ResourceBundle bundle = ResourceBundle.getBundle("message", new Locale("zh", "CN"));
		System.out.println(bundle.getString("region"));
		```
	- 配置文件名称为: baseName_language_country.properties。例如message_zh_CN.properties
- Spring国际化

# 数据校验
- 
