---
title: The Domain Language of Batch
date: 2022-11-18 20:00:00 +0900
categories: [SlipBox, Spring]
tags: [Batch, Domain]
---
![Figure 1. Batch Stereotypes](/assets/img/spring-batch-reference-model.jpg)

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

![Figure 2. Job Hierarchy](/assets/img/job-stereotypes-parameters.jpg)

- The contract can be defined as: `JobInstance = Job + JobParameters`.

# JobExecution
- A `JobExecution` refers to the technical concept of a single attempt to run a Job.
> Using the EndOfDay Job described previously as an example, consider a JobInstance for 01-01-2017 that failed the first time it was run. If it is run again with the same identifying job parameters as the first run (01-01-2017), a new JobExecution is created. However, there is still only one JobInstance.
{: .prompt-info }