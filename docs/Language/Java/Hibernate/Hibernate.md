---
title: Hibernate入门
permalink: /docs/Java/Hibernate
---

本教程基于Hibernate5.6.14.Final. 觉得老? 都用SSH框架了还有啥老不老的
## Hibernaten入门
创建持久化类
```java
@Data
@ToString
public class Student {
    private Integer id;
    private String studentName;
    private String gender;
    private Integer age;
}
```

创建映射文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>    
<class name="com.hibernatelearn.entity.Student" table="student">        
<id name="id" column="id">            <generator class="increment"/>        
</id>        
<property name="studentName" column="studentname" type="string"/>        
<property name="gender" column="gender" type="string"/>        
<property name="age" column="age" type="integer"/>    
</class>
</hibernate-mapping>
```

创建核心配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">
<hibernate-configuration>    
<session-factory>        
<property name="connection.driver_class"com.mysql.cj.jdbc.Driver/>      
<property name="connection.url"jdbc:mysql://localhost:3306/studentdb?useSSL=false&amp;serverTimezone=UTC/>       
<property name="connection.username"root/>
<property name="connection.password"80592955/>
<property name="dialect"org.hibernate.dialect.MySQL8Dialect/>
<property name="show_sql"true/>
<property name="hbm2ddl.auto"update/>
<mapping resource="mapper/student.hbm.xml"/>
</session-factory>
</hibernate-configuration>
```

测试增删查改
```java
package com.hibernatelearn;

import com.hibernatelearn.entity.Student;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.hibernate.SessionFactory;
import org.junit.jupiter.api.Test;

public class Main {
    // 添加
    @Test
    public void insertTest() {
        // 1. 加载核心配置文件
        Configuration config = new Configuration().configure();
        // 2. 创建 SessionFactory 对象
        SessionFactory sessionFactory = config.buildSessionFactory();
        // 3. 得到 Session 对象
        Session session = sessionFactory.openSession();
        // 4. 开启事务
        Transaction t = session.beginTransaction();
        // 5. 操作对象
        // 5.1 创建对象
        Student s = new Student();
        s.setStudentName("张三");
        s.setGender("男");
        s.setAge(20);
        // 5.2 保存对象
        session.save(s);
        // 6. 提交事务
        t.commit();
        // 7. 关闭资源
        session.close();
        sessionFactory.close();
    }

    // 更新
    @Test
    public void updateTest() {
        // 1. 加载核心配置文件
        Configuration config = new Configuration().configure();
        // 2. 创建 SessionFactory 对象
        SessionFactory sessionFactory = config.buildSessionFactory();
        // 3. 得到 Session 对象
        Session session = sessionFactory.openSession();
        // 4. 开启事务
        Transaction t = session.beginTransaction();
        // 5. 操作对象
        // 5.1 根据 id 查询对象
        Student s = session.get(Student.class, 1);
        // 5.2 修改对象
        s.setStudentName("李四");
        // 5.3 更新对象
        session.update(s);
        // 6. 提交事务
        t.commit();
        // 7. 关闭资源
        session.close();
        sessionFactory.close();
    }

    // 查找单个对象
    @Test
    public void getByIdTest(){
        // 1. 加载核心配置文件
        Configuration config = new Configuration().configure();
        // 2. 创建 SessionFactory 对象
        SessionFactory sessionFactory = config.buildSessionFactory();
        // 3. 得到 Session 对象
        Session session = sessionFactory.openSession();
        // 4. 开启事务
        Transaction t = session.beginTransaction();
        // 5. 操作对象
        // 5.1 根据 id 查询对象
        Student s = session.get(Student.class, 1);
        System.out.println("学生姓名：" + s.getStudentName());
        System.out.println("学生性别：" + s.getGender());
        System.out.println("学生年龄：" + s.getAge());
        // 6. 提交事务
        t.commit();
        // 7. 关闭资源
        session.close();
        sessionFactory.close();
    }

    // 删除
    @Test
    public void deleteByIdTest(){
        // 1. 加载核心配置文件
        Configuration config = new Configuration().configure();
        // 2. 创建 SessionFactory 对象
        SessionFactory sessionFactory = config.buildSessionFactory();
        // 3. 得到 Session 对象
        Session session = sessionFactory.openSession();
        // 4. 开启事务
        Transaction t = session.beginTransaction();
        // 5. 操作对象
        // 5.1 根据 id 查询对象
        Student s = session.get(Student.class, 1);
        // 5.2 删除对象
        session.delete(s);
        // 6. 提交事务
        t.commit();
        // 7. 关闭资源
        session.close();
        sessionFactory.close();
    }
}
```
利用HibernateUtil快速获取Session
```java
public class HibernateUtil {
    private static final Configuration config;
    private static final SessionFactory sessionFactory;
    static {
        config = new Configuration().configure();
        sessionFactory = config.buildSessionFactory();
    }
    public static Session getSession(){
        return sessionFactory.openSession();
    }
}
```
简化后的CRUD
```java
package com.hibernatelearn;

import com.hibernatelearn.entity.Student;
import com.hibernatelearn.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.Transaction;
import org.junit.jupiter.api.Test;

public class Main2 {
    @Test
    public void insertTest(){
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        Student s = new Student();
        s.setStudentName("张三");
        s.setGender("男");
        s.setAge(20);
        session.save(s);
        t.commit();
        session.close();
    }

    @Test
    public void updateTest(){
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        Student s = session.get(Student.class, 1);
        s.setStudentName("李四");
        session.update(s);
        t.commit();
        session.close();
    }

    @Test
    public void getByIdTest(){
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        Student s = session.get(Student.class, 1);
        System.out.println("姓名: "+s.getStudentName());
        System.out.println("性别: "+s.getGender());
        System.out.println("年龄: "+s.getAge());
        t.commit();
        session.close();
    }

    @Test
    public void deleteTest(){
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        Student s = session.get(Student.class, 2);
        session.delete(s);
        t.commit();
        session.close();
    }
}
```
## HQL查询
HQL查询
```java
public class HQLTest {
    @Test
    public void TestList() {
        // 1. 得到 Session 对象
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        // 2. 构建 HQL 语句
        String hql = "from Student";
        // 3. 创建 Query 对象
        Query query = session.createQuery(hql);
        // 4. 执行查询
        List<Studentlist = query.list();
        for (Student student: list) {
            System.out.println(student);
        }
        t.commit();
        session.close();
    }

    @Test
    public void TestIterate() {
        // 1. 得到 Session 对象
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        // 2. 构建 HQL 语句
        String hql = "from Student";
        // 3. 创建 Query 对象
        Query query = session.createQuery(hql);
        // 4. 执行查询
        Iterator<Studentstudents = query.iterate();
        Student student = null;
        while(students.hasNext()) {
            student = students.next();
            System.out.println(student);
        }
        t.commit();
        session.close();
    }

    @Test
    public void TestUniqueResult() {
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        String hql = "from Student where studentName='李四'";
        Query query = session.createQuery(hql);
        Student student = (Student) query.uniqueResult();
        System.out.println(student);
        t.commit();
        session.close();
    }

    @Test
    public void findStudent() {
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        String hql = "from Student where age=:age and gender=:gender";
        Query query = session.createQuery(hql);
        Student s = new Student();
        s.setAge(20);
        s.setGender("男");
        query.setProperties(s);
        List<Studentlist = query.list();
        for (Student student: list) {
            System.out.println(student);
        }
        t.commit();
        session.close();
    }
}
```
分页查询
```java
    @Test
    public void findStudent2() {
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        Scanner input = new Scanner(System.in);
        System.out.println("一页显示几条数据？");
        int pageSize = input.nextInt();
        System.out.println("查询第几页？");
        int pageNum = input.nextInt();
        String hql = "from Student";
        Query query = session.createQuery(hql);
        List<Studentlist = query.setFirstResult((pageNum-1)*pageSize).setMaxResults(pageSize).list();
        for (Student student: list) {
            System.out.println(student);
        }
        t.commit();
        session.close();
    }
```
## Hibernate关联映射
### 一对多关联
班级
```java
@Data
@EqualsAndHashCode(exclude = "students")
public class Classes {
    private String cid;
    private String cname;
    private Set<Studentstudents = new HashSet<);
}
```
学生
```java
@Data
@EqualsAndHashCode(exclude = "classes")
public class Student {
    private Integer id;
    private String studentName;
    private String gender;
    private Integer age;
    private Classes classes = new Classes();
}
```
班级配置表
```xml
<?xml version="1.0" encoding="UTF-8"> ?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.hibernatelearn.entity.Classes" table="classes">        
    <id name="cid" column="cid">            
        <generator class="assigned"/>        
    </id>
        <property name="cname" column="cname" length="20"/> 
        <set name="students">            
            <key column="classno"/>            
            <one-to-many class="com.hibernatelearn.entity.Student"/>        
        </set>    
    </class>
</hibernate-mapping>
            ```
子标签\<keycolumn>属性值对应数据库外键表中的外键列名

学生表配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <class name="com.hibernatelearn.entity.Student" table="student">        
        <id name="id" column="id">            
            <generator class="increment"/>        
        </id>        
        <property name="studentName" column="studentname" type="string"/>        <property name="gender" column="gender" type="string"/>        
        <property name="age" column="age" type="integer"/>
    <!-- 多对一关系映射 -->
    <many-to-one name="classes" column="classno" class="com.hibernatelearn.entity.Classes"/>    
    </class>
</hibernate-mapping>
```

```java
public class oneToManyTest {
    @Test
    public void test1() {
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        // 创建班级对象
        Classes c = new Classes();
        c.setCid(202106);
        c.setCname("Java");

        // 创建学生对象
        Student s1 = new Student();
        s1.setStudentName("张三");
        s1.setGender("男");
        s1.setAge(20);
        Student s2 = new Student();
        s2.setStudentName("王五");
        s2.setGender("女");
        s2.setAge(21);

        // 描述关系: 学生属于某个班级
        s1.setClasses(c);
        s2.setClasses(c);

        // 描述关系: 班级拥有多个学生
        c.getStudents().add(s1);
        c.getStudents().add(s2);

        // 先保存班级, 再保存学生
        session.save(c);
        session.save(s1);
        session.save(s2);

        t.commit();
        session.close();
    }
}
```

### 多对多关联

学生

```java
@Data
@EqualsAndHashCode(exclude = "courses")
public class Student {
    private Integer id;
    private String studentName;
    private String gender;
    private Integer age;
    private Set<Coursecourses = new HashSet<Course);
}
```



课程

```java
@Data
@EqualsAndHashCode(exclude = "students")
public class Course {
    private Integer courseId;
    private String courseName;
    private Set<Studentstudents = new HashSet<Student);
}
```



学生配置表

```xml
<?xml version="1.0" encoding="UTF-8" ?><!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>    
<class name="com.hibernatelearn.entity.Student" table="student">        
    <id name="id" column="id">            
        <generator class="increment"/>            
    </id>        
    <property name="studentName" column="studentname" type="string"/>        
    <property name="gender" column="gender" type="string"/>        
    <property name="age" column="age" type="integer"/>
    <!--        多对多关系映射-->        
    <set name="courses" table="studentcourse">            
        <key column="studentid"/>            
        <many-to-many column="courseid" class="com.hibernatelearn.entity.Course"/>        
    </set>

    </class>
    </hibernate-mapping>
```



课程配置表

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-mapping-3.0.dtd">
<hibernate-mapping>    
<!-- 多对多关系映射 -->    
<class name="com.hibernatelearn.entity.Course" table="course">        
    <id name="courseId" column="id">            
        <generator class="increment"/>        
    </id>        
    <property name="courseName" column="coursename" type="string"/>        
        <set name="students" table="studentcourse">            
            <key column="courseid"/>            
            <many-to-many column="studentid" class="com.hibernatelearn.entity.Student"/>
        </set>    
</class>
</hibernate-mapping>
```



多对多映射

```java
 @Test
    public void test1() {
        Session session = HibernateUtil.getSession();
        Transaction t = session.beginTransaction();
        // 创建学生对象
        Student s1 = new Student();
        s1.setStudentName("张三");
        s1.setGender("男");
        s1.setAge(20);
        Student s2 = new Student();
        s2.setStudentName("王五");
        s2.setGender("女");
        s2.setAge(21);

        // 创建课程对象
        Course c1 = new Course();
        c1.setCourseName("Java");
        Course c2 = new Course();
        c2.setCourseName("C++");

        // 设置关联关系
        s1.getCourses().add(c1);
        s1.getCourses().add(c2);
        s2.getCourses().add(c1);
        s2.getCourses().add(c2);

        session.save(c1);
        session.save(c2);
        session.save(s1);
        session.save(s2);
        t.commit();
        session.close();
    }
```

