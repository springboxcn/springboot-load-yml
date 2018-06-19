```java
@Target(value={TYPE,METHOD})
@Retention(value=RUNTIME)
@Documented
public @interface ConfigurationProperties
```

外部配置的注释。 如果要绑定和验证某些外部属性（例如，来自 `.properties` 文件），请将其添加到 `@Configuration` 类中的类定义或 `@Bean` 方法。
请注意，与 `@Value` 相反，由于属性值是外部化的，因此不会评估 `SpEL` 表达式。