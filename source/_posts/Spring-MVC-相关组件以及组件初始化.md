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

