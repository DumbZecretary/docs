# Widgets vs Helper Methods

## Why we should compare them

You are used to implementing features by wrapping parts of your UI in new widgets

```dart
ListView(
    children: <Widget>[
        Dismissible(
            key: ValueKey(...),
            child: Card(
                child: ListTile(
                    child: Text("some text"),
                ),
            ),
        ),
    ],
),
```

Then, breaking up an unwieldy build method

```dart
ListView(
    children: <Widget>[
        for(final obj in myObj) {
            _buildWidget(obj);
        }
    ],
),

// In the same class
Widget _buildWidget(MyObj myObj) {
    return Dismissible(...),
}
```

This is called the "helper method"

The second way is creating a new StateLessWidget which returns `Dismissible(...);` 

They are simillar but not the same. When rebuilding entire parent widgets or the wrapping widgets will re-render all things of helper methods, and watse precious CPU time . When creating a new widget and using it, remember to use const contructors whenever possiple.

## Advantages

### Widgets

- Good performance

- Prevent multi context

### Helper Methods

- Easy to code

## Disadvantages

### Widgets

- More code

- Hard to maintain if you are a new developer

### Helper Methods

- Bad performance

- Watse resource

## Conclusion

Classes have a better default behavior. The only benefit of methods is having to write a tiny bit less code. There's no functional benefit. - Remi Rousselet - Author of Provider, RiverPop, Frezzed

## Preferences

[docs]([Widgets vs helper methods | Decoding Flutter - YouTube](https://youtu.be/IOyq-eTRhvo))
