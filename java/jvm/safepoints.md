#JVM Safepoints

In HotSpot JVM <b>Stop-the-World pause mechanism</b> is called safepoint. During safepoint all threads running java code are suspended. 
Threads running native code may continue to run as long as they do not interact with JVM (attempt to access Java objects via JNI, call 
Java method or return from native to java, will suspend thread until end of safepoint).

Stopping all threads are required to ensure what safepoint initiator have exclusive access to JVM data structures and can do crazy things
like moving objects in heap or replacing code of method which is currently running (On-Stack-Replacement).

####How safepoints work?

Safepoint protocol in HotSpot JVM is collaborative. Each application thread checks safepoint status and suspends itself in safe state if
safepoint is required.

For compiled code, JIT inserts safepoint checks in code at certain points (usually, after return from calls or at back jump of loop). 

For interpreted code, JVM have two byte code dispatch tables and if safepoint is required, JVM switches tables to enable safepoint check.

Safepoint status check itself is implemented in very cunning way. Normal memory variable check would require expensive memory barriers. 
Though, safepoint check is implemented as memory reads a barrier. Then safepoint is required, JVM unmaps page with that address provoking 
page fault on application thread (which is handled by JVM’s handler). This way, HotSpot maintains its JITed code CPU pipeline friendly, 
yet ensures correct memory semantic (page unmap is forcing memory barrier to processing cores).

####When safepoints are used?

1. Garbage collection pauses
2. Code deoptimization
3. Flushing code cache
4. Class redefinition (e.g. hot swap or instrumentation)
5. Biased lock revocation
6. Various debug operation (e.g. deadlock check or stacktrace dump)

#### Introspect JVM Safepoints
```-XX:+PrintGCApplicationStoppedTime``` – this will actually report pause time for all safepoints (GC related or not). 
Unfortunately output from this option lacks timestamps, but it is still useful to narrow down problem to safepoints.

```-XX:+PrintSafepointStatistics –XX:PrintSafepointStatisticsCount=1``` – this two options will force JVM to report reason and timings after 
each safepoint (it will be reported to stdout, not GC log).
References

##References
(http://blog.ragozin.info/search/label/performance)
