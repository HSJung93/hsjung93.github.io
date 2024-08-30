---
title: How to Data Access with Spring Boot
date: 2022-11-05 16:09:00 +0900
categories: [SlipBox, Spring]
tags: [Boot, DataSource]
publish: false
---
# Data Access
## 1. Configure a Custom DataSource
- Spring Boot reuses your `DataSource` anywhere one is required, including database initialization.

- how to define a data source in a bean:

```java
@Bean
@ConfigurationProperties(prefix="app.datasource")
public DataSource dataSource() {
	return new FancyDataSource();
}
```

- how to define a data source by setting properties:

```
app.datasource.url=jdbc:h2:mem:mydb
app.datasource.username=sa
app.datasource.pool-size=30
```

- how to create a data source by using a `DataSourceBuilder`:

```java
@Bean
@ConfigurationProperties("app.datasource")
public DataSource dataSource() {
	return DataSourceBuilder.create().build();
}
```

- To run an app with that `DataSource`, all you need is the connection information.

```
app.datasource.url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.pool-size=30
```

- if you happen to have Hikari on the classpath, this basic setup does not work, because Hikari has no `url` property (but does have a `jdbcUrl` property):

```
app.datasource.jdbc-url=jdbc:mysql://localhost/test
app.datasource.username=dbuser
app.datasource.password=dbpass
app.datasource.maximum-pool-size=30
```

- leveraging what `DataSourceProperties` does by providing a default embedded database with a sensible username and password if no URL is provided.
- initialize a `DataSourceBuilder` from the state of any `DataSourceProperties` object.
- However, that would split your configuration into two namespaces: `url`, `username`, `password`, `type`, and `driver` on `spring.datasource` and the rest on your custom namespace (`app.datasource`). 
- To avoid that, you can redefine a custom `DataSourceProperties` on your custom namespace, as shown in the following example:

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource")
public DataSourceProperties dataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.configuration")
public HikariDataSource dataSource(DataSourceProperties properties) {
	return properties.initializeDataSourceBuilder().type(HikariDataSource.class).build();
}
```

## 85.2 Configure Two DataSources
- You must mark one of the `DataSource` instances as `@Primary`.
- configure them as follows:

```
app.datasource.first.url=jdbc:mysql://localhost/first
app.datasource.first.username=dbuser
app.datasource.first.password=dbpass
app.datasource.first.configuration.maximum-pool-size=30

app.datasource.second.url=jdbc:mysql://localhost/second
app.datasource.second.username=dbuser
app.datasource.second.password=dbpass
app.datasource.second.max-total=30
```

```java
@Bean
@Primary
@ConfigurationProperties("app.datasource.first")
public DataSourceProperties firstDataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@Primary
@ConfigurationProperties("app.datasource.first.configuration")
public HikariDataSource firstDataSource() {
	return firstDataSourceProperties().initializeDataSourceBuilder().type(HikariDataSource.class).build();
}

@Bean
@ConfigurationProperties("app.datasource.second")
public DataSourceProperties secondDataSourceProperties() {
	return new DataSourceProperties();
}

@Bean
@ConfigurationProperties("app.datasource.second.configuration")
public BasicDataSource secondDataSource() {
	return secondDataSourceProperties().initializeDataSourceBuilder().type(BasicDataSource.class).build();
}
```

# Reference
- https://docs.spring.io/spring-boot/docs/2.1.17.RELEASE/reference/html/howto-data-access.html