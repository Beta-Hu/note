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
	- 动态代理
