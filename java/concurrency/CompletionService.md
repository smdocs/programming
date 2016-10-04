#Completion Service

A [CompletionService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorCompletionService.html) that uses a supplied Executor to execute tasks. This class arranges that submitted tasks are, upon completion, placed on a queue accessible using take. The class is lightweight enough to be suitable for transient use when processing groups of tasks.

<b>ExecutorService = incoming queue + worker threads</b>

<b>CompletionService = incoming queue + worker threads + output queue</b>
  
- The difference between  Normal Executor Service and Completion Service is visible at runtime, Normal Executor Service always waits  for the
next thread to finish the job, the client needs to wait until a big bunch of threads return all together.

- Completion Service is behaves differently, it follows a producer/consumer philosophy: as soon a thread is done, it puts the result into a non blocking queue so that the consumer can take it.

```java
ExecutorService svc = Executors.newFixedThreadPool(NUM_OF_THREADS);
@SuppressWarnings("rawtypes")
CompletionService<String> csvc = new ExecutorCompletionService(svc);
```
