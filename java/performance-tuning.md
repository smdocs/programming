Jave Performance Tuning
========================
Tuning Java
============

####Measuring Performance

In order for our tuning to be meaningful we need  a way to measure the performance improvement. Lets understand two key performance
metrics: latency and throughput.

**Latency** measures the end-to-end processing time for an operation. In a distributed environment we determine latency by measuring the time for the full round-trip between sending the request and receiving the response. In those cases, latency is measured from the client machine and includes the network overhead as well.
Throughput measures the  number of messages that a server processes during a specific time interval (e.g. per second). 

**Throughput** is calculated using the equation:
Throughput = number of requests / time to complete the requests

Ideally we like to have maximum throughput with minimal latency. However, in a well-designed server, there is a tradeoff 
between throughput and latency. For example as shown by the graph below, if you need more throughput, you have to the run 
system with more concurrency, but then average latency also increases. Usually, you need to achieve maximum throughput 
while keeping latency within some acceptable limit. For example, you might choose maximum throughput in a range where  latency 
is less than 10ms.

####Performance Tuning Checklist

Following is a summary checklist of the steps we discussed. This may not tell you how to profile everything, but it can save you from many pitfalls.

1. Check the load average of the machine. If it is > 4XNumberOfCores, then your machine is running at full throttle and you can skip to the CPU tuning part.

2. Are you hitting the system hard enough? Try simulate more client by increasing the number of threads in your client programs. If that improves throughput then increase the load until throughput is maximized.

3. Are your threads idling? If you have too many locks or too few threads, the system might not provide full throughput. Use a profiler to view the lock profile and try to remove them. While some locks are neededmost are not. 

4. Try increasing the number of  threads in your thread pools, and check  whether  that improves throughput.

5. Now check the CPU profile for hotspots

6. Study the tree view and make sure any hot spots are where expected. For example, an  XML parser would be expected to consume a  lot of CPU. But if you see something unexpected taking too much CPU, there is something to fix

7. Check memory and GC, if GC throughput is less than 90% it is time to tune the GC. If memory runs on the brink (check for paging), add more and see if that helps.

8. Look at DB profiles, and ensure that what takes most load are expected.

9. Look at network IO profile. Go to hot spots, make sure all major writes are what you expect. If there is some unnecessary IO, this will show it.

10. Verify that resources such as underlying network and disk mounting are in place

####Resources
1. [Tuning Java servers](http://www.infoq.com/articles/Tuning-Java-Servers)
