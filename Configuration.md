```java
@Target(value=TYPE)
@Retention(value=RUNTIME)
@Documented
@Component
public @interface Configuration
```

指示一个类声明一个或多个 `@Bean` 方法，并且可以由 `Spring` 容器处理，以在运行时为这些 `bean` 生成 `bean` 定义和服务请求，例如：

```java
@Configuration
 public class AppConfig {

     @Bean
     public MyBean myBean() {
         // instantiate, configure and return bean ...
     }
 }
```

## 引导 `@Configuration` 类

通过 `AnnotationConfigApplicationContext`

`@Configuration` 类通常使用 `AnnotationConfigApplicationContext` 或其支持 `Web` 的变体 `AnnotationConfigWebApplicationContext` 进行引导。 
前者的一个简单例子如下：

```java
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext();
  ctx.register(AppConfig.class);
  ctx.refresh();
  MyBean myBean = ctx.getBean(MyBean.class);
  // use myBean ...
```

有关更多详细信息，请参阅 `AnnotationConfigApplicationContext` `Javadoc`，有关 `web.xml` 配置说明，请参阅 `AnnotationConfigWebApplicationContext`。

## 通过 `Spring <beans> XML`

作为直接针对 `AnnotationConfigApplicationContext` 注册 `@Configuration` 类的替代方法，可以在 `Spring XML` 文件中将 `@Configuration` 类声明为正常的 `<bean>` 定义：

```xml
<beans>
   <context:annotation-config/>
   <bean class="com.acme.AppConfig"/>
</beans>
```

在上面的例子中，为了启用 `ConfigurationClassPostProcessor` 和其他注释相关的后置处理器，需要 `<context：annotation-config />` 来处理 `@Configuration` 类。


## 通过组件扫描

`@Configuration` 用 `@Component` 进行元注释，因此 `@Configuration` 类是组件扫描的候选对象（通常使用 `Spring XML` 的 `<context：component-scan />` 元素），
因此也可以像任何常规 `@Component` 那样利用 `@Autowired`/`@Inject` 零件。 特别是，如果存在单个构造函数，则自动装配语义将透明地应用：

```java
@Configuration
 public class AppConfig {
     private final SomeBean someBean;

     public AppConfig(SomeBean someBean) {
         this.someBean = someBean;
     }

     // @Bean definition using "SomeBean"

 }
```

`@Configuration` 类不仅可以使用组件扫描进行引导，还可以使用 `@ComponentScan` 注释配置组件扫描：

```java
@Configuration
@ComponentScan("com.acme.app.services")
public class AppConfig {
     // various @Bean definitions ...
}
```

有关详细信息，请参阅 `@ComponentScan javadoc`。

## 使用外部化值
## 使用环境API

外部化的值可以通过像往常一样将 `Spring Environment` 注入 `@Configuration` 类来查找（例如使用 `@Autowired` 注解）：

```java
@Configuration
 public class AppConfig {

     @Autowired Environment env;

     @Bean
     public MyBean myBean() {
         MyBean myBean = new MyBean();
         myBean.setName(env.getProperty("bean.name"));
         return myBean;
     }
 }
```

通过 `Environment` 解决的属性驻留在一个或多个“属性源”对象中，并且 `@Configuration` 类可以使用 `@PropertySource` 注释向 `Environment` 对象提供属性源：

```java
@Configuration
@PropertySource("classpath:/com/acme/app.properties")
public class AppConfig {

 @Inject Environment env;

 @Bean
 public MyBean myBean() {
     return new MyBean(env.getProperty("bean.name"));
 }
}
```

有关更多详细信息，请参阅 `Environment` 和 `@PropertySource Javadoc`。

## 使用 `@Value` 注释

使用 `@Value` 批注可以将外部化的值“连接到” `@Configuration` 类中：

```java
@Configuration
@PropertySource("classpath:/com/acme/app.properties")
public class AppConfig {

 @Value("${bean.name}") String beanName;

 @Bean
 public MyBean myBean() {
     return new MyBean(beanName);
 }
}
```

这种方法在使用 `Spring` 的 `PropertySourcesPlaceholderConfigurer` 时非常有用，通常通过 `XML` 通过 `<context：property-placeholder />`启用。 
有关使用 `BeanFactoryPostProcessor` 类型（如`PropertySourcesPlaceholderConfigurer`）的详细信息，请参阅以下有关使用 `@ImportResource` 
使用 `Spring XML` 构造 `@Configuration` 类的部分，请参阅 `@Value Javadoc`，并参阅 `@Bean Javadoc`。

## 撰写 `@Configuration` 类

## 使用 `@Import` 注释

`@Configuration` 类可以使用 `@Import` 注释组合，与 `<import>` 在 `Spring XML` 中的工作方式不同。 由于 `@Configuration` 对象是作为
容器内的 `Spring bean` 进行管理的，导入的配置可以通过常规方式注入（例如通过构造函数注入）：

```java
@Configuration
public class DatabaseConfig {

 @Bean
 public DataSource dataSource() {
     // instantiate, configure and return DataSource
 }
}

@Configuration
@Import(DatabaseConfig.class)
public class AppConfig {

 private final DatabaseConfig dataConfig;

 public AppConfig(DatabaseConfig dataConfig) {
     this.dataConfig = dataConfig;
 }

 @Bean
 public MyBean myBean() {
     // reference the dataSource() bean method
     return new MyBean(dataConfig.dataSource());
 }
 }
```

现在 `AppConfig` 和导入的 `DatabaseConfig` 都可以通过仅针对 `Spring` 上下文注册 `AppConfig` 来引导：

```java
new AnnotationConfigApplicationContext(AppConfig.class);
```

## 使用 `@Profile` 注释

`@Configuration` 类可以使用 `@Profile` 注释来标记，以表明只有给定的一个或多个配置文件处于活动状态时才应该处理它们：

```java
@Profile("development")
@Configuration
public class EmbeddedDatabaseConfig {

 @Bean
 public DataSource dataSource() {
     // instantiate, configure and return embedded DataSource
 }
}

@Profile("production")
@Configuration
public class ProductionDatabaseConfig {

 @Bean
 public DataSource dataSource() {
     // instantiate, configure and return production DataSource
 }
}
```

或者，您也可以在 `@Bean` 方法级别声明配置文件条件，例如 用于相同配置类中的替代 `bean` 变体：

```java
@Configuration
 public class ProfileDatabaseConfig {

     @Bean("dataSource")
     @Profile("development")
     public DataSource embeddedDatabase() { ... }

     @Bean("dataSource")
     @Profile("production")
     public DataSource productionDatabase() { ... }
 }
```

有关更多详细信息，请参阅 `@Profile` 和 `Environment javadocs`。

## 使用 `Spring XML` 使用 `@ImportResource` 注释

如上所述，`@Configuration` 类可以在 `Spring XML` 文件中声明为常规的 `Spring <bean>` 定义。 也可以使用 `@ImportResource` 批注将 
`Spring XML` 配置文件导入到 `@Configuration` 类中。 从 `XML` 导入的 `Bean` 定义可以通常的方式注入（例如使用 `Inject` 注解）：

```java
@Configuration
 @ImportResource("classpath:/com/acme/database-config.xml")
 public class AppConfig {

     @Inject DataSource dataSource; // from XML

     @Bean
     public MyBean myBean() {
         // inject the XML-defined dataSource bean
         return new MyBean(this.dataSource);
     }
 }
```

## 嵌套的 `@Configuration` 类

`@Configuration` 类可以如下嵌套在另一个中：

```java
@Configuration
 public class AppConfig {

     @Inject DataSource dataSource;

     @Bean
     public MyBean myBean() {
         return new MyBean(dataSource);
     }

     @Configuration
     static class DatabaseConfig {
         @Bean
         DataSource dataSource() {
             return new EmbeddedDatabaseBuilder().build();
         }
     }
 }
```

当引导这样的安排时，只需要对应用程序上下文注册 `AppConfig`。由于是一个嵌套的 `@Configuration` 类，`DatabaseConfig` 将自动注册。
这可避免在 `AppConfig DatabaseConfig` 之间的关系已隐式清除时使用 `@Import` 注释。

还要注意，可以使用嵌套的 `@Configuration` 类对 `@Profile` 注释产生良好影响，以将同一个 `bean` 的两个选项提供给封闭的 `@Configuration` 类。

## 配置延迟初始化

默认情况下，`@Bean` 方法将在容器引导时间被迫切地实例化。为了避免这种情况，可以将 `@Configuration` 与 `@Lazy` 注释结合使用，
以表示在类中声明的所有 `@Bean` 方法在默认情况下都会被懒惰地初始化。请注意 `@Lazy` 也可以用于单独的 `@Bean` 方法。

## 测试对 `@Configuration` 类的支持

`Spring-Test` 模块中提供的 `Spring TestContext` 框架提供了 `@ContextConfiguration` 注解，从 `Spring 3.1` 开始，它可以接受一组 `@Configuration Class`对象：

```java
@RunWith(SpringJUnit4ClassRunner.class)
 @ContextConfiguration(classes={AppConfig.class, DatabaseConfig.class})
 public class MyTests {

     @Autowired MyBean myBean;

     @Autowired DataSource dataSource;

     @Test
     public void test() {
         // assertions against myBean ...
     }
 }
```

有关详细信息，请参阅TestContext框架参考文档。

## 使用 `@Enable` 注释启用内置的 `Spring` 功能

诸如异步方法执行，计划任务执行，注解驱动事务管理，甚至 `Spring MVC` 等 `Spring` 特性可以使用各自的 `@Enable` 注释从 `@Configuration` 类启用和配置。有关详细信息，请参阅 
`@EnableAsync`，`@EnableScheduling`，`@EnableTransactionManagement`，`@EnableAspectJAutoProxy` 和 `@EnableWebMvc`。

## 创作 `@Configuration` 类时的约束

- 配置类必须以类的形式提供（即不作为从工厂方法返回的实例），允许通过生成的子类进行运行时增强。
- 配置类必须是非 `final`。
- 配置类必须是非本地的（即不能在方法中声明）。
- 任何嵌套的配置类必须声明为静态。
- `@Bean` 方法可能不会创建更多的配置类（任何这样的实例将被视为常规 `bean`，其配置注释仍未被检测到）。





