# Android Style Guide

## Project structure

Projects should follow the Android Gradle project structure that is defined on the [Android Gradle plugin user guide](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Project-Structure).

# Naming conventions

## Class files
Class names are formatted using [UpperCamelCase](http://en.wikipedia.org/wiki/CamelCase).

For classes that extend an Android component, the name of the class should end with the name of the component; for example: `LoginActivity`, `LoginFragment`, `LoginService`, `LoginDialog`.

## Resources files

Resource file names are formatted in `lowercase_underscore`.

### Drawable files

Naming conventions for drawables:


| Asset Type   | Prefix            |		Example               |
|--------------| ------------------|-----------------------------|
| Button       | `btn_`	            | `btn_send_pressed.9.png`    |
| Dialog       | `dialog_`         | `dialog_top.9.png`          |
| Divider      | `divider_`        | `divider_horizontal.9.png`  |
| Icon         | `ic_`	            | `ic_star.png`               |
| Menu         | `menu_	`           | `menu_submenu_bg.9.png`     |
| Notification | `notification_`	| `notification_bg.9.png`     |
| Tabs         | `tab_`            | `tab_pressed.9.png`         |

### Icons

From [Android iconography guidelines](http://developer.android.com/design/style/iconography.html)):

| Asset Type                      | Prefix             | Example                      |
| --------------------------------| ----------------   | ---------------------------- |
| Icons                           | `ic_`              | `ic_star.png`                |
| Launcher icons                  | `ic_launcher`      | `ic_launcher_calendar.png`   |
| Menu icons and Action Bar icons | `ic_menu`          | `ic_menu_archive.png`        |
| Status bar icons                | `ic_stat_notify`   | `ic_stat_notify_msg.png`     |
| Tab icons                       | `ic_tab`           | `ic_tab_recent.png`          |
| Dialog icons                    | `ic_dialog`        | `ic_dialog_info.png`         |

### Layout files

Layout files should match the name of the Android components that they are intended for but moving the top level component name to the beginning. For example, if we are creating a layout for the `LoginActivity`, the name of the layout file should be `activity_login.xml`.

One exception is items that will be displayed in a `ListView` or `RecyclerView`. In this case, start with the prefix `item_`.

| Component        | Class Name             | Layout Name                   |
| ---------------- | ---------------------- | ----------------------------- |
| Activity         | `LoginActivity`        | `activity_login.xml`          |
| Fragment         | `LoginFragment`        | `fragment_login.xml`          |
| Dialog           | `LoginDialog`          | `dialog_login.xml`            |
| Adapter row item | ---                    | `item_user.xml`               |
| Partial layout   | ---                    | `partial_user_details.xml`    |

Sometimes these rules will not be possible to apply. For example, when creating layout files that are intended to be part of other layouts. In this case you should use the prefix `partial_`.

### Menu files

Similar to layout files, menu files should match the name of the component. For example, if we are defining a menu file that is going to be used in the `UserActivity`, then the name of the file should be `activity_user.xml`

A good practice is to not include the word `menu` as part of the name because these files are already located in the `menu` directory.

### Values files

Resource files in the values folder should be __plural__, e.g. `strings.xml`, `styles.xml`, `colors.xml`, `dimens.xml`, `attrs.xml`

# Code guidelines

## Java language rules

Some of these are covered in the Java style guide, but are worth repeating here.

### Don't ignore exceptions

You must never do the following:

	void doSomething(String value) {
	    try {
	        int number = Integer.parseInt(value);
	    } catch (NumberFormatException e) { }
	}

_You may think this exception will never occur in your code, but if it does, debugging this is a complete nightmare. You should handle every exception in a sensible way._

The [Android code style guidelines](https://source.android.com/source/code-style.html#dont-ignore-exceptions) has more to say about this.

### NEVER catch generic exception

You should never do this:

	try {
	    someComplicatedIOFunction();        // may throw IOException
	    someComplicatedParsingFunction();   // may throw ParsingException
	    someComplicatedSecurityFunction();  // may throw SecurityException
	    // phew, made it all the way
	} catch (Exception e) {                 // I'll just catch all exceptions
	    handleError();                      // with one generic handler!
	}

See the why and some alternatives in the [Android code style guidelines](https://source.android.com/source/code-style.html#dont-catch-generic-exception)

### Don't use finalizers

There are no guarantees as to when a `finalizer` will be called, or even that it will be called at all. Usually you can replace a `finalizer` with good exception handling. Once again, see the [Android code style guidelines](https://source.android.com/source/code-style.html#dont-use-finalizers) for more.

### Annotations

#### Annotations practices

According to the Android code style guide, the standard practices for some of the predefined annotations in Java are:

* `@Override`: The @Override annotation __must be used__ whenever a method overrides the declaration or implementation from a super-class. For example, if you use the @inheritdocs Javadoc tag, and derive from a class (not an interface), you must also annotate that the method @Overrides the parent class's method.

* `@SuppressWarnings`: The @SuppressWarnings annotation should only be used under circumstances where it is impossible to eliminate a warning. If a warning passes this "impossible to eliminate" test, the @SuppressWarnings annotation must be used, so as to ensure that all warnings reflect actual problems in the code.

Code should also be refactored so that the code where the warning needs to be surpressed is isolated.

More information about annotation guidelines can be found [here](http://source.android.com/source/code-style.html#use-standard-java-annotations).

### Order import statements

If you are using an IDE such as Android Studio, you don't have to worry about this because your IDE is already obeying these rules. If not, have a look below.

The ordering of import statements is:

1. Android imports
2. Imports from third parties (com, junit, net, org)
3. java and javax
4. Same project imports

To exactly match the IDE settings, the imports should be:

* Alphabetically ordered within each grouping, with capital letters before lower case letters (e.g. Z before a).
* There should be a blank line between each major grouping (android, com, junit, net, org, java, javax).

### Logging guidelines

Use the logging methods provided by the `Log` class to print out error messages or other information that may be useful for developers to identify issues:

* `Log.v(String tag, String msg)` (verbose)
* `Log.d(String tag, String msg)` (debug)
* `Log.i(String tag, String msg)` (information)
* `Log.w(String tag, String msg)` (warning)
* `Log.e(String tag, String msg)` (error)

As a general rule, we use the class name as tag and we define it as a `static final` field at the top of the file. For example:

	public class MyClass {
    	private static final String TAG = MyClass.class.getSimpleName();

    	public myMethod() {
    	    Log.e(TAG, "My error message");
    	}
	}
VERBOSE and DEBUG logs __must__ be disabled on release builds. It is also recommended to disable INFORMATION, WARNING and ERROR logs but you may want to keep them enabled if you think they may be useful to identify issues on release builds. If you decide to leave them enabled, you have to make sure that they are not leaking private information such as email addresses, user ids, etc.

Proguard can be used to strip out log statements.

### Class member ordering

As per the Java style guide, There is no single correct solution for this but using a __logical__ and __consistent__ order will improve code learnability and readability. It is recommendable to use the following order:

1. Constants
2. Fields
3. Constructors
4. Override methods and callbacks (public or private)
5. Public methods
6. Private methods
7. Inner classes or interfaces

Example:

	public class MainActivity extends Activity {
		private String title;
	    private TextView textViewTitle;

	    public void setTitle(String title) {
	    	this.title = title;
	    }

	    @Override
	    public void onCreate() {
	        ...
	    }

	    private void setUpView() {
	        ...
	    }

	    static class AnInnerClass {
	
	    }

	}

If your class is extending an __Android component__ such as an Activity or a Fragment, it is a good practice to order the override methods so that they __match the component's lifecycle__. For example, if you have an Activity that implements `onCreate()`, `onDestroy()`, `onPause()` and `onResume()`, then the correct order is:

	public class MainActivity extends Activity {

		//Order matches Activity lifecycle
	    @Override
	    public void onCreate() {}

	    @Override
	    public void onResume() {}
	
	    @Override
	    public void onPause() {}
	
	    @Override
	    public void onDestroy() {}

	}

### Parameter ordering in methods

When programming for Android, it is quite common to define methods that take a `Context`. If you are writing a method like this, then the __Context__ must be the __first__ parameter.

The opposite case are __callback__ interfaces that should always be the __last__ parameter.

Examples:

	// Context always goes first
	public User loadUser(Context context, int userId);

	// Callbacks always go last
	public void loadUserAsync(Context context, int userId, UserCallback callback);

### String constants, naming, and values

Many elements of the Android SDK such as `SharedPreferences`, `Bundle`, or `Intent` use a key-value pair approach so it's very likely that even for a small app you end up having to write a lot of String constants.

When using one of these components, you __must__ define the keys as a `static final` fields and they should be prefixed as indicated below.

| Element            | Field Name Prefix |
| -----------------  | ----------------- |
| SharedPreferences  | `PREF_`             |
| Bundle             | `BUNDLE_`           |
| Fragment Arguments | `ARGUMENT_`         |
| Intent Extra       | `EXTRA_`            |
| Intent Action      | `ACTION_`           |

Note that the arguments of a Fragment - `Fragment.getArguments()` - are also a Bundle. However, because this is a quite common use of Bundles, we define a different prefix for them.

Example:

	// Note the value of the field is the same as the name to avoid duplication issues
	static final String PREF_EMAIL = "PREF_EMAIL";
	static final String BUNDLE_AGE = "BUNDLE_AGE";
	static final String ARGUMENT_USER_ID = "ARGUMENT_USER_ID";
	
	// Intent-related items use full package name as value
	static final String EXTRA_SURNAME = "com.myapp.extras.EXTRA_SURNAME";
	static final String ACTION_OPEN_USER = "com.myapp.action.ACTION_OPEN_USER";

### Arguments in Fragments and Activities

When data is passed into an `Activity `or `Fragment` via an `Intent` or a `Bundle`, the keys for the different values __must__ follow the rules described in the section above.

When an `Activity` or `Fragment` expects arguments, it should provide a `public static` method that facilitates the creation of the relevant `Intent` or `Fragment`.

In the case of Activities the method is usually called `getStartIntent()`:

	public static Intent getStartIntent(Context context, User user) {
		Intent intent = new Intent(context, ThisActivity.class);
		intent.putParcelableExtra(EXTRA_USER, user);
		return intent;
	}

For Fragments it is named `newInstance()` and handles the creation of the Fragment with the right arguments:

	public static UserFragment newInstance(User user) {
		UserFragment fragment = new UserFragment;
		Bundle args = new Bundle();
		args.putParcelable(ARGUMENT_USER, user);
		fragment.setArguments(args)
		return fragment;
	}

__Note 1__: These methods should go at the top of the class before `onCreate()`.

__Note 2__: If we provide the methods described above, the keys for extras and arguments should be `private` because there is not need for them to be exposed outside the class.

### Line length limit

Code lines should not exceed __120 characters__. If the line is longer than this limit there are usually two options to reduce its length:

* Extract a local variable or method (preferable).
* Apply line-wrapping to divide a single line into multiple ones.

There are two __exceptions__ where it is possible to have lines longer than 100:

* Lines that are not possible to split, e.g. long URLs in comments.
* `package` and `import` statements.

#### Line-wrapping strategies

There isn't an exact formula that explains how to line-wrap and quite often different solutions are valid. However there are a few rules that can be applied to common cases.

__Break at operators__

When the line is broken at an operator, the break comes __before__ the operator. For example:


	int longName = anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne
	        + theFinalOne;

__Assignment Operator Exception__

An exception to the `break at operators` rule is the assignment operator `=`, where the line break should happen __after__ the operator.


	int longName =
	        anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne + theFinalOne;

__Method chain case__

When multiple methods are chained in the same line - for example when using Builders - every call to a method should go in its own line, breaking the line before the `.`

	// This is bad:
	Picasso.with(context).load("http://ribot.co.uk/images/sexyjoe.jpg").into(imageView);

	// This is good:
	Picasso.with(context)
        .load("http://ribot.co.uk/images/sexyjoe.jpg")
        .into(imageView);

__Long parameters case__

When a method has many parameters or its parameters are very long, we should break the line after every comma `,`

	loadPicture(context, "http://ribot.co.uk/images/sexyjoe.jpg", mImageViewProfilePicture, clickListener, "Title of the picture");
Becomes:

	loadPicture(context,
    	    "http://ribot.co.uk/images/sexyjoe.jpg",
    	    mImageViewProfilePicture,
    	    clickListener,
    	    "Title of the picture");

### RxJava chains styling

Rx chains of operators require line-wrapping. Every operator must go in a new line and the line should be broken before the `.`

	public Observable<Location> syncLocations() {
    	return mDatabaseHelper.getAllLocations()
    	        .concatMap(new Func1<Location, Observable<? extends Location>>() {
    	            @Override
    	             public Observable<? extends Location> call(Location location) {
    	                 return mRetrofitService.getLocation(location.id);
    	             }
    	        })
    	        .retry(new Func2<Integer, Throwable, Boolean>() {
    	             @Override
    	             public Boolean call(Integer numRetries, Throwable throwable) {
    	                 return throwable instanceof RetrofitError;
    	             }
    	        });
	}


## XML style rules

### Use self closing tags

When an XML element doesn\'t have any contents, you __must__ use self closing tags.

This is good:

	<TextView
		android:id="@+id/text_view_profile"
		android:layout_width="wrap_content"
		android:layout_height="wrap_content" />

This is __bad__ :

	<!-- Don\'t do this! -->
	<TextView
	    android:id="@+id/text_view_profile"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content" >
	</TextView>


### Resources naming

Resource IDs and names are written in __lowercase_underscore__.

#### ID naming

IDs should be prefixed with the name of the element in lowercase underscore. For example:

| Element            | Prefix            |
| -----------------  | ----------------- |
| `TextView`           | `text_`             |
| `ImageView`          | `image_`            |
| `Button`             | `button_`           |
| `Menu`               | `menu_`             |

Image view example:

	<ImageView
	    android:id="@+id/image_profile"
	    android:layout_width="wrap_content"
	    android:layout_height="wrap_content" />

Menu example:

	<menu>
		<item
	        android:id="@+id/menu_done"
	        android:title="Done" />
	</menu>

#### Strings

String names start with a prefix that identifies the section they belong to. For example `registration_email_hint` or `registration_name_hint`. If a string __doesn\'t belong__ to any section, then you should follow the rules below:

| Prefix             | Description                           |
| -----------------  | --------------------------------------|
| `error_`             | An error message                      |
| `msg_`               | A regular information message         |
| `title_`             | A title, i.e. a dialog title          |
| `action_`            | An action such as "Save" or "Create"  |



#### Styles and Themes

Unlike the rest of resources, style names are written in __UpperCamelCase__.

### Attributes ordering

As a general rule you should try to group similar attributes together. A good way of ordering the most common attributes is:

1. View Id
2. Layout width and layout height
3. Other layout attributes, sorted alphabetically
4. Remaining attributes, sorted alphabetically
5. Style

## Tests style rules

### Unit tests

Test classes should match the name of the class the tests are targeting, followed by `Test`. For example, if we create a test class that contains tests for the `DatabaseHelper`, we should name it `DatabaseHelperTest`.

Test methods are annotated with `@Test` and should generally start with the name of the method that is being tested, followed by a precondition and/or expected behaviour.

* Template: `@Test void methodNamePreconditionExpectedBehaviour()`
* Example: `@Test void signInWithEmptyEmailFails()`

Precondition and/or expected behaviour may not always be required if the test is clear enough without them.

Sometimes a class may contain a large amount of methods, that at the same time require several tests for each method. In this case, it\'s recommendable to split up the test class into multiple ones. For example, if the `DataManager` contains a lot of methods we may want to divide it into `DataManagerSignInTest`, `DataManagerLoadUsersTest`, etc. Generally you will be able to see what tests belong together because they have common [test fixtures](https://en.wikipedia.org/wiki/Test_fixture).

### Espresso tests

Every Espresso test class usually targets an Activity, therefore the name should match the name of the targeted Activity followed by `Test`, e.g. `SignInActivityTest`

When using the Espresso API it is a common practice to place chained methods in new lines.

	onView(withId(R.id.view))
	        .perform(scrollTo())
	        .check(matches(isDisplayed()))

# License

Copyright 2015 Riaan Cornelius.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
