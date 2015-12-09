# android-style-guide

# Project structure

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

### Don't ignore exceptions

You must never do the following:

```java
void doSomething(String value) {
    try {
        int number = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```

_You may think this exception will never occur in your code, but if it does, debugging this is a complete nightmare. You should handle every exception in a sensible way._

The [Android code style guidelines](https://source.android.com/source/code-style.html#dont-ignore-exceptions) has more to say about this.

### NEVER catch generic exception

You should never do this:

```java
try {
    someComplicatedIOFunction();        // may throw IOException
    someComplicatedParsingFunction();   // may throw ParsingException
    someComplicatedSecurityFunction();  // may throw SecurityException
    // phew, made it all the way
} catch (Exception e) {                 // I'll just catch all exceptions
    handleError();                      // with one generic handler!
}
```

See the why and some alternatives in the [Android code style guidelines](https://source.android.com/source/code-style.html#dont-catch-generic-exception)

### Don't use finalizers

There are no guarantees as to when a finalizer will be called, or even that it will be called at all. Usually you can replace a finalizer with good exception handling. Once again, see the [Android code style guidelines](https://source.android.com/source/code-style.html#dont-use-finalizers) for more.

### Import statements

The Android code style guidelines states that:

This is bad: `import foo.*;`

This is good: `import foo.Bar;`

See more info [here](https://source.android.com/source/code-style.html#fully-qualify-imports)

I disagree. The logic is that having fully qualified imports makes it obvious which classes are used, which makes the code more readable and maintainable. 

I **very rarely** look at the import statements in code I'm working on. I CTRL-click on any functionality I'm trying to understand. Once I'm in that class I occasionally look at the package name.

Personally, I don't particularly care which are used. I usually go for `import foo.*` if I'm importing multiple classes from the package and `import foo.Bar` if I'm only importing the one class. I prefer having less code.
 
## Java style rules

For Java style I usually stick with the original Java style guide (from 1997!) with minor modifications.

### Fields definition and naming

Fields should be defined at the __top of the file__

* Fields start with a lower case letter.
* Static final fields (constants) are ALL_CAPS_WITH_UNDERSCORES.

Example:

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
}
```

### Treat acronyms as words

| Good             | Bad            |
| --------------   | -------------- |
| `XmlHttpRequest` | `XMLHTTPRequest` |
| `getCustomerId`  | `getCustomerID`  |
| `String url`     | `String URL`     |
| `long id`        | `long ID`        |

### Use spaces for indentation

Use __4 space__ indents for blocks:

```java
if (x == 1) {
    x++;
}
```

Use __8 space__ indents for line wraps:

```java
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

### Use standard brace style

Braces go on the same line as the code before them.

```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```
Braces around the statements are required.

This is __bad__:

```java
if (condition)
    body();  // bad!
```

### Annotations

#### Annotations practices

According to the Android code style guide, the standard practices for some of the predefined annotations in Java are:

* `@Override`: The @Override annotation __must be used__ whenever a method overrides the declaration or implementation from a super-class. For example, if you use the @inheritdocs Javadoc tag, and derive from a class (not an interface), you must also annotate that the method @Overrides the parent class's method.

* `@SuppressWarnings`: The @SuppressWarnings annotation should only be used under circumstances where it is impossible to eliminate a warning. If a warning passes this "impossible to eliminate" test, the @SuppressWarnings annotation must be used, so as to ensure that all warnings reflect actual problems in the code.

Code should also be refactored so that the code where the warning needs to be surpressed is isolated.

More information about annotation guidelines can be found [here](http://source.android.com/source/code-style.html#use-standard-java-annotations).

