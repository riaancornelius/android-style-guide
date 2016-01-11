Read more at [Developing for Android VII - The Rules: Framework](https://medium.com/google-developers/developing-for-android-vii-the-rules-framework-concerns-d0210e52eee3#.zfq18q30m)

# Avoid Architecting Around Application Components
Application components (activities, services, providers, receivers) are interfaces for your application to interact with the operating system; don’t take them as a recommendation of the facilities you should architect your entire application around. Each of these components has very specific semantics with the operating system, and should typically only be used when those semantics are desired:

* **Activity** is a top-level UI entry point into your application. It corresponds roughly to the main() entry of a traditional operating system (that is, what runs when the user clicks on your app’s icon), except Android allows you to have multiple such entry-points for different purposes. You should implement an Activity when you want to allow others to launch specific parts of your app, such as for executing a share operation or opening some content contained in the app. If you have UI that is only going to be launched by your own app, there is no need to implement it as an Activity, you can just as well do this by transforming the state of the currently shown UI (such as through fragments). Before fragments existed in the platform, architecting an application around multiple activities was fairly standard, but it is no longer necessary unless you need that specific design.
* **Service** is how an application indicates to the operating system that it needs to perform a longer-running operation outside of the normal Activity UI flow. It may be self-running (through Context.startService()), or running on behalf of of another process (through Context.bindService()). If you do not need either of these behaviors, you should not use a Service.
For example, if you have background work to do that does not need to keep itself running (such as downloading content for your UI that you could resume later when the user returns to the UI) you should use local threading primitives such as AsyncTask, Loader, HandlerThread, etc. Using a Service is much more resource-intensive (as your service needs to be tracked as part of the global system state) and opens the door to all kinds of bad bugs where your services is left running when it shouldn’t be (this is a very common problem in Android apps, and is harmful to the system). Likewise, if you want to have an association between two pieces of code but they are running in the same local process, don’t use bindService(); instead, use regular callbacks and other facilities. This will be easier to write, easier to understand, and less resource-intensive.
Also, beware a common gotcha with AsyncTask: given the asynchronous nature of AsyncTask, it’s possible that an Activity may finish by the time that results come back in. Make sure that you check the status of the Activity before using those results.
* **Broadcast Receiver** is how an application can indicate that it is interested in certain events that may happen, and have itself run when they occur.

# Services Should Be Bound or Started, Not Both
Use Service either as a bound-service or as a started-service, avoid blending both styles.
If necessary, a bound-service may self-start in order to continue performing work in the background, but it must also finish itself when that work is completed. (This is somewhat error-prone as it may conflict with external requests to start-service.)

# Prefer Broadcast over Service for Independent Events
Use broadcasts for delivering independent events; use services for processes with state and on-going lifecycle.

# Avoid Passing Large Objects Through Binder
Objects are copied as they are serialized through the Binder, so the larger the object, the longer the process of passing it through.

# Isolate UI processes from Background Services
The activity manager on Android will aggressively kill activities when the system is resource-constrained. If your activity is tied too closely to its background service, you could get into a situation where killing one made the other unusable. An example of good separation of functionality includes a music app with a UI app and a service for playing content, where the UI activity could be killed without affecting music playback from its background service.