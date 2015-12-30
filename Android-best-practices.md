This will use Chet Haase's [Developing for Android](https://medium.com/google-developers/developing-for-android-introduction-5345b451567c#.caca9go6g) series as a backbone for topics to cover, so will probably end up being mostly a summary of his work littered with links back to the specific topics in his series of articles.

I'll also try to add links to relevant videos or articles for those who wants more information. A great source of information is the [Google Developers Youtube channel](https://www.youtube.com/channel/UC_x5XG1OV2P6uZZ5FSM9Ttw).

# Understanding the Mobile Context

It is important to understand the context in which these best practices are necessary. 

In particular, it is critical to understand that mobile devices have severe limitations that are completely different from the world of desktop and server computing. 

Moreover, failure to take these constraints into account when developing applications can lead to poor performance and memory consumption not only for any given application, but also for the entire device, because many apps with similar performance problems contribute together to create a poorly-performing device overall.

One particularly important fact to remember is that the devices that we, the application developers, use are probably significantly faster/better/newer than the majority of devices that users either own now or will own in the future. 

For example, while typical reasonable devices that we might own, such as the Nexus 5, have 1GB — 2GB of memory, 512MB is a very common memory configuration both in the U.S. as well as in emerging markets where low-end phones are prevalent.

You also need to consider that most of your users are probably using slower devices than you (a typical software developer) own. Low-end devices abound in the world and are being sold all the time. So don’t benchmark your app’s performance on your relatively new and decent Nexus device, because chances are great that most of your users will have devices with slower processors and smaller memory configurations.

Read [Developing for Android I - Understanding the Mobile Context](https://medium.com/google-developers/developing-for-android-i-understanding-the-mobile-context-fd2351b131f8#.1n1nvzpgo) for an overview of the important constraints, limitations, and realities of past, current, and foreseeable-future mobile devices.

# Memory guidelines

The use of memory in an application can be the single-most important determinant of how well that application behaves, how good the user experience of that application is, and how good the overall device experience is as well.

Memory factors include how large the application footprint is (in storage and when resident in memory) and how the application churns memory (causing garbage collections, which has an effect on runtime performance).

[Android best practices - Memory](https://github.com/riaancornelius/android-style-guide/blob/master/Android-best-practices-1-Memory.md)

# Performance guidelines

On Android, performance and memory are closely intertwined, since the memory footprint of the overall system can affect the performance of all of the processes, and since the garbage collector can have a significant impact on runtime performance. But the items in this section are targeted more specifically at runtime performance that is not necessarily memory-related.

[Android best practices - Performance](https://github.com/riaancornelius/android-style-guide/blob/master/Android-best-practices-2-Performance.md)

