# Stateful widget

This artical will show my knowledge of Stateful widget - One of the powerful things of Flutter - Dart

## What is Stateful widget

A widget that has mutable state.

State is information that (1) can be read synchronously when the widget is built and (2) might change during the lifetime of the widget. It is the responsibility of the widget implementer to ensure that the [State](https://api.flutter.dev/flutter/widgets/State-class.html) is promptly notified when such state changes, using [State.setState](https://api.flutter.dev/flutter/widgets/State/setState.html).

A stateful widget is a widget that describes part of the user interface by building a constellation of other widgets that describe the user interface more concretely. The building process continues recursively until the description of the user interface is fully concrete (e.g., consists entirely of [RenderObjectWidget](https://api.flutter.dev/flutter/widgets/RenderObjectWidget-class.html)s, which describe concrete [RenderObject](https://api.flutter.dev/flutter/rendering/RenderObject-class.html)s).

## Why we should use Stateful widget

Stateful widgets are useful when the part of the user interface you are describing can change dynamically, e.g. due to having an internal clock-driven state, or depending on some system state. For compositions that depend only on the configuration information in the object itself and the [BuildContext](https://api.flutter.dev/flutter/widgets/BuildContext-class.html) in which the widget is inflated, consider using [StatelessWidget](https://api.flutter.dev/flutter/widgets/StatelessWidget-class.html).

## How we use Stateful widget

[StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) instances themselves are immutable and store their mutable state either in separate [State](https://api.flutter.dev/flutter/widgets/State-class.html) objects that are created by the [createState](https://api.flutter.dev/flutter/widgets/StatefulWidget/createState.html) method, or in objects to which that [State](https://api.flutter.dev/flutter/widgets/State-class.html) subscribes, for example [Stream](https://api.flutter.dev/flutter/dart-async/Stream-class.html) or [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html) objects, to which references are stored in final fields on the [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) itself.

The framework calls [createState](https://api.flutter.dev/flutter/widgets/StatefulWidget/createState.html) whenever it inflates a [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html), which means that multiple [State](https://api.flutter.dev/flutter/widgets/State-class.html) objects might be associated with the same [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) if that widget has been inserted into the tree in multiple places. Similarly, if a [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) is removed from the tree and later inserted in to the tree again, the framework will call [createState](https://api.flutter.dev/flutter/widgets/StatefulWidget/createState.html) again to create a fresh [State](https://api.flutter.dev/flutter/widgets/State-class.html) object, simplifying the lifecycle of [State](https://api.flutter.dev/flutter/widgets/State-class.html) objects.

A [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) keeps the same [State](https://api.flutter.dev/flutter/widgets/State-class.html) object when moving from one location in the tree to another if its creator used a [GlobalKey](https://api.flutter.dev/flutter/widgets/GlobalKey-class.html) for its [key](https://api.flutter.dev/flutter/widgets/Widget/key.html). Because a widget with a [GlobalKey](https://api.flutter.dev/flutter/widgets/GlobalKey-class.html) can be used in at most one location in the tree, a widget that uses a [GlobalKey](https://api.flutter.dev/flutter/widgets/GlobalKey-class.html) has at most one associated element. The framework takes advantage of this property when moving a widget with a global key from one location in the tree to another by grafting the (unique) subtree associated with that widget from the old location to the new location (instead of recreating the subtree at the new location). The [State](https://api.flutter.dev/flutter/widgets/State-class.html) objects associated with [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) are grafted along with the rest of the subtree, which means the [State](https://api.flutter.dev/flutter/widgets/State-class.html) object is reused (instead of being recreated) in the new location. However, in order to be eligible for grafting, the widget must be inserted into the new location in the same animation frame in which it was removed from the old location.

## Performance considerations

There are two primary categories of [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html)s.

The first is one which allocates resources in [State.initState](https://api.flutter.dev/flutter/widgets/State/initState.html) and disposes of them in [State.dispose](https://api.flutter.dev/flutter/widgets/State/dispose.html), but which does not depend on [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)s or call [State.setState](https://api.flutter.dev/flutter/widgets/State/setState.html). Such widgets are commonly used at the root of an application or page, and communicate with subwidgets via [ChangeNotifier](https://api.flutter.dev/flutter/foundation/ChangeNotifier-class.html)s, [Stream](https://api.flutter.dev/flutter/dart-async/Stream-class.html)s, or other such objects. Stateful widgets following such a pattern are relatively cheap (in terms of CPU and GPU cycles), because they are built once then never update. They can, therefore, have somewhat complicated and deep build methods.

The second category is widgets that use [State.setState](https://api.flutter.dev/flutter/widgets/State/setState.html) or depend on [InheritedWidget](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)s. These will typically rebuild many times during the application's lifetime, and it is therefore important to minimize the impact of rebuilding such a widget. (They may also use [State.initState](https://api.flutter.dev/flutter/widgets/State/initState.html) or [State.didChangeDependencies](https://api.flutter.dev/flutter/widgets/State/didChangeDependencies.html) and allocate resources, but the important part is that they rebuild.)

There are several techniques one can use to minimize the impact of rebuilding a stateful widget:

- Push the state to the leaves. For example, if your page has a ticking clock, rather than putting the state at the top of the page and rebuilding the entire page each time the clock ticks, create a dedicated clock widget that only updates itself.

- Minimize the number of nodes transitively created by the build method and any widgets it creates. Ideally, a stateful widget would only create a single widget, and that widget would be a [RenderObjectWidget](https://api.flutter.dev/flutter/widgets/RenderObjectWidget-class.html). (Obviously this isn't always practical, but the closer a widget gets to this ideal, the more efficient it will be.)

- If a subtree does not change, cache the widget that represents that subtree and re-use it each time it can be used. To do this, simply assign a widget to a `final` state variable and re-use it in the build method. It is massively more efficient for a widget to be re-used than for a new (but identically-configured) widget to be created. Another caching stragegy consists in extracting the mutable part of the widget into a [StatefulWidget](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html) which accepts a child parameter.

- Use `const` widgets where possible. (This is equivalent to caching a widget and re-using it.)

- Avoid changing the depth of any created subtrees or changing the type of any widgets in the subtree. For example, rather than returning either the child or the child wrapped in an [IgnorePointer](https://api.flutter.dev/flutter/widgets/IgnorePointer-class.html), always wrap the child widget in an [IgnorePointer](https://api.flutter.dev/flutter/widgets/IgnorePointer-class.html) and control the [IgnorePointer.ignoring](https://api.flutter.dev/flutter/widgets/IgnorePointer/ignoring.html) property. This is because changing the depth of the subtree requires rebuilding, laying out, and painting the entire subtree, whereas just changing the property will require the least possible change to the render tree (in the case of [IgnorePointer](https://api.flutter.dev/flutter/widgets/IgnorePointer-class.html), for example, no layout or repaint is necessary at all).

- If the depth must be changed for some reason, consider wrapping the common parts of the subtrees in widgets that have a [GlobalKey](https://api.flutter.dev/flutter/widgets/GlobalKey-class.html) that remains consistent for the life of the stateful widget. (The [KeyedSubtree](https://api.flutter.dev/flutter/widgets/KeyedSubtree-class.html) widget may be useful for this purpose if no other widget can conveniently be assigned the key.)

- When trying to create a reusable piece of UI, prefer using a widget rather than a helper method. For example, if there was a function used to build a widget, a [State.setState](https://api.flutter.dev/flutter/widgets/State/setState.html) call would require Flutter to entirely rebuild the returned wrapping widget. If a [Widget](https://api.flutter.dev/flutter/widgets/Widget-class.html) was used instead, Flutter would be able to efficiently re-render only those parts that really need to be updated. Even better, if the created widget is `const`, Flutter would short-circuit most of the rebuild work.

## Life circle

![](/Users/tantran/Documents/personal_docs/init%20knowledge%20flutter/assets/stateful.jpg)

Using: init, didUpdateDependencies, deactivate, dispose 

## Preferences

[Docs]([StatefulWidget class - widgets library - Dart API](https://api.flutter.dev/flutter/widgets/StatefulWidget-class.html))
