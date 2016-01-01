Read more at [Developing for Android VI - The Rules: Storage](https://medium.com/google-developers/developing-for-android-vi-c0b1539f0e98#.5smh4o1r8)

# Avoid Hard-coded File Paths
Construct paths, instead, from the Context or the Environment.

* Don’t hardcode global paths like “/sdcard”, use `Environment.getExternalStorageDirectory()` and related methods instead.
* Don’t hardcode application paths like “/data/data/myapp/databases”, use `Context.getDatabasePath()`, `Context.getFilesDir()` and related methods instead.

# Persist Relative Paths Only
If you need to persist a path, persist only its relative location to protect against path changes. For example, when your app is backed up and restored onto a new device, the data paths may be slightly different.

Also, avoid heavy canonicalization except for specific security use-cases.

# Use Storage Cache for Temporary Files
Separate your application’s temporary files from other persistent data by storing them in the cache directory indicated by `Context.getCacheDir()`. 

This allows the system to be more efficient about how it uses storage.

Files in the cache directory may be deleted by the system to reclaim space when low on storage, unlike files in the data directory which are preserved until the application is uninstalled or the user explicitly requests to clear application data.

Files in the cache directory are never backed up, unlike files in the data directory which may be backed up automatically.

# Avoid SQLite for Simple Requirements
SQLite is a full relational database engine, and is overkill for simple data structures or key/value pairs which don’t require relational structure or indexing. 

If your data needs are simple, consider alternatives like the following:

* Use SharedPreferences for simple keys and values.
(Important things to note about SharedPreferences. First, they are cached statically after they are read for the first time. This speeds up access, but can contribute to leaks and memory issues if they are used for anything complex. Second, committing changes rewrites the entire SharedPreferences file from scratch, so small/frequent updates can cause more work than you might expect.)
* Use append-only log files and periodically rotate them when storing time-series event data.
* Use [LevelDB](https://github.com/google/leveldb) if all you need is NoSQL and you can deal with JNI correctly and minimally.

# Avoid Using Too Many Databases
SQLite databases are expensive on disk and in memory. Don’t create a separate database for each table. Most applications should have at most one database.

# Let User Choose Content Storage Location
Devices often have more than one storage location, including multiple SD cards, USB drives, and cloud storage backends. Give the user the choice of where to open/save their data by using the [Storage Access 
Framework](https://developer.android.com/guide/topics/providers/document-provider.html).

You can launch simple intents to prompt the user to open or save a file, and receive back a content:// URI ready to be used for data storage. The [DocumentFile](https://developer.android.com/reference/android/support/v4/provider/DocumentFile.html) support library class can make it easier to adapt existing code that expects a traditional File-style API.