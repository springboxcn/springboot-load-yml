## Bean

```java
@Target(value={METHOD,ANNOTATION_TYPE})
@Retention(value=RUNTIME)
@Documented
public @interface Bean
```

指示一个方法产生一个由 `Spring` 容器管理的 `bean`。

## 概观

这个注解的属性的名称和语义有意地类似于 `Spring XML` 模式中 `<bean />` 元素的名称和语义。 例如：

```java
@Bean
public MyBean myBean() {
  // instantiate and configure MyBean obj
  return obj;
}
```

## `Bean` 别名

虽然 `name()` 属性可用，但确定 `bean` 名称的默认策略是使用 `@Bean` 方法的名称。 这很方便直观，但是如果需要明确的命名，可以使用 `name` 属性（或其别名值）。 
还要注意，名称接受一个字符串数组，允许为一个 `bean` 提供多个名称（即一个主要的 `bean` 名称加上一个或多个别名）。

```java
@Bean({"b1", "b2"}) // bean available as 'b1' and 'b2', but not 'myBean'
public MyBean myBean() {
  // instantiate and configure MyBean obj
  return obj;
}
```

## Profile, Scope, Lazy, DependsOn, Primary, Order

请注意，`@Bean` 注释不提供配置文件，范围，懒惰，依赖或主要属性。 相反，它应该与 `@Scope`，`@Lazy`，`@DependsOn` 和 `@Primary` 注释一起使用来声明这些语义。 例如：

```java
@Bean
@Profile("production")
@Scope("prototype")
public MyBean myBean() {
  // instantiate and configure MyBean obj
  return obj;
}
```

上述注释的语义与它们在组件类级别的使用相匹配：`@Profile` 允许选择性包含某些 `bean`。 `@Scope` 将 `bean` 的作用域从 `singleton` 更改为指定的作用域。
默认情况下，`@Lazy` 只有实际效果。 `@DependsOn` 除了通过直接引用表示的任何依赖关系外，还会在创建此 `bean` 之前强制创建特定的其他 `bean`，
这通常对单例启动有帮助。 `@Primary` 是一种解决注入点级别歧义的机制，如果需要注入单个目标组件，但多个 `bean` 按类型匹配。

另外，`@Bean` 方法还可以声明限定符注释和 `@Order` 值，在注入点解析过程中将其考虑在内，就像对应组件类上的相应注释一样，但每个 `bean` 定义可能非常单独（在多个定义相同的情况下 `bean` 类）。
限定符在初始类型匹配之后缩小候选集;顺序值决定集合注入点（有几个目标bean按类型和限定符匹配）的已解析元素的顺序。

注：`@Order` 值可能会影响注入点的优先级，但请注意，它们不影响单身启动顺序，这是由上述依赖关系和 `@DependsOn` 声明确定的正交关系。
此外，由于无法在方法上声明优先级，因此无法在此级别使用;它的语义可以通过 `@Order` 值和 `@Primary` 在每个类型的单个 `bean` 上建模。

## `@Configuration` 类中的 `@Bean` 方法

通常，`@Bean` 方法在 `@Configuration` 类中声明。 在这种情况下，`bean` 方法可以通过直接调用它们来引用同一类中的其他 `@Bean` 方法。 
这确保了 `bean` 之间的引用是强类型和可导航的。 这种所谓的“ `bean` 间引用”保证像 `getBean()` 查找那样尊重范围检查和 `AOP` 语义。 
这些是最初的'`Spring JavaConfig`'项目中已知的语义，它们需要在运行时对每个这样的配置类进行 `CGLIB` 子类化。 因此，在这种模式下，
`@Configuration` 类及其工厂方法不能被标记为 `final` 或 `private`。 例如：

```java
@Configuration
 public class AppConfig {

     @Bean
     public FooService fooService() {
         return new FooService(fooRepository());
     }

     @Bean
     public FooRepository fooRepository() {
         return new JdbcFooRepository(dataSource());
     }

     // ...
 }
```

## `@Bean` 精简版模式

`@Bean` 方法也可以在没有用 `@Configuration` 注释的类中声明。例如，`bean` 方法可以在 `@Component` 类中声明，甚至可以在普通的旧类中声明。在这种情况下，
`@Bean` 方法将以所谓的“精简”模式进行处理。

在精简模式下的 `Bean` 方法将被容器视为纯工厂方法（类似于 `XML` 中的工厂方法声明），同时正确应用范围和生命周期回调。包含的类在这种情况下保持不变，
对于包含的类或工厂方法没有特殊的限制。

与 `@Configuration` 类中的 `bean` 方法的语义相比，精简模式不支持“ `bean` 间引用”。相反，当一个 `@Bean` 方法在精简方式下调用另一个 `@Bean` 方法时，
调用是标准的 `Java` 方法调用; `Spring` 不会通过 `CGLIB` 代理拦截调用。这与 `inter-@ Transactional` 方法调用类似，在代理模式下，
`Spring`不会拦截调用--`Spring` 只能在 `AspectJ` 模式下执行。

例如：

```java
@Component
 public class Calculator {
     public int sum(int a, int b) {
         return a+b;
     }

     @Bean
     public MyBean myBean() {
         return new MyBean();
     }
 }
```

## 引导

有关更多详细信息，请参阅 `@Configuration javadoc`，其中包括如何使用 `AnnotationConfigApplicationContext` 和朋友引导容器。

`BeanFactoryPostProcessor` 返回 `@Bean` 方法
对于返回 `Spring` `BeanFactoryPostProcessor`（`BFPP`）类型的 `@Bean` 方法，必须特别考虑。 因为 `BFPP` 对象必须在容器生命周期的早期实例化，
所以它们可能会干扰 `@Configuration` 类中的 `@Autowired`，`@Value` 和 `@PostConstruct` 之类的注释处理。 为避免这些生命周期问题，
请将 `BFPP` 返回的 `@Bean` 方法标记为静态。 例如：

```java
@Bean
public static PropertySourcesPlaceholderConfigurer pspc() {
 // instantiate, configure and return pspc ...
}
```

通过将此方法标记为静态，可以在不引起实例化其声明的 `@Configuration` 类的情况下调用此方法，从而避免上述生命周期冲突。 但是请注意，如上所述，
静态 `@Bean` 方法不会被增强用于范围检查和 `AOP` 语义。 这可以在 `BFPP` 的情况下解决，因为它们通常不被其他 `@Bean` 方法引用。 提醒一下，
`WARN` 级别的日志消息将针对任何具有可分配给 `BeanFactoryPostProcessor` 的返回类型的非静态 `@Bean` 方法发布。