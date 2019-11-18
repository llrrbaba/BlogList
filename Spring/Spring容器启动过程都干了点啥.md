#### Spring容器启动过程都干了点啥

1.首先看一个大家都很熟悉的构造函数--ClassPathXmlApplicationContext，这是Spring使用xml年代，经常会见到的一个容器实现：

~~~java
	/**
	 * Create a new ClassPathXmlApplicationContext, loading the definitions
	 * from the given XML file and automatically refreshing the context.
	 * @param configLocation resource location
	 * @throws BeansException if context creation failed
	 *
	 * 1.从给定的xml文件中加载配置
	 * 2.自动刷新context(refresh默认为true)
	 * 3.创建一个新的容器(ClassPathXmlApplicationContext)
	 */
	public ClassPathXmlApplicationContext(String configLocation) throws BeansException {
		this(new String[] {configLocation}, true, null);
	}
~~~

在看一个SpringBoot年代，使用注解，经常会见到的一个容器的构造函数实现：

~~~java
	/**
	 * Create a new AnnotationConfigApplicationContext, deriving bean definitions
	 * from the given annotated classes and automatically refreshing the context.
	 * @param annotatedClasses one or more annotated classes,
	 * e.g. {@link Configuration @Configuration} classes
	 *
	 * 1.从使用了@Configuration注解的类(class)中加载配置
	 * 2.自动刷新context
	 * 3.创建一个新的容器(AnnotationConfigApplicationContext)
	 */
	public AnnotationConfigApplicationContext(Class<?>... annotatedClasses) {
		this();
		register(annotatedClasses);
		refresh();
	}
~~~

2.继续往下跟`ClassPathXmlApplicationContext(String configLocation)`这个构造函数

~~~java
	/**
	 * Create a new ClassPathXmlApplicationContext with the given parent,
	 * loading the definitions from the given XML files.
	 * @param configLocations array of resource locations
	 * @param refresh whether to automatically refresh the context,
	 * loading all bean definitions and creating all singletons.
	 * Alternatively, call refresh manually after further configuring the context.
	 * @param parent the parent context
	 * @throws BeansException if context creation failed
	 * @see #refresh()
	 */
	public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {

		super(parent);
		setConfigLocations(configLocations);
		if (refresh) {
			refresh();
		}
	}
~~~

2.1 看`setConfigLocations(configLocations)`方法

~~~java
	/**
	 * Set the config locations for this application context.
	 * <p>If not set, the implementation may use a default as appropriate.
	 *
	 * 设置容器(application context)使用的配置文件地址(config locations)
	 * 入股没有设置，会使用默认的地址
	 */
	public void setConfigLocations(@Nullable String... locations) {
		if (locations != null) {
			Assert.noNullElements(locations, "Config locations must not be null");
			this.configLocations = new String[locations.length];
            // 解析location，并设置到configLocations数组
			for (int i = 0; i < locations.length; i++) {
				this.configLocations[i] = resolvePath(locations[i]).trim();
			}
		}
		else {
			this.configLocations = null;
		}
	}
~~~

2.1.1看下`resolvePath(String path)`怎么解析这些地址(location)的

~~~java
	/**
	 * Resolve the given path, replacing placeholders with corresponding
	 * environment property values if necessary. Applied to config locations.
	 * @param path the original file path
	 * @return the resolved file path
	 * @see org.springframework.core.env.Environment#resolveRequiredPlaceholders(String)
	 *
	 * 
	 */
	protected String resolvePath(String path) {
		return getEnvironment().resolveRequiredPlaceholders(path);
	}
~~~

