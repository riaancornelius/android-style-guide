Read more at [Developing for Android V - The Rules: Language and Libraries](https://medium.com/google-developers/developing-for-android-v-f6b8038b42f5#.bgsv8i7nu)

# Use Android-Appropriate Data Structures
Traditional collections classes tend to be sub-optimal on Android due to the overhead of allocations (which runs into Garbage Collector concerns, as discussed in the previous Memory section).

Collections classes written specifically for Android are better in most cases, such as ArrayMap (prefer over HashMap) and SparseArray. 

The more general classes are still appropriate when collections are particularly large, but smaller collections benefit from avoiding autoboxing and other allocation concerns.

# Serialization
There are several issues and options around serialization formats, which are discussed separately in the sections below.

## Parcelable
Parcelable is Android’s IPC serialization format, or rather, it is an interface for writing out your data so it can be sent over Binder. It does have limits:

* It is not safe to write Parcels to disk.
* You can implement your own Parcelables, but if the receiving process doesn’t have access to the same classes, unparceling will fail. (This is true of Parcels you send to the framework, too.)
* Some types of objects (bitmaps and CursorWindows) placed into Parcels are instead written to ashmem (Android shared memory) files, and a file descriptor is written into the Parcel in their place. This is an important performance optimization but can hide the true memory cost of big Parcels (and will take up space in ashmem until unparceled).

## Persistable Bundles
Starting in API 21, there is a new PersistableBundle class that is a variant of Bundle with a stable data format that supports serialization as XML. It only accepts a subset of the data types supported by Bundle. In particular, it does not support Parcelable objects.

PersistableBundles are particularly useful when working with data that is being passed through the Binder IPC, which requires Bundles to begin with.

## Avoid Java Serialization
Serializable, ObjectOutputStream, and friends will all work, but these approaches have very significant overhead, which increases with the raw number of fields being serialized. 

There are almost always better options to use where you can more carefully control the information that is being stored:

* Use Parcelable for anything that needs to be exchanged between running processes.
* SharedPreferences is a good, quick key-value store for situations that do not require large numbers of elements.
* Use SQLite for more complex, row-oriented data.

There is still a specific case where Java serialization may be required: if your app is talking to legacy servers that require that specific protocol. In that case, you should be looking into upgrading the system to use a more efficient mechanism.

## XML and JSON
These text-based formats tend to be slower and more verbose than other options, so they are poorly suited for large, complex data, IPC (use Parcelable instead), or data that needs to be queried (use SQLite). 

Integrating with web services that speak JSON or XML is fine. 

It’s also okay to use a little bit of XML to store a small amount of rarely-modified data (although SharedPreferences might be even easier).

Note that the XML used in Android resource files is compiled at build time into a compact format that is more efficiently parsed at runtime. The advice here against using XML is specific to parsing actual runtime XML data.

# Avoid JNI
JNI is problematic for many reasons. For one thing, the native code required for JNI must be compiled for all architectures (ARM, ARM64, MIPS, etc.), unlike the normal Java programming language code which runs across all Android platforms. 

Also, there is a high performance penalty for JNI. 

Finally, there is increased potential for difficult bugs due to non-obvious semantics of memory accesses at the native layer.

But if you do have to use JNI:

* Use long for pointers to ensure 64-bit compatibility.
* Native methods should almost always be static, with the native object pointer passed as first argument.
* The Java programming language object should determine the lifetime of its native peer, not the other way around.
* Beware of global object reference cycles causing leaks.
* Check arguments before calling into JNI rather than down in JNI where it’s tedious and error prone.
* Minimize the number of JNI crossings. Try to do more work per JNI call to amortize the overhead of of each call.
* Pass native pointers by value instead of querying members from native. Note that this technique is only valid for non-static methods; static methods could run into problems where the object in question is collected before or while it is being used in the native code.
* Consider RenderScript Compute for computationally-intensive operations that would benefit from it.

# Prefer Primitive Types
This was discussed previously in the section Avoid Allocations When Possible of the Memory section, but is worth noting here as well. 

Always use primitive types (int, float, boolean, etc.) instead of their Object equivalents (Integer, Float, Boolean, etc.) when you have the choice. 

The overhead in terms of memory (object instances always cost more than their primitive type equivalents) and performance (object instances require more runtime overhead to access the underlying value) means that you should always use primitive types when you can on Android.

Generics and some data structures (e.g., the traditional collections classes) require object types. But note that there are various collections classes on Android (`ArrayMap`, `SimpleArrayMap`, `SparseArray`, `SparseBooleanArray`, `SparseIntArray`, `SparseLongArray`, and `LongSparseArray`) that avoid the object type requirement for specific use cases.