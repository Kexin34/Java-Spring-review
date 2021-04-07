

# 组件注册-`@Lazy-bean`懒加载

* 懒加载：是专门针对于单实例的bean的
  * 单实例的bean：默认是在容器启动的时候创建对象；
  * 懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化
    当我们还没有配置懒加载的时候：作用域为单例的bean，默认是在容器启动的时候创建实例对象


------

当我们还没有配置懒加载的时候：作用域为单例的bean，默认是在容器启动的时候创建实例对象

```java
@Configuration
public class MainConfig2 {

    /**
     * 懒加载：是专门针对于单实例的bean的
     *       1.单实例的bean：默认是在容器启动的时候创建对象；
     *       2.懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化
     * @return
     */
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }
}

```

测试：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
    }

```

测试结果：

> 给IOC容器中添加Person…
> IOC容器创建完成…

从上面的测试结果，我们可以看出：**作用域为单例的bean，默认是在容器启动的时候创建实例对象**

------

而现在，我加上懒加载的注解：`@Lazy`

```java
@Configuration
public class MainConfig2 {

    /**
     * 懒加载：是专门针对于单实例的bean的
     *       1.单实例的bean：默认是在容器启动的时候创建对象；
     *       2.懒加载：容器启动的时候，不创建对象，而是在第一次使用（获取）Bean的时候来创建对象，并进行初始化
     * @return
     */
    @Lazy
    @Bean("person")
    public Person person() {
        System.out.println("给IOC容器中添加Person...");
        return new Person();
    }

}

```

我们再来运行以下的测试方法：

```java
    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
    }

```

这个时候，运行结果如下：

> IOC容器创建完成…

我们可以看到，这个时候，只有IOC容器启动了，而作用域为单例的bean并没有被创建；

而当我们第一次获取的时候，我们再来看看运行结果：

```java    @Test
    public void test02() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("IOC容器创建完成...");
        Person person = (Person) applicationContext.getBean("person");
    }
```

运行结果：

> IOC容器创建完成…
> 给IOC容器中添加Person…

从运行结果，我们可以看到，当我们调用了getBean()方法获取的时候，bean的实例对象才会被创建；而且只会被创建一次；作用域还是单实例的！

------

# 组件注册-`@Conditional`-按照条件注册bean

现在有两个bean： bill 和 linus ，现在我们想按照操作系统的不同来选择是否在IOC容器里面注册bean：

> 现在下面的两个bean注册到IOC容器是要条件的：
> 1.如果系统是windows，给容器注册(“bill”)
> 1.如果系统是linux，给容器注册(“linus”)

```java
@Configuration
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}

```

我们可以看到这个注解：value值传的是实现了Condition这个接口的类的数组

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition}s that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}

```

这是我们写的两个条件：

```java
/**
 * 判断操作系统是否为Linux系统
 */
public class LinuxCondition implements Condition {
    /**
     *  ConditionContext : 判断条件能使用的上下文(环境)
     *  AnnotatedTypeMetadata : 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断是否为Linux系统
        //1.能获取到IOC容器里面的BeanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();

        //2.获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //4.获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        //获取操作系统
        String property = environment.getProperty("os.name");
        if (property.contains("linux")) {
            return true;
        }
        return false;
    }
}

```

------

```java
/**
 * 判断操作系统是否为Windows系统
 */
public class WindowsCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //获取操作系统
        String property = environment.getProperty("os.name");
        if (property.contains("Windows")) {
            return true;
        }
        return false;
    }
}

```

两个条件写好了之后，我们给容器注册bean就可以写了：

```java
@Configuration
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */
    @Conditional({WindowsCondition.class})
    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}

```

这个时候，我们就可以来测试一下：因为当前使用的环境是Windows 7 系统，所以只会出现bill这一个bean会被注册到IOC容器当中：

```java
@Test
    public void test03() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);

        //我们可以获取当前的操作系统是什么：
        Environment environment = applicationContext.getEnvironment();
        //动态的获取环境变量的值：Windows 7
        String property = environment.getProperty("os.name");
        System.out.println(property);

        //获取IOC容器类型为Person类型的bean的名字一共有哪些
        String[] definitionNames = applicationContext.getBeanNamesForType(Person.class);
        for (String name : definitionNames) {
            System.out.println(name);
        }
        //我们也可以来获取类型为Person类型的对象
        Map<String, Person> persons = applicationContext.getBeansOfType(Person.class);
        persons.values().forEach(System.out::println);
    }

```

测试结果如下：

> Windows 7
> bill
> Person{name=‘Bill Gates’, age=62}

而对于Linux系统，我们也可以在运行环境参数里面进行模拟：

![image-20210407120026397](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120026397.png)

这个时候，我们再来运行测试类方法，结果如下：

> linux
> linus
> Person{name=‘linus’, age=48}

实际上，在这个条件类里面，我们还可以做很多的事情：比如说可以判断容器中bean的注册情况，也可以给容器中注册bean

```java
/**
 * 判断操作系统是否为Linux系统
 */
public class LinuxCondition implements Condition {
    /**
     *  ConditionContext : 判断条件能使用的上下文(环境)
     *  AnnotatedTypeMetadata : 注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //判断是否为Linux系统
        //1.能获取到IOC容器里面的BeanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();

        //2.获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        //3.能获取当前环境的信息
        Environment environment = context.getEnvironment();
        //4.获取到bean定义的注册类
        BeanDefinitionRegistry registry = context.getRegistry();

        //获取操作系统
        String property = environment.getProperty("os.name");

        //可以判断容器中bean的注册情况，也可以给容器中注册bean
        boolean definition = registry.containsBeanDefinition("person");
        
        if (property.contains("linux")) {
            return true;
        }
        return false;
    }
}

```

`@Conditional` 的这个注解还可以标注在类上，对容器中的组件进行统一设置：满足当前条件，这个类中配置的所有bean注册才能生效

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */

    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

}

```

现在的运行结果如下：一个bean都没有， 只打印的一个操作系统的值

> linux

# 组件注册-`@Import`-给容器中快速导入一个组件

我们先写上一个类：

```java
public class Color {
}

```

当我们没有添加`@Import`注解的时候：我们打印容器里面所有的组件

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

这个时候，是没有Color 的这个bean在IOC容器里面

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> bill
>

------

而当我们使用了 `@Import(Color.class)`注解之后，我们可以看到IOC容器里面就有这个组件了，id默认就是这个组件的全类名：

```java'
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import(Color.class)
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */

    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     */
}

```

我们可以看到现在 IOC 容器有这个com.ldc.bean.Color的这个bean的实例了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> bill

`@Import`这个注解也是可以导入多个组件的：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration}, {@link ImportSelector}, {@link ImportBeanDefinitionRegistrar}
	 * or regular component classes to import.
	 */
	Class<?>[] value();

}

```

这个时候，我们可以再来写上一个Red类：

```java
public class Red {
}

```

我们就可以这样用一个数组来导入多个组件：

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     */
}

```

测试结果：这个时候IOC容器里面就有Color和Red这两个bean的实例了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> bill
>

# 组件注册-`@Import`-使用`ImportSelector`

ImportSelector是一个接口：**返回需要的组件的全类名的数组；**

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);

}

```

现在，我们就来自定义逻辑返回需要导入的组件，需要实现 ImportSelector 这个接口：

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[0];
    }
}

```

现在，我们就可以这样来写：@Import({Color.class,Red.class,MyImportSelector.class})

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     */
}

```

我们打断点运行这个测试方法：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

![image-20210407120328520](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120328520.png)

也就是对应着这个类里面所有注解的信息：

![image-20210407120346214](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120346214.png)

![image-20210407120401150](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120401150.png)

![image-20210407120416775](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120416775.png)

![image-20210407120428440](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120428440.png)

------

现在，我们再写两个类：

```java
public class Blue {
}

```

```java
public class Yellow {
}

```

这个时候，我们就可以这样来在IOC容器里面加入bean,纳入IOC容器的管理：

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector {
    //返回值就是要导入到容器中的组件的全类名
    //AnnotationMetadata ：当前标注@Import注解的类的所有注解信息
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //方法不要返回null值
        return new String[]{"com.ldc.bean.Blue","com.ldc.bean.Yellow"};
    }
}
```

现在，我们再来运行测试方法，看看IOC容器里面有哪些组件：这个时候Blue和Yellow也就进来了

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill

# 组件注册- `@Import`-使用`ImportBeanDefinitionRegistrar`

ImportBeanDefinitionRegistrar是一个接口：

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}

```

我们可以再定义一个类：

```java
public class RainBow {
}

```

这个时候，我们就可以自定义一个MyImportBeanDefinitionRegistrar类并且实现ImportBeanDefinitionRegistrar这个接口：

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    /**
     *
     * AnnotationMetadata 当前类的注解信息
     * BeanDefinitionRegistry BeanDefinition注册类
     *
     * 我们把所有需要添加到容器中的bean通过BeanDefinitionRegistry里面的registerBeanDefinition方法来手动的进行注册
     */
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        //判断IOC容器里面是否含有这两个组件
        boolean definition = registry.containsBeanDefinition("com.ldc.bean.Red");
        boolean definition2 = registry.containsBeanDefinition("com.ldc.bean.Blue");
        //如果有的话，我就把RainBow的bean的实例给注册到IOC容器中
        if (definition && definition2) {
            //指定bean的定义信息，参数里面指定要注册的bean的类型：RainBow.class
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(RainBow.class);
            //注册一个bean，并且指定bean名
            registry.registerBeanDefinition("rainBow", rootBeanDefinition );
        }
    }
}

```

![image-20210407120627834](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407120627834.png)

同样，我们在@Import的这个注解里面添加我们自定义的MyImportBeanDefinitionRegistrar

> @Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     *      （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中
     */
}

```

现在，我们来进行测试：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

测试结果如下：这个时候，我们发现rainBow的这个组件也进来了：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> rainBow

以上就是@Import的三种用法：

* @Import[快速的给容器中导入一个组件]
  * （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
  * （2）、 ImportSelector ：返回需要的组件的全类名的数组；
  * （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中

ImportSelector 这种用法在SpringBoot里面用的非常的多；



# 组件注册-使用FactoryBean注册组件

FactoryBean：

```java
public interface FactoryBean<T> {

	/**
	 * Return an instance (possibly shared or independent) of the object
	 * managed by this factory.
	 * <p>As with a {@link BeanFactory}, this allows support for both the
	 * Singleton and Prototype design pattern.
	 * <p>If this FactoryBean is not fully initialized yet at the time of
	 * the call (for example because it is involved in a circular reference),
	 * throw a corresponding {@link FactoryBeanNotInitializedException}.
	 * <p>As of Spring 2.0, FactoryBeans are allowed to return {@code null}
	 * objects. The factory will consider this as normal value to be used; it
	 * will not throw a FactoryBeanNotInitializedException in this case anymore.
	 * FactoryBean implementations are encouraged to throw
	 * FactoryBeanNotInitializedException themselves now, as appropriate.
	 * @return an instance of the bean (can be {@code null})
	 * @throws Exception in case of creation errors
	 * @see FactoryBeanNotInitializedException
	 */
	T getObject() throws Exception;

	/**
	 * Return the type of object that this FactoryBean creates,
	 * or {@code null} if not known in advance.
	 * <p>This allows one to check for specific types of beans without
	 * instantiating objects, for example on autowiring.
	 * <p>In the case of implementations that are creating a singleton object,
	 * this method should try to avoid singleton creation as far as possible;
	 * it should rather estimate the type in advance.
	 * For prototypes, returning a meaningful type here is advisable too.
	 * <p>This method can be called <i>before</i> this FactoryBean has
	 * been fully initialized. It must not rely on state created during
	 * initialization; of course, it can still use such state if available.
	 * <p><b>NOTE:</b> Autowiring will simply ignore FactoryBeans that return
	 * {@code null} here. Therefore it is highly recommended to implement
	 * this method properly, using the current state of the FactoryBean.
	 * @return the type of object that this FactoryBean creates,
	 * or {@code null} if not known at the time of the call
	 * @see ListableBeanFactory#getBeansOfType
	 */
	Class<?> getObjectType();

	/**
	 * Is the object managed by this factory a singleton? That is,
	 * will {@link #getObject()} always return the same object
	 * (a reference that can be cached)?
	 * <p><b>NOTE:</b> If a FactoryBean indicates to hold a singleton object,
	 * the object returned from {@code getObject()} might get cached
	 * by the owning BeanFactory. Hence, do not return {@code true}
	 * unless the FactoryBean always exposes the same reference.
	 * <p>The singleton status of the FactoryBean itself will generally
	 * be provided by the owning BeanFactory; usually, it has to be
	 * defined as singleton there.
	 * <p><b>NOTE:</b> This method returning {@code false} does not
	 * necessarily indicate that returned objects are independent instances.
	 * An implementation of the extended {@link SmartFactoryBean} interface
	 * may explicitly indicate independent instances through its
	 * {@link SmartFactoryBean#isPrototype()} method. Plain {@link FactoryBean}
	 * implementations which do not implement this extended interface are
	 * simply assumed to always return independent instances if the
	 * {@code isSingleton()} implementation returns {@code false}.
	 * @return whether the exposed object is a singleton
	 * @see #getObject()
	 * @see SmartFactoryBean#isPrototype()
	 */
	boolean isSingleton();

}

```

首先，我们创建一个类，并且实现 FactoryBean 接口：

```java
//创建一个Spring定义的FactoryBean
public class ColorFactoryBean implements FactoryBean<Color> {

    //返回一个Color对象，这个对象会添加到容器中
    @Override
    public Color getObject() throws Exception {
        System.out.println("ColorFactoryBean...getBean...");
        return new Color();
    }

    //返回的类型
    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    //控制是否为单例
    // true：表示的就是一个单实例，在容器中保存一份
    // false:多实例，每次获取都会创建一个新的bean
    @Override
    public boolean isSingleton() {
        return true;
    }
}

```

这个时候，我们在配置类里面进行配置：可以看到表面上我们装配的是ColorFactoryBean这个类型，但是实际上我们装配的是Color这个bean的实例：

```java
@Configuration
//满足当前条件，这个类中配置的所有bean注册才能生效
@Conditional({WindowsCondition.class})
//快速导入组件，id默认是组件的全类名
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class})
public class MainConfig2 {

    /**
     * @Conditional:是SpringBoot底层大量使用的注解，按照一定的条件来进行判断，满足条件 给容器注册bean
     */

    /**
     *  现在下面的两个bean注册到IOC容器是要条件的：
     *  1.如果系统是windows，给容器注册("bill")
     *  1.如果系统是linux，给容器注册("linus")
     * @return
     */


    @Bean("bill")
    public Person person01() {
        return new Person("Bill Gates",62);
    }

    @Conditional({LinuxCondition.class})
    @Bean("linus")
    public Person person02() {
        return new Person("linus",48);
    }

    /**
     * 给容器中注册组件：
     * 1）、扫描+组件标注注解（@Controller/@Service/@Repository/@Component）
     * 【局限于要求是自己写的类，如果导入的第三方没有添加这些注解，那么就注册不上了】
     *
     * 2）、@Bean[导入的第三方包里面的组件]
     * 3）、@Import[快速的给容器中导入一个组件]
     *      （1）、 @Import(要导入容器中的组件);容器中就会自动的注册这个组件，id默认是全类名
     *      （2）、 ImportSelector ：返回需要的组件的全类名的数组；
     *      （3）、 ImportBeanDefinitionRegistrar : 手动注册bean到容器中
     *
     * 4）、使用Spring提供的FactoryBean（工厂bean）
     *      （1）、默认获取到的是工厂bean调用getObject创建的对象
     *      （2）、要获取工厂bean本身，我们需要给id前面加上一个“&”符号：&colorFactoryBean
     */

    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }
}

```

最后，我们来进行测试：

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);
        Object colorFactoryBean = applicationContext.getBean("colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean.getClass());
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

测试结果为：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> colorFactoryBean
> rainBow
> ColorFactoryBean…getBean…
> bean的类型：class com.ldc.bean.Color
>

------

如果我们就想要获取这个工厂bean，我们就可以在id的前面加上一个"&"符号

```java
    @Test
    public void test4() {
        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig2.class);
        printBeans(applicationContext);

        //工厂bean获取的是调用getObject方法创建的对象
        Object colorFactoryBean = applicationContext.getBean("colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean.getClass());

        //如果我们就想要获取这个工厂bean，我们就可以在id的前面加上一个"&"符号
        Object colorFactoryBean2 = applicationContext.getBean("&colorFactoryBean");
        System.out.println("bean的类型："+colorFactoryBean2.getClass());
    }

    private void printBeans(ApplicationContext applicationContext) {
        String[] definitionNames = applicationContext.getBeanDefinitionNames();
        for (String name : definitionNames) {
            System.out.println(name);
        }
    }

```

测试结果：

> org.springframework.context.annotation.internalConfigurationAnnotationProcessor
> org.springframework.context.annotation.internalAutowiredAnnotationProcessor
> org.springframework.context.annotation.internalRequiredAnnotationProcessor
> org.springframework.context.annotation.internalCommonAnnotationProcessor
> org.springframework.context.event.internalEventListenerProcessor
> org.springframework.context.event.internalEventListenerFactory
> mainConfig2
> com.ldc.bean.Color
> com.ldc.bean.Red
> com.ldc.bean.Blue
> com.ldc.bean.Yellow
> bill
> colorFactoryBean
> rainBow
> ColorFactoryBean…getBean…
> colorFactoryBean的类型：class com.ldc.bean.Color
> colorFactoryBean2的类型：class com.ldc.bean.ColorFactoryBean
>

# 生命周期-`@Bean`指定初始化和销毁方法

首先，我们有一个Car类：

```java
public class Car {
    public Car() {
        System.out.println("car constructor...");
    }

    public void init() {
        System.out.println("car...init...");
    }

    public void destroy() {
        System.out.println("car...destroy...");
    }

}

```

有一个配置类：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 *
 *
 *
 */
@Configuration
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

我们再来写一个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
    }

```

此时，还没有关闭IOC容器的时候，运行结果为：

> car constructor…
> car…init…
> 容器创建完成

而当我们调用了容器的close方法关闭容器的时候：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

测试的运行结果为：

> car constructor…
> car…init…
> 容器创建完成
> car…destroy…

而当bean的作用域为多例的时候：

```java
@Configuration
public class MainConfigOfLifeCycle {

    @Scope("prototype")
    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

这个时候，我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果为：

> 容器创建完成

当bean的作用域为多例的时候，只有在获取的时候，才会创建对象，而且在IOC容器关闭的时候，是不进行销毁的 ：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.getBean("car");
        applicationContext.close();
    }

```

此时，运行结果为：

> 容器创建完成
> car constructor…
> car…init…

------

# 生命周期-`InitializingBean`和`DisposableBean`

InitializingBean接口：在bean的初始化方法调用之后进行调用

```java
public interface InitializingBean {

	/**
	 * Invoked by a BeanFactory after it has set all bean properties supplied
	 * (and satisfied BeanFactoryAware and ApplicationContextAware).
	 * <p>This method allows the bean instance to perform initialization only
	 * possible when all bean properties have been set and to throw an
	 * exception in the event of misconfiguration.
	 * @throws Exception in the event of misconfiguration (such
	 * as failure to set an essential property) or if initialization fails.
	 */
	void afterPropertiesSet() throws Exception;

}

```

DisposableBean接口：

```java
public interface DisposableBean {

	/**
	 * Invoked by a BeanFactory on destruction of a singleton.
	 * @throws Exception in case of shutdown errors.
	 * Exceptions will get logged but not rethrown to allow
	 * other beans to release their resources too.
	 */
	void destroy() throws Exception;

}

```

有一个cat类：实现这两个接口

```java
@Component
public class Cat implements InitializingBean, DisposableBean {
    public Cat() {
        System.out.println("Cat...Contrustor...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Cat...destroy...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Cat...afterPropertiesSet...");
    }
}

```

这次，我们就用包扫描的方式来进行装配：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 * 2)通过bean实现InitializingBean（定义初始化逻辑）
 *               DisposableBean（定义销毁逻辑）；
 *
 *
 */
@Configuration
@ComponentScan("com.ldc.bean")
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

测试：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果：

> Cat…Contrustor…
> Cat…afterPropertiesSet…
> car constructor…
> car…init…
> 容器创建完成
> car…destroy…
> Cat…destroy…

# 生命周期- `@PostConstruct`&`@PreDestroy`

3）可以使用JSR250规范里面定义的两个注解：

- @PostConstruct :在bean创建完成并且属性赋值完成，来执行初始化方法
- @PreDestroy ：在容器销毁bean之前通知我们来进行清理工作

------

`@PostConstruct`：在bean创建完成并且属性赋值完成，来执行初始化方法

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```

`@PreDestroy`：在容器销毁bean之前通知我们来进行清理工作

```java
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}

```

我们再来定义一个Dog类：

```java
@Component
public class Dog {
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
}

```

我们再来运行这个测试方法：

```java
@Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

测试结果如下：

> Cat…Contrustor…
> Cat…afterPropertiesSet…
> Dog…Contructor…
> Dog…@PostConstruct…
> car constructor…
> car…init…
> 容器创建完成
> car…destroy…
> Dog…@PreDestroy…
> Cat…destroy…



# 生命周期-`BeanPostProcessor`-后置处理器

4）BeanPostProcessor接口：bean的后置处理器，在bean初始化前后做一些处理工作，这个接口有两个方法：

* postProcessBeforeInitialization：在初始化之前工作
* postProcessAfterInitialization：在初始化之后工作

------

BeanPostProcessor接口：

```java
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}

```

同样，还是使用包扫描的方式来进行装配：

```java
/**
 * bean的生命周期：bean的创建->初始化->销毁的过程
 * 容器管理bean的生命周期：
 * 我们可以自定义初始化方法和销毁的方法：容器在bean进行到当前的生命周期的时候，来调用我们自定义的初始化方法和销毁方法
 * 构造（对象创建）：
 *      单实例：在容器启动的时候创建对象
 *      多实例：在每次获取的时候来创建对象
 * 初始化方法：
 *      对象创建完成，并赋值好，调用初始化方法
 * 销毁方法：
 *      单实例的bean:在容器关闭的时候进行销毁
 *      多实例的bean:容器不会管理这个bean,容器不会调用销毁的方法
 *
 * 1)指定初始化方法和销毁方法；
 *          -我们可以通过@Bean(initMethod = "init",destroyMethod = "destroy")来指定初始化方法和销毁方法
 *          相当于xml配置文件bean标签里面的 init-method="" 和 destroy-method="" 属性
 * 2)通过bean实现InitializingBean（定义初始化逻辑）
 *               DisposableBean（定义销毁逻辑）；
 *
 * 3）可以使用JSR250规范里面定义的两个注解：
 *      @PostConstruct :在bean创建完成并且属性赋值完成，来执行初始化方法
 *      @PreDestroy ：在容器销毁bean之前通知我们来进行清理工作
 * 4）BeanPostProcessor接口：bean的后置处理器
 * 在bean初始化前后做一些处理工作，这个接口有两个方法：
 *      - postProcessBeforeInitialization：在初始化之前工作
 *      - postProcessAfterInitialization：在初始化之后工作
 */
@Configuration
@ComponentScan("com.ldc.bean")
public class MainConfigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destroy")
    public Car car() {
        return new Car();
    }
}

```

我们来写一个类并且实现这个后置处理器接口 BeanPostProcessor ：

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization..."+beanName+"=>"+bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization..."+beanName+"=>"+bean);
        return bean;
    }
}

```

此时，我们再来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

运行结果如下：

> postProcessBeforeInitialization…org.springframework.context.event.internalEventListenerProcessor=>org.springframework.context.event.EventListenerMethodProcessor@36f0f1be
> postProcessAfterInitialization…org.springframework.context.event.internalEventListenerProcessor=>org.springframework.context.event.EventListenerMethodProcessor@36f0f1be
> postProcessBeforeInitialization…org.springframework.context.event.internalEventListenerFactory=>org.springframework.context.event.DefaultEventListenerFactory@6ee12bac
> postProcessAfterInitialization…org.springframework.context.event.internalEventListenerFactory=>org.springframework.context.event.DefaultEventListenerFactory@6ee12bac
> postProcessBeforeInitialization…mainConfigOfLifeCycle=>com.ldc.config.MainConfigOfLifeCycle
> E n h a n c e r B y S p r i n g C G L I B EnhancerBySpringCGLIB
> EnhancerBySpringCGLIB
> 27d6c7d3@64c87930
> postProcessAfterInitialization…mainConfigOfLifeCycle=>com.ldc.config.MainConfigOfLifeCycle
> E n h a n c e r B y S p r i n g C G L I B EnhancerBySpringCGLIB
> EnhancerBySpringCGLIB
> 27d6c7d3@64c87930
> Cat…Contrustor…
> postProcessBeforeInitialization…cat=>com.ldc.bean.Cat@525f1e4e
> Cat…afterPropertiesSet…
> postProcessAfterInitialization…cat=>com.ldc.bean.Cat@525f1e4e
> Dog…Contructor…
> postProcessBeforeInitialization…dog=>com.ldc.bean.Dog@5ea434c8
> Dog…@PostConstruct…
> postProcessAfterInitialization…dog=>com.ldc.bean.Dog@5ea434c8
> car constructor…
> postProcessBeforeInitialization…car=>com.ldc.bean.Car@1d548a08
> car…init…
> postProcessAfterInitialization…car=>com.ldc.bean.Car@1d548a08
> 容器创建完成
> car…destroy…
> Dog…@PreDestroy…
> Cat…destroy…



# 生命周期-`BeanPostProcessor`原理

我们在这个后置处理器的两个方法上面打上断点：

![image-20210407121353352](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121353352.png)

然后，我们以debug的方式来运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

我们大概走一下这个方法调用栈：

![image-20210407121421471](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121421471.png)

![image-20210407121446668](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121446668.png)

![image-20210407121504912](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121504912.png)

------

前置处理器调用的方法：调用getBeanPostProcessors()方法找到容器里面的所有的BeanPostProcessor，挨个遍历，调用BeanPostProcessor的postProcessBeforeInitialization方法，一旦调用postProcessBeforeInitialization方法的返回值为null的时候，就直接跳出遍历 ，后面的BeanPostProcessor 的postProcessBeforeInitialization也就不会执行了：

```java
	@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
			result = beanProcessor.postProcessBeforeInitialization(result, beanName);
			if (result == null) {
				return result;
			}
		}
		return result;
	}

```

后置处理器调用的方法：调用getBeanPostProcessors()方法找到容器里面的所有的BeanPostProcessor，挨个遍历，调用BeanPostProcessor的postProcessAfterInitialization方法，一旦调用postProcessAfterInitialization方法的返回值为null的时候，就直接跳出遍历 ，后面的BeanPostProcessor 的postProcessAfterInitialization也就不会执行了：

```java
	@Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
			result = beanProcessor.postProcessAfterInitialization(result, beanName);
			if (result == null) {
				return result;
			}
		}
		return result;
	}

```

> //前置处理器执行
> wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
> //初始化方法执行
> invokeInitMethods(beanName, wrappedBean, mbd);
> //后置处理器执行
> wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);

在看源码的时候，我们可以发现：
在执行exposedObject = initializeBean(beanName, exposedObject, mbd);方法之前先调用了populateBean(beanName, mbd, instanceWrapper);这个方法：

populateBean(beanName, mbd, instanceWrapper);–给bean进行属性赋值
exposedObject = initializeBean(beanName, exposedObject, mbd);–初始化方法，在这个初始化方法内部包括了前置处理器、初始化方法、后置处理器的调用：


![image-20210407121556059](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121556059.png)



# 生命周期- `BeanPostProcessor`在Spring底层的使用

- Spring底层对 BeanPostProcessor 的使用：
  - bean赋值，注入其他组件，`@Autowired`,生命周期注解等功能,`@Async`等等都是使用`BeanPostProcessor`来完成的

BeanPostProcessor 这个接口有很多的实现类：

![image-20210407121624065](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121624065.png)

如果我们想要获取IOC容器，我们可以这样做：

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

我们可以debug来调试：

![image-20210407121652490](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121652490.png)

我们还是debug运行这个测试方法：

```java
    @Test
    public void test01() {
        //1.创建IOC容器
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfigOfLifeCycle.class);
        System.out.println("容器创建完成");
        applicationContext.close();
    }

```

实际上BeanPostProcessor接口还有很多的实现类，比如说BeanValidationPostProcessor，这个是用来做数据校验的：

![image-20210407121719311](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121719311.png)

像InitDestroyAnnotationBeanPostProcessor这个实现类就是实现了@PostConstruct和@PreDestroy这两个注解的功能：

![image-20210407121733619](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121733619.png)

我们在这个地方打个断点：

![image-20210407121747077](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121747077.png)

![image-20210407121800023](C:\Users\kexin\AppData\Roaming\Typora\typora-user-images\image-20210407121800023.png)