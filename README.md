
Thread Pool library for iOS
===========================

The thread pool and URL dispatch library used by Lightstreamer's iOS client library since v. 1.2


What this library does
----------------------

This code was written to solve two specific problems of iOS' SDK and runtime:

1. NSOperationQueues are unbound with non-concurrent operations, and even with concurrent
   operations their maxConcurrentOperationCount is not strictly enforced;

2. iOS runtime has a limit of 5 concurrent NSURLConnections to the same end-point; above
   this limit connections are going to time-out.
   
For more informations on this topic, please read the article on Lightstreamer's blog on this
specific topic:

* http://blog.lightstreamer.com/2013/01/on-ios-url-connection-parallelism-and.html

What is included:

* an `LSThreadPool` class that lets you execute invocations on a fixed thread pool, and

* an `LSURLDispatcherPool` class that uses a thread pool to keep the number of concurrent
  connections by end-point under control.
  
* a bonus `LSTimerThread` class that lets you run timed invocation without using the
  main thread.
  

LSThreadPool
------------

Use of `LSThreadPool` is really simple. Create it with a defined size and name (name 
will be used for logging):

    // Create thread pool
    LSThreadPool *threadPool= [[LSThreadPool alloc] initWithName:@"Test" size:4];
	
Then schedule invocations with its `scheduleInvocationForTarget:selector:` or 
`scheduleInvocationForTarget:selector:withObject:` methods. E.g.:

    [threadPool scheduleInvocationForTarget:self selector:@selector(addOne)];
	
Finally, dispose of the thread pool before releasing it when done:

    [threadPool dispose];
	[threadPool release];


LSURLDispatcher
---------------

The `LSURLDispatcher` is a singleton and is able to automatically initialize itself. Use it to
start a connection request toward a NSURLRequest in one of three possible ways:

* as a *synschronous request*: in this case the dispatcher will download the request URL
  and deliver it as a NSData; if the end-point is already at its connection limit,
  the dispatcher will wait until it can connect;

* as a *short request*: the dispatcher will connect and send events to your delegate
  as the connection proceeds; if the end-point is already at its connection limit,
  the dispatcher will wait until it can connect;

* as a *long request*: the dispatcher will connect only if the end-point is below its
  connection limit, otherwise it will raise an exception.
  
E.g.:

	NSURL *url= [NSURL URLWithString:@"http://some/url"];
	NSURLRequest *req= [NSURLRequest requestWithURL:url];

    LSURLDispatchOperation *op= [[LSURLDispatcher sharedDispatcher] dispatchShortRequest:req delegate:self];

You can also query the dispatcher to know if the a long operation is going to succeed
or not (that it: to know if the connection has been reached or not). E.g.:

	if (![[LSURLDispatcher sharedDispatcher] isLongRequestAllowed:req])
		NSLog(@"Connection limit reached");

All request will be operated on a separate thread. Each end-point has its own thread pool.


Test cases
----------

A couple of simple test cases are included, which will show the strict enforcement on thread
pool size and the strict enforcement of the connection limit per end-point.


License
-------

This software is part of Lightstreamer's iOS client library since version 1.2. It is released
as open source under the Apache License 2.0. See LICENSE for more informations.

