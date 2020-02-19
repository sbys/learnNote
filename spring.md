Spring的整体架构
	核心容器：beans，core，context，expression language
	基于核心容器的：aop，aspects
	web部分：web，servlet
	data：jdbc，orm，oxm，jms，transactions
完成spring的功能，至少需要的类：
	configReader：配置读取以及验证
	ReflectionUtil：反射实例化
	App：完成整个逻辑的串联
spring的作用也就是配置文件，然后根据配置文件实例化
三个核心部分：
	ConfigReader：读取及验证配置文件
	ReflectionUtil：根据配置文件中的配置进行反射实例化
	App:用于完成整个逻辑的串联 
DefaultListtableBeanFactory，xmBeanFactory继承自这个类，是整个bean加载的核心部分，是spring注册及加载bean的默认实现，xmlBeanFactory中使用了自定义的xml读取器XmlBeanDefinitionReader
读取配置的过程：
	获取XML文件验证模式
	加载xml文件，并得到对应的document
	根据返回的Document注册bean信息
得到Document，解析：
	profile解析
	解析element，得到beanDefinitionHolder 
		提取id及name属性，解析其他属性（class，scoe，，singeton，lazy-init，property这些）到BeanDefinition类型的实例中，指定beanName，将信息封装到BeanDefinitionHoder       
			lookupmethod，为某个方法指定其返回的bean
			replaced-method，替换方法                     
注册解析的BeanDefinition：
	通过beanName注册BeanDefinition
		校验beanDefinition
		如果beanName已经存在，是否覆盖
		加入map缓存
	通过别名注册BeanDefinition
通知监听器解析及注册完成


Bean加载
	转换对应beanName
	尝试从缓存中加载单例
	bean的实例化
	原型模式的依赖检查
	检测parentBeanFactory
	将存储XML配置文件的BeanDefinition转换为RootBeanDefinition
	寻找依赖
	针对不同的scope进行bean的创建
	类型转换

FactoryBean：
	public interface FactoryBean<T>{
		T getObject() throws Exception;
		Class<?> getObjectType();
		boolean isSingleton();
	}
FactoryBean可以用于解决单例循环依赖问题只要开始创建就暴露FactoryBean的getObject方法
	            
单例加载过程：
	首先尝试从singletonObjects里面获取实例，如果获取不到再从earlySingletonObjects里面获取，如果还获取不到，尝试从singleFactoryiesi里面获取beanName对应的ObjectFactory，然后调用这个ObjectFactory的getObject来创建bean，
	并放到earlySiingletonObjects里面如，然后从singletonFactories里面remove掉这个objectFactory
	其中：
		singletonObjects：用于保存BeanName和创建bean实例之间的关系
		singletonFactories：用于保存BeanName和创建bean的工厂之间的关系
		earlySingletonObjects：也是保存BeanName和创建bean实例之间的关系，与singletonObjects的不同之处在于，当一个单例bean被放到这里面后，那么当bean还在创建过程中，就可以通过getBean方法获取了
		registeredSingletons：用来保存当前所有已经注册的bean
createbean的参数就是beanname，rootbeandefinition


循环依赖：
	Spring 容器循环依赖包括构造器循环依赖和setter循环依赖
	构造器循环依赖：
		表示通过构造器注入构成的循环依赖，此依赖无法解决，BeanCurrentlyInCreationException 异常表示
		Spring 容器将每一个正在创建的bean标识符放在一个当前创建bean池中，bean标识符再创建过程中一直保存在这个池中，因此如果创建bean过程中发现自己已经在当前创建bean池里，将抛出BeanCurrentlyInCreationException异常表示循环依赖，而对于创建完毕的bean将从当前创建bean池中清除掉
	setter循环依赖：
		     表示通过setter注入方式构成的循环依赖，对于setter注入造成的依赖是通过spring容器提前暴露刚完成构造器注入但未完成其他步骤的bean来完成的，而且只能解决单例作用域的bean循环依赖，通过提前暴露一个简单工厂方法
 
	prototype范围的依赖处理
		对于prototype作用域bean,Spring作用域无法完成依赖注入，因为spring容器不进行缓存prototype作用域的bean

doCreateBean:
	如果是单例则需要首先清除缓存
	实例化bean，将beanDefinition转换为BeanWrapper
	MergedBeanDefinitionPostProcessor的应用
	依赖处理
	属性填充
	循环依赖检查
	注册DisposableBean
	完成创建并返回
创建过程，如果没有使用lookupmethod或者repalce这些，直接使用反射创建，如果使用了要用cglib动态代理织入方法
属性注入：
	byname
		getBean（property）递归
	bytype
		对比class

ApplicationContext 实现了XmlBeanFactory


AOP

使用实例：
	public class TestBean{
		private String testStr="testStr";
		public String getTestStr(){
			return testStr;
		}
		public void setTestStr(String testStr){
			this.testStr=testStr;
		}
		public void test(){
			System.out.println("test");
		}
	}
	创建Advisor：
	@Aspect
	public class AspectJtest{
		@PointCut（“execution （* *.test(...)）”）
		public void test(){
		}
		@Befor("test()")
		public void beforeTest(){
			System.out.println("beforeTest");
		}	
		@After("test()")
		pubilc void afterTest(){
			System.out.println("afterTest");
		}
		@Around("test()")
		public Object arountTest(ProceedingJoinPoint p){
			System.out.println("before1");
			Object o=null;
			try{
				o=p.proceed();
			}catch(Throwable e){
				e.printStackTrace();
			}
			System.out.println("after1");
			return o;
		}
	}

Spring AOP部分使用JDK动态代理或者CGILB来为目标对象创建代理。如果被代理的目标对象实现了至少一个接口，则会使用JDK动态代理，所有
该目标类型实现的接口都将被代理，若该目标对象没有实现任何接口，则创建一个CGLIB代理
JDK:只能带来接口中的方法
CGLIB:底层是继承，无法通知final方法，需要导包


JDBC连接数据库的流程极其原理如下：
	在开发环境中加载指定数据库的驱动程序
	在java中加载驱动程序，class.forName
	创建数据库连接对象
	创建statement对象
	调用statement对象的相关方法来执行相对应的SQL语句，通过execuUpdate（）方法来对数据更新，包括插入和删除等操作，executeQuery执行查询
	关闭数据库连接
Spring 连接数据库程序实现：
	创建对应数据表的PO
	public class User{
		private int id;
		private String name;
		private int age;
		private String sex;
		//省略get，set
	}
	创建表与实体间的映射：
	public class UserRowMapper implements RowMapper{
		@Override
		public Object mapRow(ResultSet set,int index) throws SQLException{
			User person=new User(set.getInt("id"),set.getString("name")set.getInt("age"),set.GetString("sex"));
			return person;	
		}
	}
	创建数据库操作接口
	public interface UserService{
		public void save(User user);
		public List<User> getUsers();
	}
	创建数据接口实现类
	public class UserServiceImpl implements UserService{
		private JdbcTemplate jdbcTemplate;
		public void setDataSource(DataSource dataSource){
			this.jdbcTemplate=new JdbcTemplate(dataSource);
		}
		public void save(User user){
			jdbcTempate.update("insert into user(name,age,sex) values(?,?,?)",new Object[]{user.getName,user.getAge,user.getSex},new int[]{java.sql.Types.VARCHAR,java.sql.Types.INTEGER,java.sql.Types.VARCHAR});
		}
		@SuppressWarnings("unchecked")
		pubilc List<User> getUsers(){
			List<User> list=jdbcTemplate.query("select * from user",new UserMapper());
			return lis;
		}
	}

核心jdbcTemplate
execute执行过程：
	获取数据库连接：需要保证线程中的数据库操作都是使用同一个事务连接
	应用用户设定的输入参数
	调用回调函数
	警告处理
	资源释放	···	·	



整合MyBatis：
	Mybatis独立使用：
		建立PO
		建立Mapper
		建立配置文件	
		建立映射文件
	使用:
		SqlSessionFactory sqlsessionFactory=new SqlSessionFactoryBuilder().build(Resources.getResourceAsReader("mybatis-config.xml"));
		SqlSession sqlSession=sqlSessionFactory.openSession();
		UserMapper userMapper=sqlSession.getMapper(UserMapper.class);
		User user=new User("tom",new Integer(5));
		userMapper.insertUser(user);
		sqlSession.commit();
		sqlSession.close();
Spring 整合MyBatis
	配置xml：
		datasource
		SqlSessionFactory
		mapper、
	mybatis配置文件
	映射文件
	然后就可以getBean得到bean


整合源码：
	SqlSessionFactory创建：
		封装在SqlSessionFactoryBean中封装，读取configLocation，dataSource，typeAliasesPackage
	MapperFactoryBean：
		获取Mapper bean
	MapperScannerConfigure

	mybatis原本SqlSessionFactory，和各种maper
	整合无非就是把这些bean装配好

事务:原理aop
	数据库事务
		默认情况下Spring中的事务只对RuntimeException方法进行回滚，如果替换成普通的Exception不会产生回滚效果

wrapIfNecessary函数功能：
	找出指定bean对应的增强器
		寻找候选增强器
	根据找出的增强器创建代理
BeanFactoryTransactionAttributeSourceAdvisor 事务增强器
TransactionInterceptor 拦截器继承自MethodInterceptor：
	invoke方法
	核心逻辑
	public Object invoke（final MethodInvocation invocation） throws Throwable{
		TransactionInfo txInfo=createTransactionIfNecessary(tm,txAttr,join  )
		Object retVal=null;
		try{
			retVal=invocation.proceed();
		}
		catch(Throwable ex){
			completeTransactionAfterThrowing(txInfo,ex);
			throw ex;
		}
		finally{
			cleanupTransactionInfo(txInof);
		}
		commitTransactionAfterReturning(txInfo);
		return retVal;          
	}



SpringMVC使用：
	配置web.xml	
		contextParam: applicationContext.xml
		dispatcherServlet
		ContextLoadListener
Spring是一个容器，每一个web应用都有一个ServletContext，要将两者结合起来，所以把spring的容器作为一个web的一个变量
ContextLoaderListener的作用就是启动web容器时，自动装配ApplicationContext的配置信息，ServletContextListener中的核心逻辑就是初始化WebApplicationContext实例并存放至ServletContext中


	ServletContext启动之后会调用ServletContextListener的contextInitialized
	initWebApplicationContext（）
	初始化WebApplicationContext并记录在servetContext中

DispatchServlet
	Servlet是一个java编写的程序，此程序是基于http协议的，生命周期由servlet的生命周期控制，可以分为3个阶段：
	初始化阶段
	运行阶段
	销毁阶段

DispatchServlet的init：
	封装成bean，并读取config，作为property注入bean
DispatchServlet中执行initWebApplicationContext，创建或刷新WebApplicationContext，然后刷新：initMultipartResolver，intLocaleResolver，initThemeResolver，initHandlerMappings，initHandlerAdapter，intHadlerExceptionResolvers，initRequestToViewNameTranslator，intViewResovlers，intFlashMAPmANAGER


默认HandlerAdapter：
	HttpRequestHandlerAdapter：HTTP请求处理器适配器，仅仅支持对Http请求处理器的适配，它简单地将HTttp请求对象和相应对象传递给Http请求处理器的实现，并不需要返回值
	SimplerControlllerHandlerAdapter：简单控制器处理器
	AnnotationMethodHandlerAdapter：注解方法处理器
HandlerExceptionResolvers：
	基于HandlerExceptionResolver接口的异常处理，使用这种方式只需要

请求处理过程
	把localeResolver，themeResolver等设置在request属性中
	如果是MultipartContent类型的request，则转换request为MultipartHttpServletRequest类型的request
	根据request查找对应的Handler
		HandlerExecutionChain getHandler（），由SImpleUrlHandlerMaping，根据url查找
	HanderAdapter执行，根据反射
















