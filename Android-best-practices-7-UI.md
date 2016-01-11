Read more at [Developing for Android VIII
The Rules: User Interface](https://medium.com/google-developers/developing-for-android-viii-e91ced595fac#.w77l8wmce)

# Avoid Overdraw
As discussed in the GPU section of the first chapter (Understanding the Mobile Context), many devices are fill-rate limited, and applications that have significant overdraw can suffer poor rendering performance as a result. 

Avoid situations where opaque views completely cover other views and cause the rendering engine to draw pixels too many times. Test your application’s overdraw with the Debug GPU overdraw developer option in Settings and fix issues accordingly.

# Avoid Null Window Backgrounds
One technique to avoid overdraw is to eliminate the window background in situations where the views in the hierarchy all have opaque backgrounds. That is, the user will not see the background of the window if the view hierarchy completely covers that background with one or more opaque views.

Eliminating the window background can be a valid technique, but it tends to be a complicated way to solve overdraw issues, and can often result in rendering artifacts in different situations. While it is possible to set a null window background in the application manifest, this can result in graphics artifacts due to the system not being able to draw the starting window correctly. A more correct way to do this is to leave the starting window background in place in the manifest, but set it to null in the activity’s `onCreate()` method, by calling `getWindow().setBackground(null)`. But even this approach can cause artifacts. 

The correct way to address overdraw issues for this situation is to actually use the starting window. Instead of having containers with their own opaque background color between the window background and views, put the background drawable you want on the window itself with the windowBackground theme attribute and let those intervening containers keep their default transparent backgrounds.

# Avoid Disabling the Starting Window
As discussed in the Launch Fast section of the Performance chapter, the system provides a quick launch experience via the starting window while the application is being loaded in the background. It is possible to defeat this experience by requesting that the system not do this (via the windowDisablePreview theme attribute). 

Some applications do this because they want to launch a custom splash screen or other branded experience, or because that starting window simply looks so different from the initial content of the launched activity. The problem with this is that these other approaches take much longer (because the application takes far longer to initialize and present its content than the system’s starting window for the application) and the user is forced to wait with no visible feedback while the activity launches.

# Allow Easy Exit from Immersive Mode
It is possible to enable fullscreen mode for your application that can only be exited by swiping in from the edge of the screen to expose the navigation bar. This is acceptable only in immersive games, where taps on the screen should not take the user out of the game. For all other apps, particularly content-viewing apps like media players, allow easier exit by tapping on the screen instead of making the user swipe the bars in. See the immersive mode developer guide for details on how to use immersive mode correctly.

# Set Correct Status/Navigation Bar Colors in Starting Window
If your application has colored status/navigation bars, then your theme (which is used by the starting window when your application launches) should have those same colors. This avoids an unpleasant snap from the starting window to the application content window. To set the status and navigation bar colors for the starting window, set `android:statusBarColor` and `android:navigationBarColor` respectively. If you need different colors for the activity window, change them in `onCreate` via `getWindow().setStatusBarColor()` and `getWindow.setNavigationBarColor()`.

# Use the Appropriate Context
It is possible to get the Context for the Application, but it is incorrect to use that context for creating UI, because it does not have the correct theme information. Instead, use the Context object for the Activity instead. For example, if you are retrieving resources for your activity, you should do so from the Activity context, not the Application context.

# Avoid View-Related References in Asynchronous Callbacks
Avoid referencing a View, Activity, or Fragment in a network operation or other long-running and asynchronous transaction, such as this:
  
    // ...
    // some activity code
    // 
    void onClick(View view) {
        webservice.fetchPosts(new Callback() {
            public void onResult(Response response) {
                // View may not be valid, Activity may be gone
            }
        });
    }
This view reference can lead to leaking an expensive Context (which will be implicitly referred to from the view) or crashing due to dereferencing an old object which is no longer valid when the callback is triggered. Instead, consider an event bus or weak callbacks and handle attachment and detachment situations carefully.

# Design for RTL
As of API 17, applications should be using the start/end variants of layout attributes instead of left/right. For example, you should use paddingStart and paddingEnd, not paddingLeft and paddingRight. This allows the applications to lay out correctly when displayed in right-to-left locales.
Test that your app is behaving correctly by going into Developer Options and enabling the Force RTL layout direction setting.

# Cache Data Locally
“Cache locally, sync globally”
Caching data locally is one of the most important things you can do to make your users feel that your application runs fast. Aside from caching big files like images, you should also cache the data fetched from the server. When feasible, you should create a local database that keeps the data in a meaningful way (not just as a serialized server response).
Whenever possible, update your UI from local data. This will help with data consistency and will also greatly improve user experience as your app will be less dependent on the network.
There is a talk from Google I/O 2010 on this topic, [Android REST Client Applications](https://www.youtube.com/watch?v=xHXn3Kg2IQE) and another more recent one at the Android Dev Summit 2015, [Android Application Architecture](https://www.youtube.com/watch?v=BlkJzgjzL0c&index=76&list=WL). 

There are better components to help with this approach than were available at the time (ORM, JobQueue, EventBus), but the architecture and approach described in the 2010 talk are still valid.

# Cache User Input Locally
When there is user input that needs to be relayed to the server, cache that response prior to relaying it across the network. The network communication can happen via an asynchronous mechanism (such as a network request thread pool), ideally in some batch of other accumulated responses. The main thing to avoid is having the user’s input lost because it was sent directly without being cached, resulting in the frustrating result of losing that input when there is network failure. 

Caching the input will make your application much more resilient as you can retry sending the user’s input data to the server whenever the network is available.

You should also have an indication in the UI that allows the user to distinguish between completed and pending input.

# Separate Network and Disk Background Operations
When using a single thread pool for all background operations, some quick tasks may starve because of long-running background operations like network requests. 

Consider having a separate thread pool for faster background operations like local storage access. This should guarantee that your local changes will update the user interface almost immediately without being affected by expensive network requests or IPC calls. This Android Dev Summit 2015 talk, [Android Application Architecture](https://www.youtube.com/watch?v=BlkJzgjzL0c&index=76&list=WL) also covers this. 