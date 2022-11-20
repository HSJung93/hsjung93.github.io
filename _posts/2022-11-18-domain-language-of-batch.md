---
title: The Domain Language of Batch
date: 2022-11-18 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Batch, Domain]
---

![Figure 1. Batch Stereotypes](/assets/img/spring-batch-reference-model.png)

- A Job has one to many steps
- Each of which has exactly one ItemReader, one ItemProcessor, and one ItemWriter. 
- A job needs to be launched (with JobLauncher)
- Metadata about the currently running process needs to be stored (in JobRepository).

# Job
- A Job is an entity that encapsulates an entire batch process.
- a Job is simply a container for Step instances.
- The job configuration contains:
    + The simple name of the job.
    + Definition and ordering of Step instances.
    + Whether or not the job is restartable.
- For those who use Java configuration, Spring Batch provides a default implementation of the Job interface in the form of the SimpleJob class, which creates some standard functionality on top of Job.
- When using java based configuration, a collection of builders is made available for the instantiation of a Job, as shown in the following example:

```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .build();
}
```

# JobInstance

- There is one 'EndOfDay' job, but each individual run of the Job must be tracked separately. In the case of this job, there is one logical JobInstance per day. 
- For example, there is a January 1st run, a January 2nd run, and so on. If the January 1st run fails the first time and is run again the next day, it is still the January 1st run. (Usually, this corresponds with the data it is processing as well, meaning the January 1st run processes data for January 1st). 
- Therefore, each JobInstance can have multiple executions, and only one JobInstance corresponding to a particular Job and identifying JobParameters can run at a given time.
- The definition of a `JobInstance` has absolutely no bearing on the data to be loaded. It is entirely up to the `ItemReader` implementation to determine how data is loaded. 
- Using a new `JobInstance` means 'start from the beginning', and using an existing instance generally means 'start from where you left off'.

# JobParameters
- A `JobParameters` object holds a set of parameters used to start a batch job.


![Figure 2. Job Hierarchy](/assets/img/job-stereotypes-parameters.png)

- The contract can be defined as: `JobInstance = Job + JobParameters`.

# JobExecution
- A `JobExecution` refers to the technical concept of a single attempt to run a Job.
> Using the EndOfDay Job described previously as an example, consider a JobInstance for 01-01-2017 that failed the first time it was run. If it is run again with the same identifying job parameters as the first run (01-01-2017), a new JobExecution is created. However, there is still only one JobInstance.
{: .prompt-info }

- A `Job` defines what a job is and how it is to be executed.
- A `JobInstance` is a purely organizational object to group executions together, primarily to enable correct restart semantics. 
- A `JobExecution`, however, is the primary storage mechanism for what actually happened during a run and contains many more properties that must be controlled and persisted.

# Step
- A `Step` is a domain object that encapsulates an independent, sequential phase of a batch job. Therefore, every Job is composed entirely of one or more steps.
- A simple `Step` might load data from a file into the database, requiring little or no code (depending upon the implementations used). A more complex `Step` may have complicated business rules that are applied as part of the processing. As with a `Job`, a `Step` has an individual `StepExecution` that correlates with a unique `JobExecution`.


![Figure 3. Job Hierarchy With Steps](/assets/img/jobHeirarchyWithSteps.png)

# StepExecution

- `Step` executions are represented by objects of the `StepExecution` class. Each execution contains a reference to its corresponding step and `JobExecution` and transaction related data, such as commit and rollback counts and start and end times.
- Additionally, each step execution contains an `ExecutionContext`, which contains any data a developer needs to have persisted across batch runs, such as statistics or state information needed to restart.

# ExecutionContext

- An `ExecutionContext` represents a collection of key/value pairs that are persisted and controlled by the framework in order to allow developers a place to store persistent state that is scoped to a `StepExecution` object or a `JobExecution` object.
- The `ExecutionContext` can also be used for statistics that need to be persisted about the run itself.
- Note that there is at least one `ExecutionContext` per `JobExecution` and one for every `StepExecution`.

# JobRepository
- It provides CRUD operations for JobLauncher, Job, and Step implementations. When a Job is first launched, a JobExecution is obtained from the repository, and, during the course of execution, StepExecution and JobExecution implementations are persisted by passing them to the repository.
- When using Java configuration, the @EnableBatchProcessing annotation provides a JobRepository as one of the components automatically configured out of the box.

# JobLauncher
- `JobLauncher` represents a simple interface for launching a Job with a given set of JobParameters, as shown in the following example:


```java
public interface JobLauncher {

public JobExecution run(Job job, JobParameters jobParameters)
            throws JobExecutionAlreadyRunningException, JobRestartException,
                   JobInstanceAlreadyCompleteException, JobParametersInvalidException;
}
```

- It is expected that implementations obtain a valid JobExecution from the JobRepository and execute the Job.

# Item Reader
- `ItemReader` is an abstraction that represents the retrieval of input for a Step, one item at a time. 
- When the `ItemReader` has exhausted the items it can provide, it indicates this by returning `null`.

# Item Writer
- `ItemWriter` is an abstraction that represents the output of a Step, one batch or chunk of items at a time.
- While the `ItemReader` reads one item, and the `ItemWriter` writes them, the `ItemProcessor` provides an access point to transform or apply other business processing.

# Reference
- https://docs.spring.io/spring-batch/docs/current/reference/html/domain.html#domainLanguageOfBatch