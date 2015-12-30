Read more at [Developing for Android II - The Rules: Memory](https://medium.com/google-developers/developing-for-android-ii-bb9a51f8c8b9#.fa3tqxl5f)

## Avoid Allocations in Inner Loops
Allocations are unavoidable. But avoid them when possible, particularly in code paths that will be called frequently, such as during the drawing code that may execute on every frame that a view is rendered.

## Avoid Allocations When Possible
In the interest of avoiding constant, temporary allocations, there are several strategies to consider. 

Note that some of these are not considered great coding or API decisions in traditional Java development, but they are recommended for Android with the tradeoff of style vs. garbage. 

As always, use the tools to help determine whether any particular code is worth optimizing. If there is some section of code that is executed rarely (like when the user changes some setting), but which would benefit from a clearer style, then a more traditional layer of abstraction could be the right decision. 

But if analysis shows that you are re-executing some code path often and causing lots of memory churn in the process, consider these strategies for avoiding excess allocations:

* Cached objects: Reused objects are useful in situations where the alternative involves constant re-allocations, such as in the example in the Avoid Allocations in Inner Loops section. Carefully applied, this technique is commonly used on Android to save on re-allocations.

* Object pools: If your code is in constant need for one or more objects of a certain type on a temporary basis, consider using a pool of objects instead of allocating on the fly. Note that object pools can be tricky to manage. 

* Arrays: ArrayList is a convenient collection to use, and does not suffer too much overhead. It does re-allocate and copy the underlying array as items are added, but setting a reasonable initial capacity can alleviate that overhead. If your collection does not need to be constantly dynamically resized, consider using an array instead.

* Android collections classes: Unless you require a map for a large set of items, consider using ArrayMap or SimpleArrayMap for data structures instead of HashMap. These classes are optimized to be more memory efficient and less GC-heavy than HashMap, better matching common use cases for such structures on mobile.  Also, consider setting the initial capacity of your collection to avoid automatic resizing as it grows to the appropriate size.

* Methods with mutated objects: Instead of having a method return a new object (which would have to be allocated), consider taking an object parameter of that type and mutating that object. For example, instead of:

	`Rect getRect(int w, int h)`

	consider:

	`void getRect(Rect rect, int w, int h)`

	This is ugly and dangerous as methods like this can introduce side effects, so use only when absolutely necessary!
 
* Avoid object types when primitive types will do: Using Integer instead of int, or Float instead of float, leads to allocation, autoboxing, and more memory allocated for the object itself. 
* Avoid arrays of objects: If you have an array of simple plain old data objects, consider one array for each field. For example, assume you have a drawing application which needs to keep track of a list of previous touch X/Y point data. Instead of storing a Point[] array, consider instead storing two int[] arrays, one for X and one for Y.

## Avoid Iterators
Explicit (e.g., List.iterator()) or implicit (e.g., for (Object o : myObjects)) iteration causes the small allocation of an Iterator object (with the exception of arrays, which can be used with the foreach syntax without causing an Iterator allocation). 

This single allocation is not a big deal in practice, but should be avoided in inner loops for the same reasons discussed above. Meanwhile, iterating more explicitly through the indices of a collection avoids any allocations:

    final int count = myList.size();
    for (int i = 0; i < count; ++i) {
        Object o = myList.get(i);
        // …
    }

Also, note that requesting iterators always causes an allocation, even on an empty list. So if you are using the foreach syntax on a Collection (`for (Object o : myObjects)`), you are causing an allocation even if that collection is typically empty. 

You should at least do an `isEmpty()` check on a collection first to avoid this needless allocation if you must use the foreach syntax.

## Avoid Enums
Enums are typically used to represent constants, but they are much more expensive than primitive-type representations, in terms of the code size and the memory allocated for the enum objects.

An occasional enum is not a big deal in terms of the memory it consumes or its allocation costs. And Proguard can, in some situations where it can statically analyze all usages, optimize enums to int values. 

But enums become a problem when used widely across a large application or, even worse, when used broadly in a library or an API that is then used by many other applications.

Note that using the `@IntDef` annotation, which is supported by Android Studio and Gradle 1.3+, will give your code build-time type safety (when lint errors are enabled), while retaining the size and performance benefits of using int variables.

## Avoid Frameworks and Libraries Not Written for Mobile Applications
It is tempting to use frameworks that you may be familiar with from other Java environments. For example, dependency-injection frameworks like Guice is a commonly used Java library. But since it was not written or optimized specifically for mobile, and for the constraints talked about in this document, then applications using libraries like this will suffer due to the problems described herein.
Also, note that if you are using only a small portion of a large library, you will tend to drag in larges amounts of unused code and bloat your application footprint unnecessarily. 

## Avoid Static Leaks

While static objects can be a useful means to avoid temporary allocations, beware of using static variables to cache objects that should not actually persist for the life of the entire process. Note, in particular, that the lifetime of a static variable (which is equal to the lifetime of the underlying process) is not the same as the lifetime of an Activity. This misunderstanding can, and has, led to leaking Activity objects across configuration changes (in which activities are destroyed and recreated) with static maps that held onto activity instances (directly or indirectly). Activities are quite expensive and this kind of leak can quickly lead to your application, and the system, running out of memory.

## Avoid Finalizers

Because of nuances with the Java language specification, finalizers require not just one full garbage collection, but two. This means that not only will the resources freed by the finalizer not be available until both of those garbage collections occur, but also that you are forcing the system to undergo two collections, which can be both costly and janky, depending on what else is happening in the system. There is a specific situation that requires finalization; when your object holds a native pointer. If this is not the case for your code, avoid finalizers completely.
If you do require finalizers, consider implementing the AutoCloseable interface and freeing native resources within the scope of their usage via the close() method.

## Avoid Excess Static Initialization
Expensive initialization can cause performance problems at crucial times in your application’s lifecycle, such as startup, contributing to poor user experience. Perform initialization on demand to avoid loading code in memory until you actually need it.

## Trim caches on demand
As of API level 14, ComponentCallbacks2 provides the onTrimMemory() callback that allows your app to release memory when the system is under memory pressure. The [Doing More with Less video](https://www.youtube.com/watch?v=gbQb1PVjfqM) from Google I/O 2012 shows an example of using this approach with bitmaps in an LruCache.

## Use isLowRamDevice
ActivityManager.isLowRamDevice() was added as part of the Svelte effort in KitKat to help developers detect when their application is running on a device with particularly constrained memory (currently, a return value of true typically indicates memory of 512MB or less). This condition should be used by applications to decide whether to disable features that require more memory than would reasonably be available on such a device.

## Avoid Requesting a Large Heap
Applications can request a large heap from the system in the application tag of their manifest… but they shouldn’t. 

Requesting a large heap is a classic example of Tragedy of the Commons (discussed in [Understanding the Mobile Context](https://medium.com/google-developers/developing-for-android-i-understanding-the-mobile-context-fd2351b131f8#.dod8d7aro)), as it might make sense for an application when considered on its own, but is the wrong decision for an app which is part of an overall device experience.

Requesting a larger heap may be necessary in some rare situations where the type of media content regularly needed by the user easily swamps the default heap limit. But applications that use this just to avoid having to more carefully manage their memory and resources are only causing problems for the overall user experience of the device. 

## Avoid Running Services Longer than Necessary
Every process on the system takes up limited resources. If you don’t need your service to run all the time (and ideally you don’t), shut it down whenever possible.

Android provides many mechanisms for ensuring that components only run for as long as necessary to perform a given function.

## Optimize for Code Size
Lean applications are fast applications. The less code you have to load, the less time it will take to download your application and the faster your application will start and initialize. Here are a few suggestions:

* Use Proguard to strip out unused code. If you are building with Gradle, you can also strip out unused resources, including those from libraries on which your application depends.
* Carefully consider your library dependencies. For example, don’t use a large library full of different collections options when all you need is a single, specific kind of data structure.
* Make sure to understand the cost of any code which is automatically generated.
* Prefer simple, direct solutions to problems rather than creating a lot of infrastructure and abstractions to solve those problems.