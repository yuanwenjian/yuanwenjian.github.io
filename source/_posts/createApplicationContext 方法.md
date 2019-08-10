```java

    protected ConfigurableApplicationContext createApplicationContext() {
        Class<?> contextClass = this.applicationContextClass;
        if (contextClass == null) {
            try {
                contextClass = Class.forName(this.webEnvironment ? "org.springframework.boot.context.embedded.AnnotationConfigEmbeddedWebApplicationContext" : "org.springframework.context.annotation.AnnotationConfigApplicationContext");
            } catch (ClassNotFoundException var3) {
                throw new IllegalStateException("Unable create a default ApplicationContext, please specify an ApplicationContextClass", var3);
            }
        }

        return (ConfigurableApplicationContext)BeanUtils.instantiate(contextClass);
    }
```

AnnotationConfigEmbeddedWebApplicationContext构造方法如下 ,通过以下代码得知，只是为成员变量赋值
```java
    public AnnotationConfigEmbeddedWebApplicationContext() {
        this.reader = new AnnotatedBeanDefinitionReader(this); 
        this.scanner = new ClassPathBeanDefinitionScanner(this);
    }
```
AnnotatedBeanDefinitionReader 解析并读取BeanDefinition
ClassPathBeanDefinitionScanner 用于扫描classpath目录下带注解的beanclass

AnnotatedBeanDefinitionReader有两个重要的属性BeanNameGenerator、ScopeMetadataResolver。
BeanNameGenerator默认实现类是AnnotationBeanNameGenerator，AnnotationBeanNameGenerator用于生成带有@Component、@Controller、@Service、@Repository注解的beanDefinition；
ScopeMetadataResolver默认实现类是AnnotationScopeMetadataResolver，用于处理带有@Scope注解的beanDefinition。


AnnotatedBeanDefinitionReader 构造方法
```java
    public AnnotatedBeanDefinitionReader(BeanDefinitionRegistry registry, Environment environment) {
        //赋值
        this.beanNameGenerator = new AnnotationBeanNameGenerator(); 
        this.scopeMetadataResolver = new AnnotationScopeMetadataResolver();
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        Assert.notNull(environment, "Environment must not be null");
        this.registry = registry;
        this.conditionEvaluator = new ConditionEvaluator(registry, environment, (ResourceLoader)null);
        // 注册相关的BeanPostProcessor、BeanFactoryPostProcessor、AutowireCandidateResolver到ApplicationContext
        AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
    }
```
在给定的注册表中注册所有相关的后置处理器(post processors)
```java
    public static Set<BeanDefinitionHolder> registerAnnotationConfigProcessors(BeanDefinitionRegistry registry, Object source) {
        DefaultListableBeanFactory beanFactory = unwrapDefaultListableBeanFactory(registry);
        if (beanFactory != null) {
            if (!(beanFactory.getDependencyComparator() instanceof AnnotationAwareOrderComparator)) {
                beanFactory.setDependencyComparator(AnnotationAwareOrderComparator.INSTANCE);
            }

            if (!(beanFactory.getAutowireCandidateResolver() instanceof ContextAnnotationAutowireCandidateResolver)) {
                beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());
            }
        }

        Set<BeanDefinitionHolder> beanDefs = new LinkedHashSet(4);
        RootBeanDefinition def;
        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalConfigurationAnnotationProcessor")) {
            //ConfigurationClassPostProcessor 用于处理注解了@Configuration那些类
            def = new RootBeanDefinition(ConfigurationClassPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalConfigurationAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalAutowiredAnnotationProcessor")) {
            //AutowiredAnnotationBeanPostProcessor 用于处理注解了@Autowired、@Value、@Inject那些类
            def = new RootBeanDefinition(AutowiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalAutowiredAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.annotation.internalRequiredAnnotationProcessor")) {
            //RequiredAnnotationBeanPostProcessor 用于处理注解了@Required那些类
            def = new RootBeanDefinition(RequiredAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalRequiredAnnotationProcessor"));
        }

        if (jsr250Present && !registry.containsBeanDefinition("org.springframework.context.annotation.internalCommonAnnotationProcessor")) {
            //CommonAnnotationBeanPostProcessor 用于处理注解了@PostConstruct、@PreDestroy那些类
            def = new RootBeanDefinition(CommonAnnotationBeanPostProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalCommonAnnotationProcessor"));
        }

        if (jpaPresent && !registry.containsBeanDefinition("org.springframework.context.annotation.internalPersistenceAnnotationProcessor")) {
            def = new RootBeanDefinition();

            try {
                def.setBeanClass(ClassUtils.forName("org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", AnnotationConfigUtils.class.getClassLoader()));
            } catch (ClassNotFoundException var6) {
                throw new IllegalStateException("Cannot load optional framework class: org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor", var6);
            }

            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.annotation.internalPersistenceAnnotationProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerProcessor")) {
            //EventListenerMethodProcessor 将注解了@EventListener方法注册为单独的ApplicationListener实例 
            def = new RootBeanDefinition(EventListenerMethodProcessor.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerProcessor"));
        }

        if (!registry.containsBeanDefinition("org.springframework.context.event.internalEventListenerFactory")) {
            def = new RootBeanDefinition(DefaultEventListenerFactory.class);
            def.setSource(source);
            beanDefs.add(registerPostProcessor(registry, def, "org.springframework.context.event.internalEventListenerFactory"));
        }

        return beanDefs;
    }
```

以上这些类放入DefaultListableBeanFactory.beanDefinitionMap 属性 该map为concurrentHashMap,事实上，只要是容器所负责的类都会放入这个属性，后面实例化过程中的注册的bean也会陆续放入DefaultListableBeanFactory.beanDefinitionMap中，即我们常说的Beans被spring容器所管理

