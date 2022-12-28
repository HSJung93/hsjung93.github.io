---
title: [WIP] Spring Batch 2. Configuring and Running a Job
date: 2022-12-06 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Batch, Job]
---

- many options about how a Job can be run and how its metadata can be stored during that run.

# Configuring a Job

## Restartability

- The launching of a `Job` is considered to be a “restart” if a `JobExecution` already exists for the particular `JobInstance`.

-  If a `Job` should never be restarted but should always be run as part of a new `JobInstance`, you can set the restartable property to false.

```java
@Bean
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .preventRestart()
                     ...
                     .build();
}
```

- To phrase it another way, setting restartable to false means “this Job does not support being started again”. 

- Restarting a Job that is not restartable causes a JobRestartException to be thrown:

```java
Job job = new SimpleJob();
job.setRestartable(false);

JobParameters jobParameters = new JobParameters();

JobExecution firstExecution = jobRepository.createJobExecution(job, jobParameters);
jobRepository.saveOrUpdate(firstExecution);

try {
    jobRepository.createJobExecution(job, jobParameters);
    fail();
}
catch (JobRestartException e) {
    // expected
}
```

## Intercepting Job Execution

- During the course of the execution of a `Job`, it may be useful to be notified of various events in its lifecycle so that custom code can be run.
- `SimpleJob` allows for this by calling a `JobListener` at the appropriate time:

```java
public interface JobExecutionListener {

    void beforeJob(JobExecution jobExecution);

    void afterJob(JobExecution jobExecution);
}
```

- You can add `JobListeners` to a `SimpleJob` by setting listeners on the job.

```java
@Bean
public Job footballJob(JobRepository jobRepository) {
    return new JobBuilder("footballJob", jobRepository)
                     .listener(sampleListener())
                     ...
                     .build();
}
```

- Note that the `afterJob` method is called regardless of the success or failure of the `Job`. 
- If you need to determine success or failure, you can get that information from the `JobExecution`:

```java
public void afterJob(JobExecution jobExecution){
    if (jobExecution.getStatus() == BatchStatus.COMPLETED ) {
        //job success
    }
    else if (jobExecution.getStatus() == BatchStatus.FAILED) {
        //job failure
    }
}
```

## Java Configuration

- There are three components for the Java-based configuration: the `@EnableBatchProcessing` annotation and two builders.
- `@EnableBatchProcessing` provides a base configuration for building batch jobs. Within this base configuration, an instance of `StepScope` and `JobScope` are created, in addition to a number of beans being made available to be autowired: `jobRepository`, `jobLauncher`, `jobRegistry`, `jobExplorer`, `jobOperator`.
- how to provide a custom data source and transaction manager:

```java
@Configuration
@EnableBatchProcessing(dataSourceRef = "batchDataSource", transactionManagerRef = "batchTransactionManager")
public class MyJobConfiguration {

	@Bean
	public DataSource batchDataSource() {
		return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.HSQL)
				.addScript("/org/springframework/batch/core/schema-hsqldb.sql")
				.generateUniqueName(true).build();
	}

	@Bean
	public JdbcTransactionManager batchTransactionManager(DataSource dataSource) {
		return new JdbcTransactionManager(dataSource);
	}

	public Job job(JobRepository jobRepository) {
		return new JobBuilder("myJob", jobRepository)
				//define job flow as needed
				.build();
	}

}
```

- programmatic way of configuring base infrastrucutre beans is provided through the DefaultBatchConfiguration class. 

```java
@Configuration
class MyJobConfiguration extends DefaultBatchConfiguration {

	@Bean
	public Job job(JobRepository jobRepository) {
		return new JobBuilder("job", jobRepository)
				// define job flow as needed
				.build();
	}

	@Override
	protected Charset getCharset() {
		return StandardCharsets.ISO_8859_1;
	}
}
```

> @EnableBatchProcessing should not be used with DefaultBatchConfiguration.
{: .prompt-warning}


# Reference

- https://docs.spring.io/spring-batch/docs/current/reference/html/job.html#configureJob