---
title: Spring Boot Logging Integration
date: 2024-08-30 19:05:59
tags:
  - Spring Boot
  - Log
categories:
  - [Spring Boot, Log]
cover: https://pics.findfuns.org/springboot.png
---


## Introduction

Up to now, Java has developed a complete logging system, which is divided into a logging facade and specific logging implementations. The logging facade is equivalent to an interface, while the various logging frameworks are actually different implementations of that facade.

Java logging loading relies on **SPI**. Unlike API, in the SPI pattern the interface is on the caller’s side, while developers only provide the concrete implementations. In the API pattern, both the interface and the concrete implementation are on the developer’s side, and the caller does not need to care about the specific implementation, but only needs to call the interface provided by the developer.

The SPI mechanism gives programs the ability to load dynamically. During program runtime, it dynamically scans for the corresponding implementation classes of an interface and loads them.

For example, the logging facade Slf4j in Java has corresponding implementations such as Spring Boot’s built-in and default Logback, as well as Log4j and Log4j2. When used, it dynamically loads the Slf4j implementation class that exists in the classpath and uses it. This is the SPI mechanism.

<img src="https://pics.findfuns.org/java-log-structure.png" style="zoom:50%;" />

### Logback

Logback is the default logging implementation used by Spring Boot. It can be used directly even without configuration.

```java
@SpringBootApplication
public class RedisUsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisUsageApplication.class, args);
        Logger log = LoggerFactory.getLogger("RedisUsageApplication.class");
        log.warn("Easy bro, this is a fake warn lol.");
        log.error("Easy bro, this is a fake error lol.");
    }
}
```

You can also use the annotation `@Slf4j` provided by Lombok to output logs, which is more concise.

```java
@SpringBootApplication
@Slf4j
public class RedisUsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisUsageApplication.class, args);
        log.warn("Easy bro, this is a fake warn lol.");
        log.error("Easy bro, this is a fake error lol.");
    }
}
```

<img src="https://pics.findfuns.org/logback-default.png" style="zoom:200%;" />

However, the default configuration still cannot precisely meet various needs, so custom configuration is required.

Create `logback.xml` or `logback-spring.xml` under the classpath.

```xml
<!--
Logback log levels: ERROR > WARN > INFO > DEBUG > TRACE
-->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <!-- Define the log file name -->
    <property name="APP_NAME" value="log-files" />
    <!-- Define the log file path -->
    <property name="LOG_PATH" value="${user.home}/${APP_NAME}" />
    <!-- Define the log file name -->
    <property name="LOG_FILE" value="${LOG_PATH}/redis-usage.log" />

    <!-- Rolling log recording: first record logs to the specified file, and when certain conditions are met, record logs to other files -->
    <appender name="APPLICATION"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- Specify the log file name -->
        <file>${LOG_FILE}</file>
        <!--
          When rolling occurs, determines the behavior of RollingFileAppender, involving file moving and renaming
          TimeBasedRollingPolicy: The most commonly used rolling strategy. It formulates the rolling strategy based on time and is responsible for both rolling and triggering rolling.
          -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--
           The storage location and file name of the files generated during rolling
           %d{yyyy-MM-dd}: Roll logs daily
           %i: When the file size exceeds maxFileSize, roll files according to i
           -->
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <!--
           Optional node that controls the maximum number of archived files to retain. When exceeded, old files will be deleted.
           For example, if rolling daily and maxHistory is set to 7, only the latest 7 days of files will be kept.
           Note that when deleting old files, directories created for archiving will also be deleted.
           -->
            <maxHistory>7</maxHistory>
            <!--
           When the log file exceeds the size specified by maxFileSize, roll according to %i mentioned above.
           Note that configuring SizeBasedTriggeringPolicy alone cannot achieve size-based rolling.
           You must configure timeBasedFileNamingAndTriggeringPolicy.
           -->
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>50MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <!-- Log output format: -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
        </layout>
    </appender>

    <!-- ch.qos.logback.core.ConsoleAppender represents console output -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--
       Log output format:
           %d represents date and time, %green green
           %thread represents thread name, %magenta magenta
           %-5level: level left-aligned with 5-character width, %highlight highlighted color
           %logger{36} means logger name up to 36 characters, otherwise split by dot, %yellow yellow
           %msg: log message
           %n is a newline
       -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%green(%d{yyyy-MM-dd HH:mm:ss.SSS}) [%magenta(%thread)] %highlight(%-5level) %yellow(%logger{36}): %msg%n</pattern>
        </layout>
    </appender>

    <!--
   root and logger have a parent-child relationship. If not specifically defined, the default is root.
   Any class will only correspond to one logger, either a defined logger or root.
   The key is to find that logger and then determine its appender and level.
   -->
    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="APPLICATION" />
    </root>
</configuration>
```

In log configuration, there are several tags:

- property: You can customize variable names and values to facilitate path concatenation.
- appender: Each appender corresponds to a log output destination.
  - layout: Log output format
  - rollingPolicy: Log rolling strategy

Through these tags, full customization of logging can be achieved.

When using Logback, you can add Logback configuration in `application.yml` or `application.properties`, but it is not necessary, because Spring Boot’s default behavior is to scan whether there is a `logback.xml` or `logback-spring.xml` under the classpath.

```yaml
logging:
  config: classpath:logback-spring.xml
```

![](https://pics.findfuns.org/logback-customized.png)

At the same time, the log files can be found in the corresponding directory.

<img src="https://pics.findfuns.org/log-files.png" style="zoom:50%;" />

## Log4j2

Log4j2 and Logback are both implementations of Slf4j, but in practical applications they cannot coexist; otherwise, an error will occur and the logging implementation class cannot be loaded properly.

Therefore, when using Log4j2, you need to explicitly exclude the Logback dependency in `pom.xml` and add the Log4j2 dependency.

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <!-- Exclude Logback dependency -->
  	<exclusions>
      <exclusion>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
  	</exclusions>
</dependency>
<!-- Log4j2 dependency -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

Create `log4j2.xml` under the classpath.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Log4j2 log levels: OFF > FATAL > ERROR > WARN > INFO > DEBUG > ALL -->
<Configuration>
    <!--<Configuration status="WARN" monitorInterval="30"> -->
    <properties>
        <property name="LOG_HOME">./service-logs</property>
    </properties>
    <Appenders>
        <!--*********************Console Log***********************-->
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <!-- Set log format and color -->
            <PatternLayout
                    pattern="%style{%d{ISO8601}}{bright,green} %highlight{%-5level} [%style{%t}{bright,blue}] %style{%C{}}{bright,yellow}: %msg%n%style{%throwable}{red}"
                    disableAnsi="false" noConsoleNoAnsi="false"/>
        </Console>

        <!--*********************File Log***********************-->
        <!-- all level logs -->
        <RollingFile name="allFileAppender"
                     fileName="${LOG_HOME}/all.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/all-%d{yyyy-MM-dd}-%i.log.gz">
            <!-- Set log format -->
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!-- Set log file splitting parameters -->
                <!--<OnStartupTriggeringPolicy/>-->
                <!-- Set base log file size, roll when exceeding this size -->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <!-- Set time-based rolling, depends on filePattern -->
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <!-- Set maximum number of log files -->
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>

        <!-- debug level logs -->
        <RollingFile name="debugFileAppender"
                     fileName="${LOG_HOME}/debug.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/debug-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <!-- Filter out info and higher level logs -->
                <ThresholdFilter level="info" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!--<OnStartupTriggeringPolicy/>-->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>

        <!-- info level logs -->
        <RollingFile name="infoFileAppender"
                     fileName="${LOG_HOME}/info.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/info-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <!-- Filter out warn and higher level logs -->
                <ThresholdFilter level="warn" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!--<OnStartupTriggeringPolicy/>-->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy interval="1" modulate="true" />
            </Policies>
            <!--<DefaultRolloverStrategy max="100"/>-->
        </RollingFile>

        <!-- warn level logs -->
        <RollingFile name="warnFileAppender"
                     fileName="${LOG_HOME}/warn.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/warn-%d{yyyy-MM-dd}-%i.log.gz">
            <Filters>
                <!-- Filter out error and higher level logs -->
                <ThresholdFilter level="error" onMatch="DENY" onMismatch="NEUTRAL"/>
            </Filters>
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!--<OnStartupTriggeringPolicy/>-->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>

        <!-- error and higher level logs -->
        <RollingFile name="errorFileAppender"
                     fileName="${LOG_HOME}/error.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/error-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <!--<OnStartupTriggeringPolicy/>-->
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>

        <!-- JSON format error level logs -->
        <RollingFile name="errorJsonAppender"
                     fileName="${LOG_HOME}/error-json.log"
                     filePattern="${LOG_HOME}/error-json-%d{yyyy-MM-dd}-%i.log.gz">
            <JSONLayout compact="true" eventEol="true" locationInfo="true"/>
            <Policies>
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy interval="1" modulate="true"/>
            </Policies>
        </RollingFile>
    </Appenders>

    <Loggers>
        <!-- Root logger configuration -->
        <Root level="debug">
            <AppenderRef ref="allFileAppender" level="all"/>
            <AppenderRef ref="consoleAppender" level="debug"/>
            <AppenderRef ref="debugFileAppender" level="debug"/>
            <AppenderRef ref="infoFileAppender" level="info"/>
            <AppenderRef ref="warnFileAppender" level="warn"/>
            <AppenderRef ref="errorFileAppender" level="error"/>
            <AppenderRef ref="errorJsonAppender" level="error"/>
        </Root>

        <!-- spring logs -->
        <Logger name="org.springframework" level="info"/>
        <!-- druid datasource logs -->
        <Logger name="druid.sql.Statement" level="warn"/>
        <!-- mybatis logs -->
        <Logger name="com.mybatis" level="warn"/>
    </Loggers>

</Configuration>
```

Similarly, you can configure log output using tags such as appender and PatternLayout.

Then add the Log4j2 configuration in `application.properties`.

```properties
logging.config=classpath:log4j2.xml
```

Test it. You can use Lombok’s `@Log4j2` annotation to use logging.

```java
@SpringBootApplication
@Log4j2
public class Log4j2UsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(Log4j2UsageApplication.class, args);
        log.warn("this is form log4j2.");
    }
}
```

![](https://pics.findfuns.org/log4j2.png)

At the same time, according to the configuration in `log4j2.xml`, you can find log records categorized by level in the corresponding directory.

<img src="https://pics.findfuns.org/log4j2-files.png" style="zoom:50%;" />
## Introduction

Over the years, Java has developed a complete logging system, which is divided into a logging facade and specific logging implementations. The logging facade acts like an interface, while the various logging frameworks are different implementations of that facade.

Java logging loading relies on **SPI**. Unlike API, in the SPI pattern the interface is on the caller’s side, while developers only provide concrete implementations. In the API pattern, both the interface and the implementation are provided by the developer, and the caller only needs to use the interface without caring about the implementation details.

The SPI mechanism gives programs the ability to load implementations dynamically. During runtime, the program scans for implementation classes corresponding to an interface and loads them dynamically.

For example, the logging facade Slf4j in Java has multiple implementations, such as Spring Boot’s default Logback, as well as Log4j and Log4j2. At runtime, it dynamically loads the corresponding Slf4j implementation found in the classpath. This is the SPI mechanism in action.

<img src="https://pics.findfuns.org/java-log-structure.png" style="zoom:50%;" />

### Logback

Logback is the default logging implementation used by Spring Boot. It can be used directly even without configuration.

```java
@SpringBootApplication
public class RedisUsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisUsageApplication.class, args);
        Logger log = LoggerFactory.getLogger("RedisUsageApplication.class");
        log.warn("Easy bro, this is a fake warn lol.");
        log.error("Easy bro, this is a fake error lol.");
    }
}
```

You can also use the Lombok annotation `@Slf4j` to output logs, which is more concise.

```java
@SpringBootApplication
@Slf4j
public class RedisUsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(RedisUsageApplication.class, args);
        log.warn("Easy bro, this is a fake warn lol.");
        log.error("Easy bro, this is a fake error lol.");
    }
}
```

<img src="https://pics.findfuns.org/logback-default.png" style="zoom:200%;" />

However, the default configuration cannot precisely meet various needs, so custom configuration is often required.

Create `logback.xml` or `logback-spring.xml` under the classpath:

```xml
<!--
Logback log levels: ERROR > WARN > INFO > DEBUG > TRACE
-->
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml" />
    <!-- Define the log file name -->
    <property name="APP_NAME" value="log-files" />
    <!-- Define the log file path -->
    <property name="LOG_PATH" value="${user.home}/${APP_NAME}" />
    <!-- Define the full log file name -->
    <property name="LOG_FILE" value="${LOG_PATH}/redis-usage.log" />

    <!-- Rolling file appender -->
    <appender name="APPLICATION"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_FILE}</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE}.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxHistory>7</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy 
                class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>50MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [ %thread ] - [ %-5level ] [ %logger{50} : %line ] - %msg%n</pattern>
        </layout>
    </appender>

    <!-- Console output -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>%green(%d{yyyy-MM-dd HH:mm:ss.SSS}) [%magenta(%thread)] %highlight(%-5level) %yellow(%logger{36}): %msg%n</pattern>
        </layout>
    </appender>

    <root level="info">
        <appender-ref ref="CONSOLE" />
        <appender-ref ref="APPLICATION" />
    </root>
</configuration>
```

Key tags in Logback configuration:

- `property`: Define custom variables for easier path construction.
- `appender`: Each appender represents a log output destination.
  - `layout`: Defines the log output format.
  - `rollingPolicy`: Defines the log rolling strategy.

Spring Boot automatically scans for `logback.xml` or `logback-spring.xml` under the classpath.

```yaml
logging:
  config: classpath:logback-spring.xml
```

![](https://pics.findfuns.org/logback-customized.png)

Log files can be found in the corresponding directory:

<img src="https://pics.findfuns.org/log-files.png" style="zoom:50%;" />

## Log4j2

Both Log4j2 and Logback are implementations of Slf4j, but they cannot coexist in the same project. Otherwise, an error will occur because multiple logging implementations are detected.

When using Log4j2, you need to exclude Logback in `pom.xml` and add the Log4j2 dependency:

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
      <exclusion>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-logging</artifactId>
      </exclusion>
  </exclusions>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

Create `log4j2.xml` under the classpath:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Log4j2 log levels: OFF > FATAL > ERROR > WARN > INFO > DEBUG > ALL -->
<Configuration>
    <properties>
        <property name="LOG_HOME">./service-logs</property>
    </properties>
    <Appenders>
        <Console name="consoleAppender" target="SYSTEM_OUT">
            <PatternLayout
                pattern="%style{%d{ISO8601}}{bright,green} %highlight{%-5level} [%style{%t}{bright,blue}] %style{%C{}}{bright,yellow}: %msg%n%style{%throwable}{red}"
                disableAnsi="false" noConsoleNoAnsi="false"/>
        </Console>

        <RollingFile name="allFileAppender"
                     fileName="${LOG_HOME}/all.log"
                     filePattern="${LOG_HOME}/$${date:yyyy-MM}/all-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout>
                <pattern>%d %p %C{} [%t] %m%n</pattern>
            </PatternLayout>
            <Policies>
                <SizeBasedTriggeringPolicy size="100 MB"/>
                <TimeBasedTriggeringPolicy/>
            </Policies>
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>
    </Appenders>

    <Loggers>
        <Root level="debug">
            <AppenderRef ref="allFileAppender" level="all"/>
            <AppenderRef ref="consoleAppender" level="debug"/>
        </Root>

        <Logger name="org.springframework" level="info"/>
        <Logger name="druid.sql.Statement" level="warn"/>
        <Logger name="com.mybatis" level="warn"/>
    </Loggers>
</Configuration>
```

Add the configuration in `application.properties`:

```properties
logging.config=classpath:log4j2.xml
```

Test with Lombok’s `@Log4j2`:

```java
@SpringBootApplication
@Log4j2
public class Log4j2UsageApplication {

    public static void main(String[] args) {
        SpringApplication.run(Log4j2UsageApplication.class, args);
        log.warn("this is from log4j2.");
    }
}
```

![](https://pics.findfuns.org/log4j2.png)

According to the `log4j2.xml` configuration, you can find logs categorized by level in the corresponding directory:

<img src="https://pics.findfuns.org/log4j2-files.png" style="zoom:50%;" />

