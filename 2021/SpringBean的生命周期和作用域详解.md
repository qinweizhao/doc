## SpringBean 的生命周期和作用域详解

## 一、SpringBean 的作用域

- **Singleton**在SpingIoc容器中仅存在一个Bean实例，Bean以单例方式存在，默认值
- **prototype**每次从容器中调用Bean时，都返回一个新的实例，即每次调用getBean时，相当于执行new xxBean()
- **request**每次Http请求都会创建一个新的Bean,该作用域仅适用于WebApplicationContent环境
- **ression**同一个HttpSession共享一个bean,不同Session使用不同Bean,仅用于WebApplication环境
- **globalSession**一般用于porlet应用环境，该作用域仅用于WebApplicationContext

## 二、SpringBean的生命周期

- **singleton单例bean的生命周期**当Scope=“Singleton”,即默认情况下，会在启动容器时（即容器实例化时）时实例化，但我们可以指定Bean节点的lazy-init="true"来延迟初始化bean,这时候只有在第一次获取bean时，才初始化bean,即第一次请求时初始化bean,默认情况下spring在读取XML文件的时候就会创建对象，在创建对象的时候先调用init-method属性中所指定的方法，对象在销毁的时候，会调用destroy-method属性中所指定的方法
- **prototype非单例对象的生命周期**当Scope="prototype"时，容器也会延迟初始化Bean,spring读取XML文件的时候，并不会立刻创建对象，而是在第一次请求该bean时才初始化，在第一次请求每一个prototype的bean时，Spring容器都会调用init-method属性值中所指定的方法，对象销毁的时候，Spring容器其不会帮助我们调用任何方法，因为是非单例，这个类型的对象有很多个，spring一旦把这个对象交给你之后，就不再管理这个对象了，交由GC

