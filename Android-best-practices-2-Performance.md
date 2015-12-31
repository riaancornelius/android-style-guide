Read more at [Developing for Android III - The Rules: Performance](https://medium.com/google-developers/developing-for-android-iii-2efc140167fd#.u24xgoxn2)

# Avoid Expensive Operations During Animations and User Interaction

expensive operations which happen on the UI thread can cause hiccups in the rendering process. This, in turn, causes problems for animations, which are dependent on this rendering process for every frame. This means that it is even more important than usual to avoid expensive operations on the UI thread while there are active animations. Here are some common situations to be avoided:

* **Layout:** Measurement and layout is an expensive operation, and the more complicated the view hierarchy, the more expensive it can be. Measurement and layout happens on the UI thread (as does any operation that needs to manipulate the views of an activity). This means that an application that is currently trying to run a smooth animation and is then told to layout will do both things on the same thread, and the animation will suffer.
* **Inflation:** View inflation can only happen on the UI thread, and it can be a expensive operation (the larger the view hierarchy, the more expensive the operation). Inflation can happen by manually inflating a view or view hierarchy, or by implicitly inflating it by launching a separate activity. This will happen on the same UI thread as your main activity, and will cause animations to halt while that new activity is inflated.

The moral of this story is that you should keep your layouts as simple as possible and only do view inflation when needed.

# Launch Fast
View inflation is not cheap. It’s not just the parsing of the resource data, but also the instantiation of potentially many views and their potentially expensive content along the way, including decoding bitmaps, running layout, and drawing for the first time. The more complicated your UI and the more views that are in your view hierarchy, the more expensive this inflation operation will be.

If you have parts of your UI that do not need to be visible on first launch, don’t inflate them. Use ViewStub as a placeholder for sub-hierarchies that can be optionally inflated later. Avoid expensive operations like decoding large bitmaps whenever possible. Avoid memory churn due to allocations and garbage collections when possible. Use the tools to monitor your startup time and eliminate bottlenecks when you find them.

Also, avoid initialization code in your Application object. This object is created each time your process is started, potentially causing much more work to be done than is actually needed to get the initial UI shown to the user. 

Application subclasses tend to become repositories for the full set of potential things an app may need to have initialized, doing a lot of wasted work in common cases. Instead, it is recommended that you use singletons to hold common global state, which are initialized only the first time they are accessed. 

On a related note, never make a network request in your Application object. That object may be created when one of the app’s Services or BroadcastReceivers is started; hitting the network will turn code that does a local update at a specific frequency into a regular DDoS.

# Avoid Complex View Hierarchies
The more views that are in your hierarchy, the longer it will take to do common operations, including inflation, layout, and rendering (in addition to the potentially expensive memory implications of unused content; Views are expensive by themselves, in addition to any additional data brought in by custom views). 

Find cheaper ways to represent nested content. One approach to avoiding complex nested hierarchies is to use custom views or custom layouts in some situations; it may be cheaper for a single view to draw several pieces of text and icons rather than have a series of nested ViewGroups to accomplish this. 

One rule of thumb for deciding how to combine multiple elements lies in the interaction model: if the user can interact with an element (e.g., via touch, focus, etc.), then that element should be its own view rather than being combined with other, separate elements.

Avoid RelativeLayout Near the Top of the View Hierarchy
RelativeLayout is a very convenient layout to use, because it allows developers to specify how content should be placed relative to other content. 

In many situations, this is necessary and may be the best solution for the job. However, it is important to understand that RelativeLayout is an expensive solution, because it requires two measurement passes to ensure that it has handled all of the layout relationships correctly. 

Moreover, this problem compounds with every additional RelativeLayout throughout the hierarchy. Imagine a RelativeLayout at the top of your view hierarchy; this essentially doubles the measurement work on your entire view hierarchy. Now imagine another RelativeLayout as one of the children of that first one — this doubles again the measurement passes that happen under it, requiring four measurement passes for all of the views in its sub-hierarchy.

Use a different type of layout for situations that do not require the capabilities of RelativeLayout, such as LinearLayout or even a custom layout. Or for situations in which relative alignment of child views is necessary, consider the more optimized GridLayout, which pre-processes the child view relationships and avoids the double-measurement problem.

# Avoid Expensive Operations on the UI Thread
Stalling the UI Thread leads to delays in animations and drawing, which causes visible jank for the user. Avoid doing known-expensive operations while on the UI thread (such as during onDraw(), or onLayout(), or any of the other View-related methods that are called on that thread). 

Good examples of what not to do include calling a web service, executing other network operations (which will throw a NetworkOnMainThreadException), and doing database transactions. Instead, consider using Loaders or other facilities for performing the operations on other threads with the information feeding into the UI later as it comes in. 

Another reason to avoid accessing the file system or database on the UI Thread is that Android device storage is often not good at handling multiple, concurrent reads/writes. Even if your app is idle, there might be another application doing expensive disk I/O (e.g., Play Store updating apps) and this may cause an ANR, or at least significant delays, in your application.

In general, anything that can be done asynchronously should be; the UI Thread should be used just for core UI Thread operations, like manipulating View properties and drawing.

# Minimize Wakeups
BroadcastReceivers are used to receive information and events from other applications that your application may want to respond to. But responding to more of these items than your application actually needs will cause your application to wake up too often, causing overall system performance problems and resource drain. 

Consider disabling BroadcastReceivers when your application does not care about the results, and carefully choose the Intents that your application really needs to respond to.

# Develop for the Low End
Most of your users will have lower-end devices than you probably carry for your everyday phone, by virtue of their being either older or cheaper than yours. 

It is important to develop for this market, and not miss important performance nuances due to higher performance metrics of your development device glossing over elements of your application that would be hard to miss on lower-end devices. 

Your primary device should not be the fastest/latest one available, and you should always have a variety of devices to be able to try out different speeds and form factors to ensure that your application runs adequately across all of them.

Other factors of low-end devices that are important to test against include small RAM sizes and small screen sizes. For example, 512MB is a common low-end configuration, and many Android devices have screen resolutions of 768x480 or less.

# Measure Performance
There is a plethora of tools available on Android. Use them to track important performance-related information about your application, including rendering performance (are you able to hit 60 fps?), memory allocations (are constant allocations triggering garbage collections leading to jank during animations?), and startup performance (are you doing too much at startup leading to long wait times for the user when your app first comes up?). Find the problems. Fix them.