[![Build Status](https://travis-ci.com/pavankjadda/LiquibaseDemo.svg?branch=master)](https://travis-ci.com/pavankjadda/LiquibaseDemo)

# Liquibase database evolution with with Spring Boot 
Steps to setup Liquibase from Scratch

1. Create SpringBoot project or clone this project
2. Add `liquibase.properties` under src/main/resources folder with the following content and change DB properties accordingly
```properties
outputChangeLogFile= src/main/resources/db/db.changelog-master.xml
driver=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/liquibasedemo
username=root
password=<your password>


# Reference Properties
referenceUrl=hibernate:spring:com.liquibasedemo.model?dialect=org.hibernate.dialect.MySQLDialect&hibernate.physical_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy&hibernate.implicit_naming_strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
```
3. Add Liquibase plugin to pom.xml and change change log file locations. Replace versions based on my pom.xml file
```xml
<plugin>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-maven-plugin</artifactId>
    <version>${liquibase-maven-plugin.version}</version>
    <configuration>
        <propertyFile>src/main/resources/liquibase.properties</propertyFile>
        <changeLogFile>src/main/resources/db/db.changelog-master.xml</changeLogFile>
        <diffChangeLogFile>src/main/resources/db/changelog/${maven.build.timestamp}_changelog.xml</diffChangeLogFile>
        <logging>info</logging>
    </configuration>
   <executions>
        <execution>
            <id>update-profile</id>
            <phase>process-resources</phase>
            <goals>
                <goal>update</goal>
            </goals>
        </execution>
        <execution>
            <id>diff-profile</id>
            <phase>process-test-resources</phase>
            <goals>
                <goal>diff</goal>
            </goals>
        </execution>
    </executions>
    <dependencies>
        <dependency>
            <groupId>org.liquibase.ext</groupId>
            <artifactId>liquibase-hibernate5</artifactId>
            <version>${liquibase-hibernate5.version}</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
            <version>${spring-boot.version}</version>
        </dependency>
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>${validation-api.version}</version>
        </dependency>
        <dependency>
            <groupId>org.javassist</groupId>
            <artifactId>javassist</artifactId>
            <version>${javassist.version}</version>
        </dependency>

    </dependencies>
</plugin>
```
4. If you are using Jadira consider adding
```xml
  <systemProperties>
    <property>
      <name>jadira.usertype.useJdbc42Apis</name>
      <value>false</value>
    </property>
  </systemProperties>
```
to the `<configuration>` as suggested by [Devacfr](https://stackoverflow.com/users/3489158/devacfr) in [this stackoverflow answer](https://stackoverflow.com/questions/41409410/liquibase-hibernate5-not-working-with-liquibase-maven-plugin).

5. Change `application-dev.yml` to point to custom change log file 
```yaml
  liquibase:
    contexts: dev
    change-log: classpath:db/db.changelog-master.xml
```
6. Add following dependency to `pom.xml`
```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```
7. Create empty database `liquibasedemo` in MySQL server
8. In liquibase-maven-plugin we have 2 custom profiles(You can give any name you want) but make sure to include phase and goals as is:
    - **diff-profile** (Generates Liquibase changeset based on changes JPA entities)
    - **update-profile** (Updates the Database based on change sets generated in above step)
9. Create `db.changelog-master.xml` file in **resources/db/** directory with the following content
```xml
<?xml version="1.1" encoding="UTF-8" standalone="no"?>
<databaseChangeLog xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
                   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                   xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-3.5.xsd">
    <includeAll path="db/changelog/" />
</databaseChangeLog>
```
10. Create `changelog` folder in **resources/db/** directory
11. Delete folder `target\classes\db` before executing below commands
12. Now go to project root directory and run the following command (Below command equivalent to `$mvn liquibase:diff`) . It will generate Change Sets based JPA entities
```
$ mvn process-test-resources

```  
> if you are running this for first time, it will generate change sets for all the Entities

13. Now run update command to generate tables entity, `DATABASECHANGELOG` and `DATABASECHANGELOGLOCK` tables
```angular2
$ mvn process-resources
```
14. Now go to database see all the changes applied and tables created
15. Repeat steps 8 and 9 every time you make any changes to JPA entities
