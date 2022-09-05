# Creational Design Patterns for Dart and Flutter: Singleton

[source]([Creational Design Patterns for Dart and Flutter: Singleton](https://dart.academy/creational-design-patterns-for-dart-and-flutter-singleton/))

When you need to ensure only a single instance of a class is created, and you want to provide a global point of access to it, the Singleton design pattern can be of great help. Examples of typical singletons include an app-wide debug logger or the data model for an app's configuration settings. If multiple parts of an app need access to such an object, each area could create its own instance or have one passed to it, but the Singleton pattern offers a cleaner, more memory efficient way. Often, the implementation includes lazy instantiation, which means the single instantiation occurs only upon first access.

> **About creational design patterns:** These patterns, as the name implies, help us with the creation of objects and processes related to creating objects. With these techniques in your arsenal, you can code faster, create more flexible, reusable object templates, and sculpt a universally recognizable structure for your projects. The creational design patterns are blueprints you can follow to reliably tackle object creation issues that arise for many software projects.

In this article, we'll introduce the Singleton pattern using a typical implementation before looking at how [Dart](https://dart.dev/) syntax can improve on the classic structure, then we'll examine a real-world example in the form of a debug message logger.

> The code for this article was tested with Dart 2.8.4 and Flutter 1.17.5.

## A typical singleton implementation

The following diagram illustrates the most basic structure of a singleton class:

![](https://i.imgur.com/RssSAQz.png)

A typical singleton class has a private, static variable containing a reference to the class's one instance. Singleton constructors optimally don't take any parameters. Purists argue that if configuration parameters are allowed, it's possible to create a singleton object that differs from another based on those values, which may violate the basic premise of the Singleton pattern, as a user is not guaranteed a completely predictable instance. The `getInstance()` method is also static and serves as the gateway to the singleton's cached instance.

Let's look at how we might implement this generic approach in Dart:

```dart
class Singleton {
  static Singleton _instance;

  Singleton._internal();

  static Singleton getInstance() {
    if (_instance == null) {
      _instance = Singleton._internal();
    }

    return _instance;
  }
}
```

First, we define the private `_instance` variable as a `static` property on the Singleton class. This will be accessible only within the class's library, so outside code has to use `getInstance()` to access it. If `getInstance()` finds the one allowed instance doesn't yet exist, it creates the instance using the private, named constructor `_internal()`, then it returns the cached instance.

If a code segment needs access to the singleton's instance, this code will get it:

```dart
final singleton = Singleton.getInstance();
```

If code outside the singleton's own library attempts to directly instantiate a singleton, the Dart analyzer will flag an error, pointing out that Singleton has no public default constructor. It can only be instantiated via the private constructor, and only from within the library.

Next, we'll take a look at how the pattern can be simplified using more idiomatic Dart syntax.

## Singleton the Dart way

Dart includes some features we can use to implement the Singleton pattern more elegantly. In this first example, we'll get rid of the awkward `getInstance()` static method and replace it with a getter:

```dart
class Singleton {
  static Singleton _instance;
  static get instance {
    if (_instance == null) {
      _instance = Singleton._internal();
    }

    return _instance;
  }

  Singleton._internal();
}
```

A Dart getter operates almost exactly like a method, but it doesn't require the caller to use parentheses.

The getter makes accessor code more readable, since it looks more like standard property access syntax, as in the following example:

```dart
final singleton = Singleton.instance;
```

A nice improvement, but can we do better? Using Dart's `factory` keyword, we can effectively hide the use of the Singleton pattern altogether:

```dart
class Singleton {
  static Singleton _instance;

  Singleton._internal();

  factory Singleton() {
    if (_instance == null) {
      _instance = Singleton._internal();
    }

    return _instance;
  }
}
```

In this version, we add a public default constructor that outside code can use, but we mark it as a `factory`. A factory constructor can be used in cases where the constructor doesn't always create a new instance of its class, as a standard constructor must. In our example, the factory constructor creates a new instance just once, then it returns that cached instance on every future invocation.

Now, a user of the class can use more familiar syntax to acquire a reference to a Singleton instance:

```dart
final singleton = Singleton();
```

Though it looks like a typical object instantiation, a new object will be created only the first time this constructor is used, but that is an implementation detail hidden from casual view.

## Dart Singleton with even less code

Astute readers will have noticed (and some did) that our final Singleton example could be accomplished with even less code using some of Dart's more interesting operators, such as the *if null* operator (`??`):

```dart
class Singleton {
  static Singleton _instance;

  Singleton._internal() {
    _instance = this;
  }

  factory Singleton() => _instance ?? Singleton._internal();
}
```

When we do it this way, client code gets an instance as before, by calling the `factory` constructor, but this time it's a fat arrow function to keep things short. The one expression of the factory checks if `_instance` is `null`, and if it is, it returns the result of `Singleton._internal()`, which sets the static `_instance` to the singleton's reference. If `_instance` has been set by a previous call to `Singleton._internal()`, the cached instance is returned instead. Elegant!

So, how might you use the Singleton pattern in a real application?

## Singleton example: Logger

Most apps need an easy way to log messages to the debug console during development. Creating a wrapper around Dart's most popular logger implementation, from the `logging` package, can make your app's logging solution more robust and customizable, and the Singleton pattern can be applied to prevent unneeded logger instances from cluttering up a device's memory space:

```dart
import 'package:logging/logging.dart';
import 'package:intl/intl.dart' show DateFormat;

class DebugLogger {
  static DebugLogger _instance;
  static Logger _logger;
  static final _dateFormatter = DateFormat('H:m:s.S');
  static const appName = 'my_app';

  DebugLogger._internal() {
    Logger.root.level = Level.ALL;
    Logger.root.onRecord.listen(_recordHandler);

    _logger = Logger(appName);

    _instance = this;
  };

  factory DebugLogger() => _instance ?? DebugLogger._internal();

  void _recordHandler(LogRecord rec) {
    print('${_dateFormatter.format(rec.time)}: ${rec.message}');
  }

  void log(message, [Object error, StackTrace stackTrace]) =>
      _logger.info(message, error, stackTrace);
}
```

After including the logging package in our app's dependencies, we can import that library to gain access to its Logger implementation. We also need the `intl` package, since it contains Dart's standard DateFormat service.

> **Dart code packages:** You can learn more about `logging`, `intl`, and all of the other available packages on Dart's package repository site, [Pub](https://pub.dev/).

Our DebugLogger class starts off by defining a private, static instance variable, as discussed previously in our exploration of the Singleton design pattern. Another one is declared for an instance of the Logger class that will provide logging functionality. Then we set up a date formatter that can be used to create a readable string from a DateTime object. The `logging` package's Logger constructor requires the app name value, so we set that up here as well.

Next, the private, named constructor, which we've called `_internal()` per established convention, is defined. Its body does some basic setup for global logging, including setting the severity level of messages we want to deal with and registering a callback for handling generated log records. Finally, a logger is instantiated, and its reference is stored.

The default constructor for DebugLogger is a factory constructor. Its job is to lazily construct and return the managed instance. This is what makes the class a singleton.

The private method `_recordHandler()` serves as the callback for handling new log records as they're produced. The method prints the log message to the debug console, including the formatted time stamp. A developer could alter the body of this method to customize log output for any given app.

DebugLogger has one more public method, `log()`, that uses the logger to post a message at the `Logger.INFO` severity level. More public methods could be added to allow for logging messages with other severity levels.

Any code that wants to make use of the debug logger simply has to import the library containing DebugLogger, then run the factory constructor:

```dart
final logger = DebugLogger();
```

Every time a DebugLogger is constructed in this manner, the same instance will be returned, and only a single logger will ever be created.

## Conclusion

This article introduced the Singleton pattern, which you can use to restrict instantiation of a class to one object and provide a global access point to it. To read more about creational design patterns in Dart, check out these related articles:

- [Creational Design Patterns for Dart and Flutter: Factory Method](https://dart.academy/creational-design-patterns-for-dart-and-flutter-factory-method)
- [Creational Design Patterns for Dart and Flutter: Builder](https://dart.academy/creational-design-patterns-for-dart-and-flutter-builder)
- [Creational Design Patterns for Dart and Flutter: Prototype](https://dart.academy/creational-design-patterns-for-dart-and-flutter-prototype)
