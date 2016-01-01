Read more at [Developing for Android IV - The Rules: Networking](https://medium.com/google-developers/developing-for-android-iv-e7dc4ce0a59#.v7m3y07qy)

# Don’t Over-Sync
Sending data to and from the cloud is one of the largest consumers of battery. The problem is not that any particular network transaction kills the device, but rather the sheer number of background applications and services making these requests and the side effect on the inability of the device to sleep (or for the radio to enter low power mode) for any reasonable length of time causes a significant drain on the battery in general. 

If you don’t need the data immediately, don’t get it. If you need the data sometime in the future, batch it up with other system requests with [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html) or [GCM Network Manager](https://developers.google.com/cloud-messaging/network-manager).

Here are some general tips for avoiding over-syncing:

* Use [Google Cloud Messaging](https://developer.android.com/google/gcm/index.html) (GCM). Do not establish new persistent connections to the device.
* Use [JobScheduler](https://developer.android.com/reference/android/app/job/JobScheduler.html) (on API 21 and later) or [GCM Network Manager](https://developers.google.com/cloud-messaging/network-manager) (which works across many releases, using JobScheduler internally on API 21 and later) to benefit from batching and coalescing from other sync requests on the device. These APIs can also be used to ensure that some operations only happen when the device is in an appropriate idle state.
* **Do not poll. Ever.**
* Only sync what you need to. Data syncing has been found to be one of the biggest culprits of battery usage and overall system health, so app developers need to think carefully about the type of data that really needs to be synced, as well as how often it should be synced for a good user experience.
* See these reference articles for more information and strategies: [Minimizing the Effect of Regular Updates](http://developer.android.com/training/efficient-downloads/regular_updates.html), [Transferring Data Using Sync Adapters](http://developer.android.com/training/sync-adapters/index.html), and [Optimizing Downloads for Efficient Network Access](http://developer.android.com/training/efficient-downloads/efficient-network-access.html).

# Avoid Overloading the Server
When a server request fails, use exponential back-off techniques to avoid hammering the server with constant, repeated requests. Also, never synchronize against wall-clock time to avoid problems with the server being hit by large loads at these absolute times.

# Don’t Make Assumptions about the Network
When making network calls, be sure to call [NetworkInfo.isConnected()](http://developer.android.com/reference/android/net/NetworkInfo.html#isConnected%28%29) to ensure connectivity.

Also, it is not possible to know when or if a network request will actually succeed, which is yet another good reason to not make network requests on the UI thread or in other places requiring fast or synchronous behavior.

# Develop for Low End Networks
The Develop for the Low End rule in the Performance section is as true for network performance as it is for raw device hardware performance. If your application is dependent upon network performance for content (such as a video or music application), then you should develop on a device that has a slower network connection, just to understand the implications in markets that do not have easy access to fast network speeds. 

If your application can get much of the content across with text (such as messaging or social apps), it is important to be able to make that work without being dependent on slower network times for associated media components. 2G networks, in particular, are important to test against since this is a slow network speed that is very prevalent in some markets.

# Design Back-End APIs to Suit Client Usage Patterns
There is a tendency in back-end applications to provide a general purpose API for all of their clients. 

Although this is a reasonable strategy for back-end systems, it is not good for the client because the application ends up transferring and processing a lot of data on a small device while some calculations could be made on the server.

The more information the client has about the data it is displaying, the more efficient it can be. The application can make a careful selection of what to cache, what to prune, and how to invalidate layout when new data arrives. Treating the client application like a simple HTML renderer will miss out on all of these advantages.