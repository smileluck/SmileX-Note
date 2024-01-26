[toc]

---

# 前言

对于 Spring Bean 的作用域需要有一定了解。[传送门](./Spring-Bean.md)

# InitializingBean

 实现`InitializingBean`接口的`Bean`，该`Bean`实例化后`Bean`中所有的属性被`BeanFactory`进行注入之后的，检查所有强制性属性的设置。 

1.  afterPropertiesSet() 方法在 BeanFactory 为 Bean 填充完属性后触发
2. `afterPropertiesSet()`方法允许`Bean`在最终初始化完成之后，针对`BeanFactory`的属性注入以及整体配置的验证。
3. `afterPropertiesSet()`方法可以在`Bean`发生错误配置(如未能设置一个基本属性)或者因为其他任何原因初始化失败而抛出异常。
4. InitializingBean有个功能类似的替换方案：在XML配置文件中配置init-method。

# SmartInitializingSingleton

 `SmartInitializingSingleton`接口中的`afterSingletonsInstantiated()`方法将会在所有的非惰性单实例`Bean`初始化完成之后进行回调。 

1. bean的afterSingletonsInstantiated()方法是在所有单例bean都初始化完成后才会调用`afterSingletonsInstantiated`的。
2. 此接口可以解决一些因为bean初始化太早而出现的错误和问题。
3. 此接口是从Spring4.1版本开始使用的。
4. 此接口可以看作是所有的Bean初始化完成后的`InitializingBean`接口的一种替代方案。

# 区别

1. SmartInitializingSingleton只作用于非懒加载单例bean，InitializingBean无此要求。但他们都不能用于懒加载的bean。
2. SmartInitializingSingleton是在所有单例Bean都初始化完成后调用的，InitializingBean是每个bean初始化完成后就会调用。

# 触发时机

# Spring 容器启动核心

> 源码 AbstractApplicationContext#refresh()

```java

	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                // 实例化所有剩余的（非惰性初始化）单例
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}

```

> 源码 AbstractApplicationContext#finishBeanFactoryInitialization()

```java

	/**
	 * Finish the initialization of this context's bean factory,
	 * initializing all remaining singleton beans.
	 */
	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
		// Initialize conversion service for this context.
		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
			beanFactory.setConversionService(
					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
		}

		// Register a default embedded value resolver if no BeanFactoryPostProcessor
		// (such as a PropertySourcesPlaceholderConfigurer bean) registered any before:
		// at this point, primarily for resolution in annotation attribute values.
		if (!beanFactory.hasEmbeddedValueResolver()) {
			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
		}

		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
		for (String weaverAwareName : weaverAwareNames) {
			getBean(weaverAwareName);
		}

		// Stop using the temporary ClassLoader for type matching.
		beanFactory.setTempClassLoader(null);

		// Allow for caching all bean definition metadata, not expecting further changes.
		beanFactory.freezeConfiguration();

		// Instantiate all remaining (non-lazy-init) singletons.
        // 实例化所有剩余的（非惰性初始化）单例
		beanFactory.preInstantiateSingletons();
	}
```

## SmartInitializingSingleton 的触发

> 源码 DefaultListableBeanFactory#preInstantiateSingletons() 

```java

	@Override
	public void preInstantiateSingletons() throws BeansException {
		if (logger.isTraceEnabled()) {
			logger.trace("Pre-instantiating singletons in " + this);
		}

		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
        // 创建一个 beanDefinitionNames 的副本用于遍历，以允许init方法注册新的bean定义。尽管这不是常规工厂启动的一部分但很好用。
		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);

		// 触发所有非惰性(懒加载)单例bean的初始化
		for (String beanName : beanNames) {
            
            // 返回合并的RootBeanDefinition，遍历父bean定义，如果指定的bean对应于子bean定义。
            // 根 Bean 定义本质上是运行时的“统一”Bean 定义视图。
           //  Spring 上下文包括实例化所有 Bean 使用的 AbstractBeanDefinition 是 RootBeanDefinition
            // 使用 getMergedLocalBeanDefinition 方法做了一次转化, 将非 RootBeanDefinition 转换为 RootBeanDefinition 以供后续操作
            // 注意, 如果当前 BeanDefinition 存在父 BeanDefinition, 会基于父 BeanDefinition 生成一个 RootBeanDefinition, 然后在将调用 OverrideFrom 子 BeanDefinition 的相关属性覆写进去
			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
            
            // 是否非抽象类 是否是单例， 是否非懒加载
			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
             	// 判断是否为 FacotryBean
				if (isFactoryBean(beanName)) {
                    // 调用 getBean方法，传入 Factory_bean的前缀 + Bean名称
					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                    // 判断是否是 FactoryBean方法
					if (bean instanceof FactoryBean) {
						FactoryBean<?> factory = (FactoryBean<?>) bean;
                        // 是否是需要立即初始化
						boolean isEagerInit;
                        // 判断系统安全接口是否为空和Bean 是否实现了 SmartFactoryBean 接口, 如果实现了 SmartFactoryBean 
						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
                            // 方法判断是否希望急切的进行初始化
							isEagerInit = AccessController.doPrivileged(
									(PrivilegedAction<Boolean>) ((SmartFactoryBean<?>) factory)::isEagerInit,
									getAccessControlContext());
						}
						else {
							isEagerInit = (factory instanceof SmartFactoryBean &&
									((SmartFactoryBean<?>) factory).isEagerInit());
						}
                        // 如果需要立即初始化， 则通过getBean初始化
						if (isEagerInit) {
							getBean(beanName);
						}
					}
				}
				else {
                    // 如果 beanName 对应的 Bean 不是 FactoryBean, 只是普通 Bean, 调用 getBean 方法通过 beanName 获取 Bean 实例对象
					getBean(beanName);
				}
			}
		}

        // 触发所有可用的Bean的初始化回调。
		for (String beanName : beanNames) {
			// 获取 beanName 对应的单例Bean实例对象
			Object singletonInstance = getSingleton(beanName);
            // 判断获取的 SigletonInstance 是否实现了 SmartInitializingSingleton 接口
			if (singletonInstance instanceof SmartInitializingSingleton) {
				SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
                // 判断系统安全接口是否为空，触发 afterSingletonInstantiated 初始化。
				if (System.getSecurityManager() != null) {
                	AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
						smartSingleton.afterSingletonsInstantiated();
						return null;
					}, getAccessControlContext());
				}
				else {
					smartSingleton.afterSingletonsInstantiated();
				}
			}
		}
	}

```

1. 获取所有的bean，判断是否是 beanFactory，如果是检查是否需要立即初始化，否则将其初始化。
2. 遍历触发所有可用的 Bean 初始化回调。获取单例对象，触发实现 `SmartInitializingSingleton` 接口的 `afterSingletonsInstantiated` 方法。
3. 对于 `SmartInitializingSingleton` 的触发实在所有非惰性单实例Bean初始化完成后进行的。

## InitializingBean的触发

> 源码AbstractAutowireCapableBeanFactory#invokeInitMethods() 

```java
/**
	 * Give a bean a chance to react now all its properties are set,
	 * and a chance to know about its owning bean factory (this object).
	 * This means checking whether the bean implements InitializingBean or defines
	 * a custom init method, and invoking the necessary callback(s) if it does.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the merged bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @throws Throwable if thrown by init methods or by the invocation process
	 * @see #invokeCustomInitMethod
	 */
// 现在 Bean 的所有属性已经设置好了，给 Bean 一个反射的机会，以及了解他拥有的Bean Factory(这个对象)的机会
// 这意味着检查 bean 是否实现了 InitializingBean 接口或定义了自定义的初始化方法，如果有的话，将调用必要的回调
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
    throws Throwable {
	// 判断 bean 是否实现 InitializingBean 接口
    boolean isInitializingBean = (bean instanceof InitializingBean);
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (logger.isTraceEnabled()) {
            logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }
        // 判断系统安全管理是否为空，然后调用 afterPropertiesSet 方法
        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
                    ((InitializingBean) bean).afterPropertiesSet();
                    return null;
                }, getAccessControlContext());
            }
            catch (PrivilegedActionException pae) {
                throw pae.getException();
            }
        }
        else {
            ((InitializingBean) bean).afterPropertiesSet();
        }
    }

    // 自定义初始化方法
    // 如果 mbd 不为空，且bean的类不等于 NullBean.class 
    if (mbd != null && bean.getClass() != NullBean.class) {
        // 获取初始化方法名
        String initMethodName = mbd.getInitMethodName();
       // 判断长度、
        //初始化名称是否等于“afterPropertiesSet” 和 判断是否是 isInitializingBean
        // 是否指定初始化方法
        if (StringUtils.hasLength(initMethodName) &&
            !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
            !mbd.isExternallyManagedInitMethod(initMethodName)) {
            // 调用自定义方法 init-method
            invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```

1. Bean实例如果实现了`InitializingBean`接口则直接触发`InitializingBean#afterPropertiesSet()`。
2. 如果Bean实例指定了init-method，则通过反射激活Bean指定的init-method。
3. 如果Bean实例实现了`InitializingBean`接口和`init-method`方法，则先调用 `afterPropertiesSet`，然后调用 `init-method`
4.  `Spring`对于`InitializingBean`的触发是在`Bean`所有属性已经设置好了之后。 
5. 实现`InitializingBean`接口是直接调用`afterPropertiesSet`方法，比通过反射调用`init-method`指定的方法效率要高一点，但是`init-method`方式消除了对`spring`的依赖。
6. 如果调用`afterPropertiesSet`方法时出错，则不调用`init-method`指定的方法。