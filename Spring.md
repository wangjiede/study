## Spring概述

Spring 是一个开源框架，为简化企业级应用开发而生，是一个 IOC(DI) 和 AOP 容器框架，具体描述：

- 轻量级：Spring 是非侵入性的 - 基于 Spring 开发的应用中的对象可以不依赖于 Spring 的 API
- 依赖注入(DI --- dependency injection、IOC)
- 面向切面编程(AOP --- aspect oriented programming)
- 容器： Spring 是一个容器, 因为它包含并且管理应用对象的生命周期
- 框架: Spring 实现了使用简单的组件配置组合成一个复杂的应用. 在 Spring 中可以使用 XML 和 Java 注解组合这些对象
- 一站式：在 IOC 和 AOP 的基础上可以整合各种企业应用的开源框架和优秀的第三方类库 （实际上 Spring 自身也提供了展现层的 SpringMVC 和 持久层的 Spring JDBC）

## IOC&DI概述

- IOC(Inversion of Control)：思想是反转资源获取的方向。传统方式是组件向容器发起请求查找资源，容器返回资源。而IOC是容器主动推送资源，组件选择合适的方式接收资源，也被称为查找的被动方式。
- spring的ApplicationContext 在初始化上下文时就实例化所有单例的 Bean，提供了两种类型的 IOC 容器实现：
  - BeanFactory：IOC 容器的基本实现，是 Spring 框架的基础设施，面向 Spring 本身
  - ApplicationContext：提供了更多的高级特性. 是 BeanFactory 的子接口，面向使用 Spring 框架的开发者，主要实现类：
    - ClassPathXmlApplicationContext：从 类路径下加载配置文件
    - FileSystemXmlApplicationContext：从文件系统中加载配置文件
    - ConfigurableApplicationContext ：扩展于 ApplicationContext，新增加两个主要方法：refresh() 和 close()， 让 ApplicationContext 具有启动、刷新和关闭上下文的能力
    - WebApplicationContext ：是专门为 WEB 应用而准备的，它允许从相对于 WEB 根目录的路径中完成初始化工作
- DI(Dependency Injection)：依赖注入，即依赖容器给组件注入参数、属性

## 配置Bean

### 基于XML

#### 实例Bean方式

XML中Id值说明：在 IOC 容器中必须是唯一的，若 id 没有指定，Spring 自动将权限定性类名作为 Bean 的名字，可以指定多个名字，名字之间可用逗号、分号、或空格分隔

- 通过全类名（反射）

  ```xml
  <bean id="helloWorld" class="com.spring.helloworld.HelloWorld">
  ```

- 通过工厂方法

  - 静态工厂：不需要实例化工厂就执行工厂的静态方法创建需要的对象

    ```java
    public class Factory {
        private static Map<String,Car> carMap = new HashMap<>();
        static {
            carMap.put("baoma",new Car("奔驰"));
            carMap.put("benci",new Car("宝马"));
        }
    
        public static Car getCar(String name){
            return carMap.get(name);
        }
    }
    ```

    ```xml
    <bean id="staticCar" class="com.just.Factory" factory-method="getCar">
      <constructor-arg value="baoma" type="java.lang.String"></constructor-arg>
    </bean>
    ```

  - 实例工厂：需要实例化工厂后执行工厂方法创建需要的对象

    ```java
    public class Factory {
        private Map<String,Car> carMap = new HashMap<>();
        {
            carMap.put("benci",new Car("奔驰"));
            carMap.put("baoma",new Car("宝马"));
        }
    
        public Car getCar(String name){
            return carMap.get(name);
        }
    }
    ```

    ```xml
    <bean id="factory" class="com.just.Factory"></bean>
    <bean id="dynamicCar" factory-bean="factory" factory-method="getCar">
      <constructor-arg value="baoma" type="java.lang.String"></constructor-arg>
    </bean>
    ```

- 自定义FactoryBean(静态工厂)

  ```java
  public class Factory implements FactoryBean<Car> {
      private String brand;
  
      @Override
      public Car getObject() throws Exception {
          return  new Car(brand);
      }
  
      @Override
      public Class<?> getObjectType() {
          return Car.class;
      }
  
      @Override
      public boolean isSingleton() {
          return false;
      }
  
      public void setBrand(String brand) {
          this.brand = brand;
      }
  }
  ```

  ```xml
  <!--实现了FactoryBean接口后，id可以不指定-->
  <bean id="beanCar" class="com.just.Factory">
    <property name="brand" value="baoma"></property>
  </bean>
  ```

#### 注入方式

- 属性注入：通过 setter 方法注入Bean 的属性值或依赖的对象，通常使用 \<property> 元素, 使用 name 属性指定 Bean 的属性名称，value 属性或\<value> 子节点指定属性值，如:

  ```xml
  <bean id="helloWorld" class="com.spring.helloworld.HelloWorld">
    <property name="user" value="Jerry"></property>
  </bean>
  ```

- 构造器注入：通过构造方法注入Bean 的属性值或依赖的对象，通过构造方法注入Bean 的属性值或依赖的对象，如：

  - 按索引匹配入参

    ```xml
    <bean id="car" class="com.spring.helloworld.Car">
      <constructor-arg value="KUGA" index="1"></constructor-arg>
      <constructor-arg value="ChangAnFord" index="0"></constructor-arg>
    </bean>
    ```

  - 按类型匹配入参

    ```xml
    <bean id="car" class="com.atguigu.spring.helloworld.Car">
      <constructor-arg value="KUGA" type="java.lang.String"></constructor-arg>
      <constructor-arg value="ChangAnFord" type="java.lang.String"></constructor-arg>
    </bean>
    ```

- 工厂方法注入（很少使用，不推荐）,参照**实例Bean方式**中的工厂方法

#### 基本类型注入

所有基本类型及基本类型包装类加String都可以通过value属性指定值，若指定的值中有特殊字符可以使用\<![CDATA[]]>，如：

```xml
<bean name='user' class="domain.User">
  <property name="name" value="张三"></property>
  <!--
	<constructor-arg name="name" value="张三" type="java.lang.String"></constructor-arg>
	-->
</bean>
```

#### 对象注入

可以通过 \<ref> 元素或 ref  属性为 Bean 的属性或构造器参数指定对 Bean 的引用，也可以在属性或构造器里包含 Bean 的声明, 这样的 Bean 称为内部 Bean,如：

```xml
<bean name="phone" class="domain.Phone"></bean>
<bean name='user' class="domain.User">
  <!--方式一-->
  <property name="phone" ref="phone"></property>
  <!--方式二-->
  <property name="phone">
  	<bean name="phone" class="domain.Phone"></bean>
  </property>
  <!--当指定的bean需要为一个null对象的时候,使用<null/>标签-->
  <property name="phone"><null/></property>
</bean>
```

#### 集合注入

```xml
<beans>
  <bean id="c1" class="domain.Computer"></bean>
  <bean id="c2" class="domain.Computer"></bean>
  <bean id="c3" class="domain.Computer"></bean>
  <bean id="c4" class="domain.Computer"></bean>
  
  <!--array-->
  <property name="computers">
    <array value-type="domain.Computer">
      <ref bean="c1"></ref>
      <ref bean="c2"></ref>
      <ref bean="c3"></ref>
      <ref bean="c4"></ref>
    </array>
  </property>
  
  <!--list-->
  <constructor-arg name="list" type="java.util.List">
    <list value-type="java.lang.String">
      <value>aaa</value>
      <value>bbb</value>
      <value>ccc</value>
      <value>ddd</value>
    </list>
  </constructor-arg>
  
  <!--set-->
  <constructor-arg name="set" type="java.util.Set">
    <set value-type="domain.Computer">
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
    </set>
  </constructor-arg>
  
  <!--map-->
  <property name="map">
    <map key-type="java.lang.Integer" value-type="java.lang.String">
      <entry key="1" value="aaa"></entry>
      <entry key="2" value="bbb"></entry>
      <entry key="3" value="ccc"></entry>
      <entry key="4" value="ddd"></entry>
    </map>
  </property>
  <property name="map">
    <map key-type="java.lang.String" value-type="domain.Computer">
      <entry key="a" value-ref="c1"></entry>
      <entry key="b" value-ref="c2"></entry>
      <entry key="c" value-ref="c3"></entry>
      <entry key="d" value-ref="c4"></entry>
    </map>
  </property>
  
  <!--props-->
  <constructor-arg name="properties">
    <props>
      <prop key="1">aaa</prop>
      <prop key="2">bbb</prop>
      <prop key="3">ccc</prop>
    </props>
  </constructor-arg>
</beans>
```

#### 命名空间

- p：代替property属性

  ```xml
  <bean name='user' class="domain.User" p:name="张三" p:age="18"></bean>
  ```

- c：代替constructor-arg属性

  ```xml
  <bean name="user" class="domain.User" c:name="张三" c:age="18"></bean>
  ```

- util：单纯的一个标签，通常用来定义集合模板

  ```xml
  <util:map id="computers">
    <entry key="1">
      <bean class="domain.Computer"></bean>
    </entry>
    <entry key="2">
      <bean class="domain.Computer"></bean>
    </entry>
  </util:map>
  <bean id="user" class="domain.User" p:name="张三" p:computers-ref="computer"></bean>
  ```

#### Bean关系

- 继承

  ```xml
  <bean id="base" p:computer1-ref="c1" p:computer2-ref="c2" p:computer3-ref="c3" abstract="true"></bean>
  <bean id="room1" class="domain.Room" parent="base"></bean>
  <!--也可以指定parent是一个具体的实例bean-->
  <bean id="room2" class="domain.Room" parent="room1"></bean>
  ```

- 依赖

  ```xml
  <!--room类依赖computer，在创建bean时，没有computerbean就会报错，依赖多个bean时，可以用逗号分隔-->
  <bean id="room" class="domain.Room" depends-on="computer"></bean>
  ```

#### SpEL

- ${}：主要是使用外部文件中定义好的key值，如在spring的配置文件中引入properties文件后：

  ```xml
  <context:property-placeholder location="classpath:xxx.properties"></context:property-placeholder>
  <property name="name" value="${name}"></property>
  ```

- #{}：

  - 基础类型属性值注入：#{'zzt'}  #{123}  #{123.45}  #{true}
  - 做运算：
    - 算术：+ - * / %等， ^在Java中是位运算，但^在Spring是幂运算
    - 比较：> >= < <= != ==对应gt ge lt le ne eq
    - 逻辑：没有与符号  ||  !	and or not
  - 对象类型属性赋值：#{beanID}
  - 操作对象属性：#{beanID.属性名}
  - 操作对象方法：#{beanID.方法名()}
  - 引入java中的类并执行方法或调用属性：#{2 * T(java.lang.Math).PI}

#### 自动装配

将对象中的对象类型的属性自动赋值，使用autowire

- byName：bean对象中的属性名与另一个bean对象的name或id一致即可
- byType：bean对象中的属性类型与另一个bean对象的class类型一致，当有多个类型的bean时，报错
- constructor：先按照类型匹配，如果类型发现不止一个对应，再按照属性名与bean的name或id匹配 成功匹配 就赋值

#### Bean作用域

- 在 Spring 中, 可以在 \<bean> 元素的 scope 属性里设置 Bean 的作用域，默认情况下值是singleton, Spring 只为每个在 IOC 容器里声明的 Bean 创建唯一一个实例, 整个 IOC 容器范围内都能共享该实例。

  | 类别      | 说明                                                         |
  | --------- | ------------------------------------------------------------ |
  | singleton | 单例模式                                                     |
  | prototype | 原型的，每次获取bean对象，都会创建新的对象                   |
  | request   | 每次Http请求都会创建一个新的bean，适合用于WebApplicationContext环境 |
  | session   | 同一个Http Seesion共享一个bean，适合用于WebApplicationContext环境 |

- spring底层默认采用立即加载、生命周期托管形式实现对象单例

  - 可以通过bean标签的scope属性修改是否为单例对象，默认singleton

    - scope="singleton"：单例模式
    - scope="prototype"：每次都new新对象

  - 可以通过bean标签的lazy-init属性修改是否延迟加载，默认default即false

    - lazy-init="default/false"：立即加载
    - lazy-init="true"：延迟加载

  - 以上两点也可以通过在\<beans>标签中设置，就不用针对每一个bean设置了

    ```xml
    <beans default-lazy-init="true" 
           default-autowire="byName">
      
    </beans>
    ```


#### IOC容器中Bean生命周期

- 默认生命周期管理过程，在 Bean 的声明里设置 init-method 和 destroy-method 属性, 为 Bean 指定初始化和销毁方法

  - 通过构造器或工厂方法创建 Bean 实例
  - 为 Bean 的属性设置值和对其他 Bean 的引用
  - 调用 Bean 的初始化方法
  - Bean 可以使用了
  - 当容器关闭时, 调用 Bean 的销毁方法

- 创建并配置bean后置处理器，如：

  ```xml
  <!---实现org.springframework.beans.factory.config.BeanPostProcessor接口并实现方法，在spring配置文件中配置-->
  <bean class="com.spring.ref.MyBeanPostProcessor"></bean>
  ```

  - 通过构造器或工厂方法创建 Bean 实例
  - 为 Bean 的属性设置值和对其他 Bean 的引用
  - 将 Bean 实例传递给 Bean 后置处理器的 postProcessBeforeInitialization 方法
  - 调用 Bean 的初始化方法
  - 将 Bean 实例传递给 Bean 后置处理器的 postProcessAfterInitialization方法
  - 使用bean
  - 当容器关闭时, 调用 Bean 的销毁方法

### 基于注解

- 使用context:component-scan标签进行包扫描

  - base-package属性：指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包里及其子包中的所有类，当需要扫描多个包时, 可以使用逗号分隔
  - resource-pattern属性：扫描满足指定规则的类
  - \<context:include-filter>子节点：表示要包含的目标类
  - \<context:exclude-filter> 子节点：表示要排除在外的目标类

- Spring扫描组件有默认的命名策略，使用非限定类名, 首字母小写，也可以在注解中通过 value 属性值标识组件的名称，默认扫描组件包括，可以使用use-default-filters属性设置：

  - @Component: 基本注解, 标识了一个受 Spring 管理的组件
  - @Respository: 标识持久层组件
  - @Service: 标识服务层(业务层)组件
  - @Controller: 标识表现层组件

- \<context:include-filter> 和 \<context:exclude-filter> 子节点支持类型表达式

  | 类别       | 示例                     | 说明                                                         |
  | ---------- | ------------------------ | ------------------------------------------------------------ |
  | annotation | com.spring.Controller    | 所有标注了@Controller的类                                    |
  | assinable  | com.spring.Service       | 所有继承或扩展了com.spring.Service的类                       |
  | aspectj    | com.spring..*Service+    | 所有以Service结束的类以及继承获取扩展它的类                  |
  | regex      | com\\.spring\\.anno\\..* | 所有com.spring.anno包下的类                                  |
  | custom     | com.spring.XxxTypeFilter | 通过实现org.springframework.core.type.filter.TypeFilter接口的类具体代码实现过滤。 |

- 组件装配

  \<context:component-scan> 元素还会自动注册 AutowiredAnnotationBeanPostProcessor 实例, 该实例可以自动装配具有 @Autowired 和 @Resource 、@Inject注解的属性.

  - @Autowired，可以使用在构造器、普通属性、具有参数的普通方法、数组、集合、map。
    - required属性：设置是否在IOC容器中必须存在。
    - @Qualifier注解：配和@Autowired使用，指定注入bean的名称
  - @Resource：要求提供一个 Bean 名称的属性，若该属性为空，则自动采用标注处的变量或方法名作为 Bean 的名称
  - @Inject：和@Autowired 注解一样也是按类型匹配注入的 Bean， 但没有 reqired 属性

## AOP