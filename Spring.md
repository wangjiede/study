## 介绍

Spring其实是Spring Framework的检查，由Rod Johnson创建。

ORM框架：节省JDBC开发

WEB框架：节省Servlet开发

Spring：创建、管理对象

## 搭建

- 核心包：core、beans、context

- 初始化/获取工厂

  ```java
  //以classPath为基础路径
  ApplicationContext(BeanFactory) cts = new ClassPathXmlApplicationContext("spring.xml");
  //以系统路径为基础路径
  ApplicationContext(BeanFactory) cts = new FileSystemXmlApplicationContext("spring.xml");
  ```

- 获取指定bean的对象

  ```java
  User user = (User)cts.getBean("user");
  ```

- 路径说明

  - 系统路径：当前电脑路径,{file:}代表从电脑路径查找
  - 项目路径：以项目路径为基础路径(没打包编译之前的：默认项目下的src)
  - classpath：以类路径作为基础{classpath:}，默认target下的classes

## IOC(Inversion Of Control)

- 定义：将对象的控制权交个其他程序，由其他程序来管理对象的创建销毁等。

- 初始化：Spring在初始化时，会读取spring的配置文件，然后如果配置文件过多，可以通过以下方式初始化

  - 动态参数：ClassPathXmlApplicationContext("spring.xml","spring-bean.xml","","");
  - 数组参数：ClassPathXmlApplicationContext(new String[]{"spring.xml","spring-bean.xml","",""});
  - 通配符写法：ClassPathXmlApplicationContext("spring*.xml")
  - 主配置文件引入其他文件：ClassPathXmlApplicationContext("spring.xml")
    - 主配置文件中：\<import resource="spring-bean.xml">\</import>

- 对象管理方式：spring底层默认采用立即加载、生命周期托管形式实现对象单例

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

## DI(Dependency Injection)

### 注入方式

- set方法注入：必须含有注入属性的set方法和无参构造方法
- 构造方法注入：必须有有参构造方法
- 注解注入

### 属性值注入

- set方法注入

  ```xml
  <bean name='user' class="domain.User">
    <property name="name" value="张三"></property>
    <property name="age">
      <value type="java.lang.Integer">18</value>
    </property>
  </bean>
  ```

- 构造方法注入

  ```xml
  <bean name="user" class="domain.User">
    <constructor-arg name="name" value="张三" type="java.lang.String"></constructor-arg>
    <constructor-arg name="age">
      <value type="java.lang.Integer">18</value>
  	<constructor-arg/>
  </bean>
  ```

### 对象值注入

- set方法注入

  ```xml
  <bean name="phone" class="domain.Phone"></bean>
  <bean name='user' class="domain.User">
    <!--方式一-->
    <property name="phone" ref="phone"></property>
    <!--方式二-->
    <property name="phone">
    	<bean name="phone" class="domain.Phone"></bean>
    </property>
  </bean>
  ```

- 构造方法注入：与set方法注入类似

- null对象：使用null标签

  ```xml
  <property name="age"><null/></property>
  ```

- 特殊字符：使用\<![CDATA[内容]]>

  ```xml
  <property name="age">
    <value><![CDATA[test~*&]]></value>
  </property>
  ```

### 对象值自动注入

将对象中的对象类型的属性自动赋值，使用autowire

- 构造方法注入(使用少)

  ```xml
  <bean name="controller" class="controller.StudentController" autowire="constructor"></bean>
  ```

- set方法注入

  ```xml
  <bean name="controller" class="controller.StudentController" autowire="byName/byType"></bean>
  ```

  - byName：bean对象中的属性名与另一个bean对象的name或id一致即可
  - byType：bean对象中的属性类型与另一个bean对象的class类型一致

- 抽象/接口类型属性

  由于抽象/接口类没有办法直接创建对象，所以给该类型的属性赋值一定是它们的子类对象。

  - 构造方法方式自动装配：先按照类型匹配，如果类型发现不止一个对应，再按照属性名与bean的name或id匹配 成功匹配 就赋值
  - 无参数构造方法+set方法进行自动装配
    - byName：按照名字进行找寻，找不到就没有，找到就装配
    - byType：如果有两个以上对应的类型，标签配置时直接报错，采用内部形式ref自己指定的形式

### 集合注入

```xml
<bean id="c1" class="domain.Computer"></bean>
<bean id="c2" class="domain.Computer"></bean>
<bean id="c3" class="domain.Computer"></bean>
<bean id="c4" class="domain.Computer"></bean>
```

- array

  ```xml
  <property name="computers">
    <array value-type="domain.Computer">
      <ref bean="c1"></ref>
      <ref bean="c2"></ref>
      <ref bean="c3"></ref>
      <ref bean="c4"></ref>
    </array>
  </property>
  ```

- list

  ```xml
  <constructor-arg name="list" type="java.util.List">
    <list value-type="java.lang.String">
      <value>aaa</value>
      <value>bbb</value>
      <value>ccc</value>
      <value>ddd</value>
    </list>
  </constructor-arg>
  ```

- set

  ```xml
  <constructor-arg name="set" type="java.util.Set">
    <set value-type="domain.Computer">
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
      <bean class="domain.Computer"></bean>
    </set>
  </constructor-arg>
  ```

- map

  ```xml
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
  ```

- props

  ```xml
  <constructor-arg name="properties">
    <props>
      <prop key="1">aaa</prop>
      <prop key="2">bbb</prop>
      <prop key="3">ccc</prop>
    </props>
  </constructor-arg>
  ```

### 命名空间

- [^p:]: 代替property属性

  ```xml
  <bean name='user' class="domain.User">
    <property name="name" value="张三"></property>
    <property name="age">
      <value type="java.lang.Integer">18</value>
    </property>
  </bean>
  <!--以上配置等于-->
  <bean name='user' class="domain.User" p:name="张三" p:age="18"></bean>
  ```

- [^c:]: 代替constructor-arg属性

  ```xml
  <bean name="user" class="domain.User">
    <constructor-arg name="name" value="张三" type="java.lang.String"></constructor-arg>
    <constructor-arg name="age">
      <value type="java.lang.Integer">18</value>
  	<constructor-arg/>
  </bean>
  <!--以上配置等于-->
  <bean name="user" class="domain.User" c:name="张三" c:age="18"></bean>
  ```

- [^util:]: 单纯的一个标签

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

  其他的array、list、set、map、props与以上map类似。

### 抽象模板

类似java中的继承

```xml
<bean id="base" p:computer1-ref="c1" p:computer2-ref="c2" p:computer3-ref="c3" abstract="true"></bean>
<bean id="room" class="domain.Room" parent="base"></bean>
```

### SpEL

- ${}：主要是使用外部文件中定义好的key值，如在spring的配置文件中引入properties文件后：

  ```xml
  <context:property-placeholder location="classpath:xxx.properties"></context:property-placeholder>
  ```

  可以使用${key}取得对象的配置项的值，如：

  ```xml
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

## 工厂

- 静态工厂：不需要实例化工厂就执行工厂的静态方法创建需要的对象

  ```java
  public class Factory {
      private static Map<String,Car> carMap = new HashMap<>();
      static {
          carMap.put("奔驰",new Car("奔驰"));
          carMap.put("宝马",new Car("宝马"));
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
          carMap.put("奔驰",new Car("奔驰"));
          carMap.put("宝马",new Car("宝马"));
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

- Spring中自定义工厂(静态工厂)

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
  <bean id="beanCar" class="com.just.Factory">
    <property name="brand" value="baoma"></property>
  </bean>
  ```

  自定义一个类实现FactoryBean接口，重写接口中的方法，spring发现实现了该接口，默认调用getObject方法()创建对象。

## AOP(Aspect Oriented Programming)

## 注解