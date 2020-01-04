# Todo Web Application using Spring Boot and MySQL as Database

Run com.springboot.web.SpringBootFirstWebApplication as a Java Application.

Runs on default port of Spring Boot - 8080

Application uses h2 database to run the tests.


## Can be run as a Jar or a WAR

`mvn clean install` generate a war which can deployed to your favorite web server.

We will deploy to Cloud as a WAR

## Web Application

- http://localhost:8080/login with sdannarapu/dummy as credentials
- You can add, delete and update your todos
- Spring Security is used to secure the application
- `com.springboot.web.security.SecurityConfiguration` contains the in memory security credential configuration.

## command to run sping boot app from docker commandlin ineterface without docker compose 
docker run -p 8080:8080 --link=mysql --env RDS_HOSTNAME=mysql sdannarapu/todo-web-application-mysql:0.0.1-SNAPSHOT

## KOMPOSE 
Installation - https://kompose.io/installation/

```
args: #https://github.com/docker-library/mysql/issues/69
  - "--ignore-db-dir=lost+found" #CHANGE
```
## Changes from H2 Application

#### pom.xml

```
<dependency>
	<groupId>com.h2database</groupId>
	<artifactId>h2</artifactId>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
</dependency>
```

#### src/main/resources/application.properties

```
#spring.h2.console.enabled=true
#spring.h2.console.settings.web-allow-others=true

spring.jpa.hibernate.ddl-auto=update
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL55Dialect
spring.datasource.url=jdbc:mysql://localhost:3306/todos
spring.datasource.username=todos-user
spring.datasource.password=dummytodos
```

#### src/test/resources/application.properties

```
spring.jpa.hibernate.ddl-auto=create-drop
spring.datasource.driver-class-name=org.h2.Driver
spring.datasource.url=jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1
spring.datasource.username=sa
spring.datasource.password=sa
```

#### public class Todo

```
@Size(min=10, message="Enter at least 10 Characters...")
@Column(name="description")
private String desc;
```
## My SQL

### Launching MySQL using Docker

```
docker run --detach --env MYSQL_ROOT_PASSWORD=rootpassword --env MYSQL_USER=user --env MYSQL_PASSWORD=password --env MYSQL_DATABASE=db1 --name mysql --publish 3307:3307 mysql:5.7
```

### Launching Web App using Docker

Using Link

```
docker container run -p 8080:8080 --link=mysql -e RDS_HOSTNAME=mysql  sdannarapu/todo-web-application-mysql:0.0.1-SNAPSHOT
```


### My SQL Shell Client

- https://dev.mysql.com/downloads/shell/

- Install on mac using `brew install caskroom/cask/mysql-shell`.


```
MySQL Shell 8.0.16

Copyright (c) 2016, 2019, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its affiliates.
Other names may be trademarks of their respective owners.

Type '\help' or '\?' for help; '\quit' to exit.

MySQL  JS > \connect user@localhost:3306
Creating a session to 'user@localhost:3306'
Please provide the password for 'user@localhost:3307': password
Save password for 'user@localhost:3306'? [Y]es/[N]o/Ne[v]er (default No): v
Fetching schema names for autocompletion... Press ^C to stop.
Your MySQL connection id is 37
Server version: 5.7.26 MySQL Community Server (GPL)
No default schema selected; type \use <schema> to set one.

 MySQL  localhost:3306 ssl  JS > \sql
Switching to SQL mode... Commands end with ;

 MySQL  localhost:3306 ssl  SQL > use todos
Default schema set to `todos`.
Fetching table and column names from `todos` for auto-completion... Press ^C to stop.

 MySQL  localhost:3306 ssl  todos  SQL > select * from todo ;
+----+--------------+---------+----------------------------+-------------+
| id | description  | is_done | target_date                | user        |
+----+--------------+---------+----------------------------+-------------+
|  1 | Default Desc | 0       | 2019-06-26 18:30:00.000000 | Srikanth |
+----+--------------+---------+----------------------------+-------------+
1 row in set (0.0032 sec)

```

### Create Todo Table for Production

```
create table hibernate_sequence (next_val bigint) engine=InnoDB
insert into hibernate_sequence values ( 1 )
create table todo (id integer not null, description varchar(255), is_done bit not null, target_date datetime(6), user varchar(255), primary key (id)) engine=InnoDB
```

### trouble shoot issue
```
C:\SRIKANT\kubernetes-workspace\todo-web-application-mysql>docker-compose up
Creating network "todo-web-application-mysql_default" with the default driver
Starting todo-web-application-mysql_mysql_1 ... error
ERROR: for todo-web-application-mysql_mysql_1  Cannot start service mysql: network 6b8f3b0f6e15c840b29c9313568d55612b4936a2ec574cbcd0636def1c7fa8ae not found

ERROR: for mysql  Cannot start service mysql: network 6b8f3b0f6e15c840b29c9313568d55612b4936a2ec574cbcd0636def1c7fa8ae not found
ERROR: Encountered errors while bringing up the project.



fix
docker-compose down which will delete the containers + network.

OR

docker ps -a find the stuck container and then remove it with docker rm -f CONTAINER_XX

OR

Just restart Docker
```