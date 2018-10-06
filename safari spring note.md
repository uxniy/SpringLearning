[TOC]

## Safaribooks

### Dependency Injection:

Spring favors interface/class separation

	Write your code in terms of interfaces
	
	Tell Spring which classes to provide
	
	"wire everything together"



A dependency is simple a attribute

	method argument 
	
	return type



Injection -> calling a setter method or using a constructor argument

Metadata describe the actual classes and services we want

Spring wire everything together and manages the lifecycle 

We rarely instantiate beans

We don't look them up

```xml
<!--import other xml configuration in current xml configuration file-->
<bean>
	<import resource="services.xml"/>
    ...
</bean>
```



```java
@Configuration  

//target=Type mean it can just used on the class (not a method)

@Import(InfrastructureConfig.class)

//import another configuration file into this one

@ImportResource

//Import a non-java configuration file into this one


ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

Game game = context.getBean("game",Game.class);


public DataSource datasource(){
    //only use the DriverManagerDataSource for test purpose
    DataSource dataSource = new DriverManagerDatasource();
    dataSource.setDriverClassName("org.hsqldb.jdbcDriver");
	dataSource.setUrl("jdbc:hsqldb:hsql://localhost:");
	dataSource.setUsername("sa");
	dataSource.setPassword("");
    return dataSource;
}



//Annotation Configuration
@Autowired @Qualifier("beanname")
//go and find the bean and embeded it here
//"autowire by type" first: this works if there is exactly one bean of that type(class) avialiable
//@Qualifier("beanname") specific the bean name, tell spring witch bean need to wire.

@Resource
//"autowire by name"
//from javax.annotation

@ComponentScan(basePackages = "com.demo")
//Scan the com.demo packages and its children packages for finding bean 
//default is the current package

@Component
//When componentScan, tell spring this is a bean (name = lowercase of class name)



```
```xml


<bean id= "redSox" class="com.oreilly.entities.RedSox"/>
<bean id="game" class="com.oreilly.entities.BaseballGame">
	<property name ="anyteam" ref="redSox"/>
</bean>
<bean id="dataSource" class="javax.sql.DataSource"/>  //wrong you can not use an interface
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource"/>  //right 


ApplicationContext context= new ClassPathXmlApplicationContext("applicationContext.xml");
src/main/resources/applicationContext.xml

xmlns:context ="http://www.springframework.org/schema/context"
xsi:schemaLocation="http://www.springframework.org/schema/context
    http://www.springframework.org/schema/beans/spring-context.xsd"

<context:component-scan base-package="org.oreilly"/>


```





Autowired by Constructor: if you really require the injection, for example, jdbcTemplate must have a datasource.
by setter method: optional properties



### Scope:

By default, all spring managed bean are singletons.

@Bean @Scope("prototype")

if you are in a web app there are also request and session scope.

`singleton` `prototype`   

`request` `session` `websocket` `application`  just for web-aware spring application context



### Factory methods and Factory beans:

```xml
<bean id="nf" class="java.text.NumberFormat" factory-method="getCurrencyInstance"/>
//NumberFormat is an abstract class, so it can't be initiate by constructor, need to use factory method

<bean id="factroy" class="java.xml.parser.DocumentBuilderFactory" factroy-method="newInstance"/>
<bean id="documentBuilder" class="java.xml.parsers.DocumentBuilder" factory-bean="factory" factory-method="newDocumentBuilder"/>
//Both DocumentBuilderFactroy and DocumentBuilder are abstract class, if you want a documentBuilder instance you need to create a documentBuilderFactroy instance to call the newDocumentBuilder method.
```

```java
//Annotation config
@Bean 
public NumberFormat nf(){
    return NumberFormat.getCurrencyInstance();
}
```



### Initialization and Destruction :

```java
@Bean(initMethod="startGame",destroyMethod="endGame")
//the initMethod and destroyMethod here couldn't have arguments


//Or in bean, you can add @PostConstruct @PreDestroy before the method
//both of them are java annotations
@PostConstruct
public void startGame(){
    ...
}
@PreDestroy
public void endGame(){
    ...
}

//There is a close method in AnnotationConfigurationApplicationContext
//when the application ended, the close method will be called.
//At that time the @PreDestry method will be called, unless the bean is prototype.
```



If you own the bean(you can edit the class file), you can use approach 2, but if the bean is in some other liberary( you don't have the source), you can use approach 1, but they can only use the none argument methods.



### AOP:

Code tangling

Code scattering

**JoinPoint**: all the posible location that we can apply our aspect( where to do it posibility)

public methods in Spring-managed beans

**PointCut**: The actual joinpoints we have declared 

where you want to apply the functionality

**Advice**:the functionality we want to apply

**Aspect**:combines  PointCuts and Advice

**Weaving**: the process of applying aspect to our system



```java
@EnableAspectJAutoProxy
//Add this to the java config file
//The aspectJ will generate a proxy in front of each of my beans, when invoke the advice, it will go through the proxy do what the advice is specified and then call the destination.(do this process both the way in and the way out)


//create an aspect class
@Aspect
@Componet
public class LogingAspect{
    
}
```



### Advice 

@Before @After @AfterReturning @AfterThrowing @Around

```
@Around("execution(void com.oreilly..*.set*(*))")
public Object foo(ProceedingJointPoint pjp){
    ...
    Object retVal = pjp.proceed();
	return retVal;
}
```



### Test:

Spring Test is for the Integration Test not for Unit Test.

```java
//Annotate the Test Class with following, RunWith is a java annotation, SpringJUnit4ClassRunner is in Spring. 
//SpringJUnit4ClassRunner make a applicationContext for the test class then cache it. 
//So we don't need to create applicationContext in the setUp method.
//the setUp method will run before every test method. So if we use setUp to create a applicationContext, you will new and destroy applicationContext over and over.
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=AppConfig.class)
//ContextConfiguration is to specific the config file for the JUnit Test class.
public class AppTest{
    ...
    @Autowired
    private ApplicationContext cxt;
    //if you want to use applicationContext.getBean("XX") in your test, you just need to Autowire the ApplicationContext, SpringJUnit4ClassRunner created for you.
    public void testXXX() throws Exception{
	    Game game = cxt.getBean("game");
        ...
    }
}

```



### Transactional Test:

If you add an @Trasactional on a test method, it will rollback automatically at the end of the test.

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = AppConfig.class)
@Transactional(transactionManager="txmg")
@Commit
//@Commit make the transaction to be commit at the end of the test.
public class AppTest{
    @BeforeTransaction
    //execute this method before the transaction start
    public void methodA(){
        ...
    }
    @AfterTransaction
    //execute this method after the transaction end(either commited or rollbacked)
    public void methodB(){
        ...
    }

    @Test
    @Rollback
    //override the default class-level rollback setting, this test will be rollback at the  end of the test
    public void testMethod(){
        ...
    }
    
    @Test
    @DirtiesContext
    //clear the application Context from cache, when this test is end.
    //And reloaded the applicationContext for the test executed later.
    //@DirtiesContext can be used both on class-level and method-level
    public void testChangeAppContext(){
        //change the applicaitonContext in this test
        ...
    }
}
```



### Transactions:

ACID properties->

Atomic

	All or nothing

Consistent

Isolated

Durable

	Commited changes are permanent 

### Transactions in Spring:

@Transactional -> manage a transaction manager

Spring does not provide a transaction manager, it just hook  the insisting transaction manager inside your system.(for example , database transaction manager)

1. Apply the @Transactional annotation 

   - xml format
   - programmatically

2. Declare a Platform Transaction Manager bean

   interface PlatformTransactionMananger  

3. @EnableTransactionManagement (use this on Java Config class, tell spring to generate the proxies, so that we can use it)

   ```xml
   <!-- xml config -->
   <beans>
   	<tx:annotation-driven/>
       <bean id="abc" class="com.oreilly.XXX"/>
   </beans>
   ```


### Isolation Level:

```java
@Transactional(isolation=Isolation.READ_COMMITED)
```



**READ_UNCOMMITED:** can see changes made by other transaction, even though they are not committed. Can read fast, didn't lock anything, but it is very risky, because the changes maybe rollbacked.(Dirty read)

**READ_COMMITED:** dirty read is prevented, if you see a change, it is a real change. It is committed. non-repeated read and phantom read is allowed.

[What's non-repeated read and phantom read?](http://www.cnblogs.com/zhoujinyi/p/3437475.html)

**REPEATABLE_READ:** Dirty read and repeatable read are prevented, but still allow phantom read. (Row lock)

**SERIALIZABLE:** Dirty read, repeatable read and phantom read are prevented.(Table lock)



### Propagation Level:

```java
@Transactional(propagation=Propatation.REQUIRED)
```

**REQUIRED(default):** (java provided)

tx->join tx

no->create and run tx

**REQUIRES_NEW:**(java provided)

tx1->suspend tx1;create and run tx2; resume tx1;

no->create and run tx

**SUPPORTS:**(java provided)

tx->join tx

no->nothing

**NOT_SUPPORTED:**(java provided)

tx->suspend tx; run outside tx; resume tx;

no->nothing

**MANDATORY:**(java provided)

tx->join tx

no->throw TransactionRequiredException

**NEVER:** (java provided) 

tx->throw a exception

no->nothing

**NESTED: ** (spring provided) 

almost the same as REQUIRED, but support parts rollback.

http://masatoshitada.hatenadiary.jp/entry/2015/12/05/135825



```java
@Transactional(readOnly="true")

@Transactional(rollbackFor=IOException.class)
@Transactional(noRollbackFor=IOException.class)
@Transactional(rollbackForClassName={"IOException"})
@Transactional(noRollbackForClassName={"IOException"})

```





### JdbcTemplate:

```java
int count = this.jdbcTemplate.queryForObject("select count(*) from users",Integer.class);

int countByCountry = this.jdbcTemplate.queryForObject("select count(*) from users where country_id = ? ",Integer.class,1);

Actor result = this.jdbcTemplate.queryForObject("select firstname,lastname from actor where id=?",
                                new Object[]{1212L},
                                 new RowMapper<Actor>{
                                     public Actor mapRow(ResultSet rs,int rowNum) throws SQLException{
                                         Actor actor = new Actor();
                                         actor.setFirstName(rs.getString("firstname"));
                                         actor.setLastName(rs.getString("lastname"));
                                         return actor;
                                     }
                                 });


List<Actor> results = this.jdbcTemplate.query("select firstname,lastname from actor",
                                new RowMapper<Actor>{
                                     public Actor mapRow(ResultSet rs,int rowNum) throws SQLException{
                                         Actor actor = new Actor();
                                         actor.setFirstName(rs.getString("firstname"));
                                         actor.setLastName(rs.getString("lastname"));
                                         return actor;
                                     }
                                 });


this.jdbcTemplate.update("insert into actor(firstname,lastname) values(?,?)","Mike","Lee");

this.jdbcTemplate.update("update actor set firstname = ? where id=?","Mike",1212L);

this.jdbcTemplate.update("delete from actor where id=?",1212L);


```



JdbcTemplate is thread safe.

```java
//common practice
@Repository
public class JDBCStaffRepository{
    private JdbcTemplate jdbcTemplate;

    @Autowired
    public void setJdbcTemplate(DataSource dataSource){
    	this.jdbcTemplate = new JdbcTemplate(dataSource);    
    }
    ...
}
//don't forget you also need to config a dataSource bean
```



NamedParameterJdbcTemplate

### Defining a DataSource:

**@Componet**

**@Service** is a mark annotation, didn't provide any other implementation rather than Componet.

**@Repository** provide a DataAccessException translation as long as use it avaliable with a PersistenceExceptionTranslationPostProcessor

**@Controller**



```java
...
@PropertiySource("classpath:prod.properties")
public class AppConfig{

    @Autowired
    private Environment env;

    @Bean(name = "dataSource",destroy-method="shutdown")
    @Profile("test")
    public DataSource dataSourceForTest(){
    	return new EmbededDatabaseBuilder()
    				.generateUniqueName(true)
    				.setType(EmbededDatabaseType.H2)
    				.setScriptEncoding("UTF-8")
    				.ignoreFailDrops(true)
    				.addScript("schema.sql")
    				.addScripts("data.sql")
    				.build();
        
    }
    
    @Bean(name="transactionManager")
    @Profile("test")
    public PlatformTransactionManager transactionManagerForTest(){
        return new DataSourceTransactionManager(dataSourceForTest());
    }
    
    @Bean(name = "dataSource")
    @Profile("prod")
    public DataSource dataSourceForProd(){
    	//BasicDataSource from apache commons.dbcp
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setDriverClassName(env.getProperty("db.driver"));
        dataSource.setUrl(env.getProperty("db.url"));
        dataSource.setUsername(env.getProperty("db.username"));
        dataSource.setPassword(env.getProperty("db.password"));
	    return dataSource;
    }
    
    @Bean(name="transactionManager")
    @Profile("prod")
    public PlatformTransactionManager transactionManagerForProd(){
        return new DataSourceTransactionManager(dataSourceForProd());
    }


}
```



### Testing Repositories

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(AppConfig.class)
@Transactional
@ActiveProfiles("test") //specify the profile to use
public class JdbcAccountRepositoryTest{
	@Autowired
    private AccountRepository repository;
    
    @Test
    public void testGetAccounts() throws Exception{
        List<Account> accounts = repository.getAccounts();
		//assertThat is a JUnit method, is(XX) method is a Hamcrest method, so you need to add Hamcrest dependency ,too.
        assertThat(accounts.size(),is(3));
    }
}
```





### JPA:

JPA->Java Persistence API

EntityManagerFactory

EntityManager

persistence.xml

@Entity

@Id

@Column

@Table

### Spring JPA

#### How to use Spring JPA:

**Step1:Define maping metadata**

```java

```



**Step2:Define EntityManangerFactory**

**Step3:Define TransactionManager and DataSource**

```java
//config file
@Bean 
public JpaVendorAdapter jpaVendorAdapter(){
    HibernateJpaVendorAdapter adapter = new HibernateJpaVendorAdapter();
    adapter.setShowSql(true);
    adapter.setGenerateDdl(true);
    adapter.setDatabase(Database.MYSQL);
    return adapter;
}

@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource, JpaVendorAdapter jpaVendorAdapter){
    Properties props = new Properties();
    props.setProperty("hibernate.format_sql",String.valueOf(true));//SQL log with a good format
    LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
    emf.setDataSource(dataSource);
    emf.setPackagesToScan("com.oreilly.entities"); //set package to scan the entities, if use this method you don't need to create a persistence.xml
    emf.setJpaVendorAdapter(jpaVendorAdapter);
    emf.setJpaProperties(props);
    return emf;
}
//there is also a LocalEntityManagerFactoryBean, use this you need to use persistence.xml

@Bean
public PlatformTransactionManager transactionManager(EntityManagerFactory emf){
    return new JpaTransactionManager(emf);
}

@Bean //translate to DataAccessException provided by Spring
public BeanPostProcesser persistenceTranslation(){
    return new PersistenceExceptionTranslationPostProcesser();
}
```

**Step 4: Implements Jpa Repository**

```java
@Repository
public class JpaAccountRepository implements AccountRepository{
    private static long nextId = 0L;
    @PersistenceContext //***IMPORTANT,with this, spring will generate a entityManager bean automaticlly and autowire it here
    private EntityManager entityManager;
    
    @Override
    public List<Account> getAccounts(){
        entityManager.createQuery("select a from Account a",Account.class) //create query with JPQL
            .getResultList();
    }
    
    @Override
    public Account getAccount(Long id){
		return entityManager.find(Account.class,id);
    }
    
    @Override
    public int getAccountsNumber(){
        //JPQL:Java Persistence Query Language
        String jpql = "select count(a.id) from Account a";
        Long result = (Long)entityManager.createQuery(jpql).getSingleResult();
        return result.intValue();
    }
    
    @Override
    public Long createAccount(BigDecimal initialBalance){
       	long id = nextId++;
        entityManager.persist(new Account(id,initialBalance));
        return id;
    }
    
    @Override
    public int deleteAccount(Long id){
        entityManager.remove(getAccount(id));
        return 1;
    }
    
    @Override
    public void updateAccount(Account account){
        entityManager.merge(account);
    }
    
}
```

### Spring JPA with Spring Boot

#### How to use spring boot with spring data Jpa?

**Add spring data jpa dependency** 

Spring data is not in the spring boot, it is in the spring data project, so you need to add the dependency of spring data Jpa in gradle or maven.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>1.5.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>1.5.3.RELEASE</version>
</dependency>

```

Or just

```xml
 <dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jpa</artifactId>
  </dependency>
 </dependencies>
```



Spring Boot configures Hibernate as the default JPA provider

so it’s no longer necessary to define the **entityManagerFactory** bean unless we want to customize it.

Spring Boot can also auto-configure the **dataSource** bean, depending on the database used.

Spring Boot also setup a **JpaTransactionManager**



**How to customize the entityManagerFactory bean?**

**Entity Location**:Override the `@EnableAutoConfiguration` with `@EntityScan`

```
@SpringBootApplication
@EntityScan(basePackages="rewards")
public class DemoApplication{
    ...
}
```

**Properties**: application.properties

```properties
spring.jpa.database=default
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true 
spring.jpa.properties.hibernate.xxx=???
```



**How to  use  Spring Data-Jpa**

```java
//config java class
@Configuration
@ComponentScan
@EnableJpaRepositories // first step: enable JpaRepository
public class AppConfig{
    ...
}

```

```java
// second step: extends JpaRepository
//JpaRepository<entity class name, id> 
interface AccountRepository extends JpaRepository<Account, Long>{
    //Spring data can generate method implementation for us by define the following format method.
	List<Account> findAccountsByBalenceGreaterThanEqual(BigDecimal amount);
}
//use this spring will default to implement some data operation method for you, like findAll , findOne , count , save, deleteAll etc

```



### Spring Boot

**Add spring-boot-starter-parent:** provide us pre-configured dependencies, there is a start POM to do the common configuration for spring boot project.

**Add other dependencies:**  For example, add a spring-boot-starter-web

**Create a Application class:** 

```java
@RestController //to specify this is a spring web application
@EnableAutoConfiguration // spring boot annotation to tell spring to guess and auto configuration, this annotation serves as a marker for the base package to do scaning
public class Application{
    public static void main(String... args){
        SpringApplication.run(Application.class,args);
        //SpringApplication is in spring boot, use this to bootstrap our application
    }
    
}
```

If you want to do JUnit test, you can add a spring-boot-starter-test dependency.



### Spring Security

**Principle:**Use, device or system that perform an action

**Authentication:**Establishing that a principle's credentials are valid

**Authorization**:Decide if  a principle are allowed to perform an action

**Authority:**Permission or credential enabling access

**Secured Item**:Resource that being secured.



#### How to SetUp and Configure Spring Security?

**Step1 : setup filter chain**

Require a `DelegatingFilterProxy` must be called **springSecurityFilterChain**

`DelegatingFilterProxy` is a special servlet filter, that by itself ,doesn't do much, instead it delegates to an implementation of a javax.servlet.Filter that registed as a bean in spring context.

**3 ways to setup filter chain**

1. configure it in web.xml

```xml
<filter>
	<filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.fiter.DelegatingFilterProxy</filter-class>
</filter>
    
```

2. you want to configure `DelegationFilterProxy` in Java, like following.

```java
public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer{}
```

3. spring boot does it automatically.



**Step 2: configure security rules**

Add `@EnableWebSecurity` on configuration class and make configuration class extends WebSercurityConfigurerAdapter.

If your application is Spring MVC application , you need to use `@EnableWebMVCSecurity` instead `@EnableWebSecurity`

```java
@Configuration
@EnableWebSecurity
//@EnableWebSecurity:switch off the security configuration by spring boot and customize a security configuration by ourselves
public class SecurityConfig extends WebSecurityConfigurerAdapter throws Exception{
    //WebSecurityConfigureAdapter is a convinient base class for creating a web security configure
    @Override //to configure how the requests are secured by intercepter
    protected void configure(HttpSecurity http){
        http.authorieRequests().anyRequests()
            .authenticated().and()
            .formLogin().and()
            .httpBasic();
    }
}
```



```java
//antMatchers and mvcMatchers
https.authrizeRequests()
    .antMatcher("/admin").hasRole("USER");
//match only /admin
https.authrizeRequests()
    .mvcMatchers("/admin").hasRole("USER");
//match /admin /admin/ /admin.html /admin.xxx
```

```java
//FirstMatch is used , so put specific matches first
http.authorizeRequests()
    .antMatchers("/signup","/about").permitAll()
    .antMatchers("/accounts/edit*").hasRole("ADMIN")
    .antMatchers("/accouns/**").hasRole("ADMIN","USER");
```



```java
//customiz login page and logout success url
http.authorizeRequests()
    .antMatchers("/admin/**").hasRole("ADMIN")
    .and()
    .formLogin()
    .loginPage("/login")
    .permitAll()
    .and()
    .logout()
    .logoutSuccessUrl("/home")
    .permitAll();
```

```java
//use to configure security filter chain, for example, to config no security check
protected void configure(WebSecurity web) throws Exception{
    web.ignoring().mvcMatchers("/css/**","/images/**","/js/**");
    
}
```

**Step 3: setup Web Authentication**

**Approach 1**:Configure the Web Authentication with build-in `UserDetailManagerConfigurer`

**In Memory**:

```java
	@Autowired //to define a userstore, to configure user-detail services
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception{
        auth.inMemoryAuthentication()
            .withUser("mike").password("aaa").roles("USER").and()
            .withUser("kate").password("bbb").roles("USER","ADMIN");
        //roles("USER") is the same as authorizes("ROLE_USER")
    }
```



**Jdbc**:

```java
@Autowired
private DataSource datasource;

@Autowired  //make a userstore in database
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception{
    auth.jdbcAuthentication()
        //use password encoder to secure the password
        .passwordEncoder(new StandardPasswordEncoder("53cr3t"));
        .dataSource(datasource)
        // can use the following method to customize queries
        .usersByUsernameQuery("select username, password, true from cus_users where username=?")
        .authoritiesByUsernameQuery("select username, 'ROLE_USER' from cus_authorities where username=?")
        //there is also a groupAuthoritiesByUsername() method to customize the group authorities query.
        
}
```

**Approach 2**: customize your own `UserDetailsService`

#### Method Security

Spring use AOP for method-level security

**Recommendation**: 

	Secure your services.(Services Layer).

	Do not access other layers directlly.







```java
import org.springframework.context.ApplicationListener
//<> type is for the event type we want to listen for
@Component 
public class CustomerSecurityEventListener implements ApplicationListener<AbstractAuthenticationFailureEvent>{
	@Override
	public void onApplicationEvent(AbstractAuthenticationFailureEvent event){
        System.out.println(event.getException().getMessage());
	}
    
} 
```







## Test Simulator

### Dependecy Injection

#### Xml Configuration for DI

```xml
<!---->
<bean id="exampleBean" class="com.demo.ExampleBean">
    <property name="beanOne" ref="anotherBean"/>
    <property name="beanTwo" ref="yetAnotherBean"/>
    <property name="integerProperty" value="1"/>
</bean>
<bean name="anotherBean" class="com.demo.AnotherBean"/>
<bean name="yetAnotherBean" class="com.demo.YetAnotherBean"/>
```

```xml
<!--xml config for constructor based DI-->
<bean id="exampleBean" class="com.demo.ExampleBean">
    <constructor-arg>
    	<ref bean="anotherBean">
    </constructor-arg>
    <constructor-arg ref="yetAnotherBean"/>
    <constructor-arg type="int" value="1"/>
</bean>
<bean name="anotherBean" class="com.demo.AnotherBean"/>
<bean name="yetAnotherBean" class="com.demo.YetAnotherBean"/>
```



#### Autowired

**`@Autowired(required=true)` and `@Required`  ? **

https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#beans-required-annotation

Only one annotated constructor per-class can be marked as required, but multiple non-required constructors can be annotated. In that case, each is considered among the candidates and Spring uses the greediest constructor whose dependencies can be satisfied — that is, the constructor that has the largest number of arguments.

The required attribute of `@Autowired` is recommended over the `@Required` annotation. The required attribute indicates that the property is not required for autowiring purposes. The property is ignored if it cannot be autowired. `@Required`, on the other hand, is stronger in that it enforces the property that was set by any means supported by the container. If no value is injected, a corresponding exception is raised.



#### Scope

By default, when a singleton bean depends on a prototype bean, there will be an object instance of the prototype bean, and it is that same object instance will be supplied to all users of the concerned singleton bean.

### Spring MVC

**What is the return type of @RequestMapping annotated method?**

https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc-ann-return-types

### Spring Boot

**How to create a deployable war file?**

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#howto-create-a-deployable-war-file

**What is `@SpringBootApplication`? **

`@EnableAutoConfiguration` 

`@ComponentScan`

`@Configuration`

https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#using-boot-using-springbootapplication-annotation

**What kind of embeded servlet container does spring boot have?**

tomcat, jetty, undertow

https://docs.spring.io/spring-boot/docs/current/reference/html/howto-embedded-web-servers.html

**Two means of programatic transaction management?**

https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction-programmatic

Use `TransactionTemplate` (Use TransactionCallback)

```java
public class SimpleService implements Service{
    private final TransactionTemplate transactionTemplate;
    //you don't need to put @Autowired here, it is default injected when there is only one  constructor
    public SimpleService(TransactionTemplate transactionTemplate){
        this.transactionTemplate=transactionTemplate;
    }
    public Object someServiceMethod(){
        //there is also TransactionCallbackWithoutResult for void return type method
        return transactionTemplate.execute(new TransactionCallback(){
            public Object doInTransaction(TransactionStatus status){
                updateOperation1();
                return resultOfOperation2();
            }
        });
    }
}
```

Use `PlatformTransactionMananger` implementation directly



### Spring MVC

RequestMethod includes GET POST PUT DELETE PATCH HEAD OPTIONS TRACE

`@GetMapping `  `@PostMapping` `@PutMapping` `@DeleteMapping` `@PatchMapping` 

#### Rest 

| Method Name | Safe | idempotent |
| ----------- | ---- | ---------- |
| GET         | yes  | yes        |
| HEAD        | yes  | yes        |
| OPTIONS     | yes  | yes        |
| TRACE       | yes  | yes        |
| DELETE      | no   | yes        |
| PUT         | no   | yes        |
| PATCH       | no   | no         |
| POST        | no   | no         |





### Spring data

#### JdbcTemplate

When you use the `JdbcTemplate` for your code, you need only to implement callback interfaces, giving them a clearly defined contract. Given a `Connection` provided by the `JdbcTemplate` class, the `PreparedStatementCreator` callback interface creates a prepared statement, providing SQL and any necessary parameters. The same is true for the`CallableStatementCreator` interface, which creates callable statements. The `RowCallbackHandler` interface extracts values from each row of a `ResultSet`.

`Statement`: use to execute the sql without parameter

`PreparedStatement`: use to execute the sql with parameter(except "in" )

`CallableStatment`:use to execute procedure 

```java
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
//JdbcTemplate is thread safe.
```



```java
List<Actor> list = this.jdbcTemplate.query("select first_name,last_name from t_actor", 
                        new RowMapper<Actor>(){
                            public Actor mapRow(ResultSet rs,int rowNum){
                                Actor actor = new Actor();
                                actor.setFirstName(rs.getString("first_name"));
                                actor.setLastName(rs.getString("last_name"));
                                return actor;
                            }
    
});
```



**ResultSet extracts callback interface**

`RowMapper`  best choice for map each row of the ResultSet to a domain object

`RowCallbackHandler ` best choice when no value should return for each row

`ResultSetExtractor` best choice when multiple rows of ResultSet map to a single Object



### Testing

`@SpringJUnitConfig(TestConfig.class)` combines

`@ExtendsWith(SpringExtension.class)`  from JUnit5

and `@ContextConfiguration(TestConfig.class)`  from Spring







###  problems to solve

MessageSource

Security

12   15  

Qualifier

PropertiySource 23



