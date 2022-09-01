# Stream

[blog](https://www.raywenderlich.com/32851541-dart-futures-and-streams)

Dart, like many other programming languages, has built-in support for *asynchronous programming* — writing code that runs later. Your program can wait for work to finish and perform other tasks while waiting. This technique is often used to fetch information from a device or server and not block your program.

In this tutorial, you’ll learn how to use *Futures* and *Streams* for writing asynchronous code in Dart.

## Getting Started

This tutorial is a pure Dart experience so use *DartPad* at [https://dartpad.dev/](https://dartpad.dev/). DartPad lets you edit and run Dart code in the browser. There is no corresponding Flutter app, but all the concepts and code can be used in a normal Flutter app.

For this tutorial, you’ll simulate an API that returns demographic data on world cities. The program will “load” data from this simulated API as if you were building a mapping or educational app. In reality, the data is all hard-coded and sent after a time delay.

In DartPad, create a new *Dart* pad.

![](/Users/tantran/Library/Application%20Support/marktext/images/2022-09-01-11-15-53-image.png)

You’ll see a basic `main` function that prints a count.

## Async Basics

Synchronous code has limits. You can only run one chunk of code at a time, which might be problematic if that code takes a long time to run. Asynchronous code lets you write code that responds to events that happen later, such as data becoming available from an API.

For the cities program, display a greeting and load some cities.

Replace the DartPad contents with:

```dart
List<String> fetchCityList() {
  print("[SIMULATED NETWORK I/O]");
  return ['Bangkok', 'Beijing', 'Cairo', 'Delhi', 'Guangzhou', 
'Jakarta', 'Kolkāta', 'Manila', 'Mexico City', 'Moscow', 'Mumbai', 
'New York', 'São Paulo', 'Seoul', 'Shanghai', 'Tokyo'];
}
void printCities(List<String> cities) {
 print("Cities:");
 for (final city in cities) {
 print(" " + city);
 }
}

void main() {
 print("Welcome to the Cities program!");
 final cities = fetchCityList();
 printCities(cities);
}
```

This has two helper functions: `fetchCityList` returns a list of cities, and `printCities` prints that list to the console. The `main` function displays the greeting and uses the helper method to fetch the cities. For a Flutter app, you might display the city list using a *ListView* widget.

The `fetchCityList` function is meant to simulate loading data from the network, but it finishes immediately, which isn’t how an API call normally works.

![](/Users/tantran/Library/Application%20Support/marktext/images/2022-09-01-11-16-57-image.png)

The order of operations in the main function. Each step completes before the next one.

What if that fetch was slow or error-prone?

Replace the existing `main` function with the following (try not to read into it yet):

```dart
Future<List<String>> fetchSlowCityList() async {
  print("Loading...");
  await Future.delayed(Duration(seconds: 2));
  return fetchCityList();
}

void main() async {
  print("Welcome to the Cities program!");
  final cities = await fetchSlowCityList();
  printCities(cities);
}
```

Run the code and you’ll feel the delay in the console. The 2-second delay is helpful because you don’t know your end-users’ networking conditions.

![](/Users/tantran/Library/Application%20Support/marktext/images/2022-09-01-11-17-56-image.png)

This time there is a forced wait to get the results.

If you were to run the above code on the main thread of a Flutter app, your app would be unresponsive while it waits for completion.

## Using Futures

Now that you’ve seen the Dart type [`Future`](https://api.dart.dev/stable/2.16.2/dart-async/Future-class.html) in action, let’s learn how to use it in your own code.

A *Future* is an object that will return a value (or an error) in the future. It’s critical to understand the `Future` itself is not the value, but rather the *promise of a value*.

To see this in practice, replace `main` with the following:

```dart
void main() {
  print("Welcome to the Cities program!");
  // 1
  final future = Future.delayed(
    const Duration(seconds: 3),
    fetchCityList,
  );
  // 2
  print("The future object is actually: " + future.toString());
  // 3
  future.then((cities) {
    printCities(cities);
  });
  // 4
  print("This happens before the future completes.");
}
```

This example highlights some of the properties and conditions of using futures.

1. A `Future` can be built a few ways. A `delayed` future will execute a provided computation after a specified delay. In this case, the delay is a 3-second `Duration`, and the computation is a call to `fetchCityList`.
2. This `print` statement is a reminder that a `Future` is its own type and isn’t the value of the computation.
3. The `then` function will execute a callback with a computed value when the future is complete. In this case, the value completed value is the `cities` variable, and it is then forwarded to the print-out function.
4. This `print` statement is a reminder that even though this appears after the call to `then` in program order. It’ll execute before the completion callback.

If you run the program, you’ll see the output shows the order in which the code is executed.

```
Welcome to the Cities program!
The future object is actually: Instance of '_Future<List<String>>'
This happens before the future completes.
[SIMULATED NETWORK I/O]
Cities:
   Bangkok
   Beijing
   Cairo
   Delhi
   Guangzhou
   Jakarta
   Kolkāta
   Manila
   Mexico City
   Moscow
   Mumbai
   New York
   São Paulo
   Seoul
   Shanghai
   Tokyo
```

![](/Users/tantran/Library/Application%20Support/marktext/images/2022-09-01-11-18-50-image.png)

While the “network” i/o is happening, the final print statement is printed. The city list prints later.

### Future States

A `Future` has two states: *uncompleted* and *completed*. An uncompleted `Future` is one that hasn’t produced a value (or error) yet. A completed `Future` is a `Future` after computing its value.

In this next example, you’ll use a `Timer` to show a loading indicator text in the console. At the top of the DartPad, add:

```dart
import 'dart:async';
```

This will import the `async` package so you can use a timer. Next, replace `main` with:

```dart
void main() {
  // 1
  final future = fetchSlowCityList();
  // 2
  final loadingIndicator = Timer.periodic(
    const Duration(milliseconds: 250),
    (timer) => print("."));
  // 3
  future.whenComplete(() => loadingIndicator.cancel());
  // 4
  future.then((cities) {
    printCities(cities);
  });
}
```

This code illustrates the completed state by assigning a `whenComplete` callback. This code does the following:

1. Reuses the helper function to create a city list `Future`.
2. The timer will drop a “.” in the console every 250 milliseconds.
3. When the loading future is complete, it cancels the timer so it stops printing dots.
4. When the future completes, print the completed value to the console.

`whenComplete` is called when the future finishes, regardless of whether it produced a value or an error. It’s a good place to clean up state, such as canceling a timer. On the other hand, `then` is only called when a value is present.

### Future Types

So far, you’ve seen `delayed`, which performs an operation after a time delay. Two other handy `Future` constructors are `value` and `error`.

To try them, replace `main` once again with:

```dart
void main() {
  final valueFuture = Future.value("Atlanta");
  final errorFuture = Future.error("No city found");
  valueFuture.then((value) => print("Value found: " + value));
  errorFuture.then((value) => print("Value found: " + value));
}
```

As you can see, `Future.value` needs a value and `Future.error` needs an error description.

When run, you get the following output:

```
Value found: Atlanta
Uncaught Error: No city found
```

Notice that the error future’s `then` is not evaluated and an uncaught error is recorded in the console. You’ll learn about error handling in the next section.

A few other constructor functions are available for creating `Future` instances. Of those, `sync` might be the handiest.

Add the following helper function:

```dart
int calculatePopulation() {
  return 1000;
}
```

This simulates a population calculation. It’s a regular function that returns immediately.

Next, in `main`, add to the bottom:

```dart
valueFuture
  .then((value) => Future.sync(calculatePopulation))
  .then((value) => print("Population found: $value"));
```

The `sync` constructor is similar to the `value` constructor. The main difference is it’s also possible for `sync` to error as well as complete. It’s most useful in an operation like this where you want to run some synchronous code in response to an asynchronous one.

You’ll mostly use a future in two main ways. One way is by wrapping an `async` function; the other is by transforming futures returned from APIs and packages. You’ll learn more about these in a future section.

## Dealing with Errors

As you’ve seen, some futures can produce errors. That can include network i/o errors, disconnects or data deserialization errors.

Error handling is built into the Future API. The way to handle errors is to catch them and then act. For example, add the following to `main`:

```dart
errorFuture
  .then((value) => print("Then callback called."))
  .catchError((error) => print("Error found: " + error));
```

A `catchError` callback is provided, which catches the error. If you run and look in the console, you’ll see the `then` callback was not run, but the `catchError` callback was.

```
Error found: No city found
```

Futures can have a chain of `then` callbacks that transform the results of one operation with one catch block at the end of the chain. This single block will handle any error along the way. For example:

```dart
errorFuture
  .then((value) => print("Load configuration"))
  .then((value) => print("Login with username and password."))
  .then((value) => print("Deserialize the user information."))
  .catchError((error) => print("Couldn't load user info, please try again"));
```

In this example, you can imagine a multistep startup sequence where no matter where it fails, you can prompt the user to try again.

You can also have different error handling for different errors. For example, replace `main` again with:

```dart
class NetworkError implements Exception {}
class LoginError implements Exception{}

void main() {
  final errorFuture = Future.error(LoginError());
  errorFuture.then((value) => print("Success!"))
    .catchError((error) => print("Network failed, try again."),
                test: (error) => error is NetworkError)
    .catchError((error) => print("Invalid username or password."),
                test: (error) => error is LoginError)
    .catchError((error) => print("Generic error, log it!"));
}
```

This creates two custom exception types: `NetworkError` and `LoginError`. Then, with a series of `catchError` calls, it uses the optional `test` parameter to run various callbacks depending on the type of error. You could use the `test` parameter to check HTTP codes or error descriptions.

## Using Async and Await

If you like writing code that performs asynchronous operations, but you don’t like the chaining of callbacks, the *async*/*await* keyword pair will be useful.

Denoting a function as *async* lets the compiler know that it won’t return right away. You then call such a function with an *await* keyword so the program knows to wait before continuing. You saw it earlier with the simulated network operation delay, but now you’ll learn to use it.

First, recall that when using a `Future`, the return value has to be in a `then` callback. For reference, replace `main` with:

```dart
void main() {
  print("Loading future cities...");
  // 1
  fetchSlowCityList().then(printCities);
  // 2
  print("done waiting for future cities");
}
```

In this example, you’ll notice:

1. The data handling function is set with the future’s `then` method.
2. This print is executed immediately after starting the future, so the print happens in the console before the city list prints.

This can be rewritten to be easier to read and to get the print statement to print after the work is finished. Once again, replace `main` with:

```dart
// 1
void main() async {
  print("Loading future cities...");
  // 2
  final cities = await fetchSlowCityList();
  // 3
  printCities(cities);
  // 4
  print("done waiting for future cities");
}
```

This modification uses `await` when calling the `Future`, making the code more linear. Here’s some details:

1. First, the `main` function has the `async` keyword. This indicates the function will not immediately return. It’s required for any function that uses an `await`.
2. By using the `await` keyword here, you can assign the `value` of the completed `Future` to a local variable instead of having to wrap the value handler in a completion block. Program execution will stop on this line until the `Future` completes.
3. By waiting for completion, the code can now ensure that `cities` will have a value so you can use it as an input to another function such as `printCities`.
4. The final `print` statement will happen after the city list is printed, which you can verify by running the program and checking the console.

Take a look at an `async` function. You added this function earlier:

```dart
Future<List<String>> fetchSlowCityList() async {
  print("[SIMULATED NETWORK I/O]");
  await Future.delayed(Duration(seconds: 2));
  return fetchCityList();
}
```

The consequence of using an `await` inside a function is you must annotate the function definition with the `async` keyword. That means the function does not immediately return. Also, you have to designate the function’s return type is a `Future`. In this case, the `return` value is the output of `fetchCityList()`, which is a `List`. By using the `async` keyword, this value is wrapped in a `Future`, and thus the function definition needs to indicate that.

The `return` value is a `List`. When it’s used in `main`, the assignment to `cities` with the `await` treats this value’s type as `List`. When using `async/await`, other than in function definitions, you needn’t worry about the `Future` type.

### Error Handling with Await

You previously used `catchError` blocks to handle `Future` errors. By using `await`, you can instead use a regular `try/catch`. For example, rewrite the previous error handling by replacing `main` with the following:

```dart
Future<int> getCityCount() async {
  throw NetworkError();
}

void main() async {
  try {
    final cityCount = await getCityCount();
    print("Got a value: $cityCount");
  } catch(e) {
    print("Got an error: $e");
  }
}
```

The `getCityCount` function should theoretically return the number of cities, but it instead throws a `NetworkError`. In `main`, you can wrap the call to `getCityCount` with a `try/catch` to catch the error and avoid a crash.

In the above example, the first print never executes because `getCityCount` eventually returns an exception. It’s caught in the catch block and printed to the console.

Just like using `test` with the `catchError` method on `Future`, you can also stack `catch` blocks to handle various errors in various ways. Change `main` once again:

```dart
void main() async {
  try {
    final cityCount = await getCityCount();
    print("Got a value: $cityCount");
  } on LoginError {
    print("Invalid username or password.");
  } on NetworkError {
    print("Network failed, try again.");
  } catch(e) {
    print("Got an error: $e");
  }
}
```

With this stack of `on` and `catch` blocks, the console outputs the specific `NetworkError` message instead of the generic one. You’ll also see this code is more straightforward to read and understand than the earlier example with error and test callbacks.

## Using Streams

*Streams* are another aspect to asynchronous programming in Dart. A `Future` represents a one-time value: the app performs an operation and comes back with some data. A `Stream` represents a sequence of data. For example, you can use a `Future` to get the entire contents of a file or use a `Stream` to get the contents a line at a time. A `Future` could represent the value of a web form after the user presses “Enter,” and a `Stream` can encapsulate the changing data as the user types.

Replace `main` with the following:

```dart
void main() async {
  var stream = Stream<int>.periodic(const Duration(seconds: 1), (i) => i);
  stream.forEach(print);
}
```

Here a `periodic` `Stream` is created that emits an increasing integer every second. Using `forEach`, it prints the next value as it’s available.

If you run this, the console updates each second with a new number.

### Creating Streams

You can create a `Stream` by repeatedly sending data from some source such as a file or a network server or by transforming an existing `Stream`.

Going back to the city example, add the following:

```dart
Stream<String> loadCityStream() async* {
  for(final city in fetchCityList()) {
    yield city;
  }
}
```

This has a few new factors occurring:

- The return type is now a `Stream`, which means this returns a `Stream` providing `String` values over time.
- This has the *async** annotation. It will not only return immediately like an `async` function does, but it will return a series of values and wrap them as a `Stream`.
- The body iterates over the city list and uses `yield` to send each value to the stream as the `for` loop iterates.

To see how it’s used, replace `main` once again:

```dart
void main() async {
  await for (final city in loadCityStream()) {
    print(city);
  }
}
```

This gets the stream from `loadCityStream` and uses `await for` to iterate over each value as it’s yielded. In the integer example, you used `forEach`. On a `Stream`, that is equivalent to `await for`. It’s the same relation as a list’s `forEach` is to a regular `for` operation.

When you run this code, the output will appear too quickly. Change `loadCityStream` so you see each value appear one at a time:

```dart
Stream<String> loadCityStream() async* {
  for(final city in fetchCityList()) {
    await Future.delayed(Duration(milliseconds: 500));
    yield city;
  }
}
```

Note that because the function is already `async*`, you don’t have to update the signature to accommodate the `await`.

Re-running the program will print the cities one at a time at a noticeable pace.

A common method to generate a `Stream` is by transforming an existing stream. Several list and iterator methods are also on `Stream`. You can use them to change a `Stream` by skipping, filtering and mapping values. You might do this by taking network bytes, deserializing them into objects and then extracting one field from that object. For example, if you had an address book you might load all the entries from a local database, collect the first and last names of each participant and then format them to get a single name `Stream`.

Try out this example. First add a population helper method:

```dart
int calculatePopulationOf(String city) {
  final populations = {'Tokyo': 37274000, 'Delhi': 32065760, 
'Shanghai': 28516904, 'São Paulo': 22429800, 'Mexico City': 22085140, 
'Cairo': 21750020, 'Beijing': 21333332, 'Mumbai': 20961472, 
'Kolkāta': 15133888, 'Manila': 14406059, 'Guangzhou': 13964274, 
'Moscow': 12640818, 'Jakarta': 11074811, 'Bangkok': 10899698, 
'Seoul': 9975709, 'London': 9540576, 'New York': 8177025};
  return populations[city] ?? -1;
}
```

That method returns population numbers for a city.

Now update `main`:

```dart
void main() async {
  print("CITY: POPULATION");    
  loadCityStream()
    .map((city) => "$city: " + calculatePopulationOf(city).toString())
    .forEach(print);
  loadCityStream()
    .map(calculatePopulationOf)
    .reduce((value, element) => value + element)
    .then((total) => print("Total known population: $total"));
}
```

This uses a city loading stream in two ways. The first uses `map` to construct an output string with the name and population. The second use chains a `map` to turn the city names into population numbers and a `reduce` to sum them up. Finally, it’s capped with a `then`, which takes the final value and prints it.

### Subscription and Broadcast Streams

In the previous example, you used two calls to `loadCityStream`, which looks odd. Try reusing the stream by modifying `main`:

```dart
void main() async {
  final stream = loadCityStream();
  print("CITY: POPULATION");    
  stream
    .map((city) => "$city: " + calculatePopulationOf(city).toString())
    .forEach(print);
  stream
    .map(calculatePopulationOf)
    .reduce((value, element) => value + element)
    .then((total) => print("Total known population: $total"));
}
```

When you run this, you’ll see the following in the console and the total won’t be computed:

Uncaught Error: Bad state: Stream has already been listened to.

The error is because `loadCityStream` is a *single subscription stream*. It can only be listened to once and has a finite start and end. In this example, the stream starts when the listener `map` is attached and continues to the last city.

In contrast, a *broadcast* stream can have many listeners and those listeners receive events for as long as they are attached. Creating a broadcast stream can be simple. In `main`, change the first line:

final stream = loadCityStream().asBroadcastStream();

This converts the subscription stream to a broadcast one. The list will print and the total calculated.

### Listening to Streams

The `Stream` transform functions like `map` listen to the source stream and return a new `Stream`. Any function that receives events from a `is a listener.`

With all the convenient methods, including `then`, you'll rarely have to write your own listener. You'll have finer control over what happens when data loads and you'll perform actions on events — such as when the `Stream` ends.

For example, add to `main`:

```dart
stream
  .listen(
    (city) => print("loaded `$city`"),
    onDone: () => print("all cities loaded")
);
```

The first callback occurs when data is ready, and `onDone` is called when the stream is complete. In this case, an all-done message is printed when it's complete.

One other reason to use `listen` is that it returns a `StreamSubscription` object, which gives the option to pause or cancel the stream.

Replace what you just added with:

```dart
final subscription = stream.listen(
  (city) => print("loaded `$city`"),
  onDone: () => print("all cities loaded")
);

await Future.delayed(Duration(seconds: 2));
subscription.pause();
await Future.delayed(Duration(seconds: 2));
subscription.resume();
await Future.delayed(Duration(seconds: 2));
subscription.cancel();
```

The change stores the subscription object so it can be paused, resumed and canceled after a short delay. It's worth looking at the console output to understand what the result was for performing these operations:

```
CITY: POPULATION
Bangkok: 10899698
loaded `Bangkok`
Beijing: 21333332
loaded `Beijing`
Cairo: 21750020
loaded `Cairo`
Delhi: 32065760
Guangzhou: 13964274
Jakarta: 11074811
Kolkāta: 15133888
loaded `Delhi`
loaded `Guangzhou`
loaded `Jakarta`
loaded `Kolkāta`
Manila: 14406059
loaded `Manila`
Mexico City: 22085140
loaded `Mexico City`
Moscow: 12640818
loaded `Moscow`
Mumbai: 20961472
loaded `Mumbai`
New York: 8177025
São Paulo: 22429800
Seoul: 9975709
Shanghai: 28516904
Tokyo: 37274000
Total known population: 302688710
```

Because city names appear at regular intervals, you can inspect the output to learn what's happening with the `Stream` and with each of the listeners.

An alternating printout shows the population and the "loaded" message of each city. After 2 seconds (when Cairo was loaded), the listener displaying the "loaded" message was paused. For a while, only the population messages displayed.

After another 2 seconds, the "loaded message" listener resumed. Because it's listening to a broadcast stream, all the missed events were buffered and sent at once, resulting in a few loaded messages being displayed before Manila comes along and both listeners are printing again.

Two seconds later, the subscription cancels and the loaded messages stop altogether.

In the case where you have a single subscriber stream, pausing the subscription stops generating events, and resuming will restart the event generator where it left off.

To try that out, replace `main` with:

```dart
void main() async {
  final stream = loadCityStream();
  final subscription = stream.listen(
    (city) => print("loaded `$city`"),
    onDone: () => print("all cities loaded")
  );
  await Future.delayed(Duration(seconds: 2));
  subscription.pause();
  await Future.delayed(Duration(seconds: 2));
  subscription.resume();
  await Future.delayed(Duration(seconds: 2));
  subscription.cancel();
}
```

The resulting console output will be:

```
loaded `Bangkok`
loaded `Beijing`
loaded `Cairo`
loaded `Delhi`
loaded `Guangzhou`
loaded `Jakarta`
loaded `Kolkāta`
```

No intermediate events occurred when the `Stream` pauses. When it cancels after 4 seconds, it only loads as far as Kolkāta.

*Note*: Canceling the `Stream` means `onDone` will not get called. If cleanup is needed, it's critical that you do it in `onDone` as well as the optional completion block of `cancel`.

You don't have to use listeners with `Stream`s. Instead, you can use `await for` to iterate over a `Stream`s data or even `await` to wait until the `Stream`is done.

For example, replacing `main` with this simple use brings back the old `Future` behavior of waiting for all the data to load before continuing.

```dart
void main() async {
  final cities = await loadCityStream().toList();
  printCities(cities);
}
```

### Handling Stream Errors

Error handling with ``Stream is the same as with `Future`s. You can use `catchError` or a `try/catch` with `await for`.``

For example, replace `main` with:

```dart
Stream<String> failingStream() async* {
  throw LoginError();
}

void main() async {
  failingStream()
    .forEach(print)
    .catchError((e) => print("Stream failed: $e"));
}
```

This has a new helper, `failingStream` that throws an exception. This is caught and printed with the `catchError` method.

You can instead use `catch` with an `await for` in this example. Replace main with:

```dart
void main() async {
  try {
    await for (final city in failingStream()) {
      print(city);
    }
  } catch (e) {
    print("Caught error: $e");
  }
}
```

### Adding Timeouts

Another condition you might want to handle is timeouts. A `Stream` will wait for the next event as long as there is a listener subscribed. In the case where you might have an indefinitely long computation or waiting for a remote resource to respond, you might want to build in a timeout.

The `timeout` method takes a duration. As long as an event happens before the duration expires, the `Stream` continues. Otherwise, by default a `TimeoutException` is thrown. Like everything else, the timeout behavior can be determined with a callback.

Replace `main` again:

```dart
void main() async {
  print("Loading Cities:");
  loadCityStream()
    .timeout(Duration(milliseconds: 400))
    .forEach(print)
    .catchError((e) => print("Stream timed out, try again."), 
test: (e) => e is TimeoutException);
}
```

This adds a 400 millisecond timeout to the operation. Because each city is yielded at 500 milliseconds, it will time out right away and be caught by the `catchError` block and and an error will be printed.

## Where to Go From Here?

That's it for a quick introduction to `Futures` and `Streams` in Dart.

From here you can continue to explore more interesting ways to manipulate data in useful ways. Many of the methods in the API are useful for building custom streams and event handling; you're likely to never use them and only build on the higher order transforms.

The power of streams comes in handy for routing data and user events through Flutter widgets, like [`StreamBuilder`](https://api.flutter.dev/flutter/widgets/StreamBuilder-class.html). To learn more about asynchronous programming, there is a chapter in [The Dart Apprentice](https://www.raywenderlich.com/268540/announcing-new-book-dart-apprentice) that covers streams and more in deep detail.
