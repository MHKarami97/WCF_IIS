# System.net Changes
By default, system.net has two concurrent connections per IP allowed.  
This means on a webserver, if it’s calling WCF service on another server or making any outbound call to a particular server,  
it will only be able to make two concurrent calls.  
When you have a distributed application and your webservers need to make frequent service calls to another server,  
this becomes the greatest bottleneck.  
  
Similarly, if you have a desktop application which is trying to make several webservice calls to the server,  
the system.net setting on the client computer’s machine.config will only allow two concurrent calls to your webserver.  
If your app makes many concurrent calls to your webserver,  
then this becomes the main bottleneck.  
No matter how fast your client app and server are, the two concurrent limit will kill it.  
  
n order to overcome the limit, you have to change the system.net configuration either in the web.config or at machine.config.  
For desktop applications, you need to apply it on app.Config.  

```c#
<system.net>
<connectionManagement>
<add address="*" maxconnection="100" />
</connectionManagement>
</system.net>
```

This gives all IP addresses 100 concurrent connections limit.  
You might want to be specific and specify some fixed IP address or domain name.  
If you are doing this on desktop apps, 100 might be overkill.  
Better go for a moderate value to prevent accidental flood of calls going to webserver and bringing it down.  

# WCF Throttling
Just like system.net and system.web default settings that more or less prevent you from making highly scalable applications,  
the WCF default configuration is very stingy.  
It allows only 16 concurrent calls per service by default. Moreover,  
if you are using wsHttpBinding and using sessions, then only 10 concurrent sessions are allowed.  
Too low for popular applications nowadays.  
I will highlight the most common settings that you should always change:  
  
more info: [link](https://weblogs.asp.net/paolopia/wcf-configuration-default-limits-concurrency-and-scalability)  

```c#
<serviceBehaviors>
<behavior name="defaultServiceBehavior">
<serviceThrottling maxConcurrentCalls="100"
maxConcurrentInstances="100" maxConcurrentSessions="100" />
```

- `maxConcurrentCalls` : defines the maximum number of messages actively processed by all the service instances of a ServiceHost.    
The default value is 16. Calls in excess of the limit are queued.  
  
- `maxConcurrentInstances` : defines the maximum number of service instances that can execute at the same time.  
The default value is Int32.MaxValue.  
Requests to create additional instances are queued and complete when a slot below the limit becomes available.  
  
- `maxConcurrentSessions` : defines the maximum number of sessions that a ServiceHost instance can accept at one time.  
The default value is 10.  
The service will accept connections in excess of the limit,  
but only the channels below the limit are active (messages are read from the channel).  
  
also make sure your services have InstancePerContext.PerCall set.  
This ensures each request is handled by a new instance of the service on a new thread.  
If you are hosting your WCF services via ASP.NET (e.g. using WCF REST Starter Kit),  
then you are further bogged down by the ASP.NET process model changes and you need to beef that up as well.  

# ASP.NET Process Model Changes
Process Model configuration defines some process level properties like how many number of threads ASP.NET uses,  
how long it blocks a thread before timing out, how many requests to keep waiting for IO works to complete and so on.  
The default is in most cases too limiting.  
Nowadays, hardware has become quite cheap and dual core with gigabyte RAM servers have become a common choice.  
So, the process model configuration can be tweaked to make ASP.NET process consume more system resources and gain better scalability from each server.  
A regular ASP.NET installation will create machine.config with the following configuration:  

```c#
<system.web>
<processModel autoConfig="true" />
```

You need to tweak this auto configuration and use some specific values for different attributes in order to customize the way ASP.NET worker process works.  
For example:  

```c#
<processModel
enable="true"
timeout="Infinite"
idleTimeout="Infinite"
shutdownTimeout="00:00:05"
requestLimit="Infinite"
requestQueueLimit="5000"
restartQueueLimit="10"
memoryLimit="60"
webGarden="false"
cpuMask="0xffffffff"
userName="machine"
password="AutoGenerate"
logLevel="Errors"
clientConnectedCheck="00:00:05"
comAuthenticationLevel="Connect"
comImpersonationLevel="Impersonate"
responseDeadlockInterval="00:03:00"
responseRestartDeadlockInterval="00:03:00"
autoConfig="false"
maxWorkerThreads="100"
maxIoThreads="100"
minWorkerThreads="40"
minIoThreads="30"
serverErrorMessageFile=""
pingFrequency="Infinite"
pingTimeout="Infinite"
maxAppDomains="2000"
/>
```

- `maxWorkerThreads` : This is default to 20 per process.  
On a dual core computer, there'll be 40 threads allocated for ASP.NET.  
This means at a time ASP.NET can process 40 requests in parallel on a dual core server.  
I have increased it to 100 in order to give ASP.NET more threads per process.  
If you have an application which is not that CPU intensive and can easily take in more requests per second,  
then you can increase this value.  
Especially if your Web application uses a lot of Web service calls or downloads/uploads a lot of data which does not put pressure on the CPU.  
When ASP.NET runs out of worker threads, it stops processing more requests that come in.  
Requests get into a queue and keep waiting until a worker thread is freed.  
This generally happens when a site starts receiving much more hits than you originally planned.  
In that case, if you have CPU to spare, increase the worker threads count per process.  
  
- `maxIOThreads` : This is default to 20 per process.  
On a dual core computer, there'll be 40 threads allocated for ASP.NET for I/O operations.  
This means at a time ASP.NET can process 40 I/O requests in parallel on a dual core server.  
I/O requests can be file read/write, database operations, web service calls, HTTP requests generated from within the Web application and so on.  
So, you can set this to 100 if your server has enough system resource to do more of these I/O requests.  
Especially when your Web application downloads/uploads data, it calls many external webservices in parallel.  
  
minWorkerThreads : When a number of free ASP.NET worker threads fall below this number,  
ASP.NET starts putting incoming requests into a queue.  
So, you can set this value to a low number in order to increase the number of concurrent requests.  
However, do not set this to a very low number because Web application code might need to do some background processing and parallel processing for which it will need some free worker threads.  
  
- `minIOThreads` : Same as minWorkerThreads but this is for the I/O threads.  
However, you can set this to a lower value than minWorkerThreads because there's no issue of parallel processing in case of I/O threads.  
  
- `memoryLimit` : Specifies the maximum allowed memory size,  
as a percentage of total system memory,  
that the worker process can consume before ASP.NET launches a new process and reassigns existing requests.  
If you have only your Web application running in a dedicated box and there's no other process that needs RAM,  
you can set a high value like 80.  
However, if you have a leaky application that continuously leaks memory,  
then it's better to set it to a lower value so that the leaky process is recycled pretty soon before it becomes a memory hog and thus keep your site healthy.  
Especially when you are using COM components and leaking memory.  
However, this is a temporary solution, you of course need to fix the leak.  
  
# Turn on IIS Caching and Dynamic Compression
You can enable IIS compression for static and dynamic content.  
The dynamic content compression will affect output from aspx and WCF services.  
If you find the Enable Dynamic Content Compression option disabled,  
then you need to go to “Add or Remote Features” from Windows start menu and then enable the IIS Dynamic Compression from IIS features.  
You might have to enable the compression first at server level and then at site level.  


# Monitor Your Server for Performance Issues
You can use the Windows Performance Monitor to monitor some key performance counters in order to identify performance bottlenecks.  
For example, the following counters should be monitored for ASP.NET web application hosting WCF services.  