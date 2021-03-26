# spring-boot web篇

## 一、简介

### 1、使用

- 创建springboot应用，选中我们需要的模块；
- springboot已经默认将这些场景配置好了(autoconfigure包下)，只需要在配置文件中指定少量配置就可以运行起来
- 自己编写业务代码

### 2、自动配置原理？

​	这个场景SpringBoot帮我们配置了什么，？

​	能不能修改？

​	能修改那些配置？

​	能不能拓展？

````
xxxxAutoConfiguration：帮我们给容器中自动配置组件
xxxxProperties：配置类封装配置文件的内容
````

------



## 二、静态资源

### 1、映射规则

> 参考源码	(资源映射类WebMvcAutoConfiguration.addResourceHandlers())	==>	添加资源映射
>

```java
public void addResourceHandlers(ResourceHandlerRegistry registry) {
  if (!this.resourceProperties.isAddMappings()) {
    logger.debug("Default resource handling disabled");
    return;
  }
  Duration cachePeriod = this.resourceProperties.getCache().getPeriod();
  CacheControl cacheControl = this.resourceProperties.getCache().getCachecontrol().toHttpCacheControl();
  if (!registry.hasMappingForPattern("/webjars/**")) {
    customizeResourceHandlerRegistration(registry.addResourceHandler("/webjars/**")
        .addResourceLocations("classpath:/META-INF/resources/webjars/")
        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
  }
  String staticPathPattern = this.mvcProperties.getStaticPathPattern();
  if (!registry.hasMappingForPattern(staticPathPattern)) {
    customizeResourceHandlerRegistration(registry.addResourceHandler(staticPathPattern)
        .addResourceLocations(getResourceLocations(this.resourceProperties.getStaticLocations()))
        .setCachePeriod(getSeconds(cachePeriod)).setCacheControl(cacheControl));
  }
}
```

#### wabjars

​	介绍：以jar包的形式引入静态资源；

​	链接：https://www.webjars.org/

​	作用：将jquery等前端框架以maven依赖方式交给我们

​	引入：

```xml
<dependency>
    <groupId>org.webjars</groupId>
    <artifactId>jquery</artifactId>
    <version>3.5.1</version>
</dependency>
```

​	==使用：由上面代码可知	==>	所有/webjars/**，都去classpath:/META-INF/resources/webjars/下找资源==

> 如localhost:8080\webjars\jquery\3.5.1\jquery.js

，

### 2、静态资源设置规则

> 参考源码 ConfigurationProperties类	==>	可以设置和静态资源有关的参数，缓存时间等

```java
@ConfigurationProperties(prefix = "spring.resources", ignoreUnknownFields = false)
public class ResourceProperties {
  private static final String[] CLASSPATH_RESOURCE_LOCATIONS = { "classpath:/META-INF/resources/",
			"classpath:/resources/", "classpath:/static/", "classpath:/public/" };

	private String[] staticLocations = CLASSPATH_RESOURCE_LOCATIONS;
  
}
```

​	=="/**"作用：访问当前项目的任何资源==

````
"classpath:/META-INF/resources/",
"classpath:/resources/", 
"classpath:/static/", 
"classpath:/public/"
````

==使用：localhost:8080/abc	==>	去静态资源文件夹中找abc文件==



### 3、欢迎页映射

> 参考源码	(资源映射类WebMvcAutoConfiguration.welcomePageHandlerMapping())	==>	配置欢迎页映射

````java
@Bean
public WelcomePageHandlerMapping welcomePageHandlerMapping(ApplicationContext applicationContext,
    FormattingConversionService mvcConversionService, ResourceUrlProvider mvcResourceUrlProvider) {
  WelcomePageHandlerMapping welcomePageHandlerMapping = new WelcomePageHandlerMapping(
      new TemplateAvailabilityProviders(applicationContext), applicationContext, getWelcomePage(),
      this.mvcProperties.getStaticPathPattern());
  welcomePageHandlerMapping.setInterceptors(getInterceptors(mvcConversionService, mvcResourceUrlProvider));
  welcomePageHandlerMapping.setCorsConfigurations(getCorsConfigurations());
  return welcomePageHandlerMapping;
}
````

欢迎页：静态资源文件夹下的所有index.html页面；被 “/**” 映射

==使用：localhost:8080 不指定文件  即可访问静态资源文件夹下的index.html页面==

​			若在/META-INF/resources/下需指定文件名

------

## 三、模板引擎

​	常见的：JSP、Velocity、Freemaker、Thymeleaf

​	![点击查看源网页](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=2305624367,180296730&fm=26&gp=0.jpg)

​	SpringBoot推荐的Thymeleaf:

​		语法更简单，功能更强大

### 1、引入Thymeleaf

```xml
<!--引入thymeleaf-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
<!--切换版本-->
<properties>
    <!--thymeleaf-->
    <thymeleaf.version>3.0.11.RELEASE</thymeleaf.version>
    <thymeleaf-layout-dialect.version>2.4.1</thymeleaf-layout-dialect.version>
</properties>
```



### 2、使用&语法

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;
	public static final String DEFAULT_PREFIX = "classpath:/templates/";
	public static final String DEFAULT_SUFFIX = ".html";
```

只要我们把HTML页面放在classpath:/templates/,thymeleaf就能自动渲染；

导入Thymeleaf命名空间	:	xmlns:th="http://www.thymeleaf.org"

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h2>成功!</h2>
    <!--th:text将div中的文本设置为域中的值-->
    <div th:text="${hello}"></div>
</body>
</html>
```

### 3、语法&表达式

th:text:改变当前元素里面的文本内容

####  th:任意属性 

<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200709082344946.png" alt="image-20200709082344946" style="zoom:90%;" />

#### 表达式

````properties
Simple expressions:(表达式语法)
    Variable Expressions: ${...}:获取变量值;ognl
    	1)获取对象的属性、调用方法
    	2)使用内置的基本对象
    	3)使用内置的一些工具对象
    Selection Variable Expressions: *{...}:变量的选择表达式,和${}在功能上是一样的
    	1)配合 th:Object="${session.user}" 进行使用的1
    Message Expressions: #{...}:获取国际化内容
    Link URL Expressions: @{...}:定义url连接
    Fragment Expressions: ~{...}:片段引用表达式
Literals:(字面量)
    Text literals: 'one text' , 'Another one!' ,…
    Number literals: 0 , 34 , 3.0 , 12.3 ,…
    Boolean literals: true , false
    Null literal: null
    Literal tokens: one , sometext , main ,…
Text operations:(文本操作)
    String concatenation: +
    Literal substitutions: |The name is ${name}|
Arithmetic operations:(数学运算)
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
Boolean operations:(布尔运算)
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
Comparisons and equality:(比较运算)
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
Conditional operators:(条件操作)
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
Special tokens:(特殊操作)
    Page 17 of 106
    No-Operation: _
````

------

## 四、springMVC自动装配原理

### 1、 Spring MVC Auto-configuration

Spring Boot 为 Spring MVC 提供了自动配置

以下的添加的自动配置列表：

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
  - 自动配置了ViewResolver(视图解析器：根据方法的返回值得到视图对象(VIew)),视图对象决定如何渲染（转发？重定向？）
  - ContentNegotiatingViewResolver：组合所有的视图解析器的；
  - 如何定制：我们可以给容器中添加一个试图解析器;自动的将其组合进来;
- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-spring-mvc-static-content))).
  - 支持静态资源文件夹路径和webjars
- Static `index.html` support.
  - 支持静态首页
- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-spring-mvc-favicon)).
  - 自定义图标 favicon.ico
- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
  - 转换器	转换数据类型
  - 格式化器    转换数据格式  格式化日期等(yyyy-MM-dd,yyyy.MM.dd,yyyy/MM/dd)

```java
@Override
public void addFormatters(FormatterRegistry registry) {
  ApplicationConversionService.addBeans(registry, this.beanFactory);
}
```

- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-spring-mvc-message-converters)).

  - HttpMessageConverters：spring用来转换Http请求和响应的,User--->json
  - HttpMessageConverters 是从容器中确定；获取所有的HttpMessageConverter；

  ==自己给容器中添加HttpMessageConverter，只需要将自己的组件注册容器中(@bean@Component)==

- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-spring-message-codes)).

  - 定义错误代码生成规则

- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.4.0-SNAPSHOT/reference/html/spring-boot-features.html#boot-features-spring-mvc-web-binding-initializer)).

  - 我们可以配置一个ConfigurableWebBindingInitializer来替换默认的；（添加到容器）

  ````
  初始化WebDataBinder;
  请求数据====JavaBean
  ````

  **org.springframework.boot.autoconfigure.web：web的所有自动配置场景**

If you want to keep those Spring Boot MVC customizations and make more [MVC customizations](https://docs.spring.io/spring/docs/5.3.0-M1/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`.

### 2、拓展springMVC

#### 配置文件方式

```xml
<mvc:view-controller path="/hello" view-name="success"/>
<mvc:interceptors>
    <mvc:interceptor>
        <mvc:mapping path="/hello"/>
        <bean/>
    </mvc:interceptor>
</mvc:interceptors>
```

#### 配置类方式

​	==编写一个配置类(@Configuration),是WebMvcConfigurer类型；不能标注@EnableWebMvc注解==

```java
@Configuration
//使用WebMvcConfigurer可以拓展springMVC的功能
public class MvcConfig implements WebMvcConfigurer {

    public void addViewControllers(ViewControllerRegistry registry) {
        //浏览器发送boot请求，也来到success页面
        registry.addViewController("/boot").setViewName("success");
    }
}
```

> 既保留了所有的自动配置，也能用我们拓展的配置

​	原理：

​		1）WebMvcAutoConfiguration是springMVC的自动配置类

​		2）在做其他自动配置时会导入:@Import(EnableWebMvcConfiguration.class)

​		3）容器中所有的WebMvcConfiger都会一起起作用

​		4）我们的配置类也会被调用

​	效果：springMVC的自动配置和我们的拓展配置都会起作用

### 3、全面接管springMVC

​	SpringBoot对springMVC的自动配置不需要了，所有都是我们自己配，

​	**我们需要在配置类中添加@EnableWebMvc即可**

````java
@EnableWebMvc
@Configuration
//使用WebMvcConfigurer可以拓展springMVC的功能
public class MvcConfig implements WebMvcConfigurer {

    public void addViewControllers(ViewControllerRegistry registry) {
        //浏览器发送boot请求，也来到success页面
        registry.addViewController("/boot").setViewName("success");
    }
}
````

​	原理：

​		为什么@EnableWebMvc自动配置就失效了

​		1）EnableWebMvc的核心

```java
@Import(DelegatingWebMvcConfiguration.class)
public @interface EnableWebMvc {
}
```

​		2）DelegatingWebMvcConfiguration类让自动配置失效了

```java
@Configuration(proxyBeanMethods = false)
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {}
```

​		3）再查看自动配置类WebMvcAutoConfiguration

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass({ Servlet.class, DispatcherServlet.class, WebMvcConfigurer.class })
//容器中没有这个组件的时候,这个自动配置类才生效
@ConditionalOnMissingBean(WebMvcConfigurationSupport.class)
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE + 10)
@AutoConfigureAfter({ DispatcherServletAutoConfiguration.class, TaskExecutionAutoConfiguration.class,
      ValidationAutoConfiguration.class })
public class WebMvcAutoConfiguration {}
```

​		4）EnableWebMvc注解帮我们把WebMvcConfigurationSupport类导进来了,所以springMVC的自动配置类失效了，全部由我们配置

​		5）导入的WebMvcConfigurationSupport只是springMVC最基本的功能,像映射器，适配器，解析器，拦截器都需要自己再来配置

------

## 五、如何修改springboot的默认配置

模式：

​	1）:springboot在自动配置很多组件的时候，先看容器中有没有用户自己配置的（@Bean/@Component）如果有就用用户配置的，如果没有，才自动配置；如果有些组件有多个，(ViewResolver)将用户配置的和自己默认的组合起来;

​	2）:在springboot中会有非常多的xxxConfiger帮助我们进行拓展配置

















