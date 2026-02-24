---
title: SQL Injection Risks and Solutions in MyBatis
date: 2024-08-28 16:07:19
tags:
  - SQL
  - MyBatis
categories:
  - [SQL, SQL Injection]
cover: https://pics.findfuns.org/SQL-injection.jpg
---



## SQL Injection

As a classic web security issue, SQL injection is something that almost every student who has taken a cybersecurity course has encountered to some extent.

The underlying principle is actually quite simple: by inserting specially crafted characters into an SQL statement, an attacker can manipulate the query condition into a tautology, thereby bypassing checks such as username and password validation and directly retrieving data from the database.

In more severe cases, SQL injection can even lead to database corruption or data loss, causing extremely serious consequences.

Below is a simple example of SQL injection.

Suppose we have a login form where users enter a username and password. The backend validates these credentials and returns the corresponding data.

```sql
SELECT * FROM users WHERE username = ${} AND password = ${};
```

Now, if a malicious user enters special input such as:

```sql
' OR '1'='1
```

The original SQL statement becomes:

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '';
```

Since `'1'='1'` is always true, the condition becomes a tautology, and the attacker can retrieve all records from the database.

Another example:

```sql
admin'; DROP TABLE users; --
```

After concatenation, the SQL becomes:

```sql
SELECT * FROM users WHERE username = 'admin'; DROP TABLE users; --'
```

After executing the first query, the `DROP TABLE` statement will also be executed, potentially deleting the entire `users` table.

---

## Prevention Measures

1. Use precompiled (prepared) SQL statements to avoid directly concatenating user input into SQL queries, such as Java’s `PreparedStatement`.
2. Use ORM frameworks such as MyBatis or Hibernate.

Below, we’ll introduce how to use the MyBatis framework to perform SQL queries. (Of course, MyBatis can also be integrated into Spring Boot, but that will not be covered here.)

---

## Step 1: Add Dependencies in `pom.xml`

```xml
<!-- MySQL Driver -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
</dependency>

<!-- MyBatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>

<!-- Log4j2 -->
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.20.0</version>
</dependency>
```

---

## MyBatis Configuration (`mybatis-config.xml`)

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <settings>
        <!-- Enable SQL logging -->
        <setting name="logImpl" value="LOG4J2"/>
    </settings>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://localhost:3306/person"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/zzb/mapper/PersonMapper.xml"/>
    </mappers>
</configuration>
```

---

## Logging Configuration (Log4j2 Example)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration xmlns="http://logging.apache.org/log4j/2.0/config">

    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%5level [%t] - %msg%n"/>
        </Console>
    </Appenders>

    <Loggers>
        <Logger name="com.zzb.mapper.PersonMapper" level="debug"/>
        <!-- Set log level to debug to view detailed SQL statements -->
        <Root level="error">
            <AppenderRef ref="stdout"/>
        </Root>
    </Loggers>

</Configuration>
```

---

## Mapper Interface

```java
package com.zzb.mapper;

public interface PersonMapper {
    public List<User> selectUser(@Param(value="name") String name);
}
```

---

## Mapper XML

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zzb.mapper.PersonMapper">

    <select id="selectUser" resultType="com.zzb.pojo.User">
        SELECT * FROM person_info WHERE name = #{name}
    </select>

</mapper>
```

---

## POJO Class

```java
package com.zzb.pojo;

public class User {
    private String id;
    private String name;

    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }

    public User() {
    }

    // getter, setter and toString
}
```

---

## MyBatis Configuration Class

```java
package com.zzb.config;

import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.apache.ibatis.io.Resources;

import java.io.InputStream;

public class MyBatisConfig {
    private static final SqlSessionFactory sqlSessionFactory;

    static { 
        // Static block executes when the class is loaded
        try {
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static SqlSessionFactory getSqlSessionFactory() {
        return sqlSessionFactory;
    }
}
```

---

## Test Code

```java
package com.zzb;

import com.zzb.config.MyBatisConfig;
import com.zzb.mapper.PersonMapper;
import com.zzb.pojo.User;
import org.apache.ibatis.session.SqlSession;

import java.util.List;

public class Main {
    public static void main(String[] args) {
        try (SqlSession session = MyBatisConfig.getSqlSessionFactory().openSession()) {
            PersonMapper mapper = session.getMapper(PersonMapper.class);
            List<User> user = mapper.selectUser("zzb");
            for(User usr: user) {
                System.out.println(usr);
            }
        }
    }
}
```

---

## Output

```java
DEBUG [main] - ==>  Preparing: SELECT * FROM person_info WHERE name = ?
DEBUG [main] - ==> Parameters: zzb(String)
DEBUG [main] - <==      Total: 1
User{id='zzb', name='zzb'}
```

From the logs, we can clearly see that a precompiled SQL statement is being used. The user-provided parameter does not become part of the SQL structure itself; instead, it is bound as a parameter value. This effectively prevents SQL injection.

MyBatis achieves this through JDBC’s underlying `PreparedStatement`. The SQL statement is precompiled before execution, and parameters are safely bound during runtime.

To better understand the difference, consider the following example.

This time, instead of using `#{}`, we use `${}`.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.zzb.mapper.PersonMapper">

    <select id="selectUser" resultType="com.zzb.pojo.User">
        SELECT * FROM person_info WHERE name = ${name}
    </select>

</mapper>
```

The output log now becomes:

```java
DEBUG [main] - ==>  Preparing: SELECT * FROM person_info WHERE name = 'zzb'
DEBUG [main] - ==> Parameters:
DEBUG [main] - <==      Total: 1
User{id='zzb', name='zzb'}
```

It is clear that when using `${}`, the SQL is directly constructed by string concatenation. This approach does not prevent SQL injection and is highly risky.

In contrast, `#{}` is much safer because it relies on precompilation and parameter binding, fundamentally preventing SQL injection.