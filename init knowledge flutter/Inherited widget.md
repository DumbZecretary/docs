# Inherited Widget

## What is Inherited widget

Base class for widgets that efficiently propagate information down the tree.

## How to use Inherited widget

To obtain the nearest instance of a particular type of inherited widget from a build context, use [BuildContext.dependOnInheritedWidgetOfExactType](https://api.flutter.dev/flutter/widgets/BuildContext/dependOnInheritedWidgetOfExactType.html).

Inherited widgets, when referenced in this way, will cause the consumer to rebuild when the inherited widget itself changes state.

```dart
class FrogColor extends InheritedWidget {
  const FrogColor({
    super.key,
    required this.color,
    required super.child,
  });

  final Color color;

  static FrogColor of(BuildContext context) {
    final FrogColor? result = context.dependOnInheritedWidgetOfExactType<FrogColor>();
    assert(result != null, 'No FrogColor found in context');
    return result!;
  }

  @override
  bool updateShouldNotify(FrogColor old) => color != old.color;
}
```

## Implementing the `of` method

The convention is to provide a static method `of` on the [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html) which does the call to [BuildContext.dependOnInheritedWidgetOfExactType](https://api.flutter.dev/flutter/widgets/BuildContext/dependOnInheritedWidgetOfExactType.html). This allows the class to define its own fallback logic in case there isn't a widget in scope. In the example above, the value returned will be null in that case, but it could also have defaulted to a value.

Sometimes, the `of` method returns the data rather than the inherited widget; for example, in this case it could have returned a [Color](https://api.flutter.dev/flutter/dart-ui/Color-class.html) instead of the `FrogColor` widget.

Occasionally, the inherited widget is an implementation detail of another class, and is therefore private. The `of` method in that case is typically put on the public class instead. For example, [Theme](https://api.flutter.dev/flutter/material/Theme-class.html) is implemented as a [StatelessWidget](https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html) that builds a private inherited widget; [Theme.of](https://api.flutter.dev/flutter/material/Theme/of.html) looks for that inherited widget using [BuildContext.dependOnInheritedWidgetOfExactType](https://api.flutter.dev/flutter/widgets/BuildContext/dependOnInheritedWidgetOfExactType.html) and then returns the [ThemeData](https://api.flutter.dev/flutter/material/ThemeData-class.html).

## Calling the `of` method

When using the `of` method, the `context` must be a descendant of the [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html), meaning it must be "below" the [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html) in the tree.

In this example, the `context` used is the one from the [Builder](https://api.flutter.dev/flutter/widgets/Builder-class.html), which is a child of the FrogColor widget, so this works.

```dart
class MyPage extends StatelessWidget {
  const MyPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: FrogColor(
        color: Colors.green,
        child: Builder(
          builder: (BuildContext innerContext) {
            return Text(
              'Hello Frog',
              style: TextStyle(color: FrogColor.of(innerContext).color),
            );
          },
        ),
      ),
    );
  }
}
```

In this example, the `context` used is the one from the MyOtherPage widget, which is a parent of the FrogColor widget, so this does not work.

```dart
class MyOtherPage extends StatelessWidget {
  const MyOtherPage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: FrogColor(
        color: Colors.green,
        child: Text(
          'Hello Frog',
          style: TextStyle(color: FrogColor.of(context).color),
        ),
      ),
    );
  }
}
```

## More to read

[Inherited Widget]([InheritedWidget class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html))

[Inherited Notifier]([InheritedNotifier class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/InheritedNotifier-class.html))

[Listenable]([Listenable class - foundation library - Dart API](https://api.flutter.dev/flutter/foundation/Listenable-class.html))

[Inherited Model]([InheritedModel class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/InheritedModel-class.html))

[Of - power]([dependOnInheritedWidgetOfExactType method - BuildContext class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/BuildContext/dependOnInheritedWidgetOfExactType.html))


