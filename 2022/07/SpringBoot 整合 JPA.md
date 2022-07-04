# SpringBoot 整合 JPA

JPA（Java Persistence API）和 JDBC 类似，是官方定义的一组接口，但是它相比传统的 JDBC，它是为了实现 ORM 而生的，它的作用是在关系型数据库和对象之间形成一个映射，这样，在具体的操作数据库的时候，就不需要再去和复杂的 SQL 语句打交道，只要像平时操作对象一样操作它就可以了。实现 JPA 规范的框架一般最常用的就是 Hibernate ，它是一个重量级框架，而 SpringDataJPA 也是采用 Hibernate 框架作为底层实现，并对其加以封装。

## 一、导包

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

## 二、配置

数据源：

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3316/jpa
    username: root
    password: Qwz#1201
```

其他：

```yml
spring:

  jpa:
		#开启SQL语句执行日志信息
    show-sql: true
    hibernate:
    	#配置为自动创建
      ddl-auto: create
```

说明：`ddl-auto`属性用于设置自动表定义，可以实现自动在数据库中为我们创建一个表，表的结构会根据我们定义的实体类决定。可选值有四种：

- create：启动时删数据库中的表，然后创建，退出时不删除数据表
- create-drop：启动时删数据库中的表，然后创建，退出时删除数据表 如果表不存在报错
- update：如果启动时表格式不一致则更新表，原有数据保留
- validate：项目启动表结构进行校验 如果不一致则报错

## 三、准备

entity：

```java
/**
 * @author qinweizhao
 * @since 2022/7/4
 */
@Data
@Entity   //表示这个类是一个实体类
@Table(name = "account")    //对应的数据库中表名称
public class Account {

    /**
     * 生成策略，这里配置为自增
     */
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")    //对应表中id这一列
    @Id     //此属性为主键
    private int id;

    @Column(name = "username")
    private String username;

    @Column(name = "password")
    private String password;

    //一对一
    @JoinColumn(name = "detail_id")
    @OneToOne//声明为一对一关系
    private AccountDetail detail;//对象类型,也可以理解这里写哪个实体类,外键就指向哪个实体类的主键

}
```

提前准备数据库，此时启动程序即可就会在数据库中创建表（users），在项目启动时会执行创建表操作（配置文件配置了 **ddl-auto** ）。表创建完成后为了方便后续操作修改项目的 yml 配置：

```yml
      # 配置为自动创建
      # ddl-auto: create
      ddl-auto: update
```

将 `ddl-auto` 修改为 update ，因为 create 策略会删除表中数据。

## 四、使用

### 1、自带方法

repository：

```java
/**
 * @author qinweizhao
 * @since 2022/7/4
 */
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {
    
}
```

注意：JpaRepository 有两个泛型，前者是具体操作的对象实体，也就是对应的表，后者是 ID 的类型，接口中已经定义了比较常用的数据库操作。

测试：

```java
    @Test
    void save() {
        Account account = new Account();
        account.setUsername("Admin");
        account.setPassword("123456");
        account = accountRepository.save(account);  //返回的结果会包含自动生成的主键值
        System.out.println("插入时，自动生成的主键ID为：" + account.getId());
    }
```

```java
    @Test
    void delete() {
        accountRepository.deleteById(2);   //根据ID删除对应记录
    }
```

```java
    @Test
    void page() {
        accountRepository.findAll(PageRequest.of(0, 1)).forEach(System.out::println);  //直接分页
    }
```

### 2、自定义方法

repository：

```java
/**
 * @author qinweizhao
 * @since 2022/7/4
 */
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    /**
     * 按照表中的规则进行名称拼接，不用刻意去记，IDEA会有提示
     *
     * @param str str
     * @return List
     */
    List<Account> findAccountByUsername(String str);
    
}
```

测试：

```java
    @Test
    void findAccountByUsername() {
        accountRepository.findAccountByUsername("Admin").forEach(System.out::println);
    }
```

说明：其他方法类似，都可以根据实体中的字段进行条件拼接。

也可以自己通过注解来指定 SQL 语句。

repository：

```java
/**
 * @author qinweizhao
 * @since 2022/7/4
 */
@Repository
public interface AccountRepository extends JpaRepository<Account, Integer> {

    /**
     * 自定义SQL语句,必须在事务环境下运行 必须有DML支持(Modifying)
     * 直接对实体类进行操作 然后实体类映射到表中
     *
     * @param id          id
     * @param newPassword newPassword
     * @return int
     * Transactional 这个注解也可以加到测试类上面 但需要跟进一个@commit提交事务的注解 因为测试类会自动回滚事务
     */
    @Transactional(rollbackFor = Exception.class)
    @Modifying
    @Query("update Account set password=?2 where id=?1")
    int updatePasswordById(int id, String newPassword);
    
}
```

测试：

```java
    @Test
    void updatePasswordById() {
        accountRepository.updatePasswordById(1, "123");
    }
```

### 3、关联查询

entity：

```java
@Data
@Entity
@Table(name = "account_details")
public class AccountDetail {
    @Column(name = "id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)//还是设置一个自增主键
    @Id
    int id;

    @Column(name = "address")
    String address;

    @Column(name = "email")
    String email;

    @Column(name = "phone")
    String phone;

    @Column(name = "real_name")
    String realName;
}
```

新增实体 AccountDetail ，在 Account 实体类中添加外键，设置懒加载功能,设置关联级别完成同时操作两张表：

```java
		//一对一
    @JoinColumn(name = "detail_id")
    @OneToOne(fetch = FetchType.LAZY, cascade = CascadeType.ALL) //设置关联操作为ALL
    AccountDetail detail;//对象类型,也可以理解这里写哪个实体类,外键就指向哪个实体类的主键
```

这里的关联级别也是有多个，一般设置为 all 就行：

- ALL：所有操作都进行关联操作。
- PERSIST：插入操作时才进行关联操作。
- REMOVE：删除操作时才进行关联操作。
- MERGE：修改操作时才进行关联操作。

测试：

```java
@Test
void add(){
    Account account = new Account();
    account.setUsername("qwz");
    account.setPassword("qwz");
    AccountDetail detail = new AccountDetail();
    detail.setAddress("zgsh");  
    detail.setPhone("112");
    detail.setEmail("yvkg@qq.com");
    detail.setRealName("wz");
    account.setDetail(detail);//这里就是传入一个对象
    account = accountRepository.save(account);
    System.out.println("插入时，自动生成的主键ID为："+account.getId()+"，外键ID为："+account.getDetail().getId());
}
```

上面为一对一的情况，进行关系映射时，常用的注解还有：

```java
@OneToMany
@OneToOne
@ManyToMany
@ManyToOne
```

## 五、总结

此次只是一些基础的用法，由于平时接触这个并不多，所以暂时不多做研究，毕竟知识是学不完的。

官网：[Spring Data JPA](https://spring.io/projects/spring-data-jpa)

## 附

>代码地址：
>
>https://github.com/qinweizhao/qwz-integration/tree/master/spring-boot-jpa
