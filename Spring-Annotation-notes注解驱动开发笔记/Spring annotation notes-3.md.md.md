# 属性赋值-`@Value`赋值

有一个Person类：

```java
public class Person {
    private String name;
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

还有一个配置类：

```java
@Configuration
public class MainConfigOfPropertyValues {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

我们来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
        System.out.println("====================");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }
```

运行结果如下：

> ====================
> Person{name=‘null’, age=null}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person

我们可以看容器中的Person{name=‘null’, age=null}值都会默认值，

现在，我们就可以这样来写：

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;
    
    @Value("#{20-2}")
    private Integer age;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

这个时候，我们再来运行以上的测试方法，测试结果如下：这个时候，Person的属性就是可以获取到值了：

> ====================
> Person{name=‘张三’, age=18}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person



# 属性赋值-`@PropertySource`加载外部配置文件

我们为Person这个类再添加一个属性nickName:

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件【properties】中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;

    @Value("#{20-2}")
    private Integer age;

    private String nickName;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }


    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", nickName='" + nickName + '\'' +
                '}';
    }
}
```

我们再来写上一个person.properties文件：

```properties
person.nickName=小张三
```

在用xml文件配置的时候，我们是这样做的：

```xml
<context:property-placeholder location="classpath:person.properties"/>
```

现在，我们用注解的方式就可以这样来做：
我们要添加这样的一个注解：`@PropertySource`，查看源码的时候，我们可以发现，这个注解是一个可重复标注的注解，可多次标注，也可以在一个注解内添加外部配置文件位置的数组，我们也可以用PropertySources内部包含多个PropertySource ：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource {

	/**
	 * Indicate the name of this property source. If omitted, a name will
	 * be generated based on the description of the underlying resource.
	 * @see org.springframework.core.env.PropertySource#getName()
	 * @see org.springframework.core.io.Resource#getDescription()
	 */
	String name() default "";

	/**
	 * Indicate the resource location(s) of the properties file to be loaded.
	 * For example, {@code "classpath:/com/myco/app.properties"} or
	 * {@code "file:/path/to/file"}.
	 * <p>Resource location wildcards (e.g. *&#42;/*.properties) are not permitted;
	 * each location must evaluate to exactly one {@code .properties} resource.
	 * <p>${...} placeholders will be resolved against any/all property sources already
	 * registered with the {@code Environment}. See {@linkplain PropertySource above}
	 * for examples.
	 * <p>Each location will be added to the enclosing {@code Environment} as its own
	 * property source, and in the order declared.
	 */
	String[] value();

	/**
	 * Indicate if failure to find the a {@link #value() property resource} should be
	 * ignored.
	 * <p>{@code true} is appropriate if the properties file is completely optional.
	 * Default is {@code false}.
	 * @since 4.0
	 */
	boolean ignoreResourceNotFound() default false;

	/**
	 * A specific character encoding for the given resources, e.g. "UTF-8".
	 * @since 4.3
	 */
	String encoding() default "";

	/**
	 * Specify a custom {@link PropertySourceFactory}, if any.
	 * <p>By default, a default factory for standard resource files will be used.
	 * @since 4.3
	 * @see org.springframework.core.io.support.DefaultPropertySourceFactory
	 * @see org.springframework.core.io.support.ResourcePropertySource
	 */
	Class<? extends PropertySourceFactory> factory() default PropertySourceFactory.class;

}
```

`@PropertySources` ：内部可以指定多个`@PropertySource`

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface PropertySources {

	PropertySource[] value();

}
```

类似于xml文件配置的这一步：`<context:property-placeholder location="classpath:person.properties"/>`
我们要用`@PropertySource`这个注解来指定外部文件的位置：`@PropertySource(value = {"classpath:/person.properties"})`

```java
//使用@PropertySource读取外部配置文件中的key/value保存到运行的环境变量中
//加载完外部配置文件以后使用${}取出配置文件的值
@PropertySource(value = {"classpath:/person.properties"})
@Configuration
public class MainConfigOfPropertyValues {

    @Bean
    public Person person() {
        return new Person();
    }
}

```

然后，我们就可以用`@Value`，里面用${}就可以引用配置文件中的值了：

```java
public class Person {

    //使用@Value赋值
    //1.基本的数值
    //2.可以写SpEL: #{}
    //3.可以写${}：取出配置文件【properties】中的值（在运行环境变量里面的值）
    @Value("张三")
    private String name;

    @Value("#{20-2}")
    private Integer age;

    @Value("${person.nickName}")
    private String nickName;

    public Person() {
    }

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }


    public String getNickName() {
        return nickName;
    }

    public void setNickName(String nickName) {
        this.nickName = nickName;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", nickName='" + nickName + '\'' +
                '}';
    }
}
```

我们再来运行测试方法，运行结果如下：这个时候，nickName就有值了：

> ====================
> Person{name=‘张三’, age=18, nickName=‘小张三’}
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person
>

我们还可以用Environment里面的getProperty()方法来获取：

```java
@Test
    public void test01() {
        //1.创建IOC容器
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfPropertyValues.class);
        System.out.println("====================");

        Person person = (Person) applicationContext.getBean("person");
        System.out.println(person);

        Environment environment = applicationContext.getEnvironment();
        String property = environment.getProperty("person.nickName");
        System.out.println(property);

        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

测试结果如下：

> ====================
> Person{name=‘张三’, age=18, nickName=‘小张三’}
> 小张三
> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfigOfPropertyValues
> person
>

# 自动装配-`@Autowired`&`@Qualifier`&`@Primary`

在原来，我们就是使用`@Autowired`的这个注解来进行自动装配；
现在，我们有一个BookController 类：

```java
@Controller
public class BookController {

    @Autowired
    private BookService bookService;

}
```

还有一个BookService：

```java
@Service
public class BookService {
    @Autowired
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}
```

还有一个BookDao：为了在自动装配的是哪一个，我们给这个BookDao加上一个标识属性：lable ，如果是通过包扫描到IOC容器中，标识为1，如果是在配置类里面通过`@Bean`装配的标识为2：

```java
//在IOC容器里面默认就是类名的首字母小写
@Repository
public class BookDao {

    private String lable = "1";

    public String getLable() {
        return lable;
    }

    public void setLable(String lable) {
        this.lable = lable;
    }

    @Override
    public String toString() {
        return "BookDao{" +
                "lable='" + lable + '\'' +
                '}';
    }
}
```

现在，我们有一个配置类：

```java
/**
 * 自动装配：
 *      Spring利用依赖注入（DI）完成对IOC容器中各个组件的依赖关系赋值
 * 1) @Autowired：自动注入
 *      (1)默认优先按照类型去容器中去找对应的组件：applicationContext.getBean(BookDao.class);如果找到了则进行赋值；
 *      public class BookService {
 *          @Autowired
 *          BookDao bookDao;
 *      }
 */
@Configuration
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller"})
public class MainConfigOfAutowired {

    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

}
```

我们再来写上一个测试类：为了区分是装配的是通过包扫描的方式还是通过在配置类里面进行装配的：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        BookService bookService = applicationContext.getBean(BookService.class);
        System.out.println(bookService);
    }

```

测试结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

自动装配：
Spring利用依赖注入（DI）完成对IOC容器中各个组件的依赖关系赋值
1）`@Autowired`：自动注入
（1）默认优先按照类型去容器中去找对应的组件：applicationContext.getBean(BookDao.class);如果找到了则进行赋值；
（2）如果找到了多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
applicationContext.getBean(“bookDao”)
（3）`@Qualifier("bookDao")`：使用 `@Qualifier` 指定需要装配的组件的id，而不是使用属性名
（4）自动装配默认一定要将属性赋值好，没有就会报错，可以使用`@Autowired(required = false);`来设置为非必须的
（5）可以利用`@Primary`：让Spring在进行自动装配的时候，默认使用首选的bean
也可以继续使用`@Qualifier("bookDao")`来明确指定需要装配的bean的名字

```java
      public class BookService {
          @Autowired
          BookDao bookDao;
      }

```

如果我们想要装配bookDao2：我们就把属性名改成bookDao2就可以了：

```java
      public class BookService {
          @Autowired
          BookDao bookDao2;
      }

```

我们就可以这样来写：

```java
@Service
public class BookService {
    @Autowired
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}

```

这个时候，我们再来进行测试：这个时候，如果找找到了多个相同类型的bean，那么就是装配的bookDao2了

> BookService{bookDao=BookDao{lable=‘2’}}

------

虽然，我们在属性名写了bookDao2，但是，我就想要装配bookDao;实际上也是可以的：我们可以使用 `@Qualifier`这个注解
`@Qualifier("bookDao")`：使用 `@Qualifier` 指定需要装配的组件的id，而不是使用属性名

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}

```

这个时候，我们再来测试：发现又装配了bookDao了

> BookService{bookDao=BookDao{lable=‘1’}}

------

而当我们的容器里面没有一个对应的bean的时候，这个时候，就是会报一个错 ：

> org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name ‘bookService’: Unsatisfied dependency expressed through field ‘bookDao2’; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type ‘com.ldc.dao.BookDao’ available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Qualifier(value=bookDao), @org.springframework.beans.factory.annotation.Autowired(required=true)}
> 

那可不可以在使用自动装配的时候，这个bean不是必须的呢？如果容器里面没有对应的bean，我就不装配，实际上也是可以的：我们要`@Autowired`注解里面添加`required = false`这个属性`@Autowired(required = false)`

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来运行测试方法，测试结果为：

> BookService{bookDao=null}

我们还可以利用一个注解来让Spring在自动装配的时候，首选装配哪个bean：`@Primary`

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Primary {

}
```

```java
@Configuration
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller"})
public class MainConfigOfAutowired {

    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

}

```

当然明确指定的注解也是不能用了：`@Qualifier("bookDao")`

```java
@Service
public class BookService {

    //@Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao2;

    public void print() {
        System.out.println(bookDao2);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao2 +
                '}';
    }
}
```

这个时候，我们再来运行测试方法，测试结果如下：这个时候，Spring就首选装配了标注了`@Primary`注解的bean：

> BookService{bookDao=BookDao{lable=‘2’}}

------

我们把装配的时候的属性名也变一下：

```java
@Service
public class BookService {

    //@Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}
```

我们再来看看测试结果：还是装配了标注了`@Primary`注解的bean

> BookService{bookDao=BookDao{lable=‘2’}}

如果是使用了`@Qualifier("bookDao")`明确指定了的：那还是按照明确指定的bean来进行装配

```java
@Service
public class BookService {

    @Qualifier("bookDao")
    @Autowired(required = false)
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

测试结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

------

# 自动装配-`@Resource`&`@Inject`

2）Spring还支持使用`@Resource`(JSR250)和`@Inject`(JSR330)

- （1）`@Resource`：可以和`@Autowired`一样实现自动的装配，默认是按照组件的名称来进行装配,没有支持`@Primary`
也没有支持和`@Autowired(required = false)`一样的功能
- （2）`@Inject`：需要导入javax.inject的包,和`@Autowired`的功能一样,没有支持和`@Autowired(required = false)`一样的功能

`AutowiredAnnotationBeanPostProcessor`是用来解析完成自动装配的功能的

`@Autowired`：是Spring定义的
`@Resource` 和 `@Inject`都是java的规范

------

`@Resource` 的使用

```java
@Service
public class BookService {

    @Resource
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}
```

我们来运行测试方法:

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        BookService bookService = applicationContext.getBean(BookService.class);
        System.out.println(bookService);
    }

```

运行结果如下：

> BookService{bookDao=BookDao{lable=‘1’}}

我们也可以用`@Resource`注解里面的name属性来指定装配哪一个：

```java
@Service
public class BookService {

    @Resource(name = "bookDao2")
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}

```

这个时候的测试结果如下：

> BookService{bookDao=BookDao{lable=‘2’}}

------

@Inject`的使用
导入jar包：

```xml
     <!-- https://mvnrepository.com/artifact/javax.inject/javax.inject -->
     <dependency>
         <groupId>javax.inject</groupId>
         <artifactId>javax.inject</artifactId>
         <version>1</version>
     </dependency>

```

我们就可以这样来使用：

```java
@Service
public class BookService {

    @Inject
    private BookDao bookDao;

    public void print() {
        System.out.println(bookDao);
    }

    @Override
    public String toString() {
        return "BookService{" +
                "bookDao=" + bookDao +
                '}';
    }
}
```

我们可以来进行测试：发现也是可以支持`@Primary`的功能的

> BookService{bookDao=BookDao{lable=‘2’}}

# 自动装配-方法、构造器位置的自动装配

我们从`@Autowired`这个注解点进去看一下源码：
我们可以发现这个注解可以标注的位置有：构造器，参数，方法，属性；都是从容器中来获取参数组件的值

```java
@Target({ElementType.CONSTRUCTOR, ElementType.METHOD, ElementType.PARAMETER, ElementType.FIELD, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Autowired {

	/**
	 * Declares whether the annotated dependency is required.
	 * <p>Defaults to {@code true}.
	 */
	boolean required() default true;

}
```

1. `@Autowired`注解标注在方法上：用的最多的方式就是在`@Bean`注解标注的方法的参数，这个参数就是会从容器中获取，在这个方法的参数前面可以加上`@Autowired`注解，也可以省略，默认是不写`@Autowired`，都能自动装配；

   ```java
   @Component
   public class Boss {
   
       private Car car;
   
       public Car getCar() {
           return car;
       }
   
       @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
       //方法使用的参数，自定义类型的值从IOC容器里面进行获取
       public void setCar(Car car) {
           this.car = car;
       }
   
       @Override
       public String toString() {
           return "Boss{" +
                   "car=" + car +
                   '}';
       }
   }
   ```

   我们来进行测试：

   ```java
       @Test
       public void test01() {
           //1.创建IOC容器
           AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
           Boss boss = applicationContext.getBean(Boss.class);
           Car car = applicationContext.getBean(Car.class);
           System.out.println(car);
           System.out.println(boss);
       }
   
   ```

   测试结果如下：

   > com.ldc.bean.Car@69930714
   > Boss{car=com.ldc.bean.Car@69930714}

2. `@Autowired`注解标注在构造器上，默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作，构造器要用的组件，也都是从容器中来获取：
   注意：如果组件只有一个有参的构造器，这个有参的构造器的 `@Autowired`注解可以省略，参数位置的组件还是可以自动从容器中获取

   ```java
   //默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
   @Component
   public class Boss {
   
       private Car car;
   
       //构造器要用的组件，也都是从容器中来获取
       @Autowired
       public Boss(Car car) {
           this.car = car;
           System.out.println("Boss的有参构造器"+car);
       }
   
       public Car getCar() {
           return car;
       }
   
       @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
       //方法使用的参数，自定义类型的值从IOC容器里面进行获取
       public void setCar(Car car) {
           this.car = car;
       }
   
       @Override
       public String toString() {
           return "Boss{" +
                   "car=" + car +
                   '}';
       }
   }
   ```

   测试结果：

   > Boss的有参构造器com.ldc.bean.Car@1460a8c0
   > com.ldc.bean.Car@1460a8c0
   > Boss{car=com.ldc.bean.Car@1460a8c0}

3. `@Autowired`注解标注在参数上：效果是一样的

   ```java
   //默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
   @Component
   public class Boss {
   
       private Car car;
   
       //构造器要用的组件，也都是从容器中来获取
   
       //我们也可以标注在参数上效果是一样的
       public Boss(@Autowired Car car) {
           this.car = car;
           System.out.println("Boss的有参构造器"+car);
       }
   
       public Car getCar() {
           return car;
       }
   
       @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
       //方法使用的参数，自定义类型的值从IOC容器里面进行获取
       public void setCar(Car car) {
           this.car = car;
       }
   
       @Override
       public String toString() {
           return "Boss{" +
                   "car=" + car +
                   '}';
       }
   }
   
   ```

   ------

   

还一种情况，如果Boss这个类里面只有一个有参构造器，在构造器里面不加`@Autowired`注解也是可以的：

```java
//默认加在IOC容器中的组件，容器启动的时候会调用无参构造器创建对象，再进行初始化赋值操作
@Component
public class Boss {

    private Car car;

    //构造器要用的组件，也都是从容器中来获取

    //我们也可以标注在参数上效果是一样的
    public Boss(Car car) {
        this.car = car;
        System.out.println("Boss的有参构造器"+car);
    }

    public Car getCar() {
        return car;
    }

    @Autowired //标注在方法上，Spring容器在创建当前对象的时候，就会调用当前方法完成赋值；
    //方法使用的参数，自定义类型的值从IOC容器里面进行获取
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}
```

------

还有一种用法：
现在有一个Color类，里面有一个Car属性：

```java
public class Color {
    private Car car;

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Color{" +
                "car=" + car +
                '}';
    }
}
```

我们在配置类里面来进行配置：

```java
@Configuration
@ComponentScan({"com.ldc.service","com.ldc.dao","com.ldc.controller","com.ldc.bean"})
public class MainConfigOfAutowired {
    @Primary
    @Bean("bookDao2")
    public BookDao bookDao() {
        BookDao bookDao = new BookDao();
        bookDao.setLable("2");
        return bookDao;
    }

    //@Bean标注的方法创建对象的时候，方法参数的值从容器中获取
    @Bean
    public Color color(Car car) {
        Color color = new Color();
        color.setCar(car);
        return color;
    }

}
```

我们来测试一下:

```java
@Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
        Boss boss = applicationContext.getBean(Boss.class);
        Car car = applicationContext.getBean(Car.class);
        System.out.println(car);
        System.out.println(boss);
        Color color = applicationContext.getBean(Color.class);
        System.out.println(color);
    }
```

测试结果：

> com.ldc.bean.Car@6e75aa0d
> Boss{car=com.ldc.bean.Car@6e75aa0d}
> Color{car=com.ldc.bean.Car@6e75aa0d}



# 自动装配-Aware注入Spring底层组件&原理

4）自定义组件想要使用Spring容器底层的一些组件（ApplicationContext、BeanFactory…）
自定义组件实现xxxAware接口就可以实现，在创建对象的时候，会调用接口规定的方法注入相关的组件;
把Spring底层的一些组件注入到自定义的bean中；

xxxAware等这些都是利用后置处理器的机制，比如ApplicationContextAware 是通过ApplicationContextAwareProcessor来进行处理的；

例如之前写的这个,实现ApplicationContextAware 接口，里面有一个setApplicationContext方法：

```java
@Component
public class Dog implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public Dog() {
        System.out.println("Dog...Contructor...");
    }

    //在对象创建并赋值之后调用
    @PostConstruct
    public void init() {
        System.out.println("Dog...@PostConstruct...");
    }

    //在对象创建并赋值之后调用
    @PreDestroy
    public void destroy() {
        System.out.println("Dog...@PreDestroy...");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}

```

Aware 是一个总接口：

```java
/**
 * Marker superinterface indicating that a bean is eligible to be
 * notified by the Spring container of a particular framework object
 * through a callback-style method. Actual method signature is
 * determined by individual subinterfaces, but should typically
 * consist of just one void-returning method that accepts a single
 * argument.
 *
 * <p>Note that merely implementing {@link Aware} provides no default
 * functionality. Rather, processing must be done explicitly, for example
 * in a {@link org.springframework.beans.factory.config.BeanPostProcessor BeanPostProcessor}.
 * Refer to {@link org.springframework.context.support.ApplicationContextAwareProcessor}
 * and {@link org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory}
 * for examples of processing {@code *Aware} interface callbacks.
 *
 * @author Chris Beams
 * @since 3.1
 */
public interface Aware {

}

```

它有这么多的子接口：

![image-20210406104441579](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210406104441579.png)

来挑几个看看：

```java
@Component
public class Red implements ApplicationContextAware, BeanNameAware , EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        //如果我们后来要用，我们就用一个变量来存起来
        System.out.println("传入的IOC容器："+applicationContext);
        this.applicationContext = applicationContext;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的名字："+name);
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String resolveStringValue = resolver.resolveStringValue("你好${os.name} 我是#{20*18}");
        System.out.println("解析的字符串"+resolveStringValue);
    }
}

```

这个时候，我们再来测试：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfAutowired.class);
    }

```

测试结果：

> 当前bean的名字：red
> 解析的字符串你好Windows 7 我是360
> 传入的IOC容器：org.springframework.context.annotation.AnnotationConfigApplicationContext@4141d797: startup date [Tue Jan 15 15:29:08 CST 2019]; root of context hierarchy
>

# 自动装配-`@Profile`环境搭建

`@Profile`注解源码：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}
```

引入数据源和mysql驱动：

```xml
      <!--数据源-->
      <!-- https://mvnrepository.com/artifact/c3p0/c3p0 -->
      <dependency>
          <groupId>c3p0</groupId>
          <artifactId>c3p0</artifactId>
          <version>0.9.1.2</version>
      </dependency>
      <!--数据库驱动-->
      <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
      <dependency>
          <groupId>mysql</groupId>
          <artifactId>mysql-connector-java</artifactId>
          <version>5.1.44</version>
      </dependency>

```

再写一个dbconfig.properties

```properties
db.user=root
db.password=12358
db.driverClass=com.mysql.jdbc.Driver
```

这个时候，我们就可以这样来进行配置：记得加上`@PropertySource("classpath:/dbconfig.properties")`告诉配置文件的位置

```java
/**
 * Profile:
 *      Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
 * 开发环境，测试环境，生产环境
 * 我们以切换数据源为例：
 * 数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）
 */
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

上面涉及到三种获取配置文件中的值：

1. 直接通过属性上面加上`@Value("${db.user}")`

   ```java
       @Value("${db.user}")
       private String user;
   
   ```

2. 在参数上面使用`@Value("${db.password}")` public DataSource dataSourceTest(@Value("${db.password}") String pwd)

   ```java
    	@Bean("testDataSource")
       public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
           ComboPooledDataSource dataSource = new ComboPooledDataSource();
           dataSource.setUser(user);
           dataSource.setPassword(pwd);
           dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
           dataSource.setDriverClass(driverClass);
           return dataSource;
       }
   
   ```

3. 实现EmbeddedValueResolverAware接口，在setEmbeddedValueResolver(StringValueResolver resolver)方法里面进行获取：里面使用 resolver.resolveStringValue("${db.driverClass}")方法解析，返回赋值给成员属性private StringValueResolver resolver;

   ```java
       @Override
       public void setEmbeddedValueResolver(StringValueResolver resolver) {
           this.resolver = resolver;
           driverClass = resolver.resolveStringValue("${db.driverClass}");
       }
   
   ```

这个时候，我们来测试一下，看看容器里面DataSource类型的bean有哪些：

```java
    @Test
    public void test01() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

测试结果：

> testDataSource
> devDataSource
> prodDataSource

# 自动装配-`@Profile`根据环境注册bean

`@Profile`:
Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
开发环境，测试环境，生产环境
我们以切换数据源为例：
数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）

`@Profile`: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件
1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了default，那么这个bean默认会被注册到容器中
2）`@Profile` 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效
3）没有标注环境标识的bean，在任何环境都是加载的

现在，我来用`@Profile`注解只激活哪一个数据源：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

	/**
	 * The set of profiles for which the annotated component should be registered.
	 */
	String[] value();

}
```

现在，我们来给数据源加上标识：

```java
/**
 * @Profile:
 *      Spring为我们提供的可以根据当前的环境，动态的激活和切换一系列组件的功能；
 * 开发环境，测试环境，生产环境
 * 我们以切换数据源为例：
 * 数据源：开发环境中(用的是A数据库)、测试环境(用的是B数据库)、而生产环境（用的又是C数据库）
 * @Profile: 指定组件在哪一个环境的情况下才能被注册到容器中，不指定任何环境都能被注册这个组件
 * 1）加了环境标识的bean，只有这个环境被激活的时候才能注册到容器中，默认是default环境，如果指定了
 * default，那么这个bean默认会被注册到容器中
 * 2）@Profile 写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效
 * 3）没有标注环境标识的bean，在任何环境都是加载的
 */
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Profile("test")
    @Bean
    public Yellow yellow() {
        return new Yellow();
    }

    @Profile("test")
    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

现在，我们去测试方法里面去指定用哪一个：

1. 使用命令行动态参数的方式：在虚拟机参数的位置加载-Dspring.profile.active=test

   ![image-20210406104847287](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210406104847287.png)

   这个时候，我们再来运行这个测试方法：

   ```java
       //1.使用命令行动态参数的方式
       @Test
       public void test01() {
           ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfProfile.class);
           String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
           Stream.of(beanNamesForType).forEach(System.out::println);
       }
   
   ```

   运行结果：

   > testDataSource

   我们再切换到开发的环境来试试：

   这个时候的运行结果为：

   > devDataSource

2. 利用代码的方式来实现激活某种环境，这个时候，我们在创建IOC容器的时候，必须要用无参构造器：

   ```java
       @Test
       public void test01() {
           AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
           //1)使用无参构造器来创建applicationContext对象
           //2)设置需要激活的环境
           applicationContext.getEnvironment().setActiveProfiles("dev");
           //3)加载主配置类
           applicationContext.register(MainConfigOfProfile.class);
           //4)启动刷新容器
           applicationContext.refresh();
   
           String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
           Stream.of(beanNamesForType).forEach(System.out::println);
       }
   
   ```

   运行结果：

   > devDataSource

------

`@Profile`写在配置类上，只有是指定的环境，整个配置类里面的所有配置才能开始生效:

```java
@Profile("test")
@Configuration
@PropertySource("classpath:/dbconfig.properties")
public class MainConfigOfProfile implements EmbeddedValueResolverAware {

    @Value("${db.user}")
    private String user;

    private StringValueResolver resolver;

    private String driverClass;

    @Profile("test")
    @Bean
    public Yellow yellow() {
        return new Yellow();
    }


    @Bean("testDataSource")
    public DataSource dataSourceTest(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/test");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("dev")
    @Bean("devDataSource")
    public DataSource dataSourceDev(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/dev");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Profile("prod")
    @Bean("prodDataSource")
    public DataSource dataSourceProd(@Value("${db.password}") String pwd) throws PropertyVetoException {
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        dataSource.setUser(user);
        dataSource.setPassword(pwd);
        dataSource.setJdbcUrl("jdbc:mysql://localhost:3306/prod");
        dataSource.setDriverClass(driverClass);
        return dataSource;
    }

    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        this.resolver = resolver;
        driverClass = resolver.resolveStringValue("${db.driverClass}");
    }
}

```

我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext();
        //1)使用无参构造器来创建applicationContext对象
        //2)设置需要激活的环境
        applicationContext.getEnvironment().setActiveProfiles("dev");
        //3)加载主配置类
        applicationContext.register(MainConfigOfProfile.class);
        //4)启动刷新容器
        applicationContext.refresh();

        String[] beanNamesForType = applicationContext.getBeanNamesForType(DataSource.class);
        Stream.of(beanNamesForType).forEach(System.out::println);
    }

```

这个时候，就没有一个配置是生效的；







# IOC-小结

**容器**：

* AnnotationConfigApplicationContext：
  * 配置类
  * 包扫描
* 组件添加：
  * @ComponentScan
  * @Bean
    * 指定初始化销毁
    * 初始化其他方式
      * （1）InitializingBean（初始化设置值之后）
      * （2）InitializingBean（初始化设置值之后）
      * （3）JSR250：@PostConstruct、@PreDestroy
    * BeanPostProcessor
  * @Configuration
  * @Component
  * @Service
  * @Controller
  * @Repository
  * @Conditional
  * @Primary
  * @Lazy
  * @Scope
  * @Import
  * ImportSelector
  * 工厂模式
    * FactoryBean：&beanName获取Factory本身
* 组件赋值
  * @Value
  * @Autowired
    * （1）@Qualifier
    * （2）@Resources（JSR250）
    * （3）@Inject（JSR330，需要导入javax.inject）
  * @PropertySource
  * @PropertySources
  * @Profile
    * （1）Environment
    * （2）-Dspring.profiles.active=test
* 组件注入
  * 方法参数
  * 构造器注入
  * xxxAware
  * ApplicationContextAware
  * ApplicationContextAwareProcessor

```java
if (bean instanceof Aware) {
			if (bean instanceof EnvironmentAware) {
				((EnvironmentAware) bean).setEnvironment(this.applicationContext.getEnvironment());
			}
			if (bean instanceof EmbeddedValueResolverAware) {
				((EmbeddedValueResolverAware) bean).setEmbeddedValueResolver(this.embeddedValueResolver);
			}
			if (bean instanceof ResourceLoaderAware) {
				((ResourceLoaderAware) bean).setResourceLoader(this.applicationContext);
			}
			if (bean instanceof ApplicationEventPublisherAware) {
				((ApplicationEventPublisherAware) bean).setApplicationEventPublisher(this.applicationContext);
			}
			if (bean instanceof MessageSourceAware) {
				((MessageSourceAware) bean).setMessageSource(this.applicationContext);
			}
			if (bean instanceof ApplicationContextAware) {
				((ApplicationContextAware) bean).setApplicationContext(this.applicationContext);
			}
		}

```

