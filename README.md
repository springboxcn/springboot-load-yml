# springboot-load-yml
springboot load yml look YamlPropertySourceLoader


https://my.oschina.net/bfleeee/blog/879209

- [spring-boot-yaml-property-binding-collection-types](https://stackoverflow.com/questions/47748801/spring-boot-yaml-property-binding-collection-types)

- [spring-configurationproperties-binding-fails-in-test](https://stackoverflow.com/questions/48265981/spring-configurationproperties-binding-fails-in-test)

https://cloud.spring.io/spring-cloud-static/spring-cloud-commons/2.0.0.M6/multi/multi__spring_cloud_context_application_context_services.html

### 1.8 Refresh Scope

- A Spring `@Bean` that is marked as `@RefreshScope` will get special treatment when there is a configuration change. This addresses the problem of stateful beans that only get their configuration injected when they are initialized. For instance if a `DataSource` has open connections when the database URL is changed via the `Environment`, we probably want the holders of those connections to be able to complete what they are doing. Then the next time someone borrows a connection from the pool he gets one with the new URL.

- Refresh scope beans are lazy proxies that initialize when they are used (i.e. when a method is called), and the scope acts as a cache of initialized values. To force a bean to re-initialize on the next method call you just need to invalidate its cache entry.

- The `RefreshScope` is a bean in the context and it has a public method `refreshAll()` to refresh all beans in the scope by clearing the target cache. This functionality is exposed in the `/refresh` endpoint (over HTTP or JMX). There is also a `refresh(String)` method to refresh an individual bean by name.

> `@RefreshScope` works (technically) on an `@Configuration` class, but it might lead to surprising behaviour: e.g. it does not mean that all the `@Beans` defined in that class are themselves `@RefreshScope`. Specifically, anything that depends on those beans cannot rely on them being updated when a refresh is initiated, unless it is itself in `@RefreshScope` (in which it will be rebuilt on a refresh and its dependencies re-injected, at which point they will be re-initialized from the refreshed `@Configuration`).

For a Spring Boot Actuator application there are some additional management endpoints:

`/refresh` for re-loading the boot strap context and refreshing the `@RefreshScope` beans


### ConfigurationProperties & Configuration

#### ConfigurationProperties

[org.springframework.boot.context.properties.ConfigurationProperties/doc](https://docs.spring.io/spring-boot/docs/2.1.0.BUILD-SNAPSHOT/api/org/springframework/boot/context/properties/ConfigurationProperties.html)

Annotation for externalized configuration. Add this to a class definition or a @Bean method in a @Configuration class if you want to bind and validate some external Properties (e.g. from a .properties file).
Note that contrary to @Value, SpEL expressions are not evaluated since property values are externalized.

#### Configuration

[org.springframework.context.annotation.Configuration/doc](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/context/annotation/Configuration.html)

[org.springframework.context.annotation.Bean/doc](https://docs.spring.io/spring/docs/5.1.0.BUILD-SNAPSHOT/javadoc-api/org/springframework/context/annotation/Bean.html)

## EnvironmentManager/Environment

[spring-cloud.html#_endpoints](http://projects.spring.io/spring-cloud/spring-cloud.html#_endpoints)

## LICENSE

### [CC-BY-SA-3.0](https://creativecommons.org/licenses/by-nc-sa/3.0/cn/)

[![](LICENSE.png)](https://creativecommons.org/licenses/by-nc-sa/3.0/cn/)
