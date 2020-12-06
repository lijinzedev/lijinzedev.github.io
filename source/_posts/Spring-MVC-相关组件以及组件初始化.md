---
title: Spring MVC 相关组件以及组件初始化
top: false
cover: false
toc: true
mathjax: true
categories:
  - 源代码 - SpringMvc源码
tags:
  - 源码 - SpringMvc
date: 2020-11-29 14:07:20
password:
summary:
---

[toc]

# 1 相关组件及其初始化

## 1 相关组件列表

* WebAsyncManager（异步请求管理器）

  > 用于异步请求和响应提供支持

* LocaleResolver（本地化解析器）

  > 用于解析请求对应的Locale。可以是其子接口LocaleContextResolver本地化上下文解析器，额外添加获取时区的功能

* ThemeResovler（主题解析器）

  > 用于封装不同主题对应的资源包，通过主题源的getTheme方法，可以根据主题名获取对应的主题资源包

* ThemeSource（主题源）

  > 用于封装不同主题对应的资源包。通过主题源的getTheme方法，可以根据主题名获取对应的主题资源包

* MessageSource（消息源）

  > 用于对国际化提供支持，消息源封装了不同Code在不同Locale中对应的消息字符串

* FlashMapMannger（闪存管理器）

  > 用于提供重定向跨请求传递Model参数，其中存储内容在下次请求发生后即清空。同时该管理器还用于获取上次请求输出的FlashMap作为本地请求输入的FlashMap，把当前请求的输出FlashMap保存起来作为下一次请求的输入FlashMap

* MultipartResolver（多块请求解析器）

  > 用于判断请求是否为多块请求，对于多块请求进行一些预先解析

* List\<HandlerMapping\> （处理器映射列表）

  > 用于根据请求查找对应的处理器执行链

* List\<HandlerAdapter\> (处理器适配器列表)

  > 用于对处理器执行链进行适配执行

* ViewNameTranslator（视图名翻译器）

  > 类型为RequestToViewNameTranslator，用于通过请求获取请求对应的默认视图名

* List\<HandlerExceptionResolver\> (处理器异常解析器列表)

  > 用于解析整个处理过程发生的异常，把异常解析为ModelAndView的模型视图结果，并执行统一的渲染逻辑
  
* List<ViewResolver\> （视图解析器列表）

  > 用于根据视图名解析获取对应的视图

**总结：**

> 除了WebAsyncManager、ThemeSource、MessageSource不是在Dispatchcher中定义的之外，其他九大组件均在DispatchServlet中定义的，这就是常说的DispatchServlet的九大组件。其属性定义的源码如下

```java
/** MultipartResolver used by this servlet. */
@Nullable
private MultipartResolver multipartResolver;

/** LocaleResolver used by this servlet. */
@Nullable
private LocaleResolver localeResolver;

/** ThemeResolver used by this servlet. */
@Nullable
private ThemeResolver themeResolver;

/** List of HandlerMappings used by this servlet. */
@Nullable
private List<HandlerMapping> handlerMappings;

/** List of HandlerAdapters used by this servlet. */
@Nullable
private List<HandlerAdapter> handlerAdapters;

/** List of HandlerExceptionResolvers used by this servlet. */
@Nullable
private List<HandlerExceptionResolver> handlerExceptionResolvers;

/** RequestToViewNameTranslator used by this servlet. */
@Nullable
private RequestToViewNameTranslator viewNameTranslator;

/** FlashMapManager used by this servlet. */
@Nullable
private FlashMapManager flashMapManager;

/** List of ViewResolvers used by this servlet. */
@Nullable
private List<ViewResolver> viewResolvers;

```

## 2 组件初始化

```java
/**
 * 初始化本类使用的多块请求解析器
 * 如果在ApplicationContext 中没有一个MultipartResovler类型且名称为multipartResolver的Bean被定义，则默认为空，不对多块请求进行特殊处理
 */
private void initMultipartResolver(ApplicationContext context) {
   try {
       // 从application中获取name为multipartResolver，类型为MultipartResolver的Bean
      this.multipartResolver = context.getBean(MULTIPART_RESOLVER_BEAN_NAME, MultipartResolver.class);
      if (logger.isTraceEnabled()) {
         logger.trace("Detected " + this.multipartResolver);
      }
      else if (logger.isDebugEnabled()) {
         logger.debug("Detected " + this.multipartResolver.getClass().getSimpleName());
      }
   }
   catch (NoSuchBeanDefinitionException ex) {
      // Default is no multipart resolver.
       //没有这种bean的时候，设置null
      this.multipartResolver = null;
      if (logger.isTraceEnabled()) {
         logger.trace("No MultipartResolver '" + MULTIPART_RESOLVER_BEAN_NAME + "' declared");
      }
   }
}
```

> ​			组件初始化是延迟加载，也就是第一个请求耗时比较长的原因，可以通过指定配置项`spring.mvc.servlet.load-on-startup`为任意非负数强制指定启动时执行初始化，默认为-1.

​			在HttpServeletBean的init方法中，先为当前DispatcherServlet实例绑定ServletContext，中初始化的一些属性（在Spring Boot中为空，此段逻辑不执行），接着执行initServletBean方法，该方法由DispatchServlet的父类FrameworkServlet实现。

​			在FrameworkServlet的initServletBean方法，执行了本类initwebApplicationContext方法，此方法就是核心的初始化方法入口，在该方法中调用onRefresh方法，即上面DisorderServlet初始化入口方法，

```java
	/**
	 * 初始化此Servlet的Web应用上下文
	 * @return the WebApplicationContext instance
	 * @see #FrameworkServlet(WebApplicationContext)
	 * @see #setContextClass
	 * @see #setContextConfigLocation
	 */
	protected WebApplicationContext initWebApplicationContext() {
        //  先尝试Servlet上下文获取已初始化的Web应用上下文作为根上文
        //  此处已经可以获取rootContext，设置该值的代码在第七章中提到的selfInitialize方法的prepareWebApplicationContext方法中
		WebApplicationContext rootContext =
				WebApplicationContextUtils.getWebApplicationContext(getServletContext());
        // 定义最终要返回的Web应用上下文
		WebApplicationContext wac = null;
		// 如果当前Web应用上下文不为空 
		if (this.webApplicationContext != null) {
			// 在默认情况下，在DispatcherServlet作为Bean初始化的时候，就通过setApplicationContext设置其值
			wac = this.webApplicationContext;
            // 如果当前Web应用上下文是可配置的Web应用上下文
			if (wac instanceof ConfigurableWebApplicationContext) {
				ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
       		// 且Web应用上下文还刷新，则执行此逻辑。默认情况下，该上下文未刷新，不执行此逻辑
				if (!cwac.isActive()) {
                    // 当其父上下文为空时
					if (cwac.getParent() == null) {
						// 则设置其父上下文未上面获取的根本上下文
						cwac.setParent(rootContext);
					}
					configureAndRefreshWebApplicationContext(cwac);
				}
			}
		}
		if (wac == null) {
			// 如果在上一逻辑未获取到Web上下文，此时执行查找逻辑获取
            // 该方法内部通过查找Servlet上下文中本类指定的contextAttribute属性名对应的属性值找到Web上下文
			wac = findWebApplicationContext();
		}
		if (wac == null) {
			 // 在上一逻辑未获取到Web上下文，则创建一个
             // 在Springboot中第一逻辑获取到了Web应用上下文，故不执行此逻辑
			wac = createWebApplicationContext(rootContext);
		}
		// 如果此时已接受过用上下文刷新事件，则不执行下面的if块逻辑
        // 在上下文刷新时间中，也会执行onRefresh的方法
        // 如果已经执行了onRefresh方法，则不再执行
		if (!this.refreshEventReceived) {
			// 执行onRefresh方法，其中执行整个Servlet中组件的初始化操作
			synchronized (this.onRefreshMonitor) {
				onRefresh(wac);
			}
		}
		// 如果配置发布Web应用上下文到Servlet上下文
		if (this.publishContext) {
			// 同时把Web应用上下文添加到Servlet上下文属性中，以供整个Web应用使用
			String attrName = getServletContextAttributeName();
			getServletContext().setAttribute(attrName, wac);
		}

		return wac;
	}

```

> 以上便是初始化此Servlet对应的Web应用上下文的过程，在这个过程中执行了DispatcherServlet中使用到的组件初始化，因为一些初始化逻辑需要使用Web应用上下文获取组件，故需先执行此处逻辑  



# 2 组件初始化过程      

### 1 MultipartResolver

> ​		对于MultipartResolver，已知是通过初始化Web应用上下文获取、
>
> ​		MultipartResolver类型为StandardServletMultipartResolver，查找此类的所有使用，发现是在MultipartAutoConfiguration类中创建了此类的实例

```java
@Configuration
// 只要是基于Servlet的Web应用，就存在这三个类
@ConditionalOnClass({Servlet.class, StandardServletMultipartResolver.class, MultipartConfigElement.class})
// 在属性spring.servlet.multipart.enabled不为flase或不存在时候此配置生效
@ConditionalOnProperty(
    prefix = "spring.servlet.multipart",
    name = {"enabled"},
    matchIfMissing = true
)
@当Web类型为Servlet的时候生效
@ConditionalOnWebApplication(
    type = Type.SERVLET
)
@EnableConfigurationProperties({MultipartProperties.class})
public class MultipartAutoConfiguration {
    private final MultipartProperties multipartProperties;

    public MultipartAutoConfiguration(MultipartProperties multipartProperties) {
        this.multipartProperties = multipartProperties;
    }

    @Bean
  @ConditionalOnMissingBean({MultipartConfigElement.class, CommonsMultipartResolver.class})
    public MultipartConfigElement multipartConfigElement() {
        return this.multipartProperties.createMultipartConfig();
    }

    @Bean(
        name = {"multipartResolver"}
    )
    @ConditionalOnMissingBean({MultipartResolver.class})
    public StandardServletMultipartResolver multipartResolver() {
        // 创建实例
        StandardServletMultipartResolver multipartResolver = new StandardServletMultipartResolver();
        // 根据属性源spring.servlet.multipart.resolvelazily的值，设置multipartProperties中的Resolverlazily属性值，表示对多块请求是否执行懒解析，如果为懒解析则在用多块钱请求数据的时候才对请求进行解析
        multipartResolver.setResolveLazily(this.multipartProperties.isResolveLazily());
        return multipartResolver;
    }
}
```

**总结：**

> ​		MultipartAutoConfiguration本身是自动配置类，在该类中用@Bean的方式标记方法，返回值都会注册到当前应用上下文中，这就应用上下文该组件的来源了

### 2 LocaleResovler

```java
    private void initLocaleResolver(ApplicationContext context) {
        try {
            // 从上下文中获取
            this.localeResolver = (LocaleResolver)context.getBean("localeResolver", LocaleResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.localeResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.localeResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            // 上下文中没有使用默认策略
            this.localeResolver = (LocaleResolver)this.getDefaultStrategy(context, LocaleResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No LocaleResolver 'localeResolver': using default [" + this.localeResolver.getClass().getSimpleName() + "]");
            }
        }

    }
```

​		获取默认组件的策略是通过加载jar包中的DispatcherServlet.properties属性文件，获取其中的key:value。key为组件的类型接口全限定名，value为默认的组件类型名，初始化策略为获取组件接口属性名对应的组件类型属性值，并通过反射，作为默认组件使用。其中DispatchServlet.properties文件内容如下

```properties
# Default implementation classes for DispatcherServlet's strategy interfaces.
# Used as fallback when no matching beans are found in the DispatcherServlet context.
# Not meant to be customized by application developers.

org.springframework.web.servlet.LocaleResolver=org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver

org.springframework.web.servlet.ThemeResolver=org.springframework.web.servlet.theme.FixedThemeResolver

org.springframework.web.servlet.HandlerMapping=org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping

org.springframework.web.servlet.HandlerAdapter=org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter,\
	org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter,\
	org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter

org.springframework.web.servlet.HandlerExceptionResolver=org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver,\
	org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver,\
	org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver

org.springframework.web.servlet.RequestToViewNameTranslator=org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator

org.springframework.web.servlet.ViewResolver=org.springframework.web.servlet.view.InternalResourceViewResolver

org.springframework.web.servlet.FlashMapManager=org.springframework.web.servlet.support.SessionFlashMapManager
```

**注意：**

> ​		上面有些key配置了用逗号隔开的多个class类名，因为有些最贱是list类型，list类型通过方法getDefaultStrategies获取

​		除此之外，在配置`spring.mvc.locale-resolver=FIXED`与`spring.mvc.locale-zh_CN`后，初始化LocaleResolver就不会通过默认策略进行初始化，而是直接通过应用上下文获取LocaleResolver类型的Bean，为FixedLocaleResolver。通过查找FixedLocaleResolver的构造器引用位置，找到了该Bean的定义位子在WebMvcAutoConfiguration自动配置类的WebMvcAutoConfigurationAdapter

```java
        @Bean
        @ConditionalOnMissingBean
// 只有配置了spring.mvc.locale属性的时候，该Bean才会被定义
        @ConditionalOnProperty(
            prefix = "spring.mvc",
            name = {"locale"}
        )
        public LocaleResolver localeResolver() {
            // 通过MvcProperties封装spirng.mvc下的所有的配置
           
            if (this.mvcProperties.getLocaleResolver() == org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties.LocaleResolver.FIXED) {
                // 使用spring.mvc.locale配置的locale作为构成参数传入，即固定取此Locale
                return new FixedLocaleResolver(this.mvcProperties.getLocale());
            } else {
                // 否则使用请求头Locale解析器
                AcceptHeaderLocaleResolver localeResolver = new AcceptHeaderLocaleResolver();
                // 并设置配置值为；localeResolver未解析到Locale时的默认值
                localeResolver.setDefaultLocale(this.mvcProperties.getLocale());
                return localeResolver;
            }
        }
```

### 3 ThemeResolver

> ​		主题解析器ThemeResolver的初始化逻辑通过方法initThemeResolver执行，该组件在默认情况下同LocaleResolver组件，在应用上下文中无此类型的Bean，最终通过执行降级的初始化方法获取该类型的默认实例

```java

    private void initThemeResolver(ApplicationContext context) {
        try {
            this.themeResolver = (ThemeResolver)context.getBean("themeResolver", ThemeResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("Detected " + this.themeResolver);
            } else if (this.logger.isDebugEnabled()) {
                this.logger.debug("Detected " + this.themeResolver.getClass().getSimpleName());
            }
        } catch (NoSuchBeanDefinitionException var3) {
            this.themeResolver = (ThemeResolver)this.getDefaultStrategy(context, ThemeResolver.class);
            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No ThemeResolver 'themeResolver': using default [" + this.themeResolver.getClass().getSimpleName() + "]");
            }
        }

    }
```

### 4 HandlerMapping

```java
	/**
	 * 如果没定义HandlerMapping组件类型的Bean的时候。默认使用组件RequestMappingHandlerMapping BeanNameUrlHandlerMapping.
	 */
	private void initHandlerMappings(ApplicationContext context) {
		this.handlerMappings = null;
// 是否查找当前BeanFactory与其所有父BeanFactory中HandlerMapping组件，默认为true
		if (this.detectAllHandlerMappings) {
			// Find all HandlerMappings in the ApplicationContext, including ancestor contexts.
            // 查找当前以及所有祖先BeanFactory中的HandlerMapping类型的Bean，返回Map类型，key为BeanName，value为Bean实例
			Map<String, HandlerMapping> matchingBeans =
					BeanFactoryUtils.beansOfTypeIncludingAncestors(context, HandlerMapping.class, true, false);
			if (!matchingBeans.isEmpty()) {
				this.handlerMappings = new ArrayList<>(matchingBeans.values());
				// We keep HandlerMappings in sorted order.
                // 排序
				AnnotationAwareOrderComparator.sort(this.handlerMappings);
			}
		}
		else {
            // 否则，只查找BeanName为handlerMapping的Bean作为唯一HandlerMapping组件使用
			try {
				HandlerMapping hm = context.getBean(HANDLER_MAPPING_BEAN_NAME, HandlerMapping.class);
				this.handlerMappings = Collections.singletonList(hm);
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Ignore, we'll add a default HandlerMapping later.
			}
		}

		// Ensure we have at least one HandlerMapping, by registering
		// a default HandlerMapping if no other mappings are found.
        // 当Bean策略查找不到的时候使用默认策略
		if (this.handlerMappings == null) {
			this.handlerMappings = getDefaultStrategies(context, HandlerMapping.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No HandlerMappings declared for servlet '" + getServletName() +
						"': using default strategies from DispatcherServlet.properties");
			}
		}
	}

```

​		在默认情况下，会通过当前BeanFactory与其祖先BeanFactory中获取所有的HandlerMapping类型的Bean作为HandlerMapping组件列表使用。默认包括以下组件

|          Bean的名称          |                          Bean的类型                          |                          作用                           |                          初始化位置                          |
| :--------------------------: | :----------------------------------------------------------: | :-----------------------------------------------------: | :----------------------------------------------------------: |
|    faviconHandlerMapping     | org.springframework.<br />web.servlet.handler.<br />SimpleUrlHandlerMapping | 简单URl映射，处理**/favicon.ico路径请求，返回网站小图标 | WebMvcAutoConfiguration.<br />WebMvcAutoConfigurationAdapter.<br />FaviconConfiguration.faviconHandlerMapping |
| requestMappingHandlerMapping | org.springframework<br />.web.servlet.mvc.<br />method.annotation.<br />RequestMappingHandlerMapping |         处理@RequestMapping注解注册的请求处理器         | WebMvcAutoConfiguration.<br />EnableWebMvcConfiguration.<br />requestMappingHandlerMapping |
|    beanNameHandlerMapping    | org.springframework.<br />web.servlet.handler.<br />BeanNameHandlerMapping |              用于通过URL映射到Bean的处理器              | EnableWebMvcConfiguration的祖先类<br />WebMVcConfigurationSupport.<br />beanNameHandlerMappinging() |
|    resourceHandlerMapping    | org.springframework.<br />web.servlet.handler.<br />SimpleUrlHandlerMapping |                  用于处理静态资源映射                   | EnableWebMvcConfiguration的祖先类<br />WebMVcConfigurationSupport.<br />resourceHandlerMapping() |
|  WelcomePageHandlerMapping   | org.springframework.<br />boot.autoconfigure.web.servlet<br />WelcomePageHandlerMapping |          用于处理默认首页请求跳转到index.html           | EnableWebMvcConfiguration的祖先类<br />WebMVcConfigurationSupport.<br />weclomePageHandlerMapping() |

> ​		大部分组件Bean的初始化都在WebMvcAutoConfiguration类、EnableWebMvcConfiguration类及其父类WebMvcConfigurationSupport中
>
> ​		在WebMvcAutoConfiguration自动配置类中，又包含WebMVCAutoConfigurationAdapter配置类，在这个配置中，通过@Import注解引入EnableWebMvcConfiguration类作为配置类，使得EnableWebMvcConfiguration配置类称为Spring Mvc 所有相关组件定义的核心配置类，在该配置类中，包括全部Mvc需要使用到组件的初始化逻辑

```java
/**
 * Configuration equivalent to {@code @EnableWebMvc}.
 */
@Configuration(proxyBeanMethods = false)
public static class EnableWebMvcConfiguration extends DelegatingWebMvcConfiguration implements ResourceLoaderAware {

   private final ResourceProperties resourceProperties;
	// 配置属性映射类
   private final WebMvcProperties mvcProperties;

   private final ListableBeanFactory beanFactory;

   private final WebMvcRegistrations mvcRegistrations;

   private ResourceLoader resourceLoader;
	
//....省略其他方法
   // 声明为Primary，当过Bean类型获取Bean时，如果有多个Bean存在，则使用标记了@primary的Bean作为结果
   @Bean
   @Primary
   @Override
   public RequestMappingHandlerMapping requestMappingHandlerMapping(
         @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,
         @Qualifier("mvcConversionService") FormattingConversionService conversionService,
         @Qualifier("mvcResourceUrlProvider") ResourceUrlProvider resourceUrlProvider) {
      // Must be @Primary for MvcUriComponentsBuilder to work
      // 标记Primary后MvcUriComponetsBuilder才能正常工作，此部分内容读者可自行了解
       // 调用父类方法初始化RequestMappingHandlerMapping组件
      return super.requestMappingHandlerMapping(contentNegotiationManager, conversionService,
            resourceUrlProvider);
   }
/
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

   private Optional<Resource> getWelcomePage() {
      String[] locations = getResourceLocations(this.resourceProperties.getStaticLocations());
      return Arrays.stream(locations).map(this::getIndexHtml).filter(this::isReadable).findFirst();
   }

   private Resource getIndexHtml(String location) {
      return this.resourceLoader.getResource(location + "index.html");
   }

   private boolean isReadable(Resource resource) {
      try {
         return resource.exists() && (resource.getURL() != null);
      }
      catch (Exception ex) {
         return false;
      }
   }


   @Override
   protected RequestMappingHandlerMapping createRequestMappingHandlerMapping() {
      if (this.mvcRegistrations != null && this.mvcRegistrations.getRequestMappingHandlerMapping() != null) {
         return this.mvcRegistrations.getRequestMappingHandlerMapping();
      }
      return super.createRequestMappingHandlerMapping();
   }



}
```

> ​		以上代码可以看到并不是在本类中创建的RequestMappingHandlerMapping实例，而是在父类中创建的。其父类为DelegatingWebMvcConfiguration，该类中也没有此方法，所以requestmappingHandlerMapping最终调用的是，WebMvcConfigurationSupport中的方法。
>
> ​		DelegatingWebMvcConfiguration类的作用是为实现通过WebMvcConfigurer类型的Bean作为配置器List，并在父类WebMvcConfigurationSupport执行组件配置方式时，遍历这个list，逐一进行组件配置。
>
>   		WebMvcConfigurationSupport类的作用则是为Mvc组件定义与配置提供支持，所有组件Bean的定义都在这个类中，同时该类在定义组件前后会执行一些配置方法。Spring boot引入的配置类EnableWebMvcConfiguration作用仅仅是为了通过spring.nvc相关配置来定制其中一些组件的特性，就如上面代码中看到的RequestMappngHandlerAdapter的定义一样，这也是通过配置来定制组件特性的实现原理。
>
> ​		DelegatingWebMvcConfiguration实现了这些配置方法，通过其中保存的WebMvcConfigur配置器列表执行所有配置器的对应方法。

### 5 HandlerAdapter

> ​		HandlerAdapter组件是处理请求的核心组件，该组件为列表。其初始化逻辑与HandlerMapping初始化逻辑相同，当BeanFactory中无HandlerAdapter组件时，使用默认组件列表：HttpRequestHandlerAdapter、SimpleControllerHandlerAdapter、RequestMappingHandlerAdapter。在SpringBoot中，默认情况是通过BeanFactory获取组件列表

| Bean名称                      | Bean类型                     | 作用                                    | 初始化配置                                                   |
| ----------------------------- | ---------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| requestMappingHandlerAdapter  | RequestMappingHandlerAdapter | 处理@RequestMapping标记注解生成的处理器 | WebMVcConfigurationSupport.<br />reqeustMappingHandlerAdapter() |
| httpRequestHandlerAdapter     | HttpRequestHandlerAdapter    | 处理HttpRequestHandler类型的处理器      | WebMVcConfigurationSupport.<br />httpRequestHandlerAdapter() |
| simpleControlerHandlerAdapter | impleControlerHandlerAdapter | 处理Controller类型的处理器              | WebMVcConfigurationSupport.<br />simpleControlerHandlerAdapter() |

```java

@Bean
public HttpRequestHandlerAdapter httpRequestHandlerAdapter() {
   return new HttpRequestHandlerAdapter();
}

@Bean
public SimpleControllerHandlerAdapter simpleControllerHandlerAdapter() {
   return new SimpleControllerHandlerAdapter();
}


```

WebConfigurationSupport仅仅是WebMvc配置支持类，本身并不是配置类，所以其内部定义@Bean标记的方法并不会产生Bean所以在实际环境中还需要创建一个配置类，继承该配置支持类，来实现该类作为配置类的目的。

在SpringMvc中，由配置类DelegatingWebMvcConfiguration中还添加了通过WebMvcConfigurer配置器组件配置WebMVc组件功能。在SpringBoot中，又通过EnableWebMvcConfiguration继承了DelegatingWebMvcConfiguration，从而实现更强大的通过配置文件定义的属性来配置组件功能。

### 6 HandlerExceptionResolver

> ​		HandlerExceptionResolver组件为一个列表，其初始化逻辑与HandlerMapping初始化逻辑相同，默认组件列表为HandlerExceptionResolver、ResponseStatusExceptionResolver、DefaultHandlerExceptionResovler，在SpringBoot默认通过BeanFactory获取组件列表如下

| Bean的名称               | Bean的类型                         | 作用                                   | 初始化位置                                           |
| ------------------------ | ---------------------------------- | -------------------------------------- | ---------------------------------------------------- |
| errorAttributes          | DefaultErrorAttributes             | 用于发生异常时保存异常信息到请求属性中 | ErrorMvcConfoguration.<br />errorAttributes()        |
| handlerExceptionResolver | HandlerExceptionResolver-Composite | 用于组合已有的处理器异常解析器组件     | WebMvcconfigurationSupport.-handlerExceptionResolver |

> ​		在handlerExceptionResolver这个HandlerExceptionResolverComposite组合组件中，还包含了三个处理器，改组件用于包装这三个处理器异常解析器，顺序执行其中三个HandlerExceptionResolver、ResponseStatusExceptionResovler、DefaultHandlerExceptionResolver，这三个组件都是在WebMvcConfigurationSupport.handlerExceptionResolver方法中初始化
>
> ​		在该方法中，调用了本类的configurationHandlerExceptionResolvers方法，配置处理器异常解析器。在实际运行时，该方法执行的实例在其子类WebMvcAutoConfiguration.EnableWebMvcConfiguration中定义，子类重写了该方法。

```java
@Override
protected void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers) {

    super.extendHandlerExceptionResolvers(exceptionResolvers);
    // 为异常添加记录
    //如果spring.mvc.log-resolved-exception为true,则为异常添加日志u记录
    if (this.mvcProperties.isLogResolvedException()) {
        for (HandlerExceptionResolver resolver : exceptionResolvers) {
            if (resolver instanceof AbstractHandlerExceptionResolver) {
                ((AbstractHandlerExceptionResolver) resolver).setWarnLogCategory(resolver.getClass().getName());
            }
        }
    }
}
@Bean
public HandlerExceptionResolver handlerExceptionResolver(
    @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager) {
    
    List<HandlerExceptionResolver> exceptionResolvers = new ArrayList<>();
        // 执行父类方法 ，其中的逻辑为执行所有WebMvcConfigur类型的Bean的configureHandlerExceptionResolvers方法，配置异常处理器，默认无此类型的Bean故不作处理器
    configureHandlerExceptionResolvers(exceptionResolvers);
    // 默认情况主席那个默认异常处理逻辑
    if (exceptionResolvers.isEmpty()) {
        addDefaultHandlerExceptionResolvers(exceptionResolvers, contentNegotiationManager);
    }
    extendHandlerExceptionResolvers(exceptionResolvers);
    HandlerExceptionResolverComposite composite = new HandlerExceptionResolverComposite();
    composite.setOrder(0);
    composite.setExceptionResolvers(exceptionResolvers);
    return composite;
}

protected final void addDefaultHandlerExceptionResolvers(List<HandlerExceptionResolver> exceptionResolvers,ContentNegotiationManager mvcContentNegotiationManager) {
 // 创建ExceptionHandlerExceptionResolver添加到异常处理器列表
    ExceptionHandlerExceptionResolver exceptionHandlerResolver = createExceptionHandlerExceptionResolver();
    // 配置异常处理器的内容协商器
    exceptionHandlerResolver.setContentNegotiationManager(mvcContentNegotiationManager);
    // 设置异常处理器用到的消息转换器
    exceptionHandlerResolver.setMessageConverters(getMessageConverters());
    // 设置异常处理器使用的参数有解析器
    exceptionHandlerResolver.setCustomArgumentResolvers(getArgumentResolvers());
    // 设置异常解析器的返回值处理器
    exceptionHandlerResolver.setCustomReturnValueHandlers(getReturnValueHandlers());
    // 如果在jackson2的库,则添加一个响应体增强其
    if (jackson2Present) {
        exceptionHandlerResolver.setResponseBodyAdvice(
            Collections.singletonList(new JsonViewResponseBodyAdvice()));
    }
    // 设置异常处理器的应用上下文
    if (this.applicationContext != null) {
        exceptionHandlerResolver.setApplicationContext(this.applicationContext);
    }
    // 执行异常处理器的初始化方法
    exceptionHandlerResolver.afterPropertiesSet();
    // 添加异常处理器列表
    exceptionResolvers.add(exceptionHandlerResolver);

    ResponseStatusExceptionResolver responseStatusResolver = new ResponseStatusExceptionResolver();
    //设置该异常解析器的信息源
    responseStatusResolver.setMessageSource(this.applicationContext);
    // 添加异常解析器列表
    exceptionResolvers.add(responseStatusResolver);

    exceptionResolvers.add(new DefaultHandlerExceptionResolver());
}


```

### 7RequestToViewNameTranslator

> ​		RequestToViewNameTranslator组件又叫做ViewNameTranslator组件,该组件在DispatcherServlet中存在一个.其初始化策略与Locale组件相同,默认情况下BeanFactory中未提供该组件,使用默认策略指定的DefalutRequestToViewNameTranslator组件

```java
	/**
	 * Initialize the RequestToViewNameTranslator used by this servlet instance.
	 * <p>If no implementation is configured then we default to DefaultRequestToViewNameTranslator.
	 * Beanfactory中没有定义时,使用默认的
	 */
	private void initRequestToViewNameTranslator(ApplicationContext context) {
		try {
            // 默认无组件,触发NoSuchBeanDefinitionEception异常
			this.viewNameTranslator =
					context.getBean(REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME, RequestToViewNameTranslator.class);
			if (logger.isTraceEnabled()) {
				logger.trace("Detected " + this.viewNameTranslator.getClass().getSimpleName());
			}
			else if (logger.isDebugEnabled()) {
				logger.debug("Detected " + this.viewNameTranslator);
			}
		}
		catch (NoSuchBeanDefinitionException ex) {
			// We need to use the default.
            // 使用默认策略
			this.viewNameTranslator = getDefaultStrategy(context, RequestToViewNameTranslator.class);
			if (logger.isTraceEnabled()) {
				logger.trace("No RequestToViewNameTranslator '" + REQUEST_TO_VIEW_NAME_TRANSLATOR_BEAN_NAME +
						"': using default [" + this.viewNameTranslator.getClass().getSimpleName() + "]");
			}
		}
	}
```

> ​		该类型组件同样可以通过在ApplicationContext中定义来实现替换该类型的默认组件

### 8 ViewResolver

> ​		ViewResolver组件为组件列表,其初始化策略与HandlerMapping组件列表相同,在只加入Thymeleaf模板引擎的依赖时.默认BeanFactory中包含五个该组件.

| Bean的名称            | Bean的类型                      | 作用                                                         | 初始化的位置                                                 |
| --------------------- | ------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| beanNameViewResovler  | BeanNameViewResovler            | 用于支持默认的Bean名称是哦图解析器                           | ErrorMvcAutoConfiguration.-whitelabelErrorViewConfiguration.-beanNameViewResolver |
| mvcViewResolver       | ViewResovlerComposite           | 提供configure-ViewResolver方法添加VIewResolver的功能<br />也支持通过BeanFactory获取所有ViewResolver类型的Bean | EnableWebMvcConfiguration-的祖先类WebMvcConfigurationSupport.<br />mvcViewResolver |
| defaultViewResolver   | InternalResource-ViewResolver   | 默认的视图解析器，用于解析html、jsp等内置的静态页面资源      | WebMvcAutoConfiguration.-WebMvcAuto-ConfigurationAdapter.defaultViewResolver |
| viewResolver          | ContentNegotiating-ViewResolver | 内容协商视图解析器，用于支持根据请求接受内容类型与视图内容类型自动匹配视图功能 | WebMvcAutoConfiguration.-WebMvcAuto-ConfigurationAdapter.ViewResolver(BeanFactory beanfactory) |
| thymeleafViewResolver | ThymeleafViewResolver           | 用于支持Thymeleaf类型的模板视图                              | ThymeleaAutoConfiguration.-ThymeleaWebMvcConfiguration.-ThymeleafViewResolverConfiguration.-thymeleafViewResolver |

> ​		其中beanNameViewResolver在WebMvcAutoConfigurationAdapter.beanNameViewResolver（）也进行了初始化，但因ErrorMvcAutoConfiguration的执行顺序在前（通过@AutoConfigureBefore（WebMvcAutoConfiguration.class）指定顺序），故ErrorMvcAutoConfiguration中的beanNameViewResolver有限
>
> ​	另外，Thymeleaf模板视图是在引用Thymeleaf依赖后自动添加的，同此原理，其他类型模板的依赖引入后，也会自动添加对应类型的视图解析器。如freeMarkerViewResolve。关于ThymeLeaf的自动配置，依赖于ThymeleftAutoConfiguration自动配置类，在该自动配置类中判断ThymeLeaf依赖是否存在。存在是则创建ThymeLeaf对应的视图解析器。

```java
// 指定配置类
@Configuration(proxyBeanMethods = false)
// 在servlet容器中使用
@ConditionalOnWebApplication(type = Type.SERVLET)
//当为true或者不存在的时候启用
@ConditionalOnProperty(name = "spring.thymeleaf.enabled", matchIfMissing = true)
static class ThymeleafWebMvcConfiguration {
	
   @Bean
   @ConditionalOnEnabledResourceChain
   @ConditionalOnMissingFilterBean(ResourceUrlEncodingFilter.class)
   FilterRegistrationBean<ResourceUrlEncodingFilter> resourceUrlEncodingFilter() {
      FilterRegistrationBean<ResourceUrlEncodingFilter> registration = new FilterRegistrationBean<>(
            new ResourceUrlEncodingFilter());
      registration.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
      return registration;
   }
// 声明ThymeLeaf的ViewResolver组件
   @Configuration(proxyBeanMethods = false)
   static class ThymeleafViewResolverConfiguration {
		// 声明ThymeLeaf的ViewResolver组件
      @Bean
       // 不存在thymeLeafViewResolver时创建
      @ConditionalOnMissingBean(name = "thymeleafViewResolver")
      ThymeleafViewResolver thymeleafViewResolver(ThymeleafProperties properties,
            SpringTemplateEngine templateEngine) {
          // 构造
         ThymeleafViewResolver resolver = new ThymeleafViewResolver();
          // 设置模板引擎
         resolver.setTemplateEngine(templateEngine);
          // 设置编码
         resolver.setCharacterEncoding(properties.getEncoding().name());
          // 设置contenttype
         resolver.setContentType(
               appendCharset(properties.getServlet().getContentType(), resolver.getCharacterEncoding()));
          // 
         resolver.setProducePartialOutputWhileProcessing(
               properties.getServlet().isProducePartialOutputWhileProcessing());
          // 设根据配置文件置需要排除的视图名
         resolver.setExcludedViewNames(properties.getExcludedViewNames());
          //根据配置文件设置需要解析的视图名
         resolver.setViewNames(properties.getViewNames());
         // This resolver acts as a fallback resolver (e.g. like a
         // InternalResourceViewResolver) so it needs to have low precedence
          // 指定顺序比最低的优先级高一个优先级，由InternalResourceViewResolver兜底
         resolver.setOrder(Ordered.LOWEST_PRECEDENCE - 5);
          // 根据配置文件设置是否启用缓存
         resolver.setCache(properties.isCache());
         return resolver;
      }

      private String appendCharset(MimeType type, String charset) {
         if (type.getCharset() != null) {
            return type.toString();
         }
         LinkedHashMap<String, String> parameters = new LinkedHashMap<>();
         parameters.put("charset", charset);
         parameters.putAll(type.getParameters());
         return new MimeType(type, parameters).toString();
      }

   }

}
```

### 9 FlashMapMannger组件

> ​		在DispatcherServlet中只存在一个FalshMapManager组件.其初始化策略与LocaleResolver组件相同,默认情况下BeanFacotry中没有提供该组件,使用默认策略指定的SessionFlashMapManager组件

```java
private void initFlashMapManager(ApplicationContext context) {
   try {
      this.flashMapManager = context.getBean(FLASH_MAP_MANAGER_BEAN_NAME, FlashMapManager.class);
      if (logger.isTraceEnabled()) {
         logger.trace("Detected " + this.flashMapManager.getClass().getSimpleName());
      }
      else if (logger.isDebugEnabled()) {
         logger.debug("Detected " + this.flashMapManager);
      }
   }
   catch (NoSuchBeanDefinitionException ex) {
      // We need to use the default.
       // 默认使用SessionFlashMapManage
      this.flashMapManager = getDefaultStrategy(context, FlashMapManager.class);
      if (logger.isTraceEnabled()) {
         logger.trace("No FlashMapManager '" + FLASH_MAP_MANAGER_BEAN_NAME +
               "': using default [" + this.flashMapManager.getClass().getSimpleName() + "]");
      }
   }
}
```

> ​		通过初始化代码可知,该类型组件可以在Application中重新定义,但该组件类型较为复杂

### 10 WebAsyncManager组件

> ​		WebAsyncManager组件并不属于DispatcherServlet中的组件,但其也是处理过程中非常重要的组件,该组件的获取是通过方法WebAsyncUtils.getAsyncManage实现

```java
// 根据请求获取该请求对应的异步管理器
public static WebAsyncManager getAsyncManager(ServletRequest servletRequest) {
    WebAsyncManager asyncManager = null;
    // 在当前请求属性中尝试获取
    Object asyncManagerAttr = servletRequest.getAttribute(WEB_ASYNC_MANAGER_ATTRIBUTE);
    if (asyncManagerAttr instanceof WebAsyncManager) {
        asyncManager = (WebAsyncManager) asyncManagerAttr;
    }
    //不存在创建，并添加到属性当中
    if (asyncManager == null) {
        asyncManager = new WebAsyncManager();
        servletRequest.setAttribute(WEB_ASYNC_MANAGER_ATTRIBUTE, asyncManager);
    }
    return asyncManager;
}
```

> ​		在配置文件中，可以通过spring.mvc.async.request.request-timeout指定异步请求超时，同时也可以通过WebMvcConfigurer接口实现类Bean的configureAsyncSupport方法配置更多的属性。
>
> ​		已知MvcProperties为spring.mvc下所有配置属性的表现类，那其中也必然有spring.mvc.async.request-timeout的绑定属性，为MvcProperties.async.requestTimeout，通过查找该属性的使用位置，找到了使用该值进行配置的逻辑，即在WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter的configureAsyncSupport方法中。
> ​		可以看到WebMvcAutoConfigurationAdapter正是WebMvcConfigurer配置器接口实现类，即改配置其实是通过配置接口的实现类Bean实现的，
>
> ​		那么配置器是如何生效的呢，在RequestMapping组件初始化中，已经讲述过EnableWebMVCConfiguration与其父类DelegatingWebMvcConfiguration及再上一级的父类WebMvcConfigurationSupport的大致作用，可以就以Async的配置为例，简述其代码的实现原理。一样通过反查法的方式推断。
>
> ​		通过查找WebMvcConfigurer.configureAsyncSupport方法的调用，可以找到两处。第一处是DelegatingWebMvcConfiguration的configureAsyncSupport方法，第二处是WebMvcConfigurerComposite的configureAsyncSupport方法，第一次调用的就是第二次的方法。

```java
public class DelegatingWebMvcConfiguration extends WebMvcConfigurationSupport {
// 创建WebMvc配置类组合类，用于组合全部配置器实例
   private final WebMvcConfigurerComposite configurers = new WebMvcConfigurerComposite();
    
      // 自定注入当前应用上下文所有MVC配置类
    @Autowired(required = false)
	public void setConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.configurers.addWebMvcConfigurers(configurers);
		}
	}
    、
    @Override
	protected void configureAsyncSupport(AsyncSupportConfigurer configurer) {
		// 配置时，调用配置类组合器的配置方法
        this.configurers.configureAsyncSupport(configurer);
	}
}
//配置器组合器，同样实现配置器接口
class WebMvcConfigurerComposite implements WebMvcConfigurer {
	// 全部配置类Bean实例列表，名字叫做delegates代理，即代理Bean的调用
	private final List<WebMvcConfigurer> delegates = new ArrayList<>();
    
    //添加需要代理执行的配置类
	public void addWebMvcConfigurers(List<WebMvcConfigurer> configurers) {
		if (!CollectionUtils.isEmpty(configurers)) {
			this.delegates.addAll(configurers);
		}
	}
    
    // 代理执行配置方法
    @Override
	public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
		for (WebMvcConfigurer delegate : this.delegates) {
			delegate.configureAsyncSupport(configurer);
		}
	}

}
```

> ​		可以看到配置器组合类的目的是代理多个配置器的配置方法调用，所以该方法的真实调用其实是DelegatingWebMvcConfigureration的ConfigureAsyncSupport
>
> 方法，该方法重写了父类WebMvcConfigureationSupport方法

```java
@Bean
public RequestMappingHandlerAdapter requestMappingHandlerAdapter(
      @Qualifier("mvcContentNegotiationManager") ContentNegotiationManager contentNegotiationManager,
      @Qualifier("mvcConversionService") FormattingConversionService conversionService,
      @Qualifier("mvcValidator") Validator validator) {
//...省略
    // 创建Async配置支持
   AsyncSupportConfigurer configurer = new AsyncSupportConfigurer();
    //调用 Async配置方法，子类重写该方法实现通过WevMvcConfigurer配置Async的目的
   configureAsyncSupport(configurer);
    // 配置完成之后获取Async相关配置与组件并设置到RequestMappingHandlerAdapter组件中
   if (configurer.getTaskExecutor() != null) {
      adapter.setTaskExecutor(configurer.getTaskExecutor());
   }
    /
   if (configurer.getTimeout() != null) {
      adapter.setAsyncRequestTimeout(configurer.getTimeout());
   }
   adapter.setCallableInterceptors(configurer.getCallableInterceptors());
   adapter.setDeferredResultInterceptors(configurer.getDeferredResultInterceptors());

   return adapter;
}
```

### 11  ThemeSource 主题源

> ​		主题源也不是DispatchServlet原始组件的一部分，其在DispatcherServlet的doService方法中设置到请求属性中，通过DispatcherServlet的getThemeSource方法获取了主题源。在默认情况，获取到的主题源是Web应用上下文自身，因为默认Web应用上下文实现类ThemeSource接口。

```java
//获取主题源
@Nullable
public final ThemeSource getThemeSource() {
   return (getWebApplicationContext() instanceof ThemeSource ? (ThemeSource) getWebApplicationContext() : null);
}

```

> ​		而在Web应用上下文中，其实现的ThemeSource接口的getTheme方法其实是个代理方法，真正执行的是当前Web应用上下文实例中保存的ThemeSource主题源实例。

```java
// 代理方法
@Override
@Nullable
public Theme getTheme(String themeName) {
   Assert.state(this.themeSource != null, "No ThemeSource available");
    // 实际执行本类主题源实例的该方法
   return this.themeSource.getTheme(themeName);
}
```

> ​		那么上面代码中主题源实例何时初始化的呢？通过查找其引用，找到了设置值的位置为GenericWebApplicationContext上下文类的onRefresh方法，通过UiApplicationContextUtils的initThemeSource方法进行初始化
>
> ​		在initThemeSource方法中，则尝试先从BeanFactory中或其父BeanFactory中获取名称为themesource，类型为ThemeSource的bean作为主题源。

### 12 MessageSource信息源

### 13 SpringBoot 配置类简单解析

![image-20201206164605785](image-20201206164605785.png)

* `WebMvcConfigurer`

> 暂时理解为WebMvc的配置扩展类

* `WebMvcConfigurerComposite`

> WebMvcConfigurer的组合器

* `WebMvcConfigurerSupport`

> 为Mvc组件的定义与组件配置提供支持，所有的Bean的定义都在这个组类中，同时该类在定义前后会执行一些配置方法。SpringBoot引入配置类`EnableWebMvcConfiguration`的作用仅仅是为了通过spring.mvc相关配置来定制其中的一些组件的特性

* `DelegatingWebMvcConfiguration`

> 为实现通过WebMvcConfigurer接口实现类Bean进行Mvc配置的目的。其中注入当前上下文中全部WebMvcConfigurer类型的Bean作为配置器List，并在父类WebMvcConfigurationSupport执行组件配置方式时，遍历这个配置器list，实现逐一配置。实现类内部@Bean定义组件的目的，同时DelegatingWebMvcconfiguration中还添加了通过WebMvcConfigurer配置器组件配置WebMvc组件功能

* `WebMvcAutoConfigurer`

> 在此类中包含`WebMvcAutoConfigurationAdapter`类

* `EnableWebMvcConfiguration`

> 为了通过spring.mvc相关配置来定制其中的一些组件的特性，并且包含全部的MVC需要使用到组件初始化逻辑

* `WebMvcAutoConfigurationAdapter`

> 引入了EnableWebMvcConfigureation类作为配置类，在该配置类中包含全部的MVC需要使用到组件初始化逻辑，并且配置一部分MvcProperties属性

**总结：**

> Spring Mvc 组件配置核心就是在SpringBoot中为WebMvcAutoconfiguration自动配置类，其中根据提供的配置属性定制、创建一些MVC中的组件。同时还提供了WebMvcAutoConfigurationAdapter，该类为WEbMvcConfigurer配置器的实现类，其中的配置方法里又根据配置文件中提供的配置属性定制了MVC组件特性，又提供了EnableWebMvcConfiguration这个核心的MVC配置类，内部提供了MVC全部组件的注册与配置逻辑，用于实现无法通过WebMvcConfigurer达到配置的功能。