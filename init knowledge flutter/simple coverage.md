# Simple coverage

# Unit Testing: Bloc Library & Codemagic

[Unit Testing: Bloc Library & Codemagic | Codemagic Blog](https://blog.codemagic.io/flutter-unit-testing-bloc-with-codemagic/)

Nov 18, 2019

---

> Written by Felix Angelov

v2.0.0 of the [bloc library](https://bloclibrary.dev/) introduced some major testing improvements, making it easier than ever to unit test blocs. Today, we’ll learn to fully test blocs as well as how to run the tests as part of the Codemagic pipeline using the new `codemagic.yaml`. Let’s get started! 🎉

---

For this tutorial, we’re going to be building a `WeatherBloc` which will be interacting with a `WeatherRepository` to retrieve weather information for a given city.

## Weather Repository

For simplicity, the `WeatherRepository` will just return mock weather data but you can refer to the [bloc library weather tutorial](https://bloclibrary.dev/#/flutterweathertutorial) for a complete example of how to hook it up to a real API and fetch real weather data.

```dart
// weather_repository.dart
import 'dart:async';
import 'package:meta/meta.dart';
import 'models/models.dart';

class WeatherRepository {
  Future<Weather> getWeather({@required String city}) async {
    await Future.delayed(Duration(seconds: 1));
    return Weather(
      temperature: 30,
      condition: Condition.sunny,
    );
  }
}
```

The `Weather` model implementation is also simplified in order to focus on unit testing and simply contains a `temperature` (in celsius) and a `condition`.

```dart
// weather.dart
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

enum Condition { sunny, rainy, cloudy }

class Weather extends Equatable {
  final double temperature;
  final Condition condition;

  const Weather({
    @required this.temperature,
    @required this.condition,
  });

  @override
  List<Object> get props => [temperature, condition];
}
```

Now that we’ve defined the `WeatherRepository`, we can move on to the `WeatherBloc` implementation.

## Weather Bloc

The `WeatherBloc` will be responsible for taking `WeatherEvents` as input and emitting `WeatherStates` as output. Be sure to refer to the bloc library [core concepts](https://bloclibrary.dev/#/coreconcepts) if you haven’t already.

Let’s start by defining the `WeatherEvents`!

### Events

In this example, the `WeatherBloc` will only need to react to a single event: `WeatherRequested`. The `WeatherRequested` event will contain the city for which the weather was requested.

```dart
// weather_event.dart
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';

abstract class WeatherEvent extends Equatable {
  const WeatherEvent();
}

class WeatherRequested extends WeatherEvent {
  final String city;

  const WeatherRequested({@required this.city});

  @override
  List<Object> get props => [city];
}
```

### States

The `WeatherBloc` will have four different states that it can emit:

- `WeatherInitial` - the initial state of the bloc before any events are added.
- `WeatherLoadInProgress` - the state of the bloc immediately after a `WeatherRequested` event is added while the weather is being retrieved asynchronously.
- `WeatherLoadSuccess` - the state of the bloc after weather has been successfully retrieved.
- `WeatherLoadFailure` - the state of the bloc after weather has been unsuccessfully retrieved.

```dart
import 'package:equatable/equatable.dart';
import 'package:meta/meta.dart';
import 'package:weather_repository/weather_repository.dart';

abstract class WeatherState extends Equatable {
  const WeatherState();

  @override
  List<Object> get props => [];
}

class WeatherInitial extends WeatherState {}

class WeatherLoadInProgress extends WeatherState {}

class WeatherLoadSuccess extends WeatherState {
  final Weather weather;

  const WeatherLoadSuccess({@required this.weather});

  @override
  List<Object> get props => [weather];
}

class WeatherLoadFailure extends WeatherState {}
```

## Bloc

Now that the `WeatherEvents` and `WeatherStates` have been defined, we can implement the `WeatherBloc`.

```dart
import 'dart:async';
import 'package:bloc/bloc.dart';
import 'package:meta/meta.dart';
import 'package:weather_repository/weather_repository.dart';
import './bloc.dart';

class WeatherBloc extends Bloc<WeatherEvent, WeatherState> {
  final WeatherRepository _weatherRepository;

  WeatherBloc({@required WeatherRepository weatherRepository})
      : assert(weatherRepository != null),
        _weatherRepository = weatherRepository;

  @override
  WeatherState get initialState => WeatherInitial();

  @override
  Stream<WeatherState> mapEventToState(
    WeatherEvent event,
  ) async* {
      // TODO: Add Logic
  }
}
```

We start by defining the `WeatherRepository` as a dependency of the `WeatherBloc`. We also set the `initialState` of the bloc to be `WeatherInitial` and now all that’s left is to implement `mapEventToState`.

```dart
@override
Stream<WeatherState> mapEventToState(
  WeatherEvent event,
) async* {
  if (event is WeatherRequested) {
    yield* _mapWeatherRequestedToState(event);
  }
}

Stream<WeatherState> _mapWeatherRequestedToState(
  WeatherRequested event,
) async* {
  yield WeatherLoadInProgress();
  try {
    final weather = await _weatherRepository.getWeather(
      city: event.city,
    );
    yield WeatherLoadSuccess(weather: weather);
  } catch (_) {
    yield WeatherLoadFailure();
  }
}
```

Awesome! At this point we have a fully functional weather bloc which we can hook up to our UI using the [flutter_bloc package](https://pub.dev/packages/flutter_bloc). For this article we’re just going to focus on unit testing the newly created `WeatherBloc`.

## Unit Testing with “bloc_test”

Finally, we’re going to learn how to unit test the `WeatherBloc` in order to ensure that it functions as we expect and to give us confidence that over time, changes to the code don’t break the `WeatherBloc` functionality.

We’ll start by setting up the test file to have an over-arching group for all `WeatherBloc` tests.

```dart
// weather_bloc_test.dart
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('WeatherBloc', () {
      // TODO: Write Tests
  });
}
```

The very first test we’ll write is just to ensure that the bloc will throw an `AssertionError` if a `WeatherRepository` is not provided.

```dart
// weather_bloc_test.dart
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('WeatherBloc', () {
    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });
  });
}
```

We can run the test via `flutter test` or directly in `VSCode` or `Android Studio` and ensure that it passes.

Next, we’ll move on to the fun part: bloc functionality tests.

Generally, it’s good to create a group per event with tests to ensure that the bloc reacts to the event in every scenario. In this case, there is only a single event so we’ll just create one group within the `WeatherBloc` group called `WeatherRequested`.

```dart
// weather_bloc_test.dart
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';

void main() {
  group('WeatherBloc', () {
    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      // TODO: Add Tests
    });
  });
}
```

Next, we’ll import [mockito](https://pub.dev/packages/mockito) and define a `MockWeatherBloc` which will allow us simulate every possible outcome.

```dart
// weather_bloc_test.dart
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_repository/weather_repository.dart';

class MockWeatherRepository extends Mock implements WeatherRepository {}

void main() {
  group('WeatherBloc', () {
    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      // TODO: Add Tests
    });
  });
}
```

We can now define and initialize a `WeatherBloc` with a `MockWeatherRepository` in the `WeatherBloc` group’s `setUp`. `setUp` will get run before each test in the group to ensure that every test starts off in the exact same state.

```dart
// weather_bloc_test.dart
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_repository/weather_repository.dart';

class MockWeatherRepository extends Mock implements WeatherRepository {}

void main() {
  group('WeatherBloc', () {
    WeatherRepository weatherRepository;
    WeatherBloc weatherBloc;

    setUp(() {
      weatherRepository = MockWeatherRepository();
      weatherBloc = WeatherBloc(weatherRepository: weatherRepository);
    });

    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      // TODO: Add Tests
    });
  });
}
```

Next, we’ll import the [bloc_test package](https://pub.dev/packages/bloc_test) which will help us test blocs in an efficient and structured manner.

```dart
// weather_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_repository/weather_repository.dart';

class MockWeatherRepository extends Mock implements WeatherRepository {}

void main() {
  group('WeatherBloc', () {
    WeatherRepository weatherRepository;
    WeatherBloc weatherBloc;

    setUp(() {
      weatherRepository = MockWeatherRepository();
      weatherBloc = WeatherBloc(weatherRepository: weatherRepository);
    });

    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      blocTest(
        'emits [WeatherInitial, WeatherLoadInProgress, WeatherLoadSuccess] when WeatherRequested is added and getWeather succeeds',
        build: () {},
        act: (_) {},
        expect: [],
      );
    });
  });
}
```

### blocTest

`blocTest` creates a new bloc-specific test case with the given description. `blocTest` will handle asserting that the `bloc` emits the expected states (in order) after `act` is executed. `blocTest` also handles ensuring that no additional states are emitted by closing the `bloc` stream before evaluating the expectation.

- `build` should be used for all `bloc` initialization and preparation and must return the `bloc` under test.

- `act` is an optional callback which will be invoked with the `bloc` under test and should be used to `add` events to the `bloc`.

- `expect` is an `Iterable<State>` which the `bloc` under test is expected to emit after `act` is executed.

Now that we’ve defined what `blocTest` does and how to use it, let’s finish the first test.

```dart
// weather_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_repository/weather_repository.dart';

class MockWeatherRepository extends Mock implements WeatherRepository {}

void main() {
  group('WeatherBloc', () {
    WeatherRepository weatherRepository;
    WeatherBloc weatherBloc;

    setUp(() {
      weatherRepository = MockWeatherRepository();
      weatherBloc = WeatherBloc(weatherRepository: weatherRepository);
    });

    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      blocTest(
        'emits [WeatherInitial, WeatherLoadInProgress, WeatherLoadSuccess] when WeatherRequested is added and getWeather succeeds',
        build: () {
          when(weatherRepository.getWeather(city: 'Chicago')).thenAnswer(
            (_) => Future.value(
              Weather(
                temperature: 10,
                condition: Condition.cloudy,
              ),
            ),
          );
          return weatherBloc;
        },
        act: (bloc) => bloc.add(WeatherRequested(city: 'Chicago')),
        expect: [
          WeatherInitial(),
          WeatherLoadInProgress(),
          WeatherLoadSuccess(
            weather: Weather(
              temperature: 10,
              condition: Condition.cloudy,
            ),
          )
        ],
      );
    });
  });
}
```

Notice that in the `build` we stub the `getWeather` method on the `weatherRepository` and we return the `weatherBloc`. Then in `act` we just add the `WeatherRequested` event to the `weatherBloc`. Lastly, in `expect` we define the list of states that we expect to be emitted by the `weatherBloc` after the `WeatherRequested` event is added.

At this point we can run the `blocTest` just like any other `test` directly in `VSCode` or `Android Studio` and we should see it pass. You can play around with the test by removing or adding states to `expect` and observe the the test will fail instantly with a detailed error.

All that’s left is to add one more test to verify that the bloc handles errors from the `WeatherRepository` properly.

```dart
// weather_bloc_test.dart
import 'package:bloc_test/bloc_test.dart';
import 'package:codemagic_bloc_unit_tests/bloc/bloc.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';
import 'package:weather_repository/weather_repository.dart';

class MockWeatherRepository extends Mock implements WeatherRepository {}

void main() {
  group('WeatherBloc', () {
    WeatherRepository weatherRepository;
    WeatherBloc weatherBloc;

    setUp(() {
      weatherRepository = MockWeatherRepository();
      weatherBloc = WeatherBloc(weatherRepository: weatherRepository);
    });

    test('throws AssertionError if WeatherRepository is null', () {
      expect(
        () => WeatherBloc(weatherRepository: null),
        throwsA(isAssertionError),
      );
    });

    group('WeatherRequested', () {
      blocTest(
        'emits [WeatherInitial, WeatherLoadInProgress, WeatherLoadSuccess] when WeatherRequested is added and getWeather succeeds',
        build: () {
          when(weatherRepository.getWeather(city: 'Chicago')).thenAnswer(
            (_) => Future.value(
              Weather(
                temperature: 10,
                condition: Condition.cloudy,
              ),
            ),
          );
          return weatherBloc;
        },
        act: (bloc) => bloc.add(WeatherRequested(city: 'Chicago')),
        expect: [
          WeatherInitial(),
          WeatherLoadInProgress(),
          WeatherLoadSuccess(
            weather: Weather(
              temperature: 10,
              condition: Condition.cloudy,
            ),
          )
        ],
      );

      blocTest(
        'emits [WeatherInitial, WeatherLoadInProgress, WeatherLoadFailure] when WeatherRequested is added and getWeather fails',
        build: () {
          when(weatherRepository.getWeather(city: 'Chicago')).thenThrow('oops');
          return weatherBloc;
        },
        act: (bloc) => bloc.add(WeatherRequested(city: 'Chicago')),
        expect: [
          WeatherInitial(),
          WeatherLoadInProgress(),
          WeatherLoadFailure(),
        ],
      );
    });
  });
}
```

We can run all the tests within `VSCode` or `Android Studio`:

![VS Code tests](https://blog.codemagic.io/vscode-tests_5160189701279040764_hub7ab1307710f682284ab47831f435cde_0_1280x1800_fit_linear_3.png)

and we can generate a coverage report using the Flutter command line tools:

```sh
flutter test --coverage && genhtml coverage/lcov.info --output=coverage
```

We can then open the coverage report in our browser and check the coverage report:

```sh
open coverage/index.html
```

![Coverage report](https://blog.codemagic.io/coverage-report_1325089907001973272_hu6fafaee08df3060728cd99936a46a369_0_1280x1800_fit_linear_3.png)

## Automating the tests using Codemagic

Now that we have the `WeatherBloc` fully tested, we will set up a Codemagic pipeline which will handle running the tests every time we push changes to the project using the new `codemagic.yaml` configuration.

### Codemagic YAML

The `codemagic.yaml` configuration file is a file we can create and commit to version control which will tell Codemagic how the project’s build should be configured. This can save lots of time when setting up new projects and is also very useful when working on larger teams because it will allow changes to the build process to be part of the code review process.

Let’s create a `codemagic.yaml` file at the root of our project!

```yaml
# codemagic.yaml
workflows:
  default-workflow:
    name: Default Workflow
    environment:
      flutter: stable
    cache:
      cache_paths:
        - $CM_BUILD_DIR/build
    triggering:
      events:
        - push
        - pull_request
      branch_patterns:
        - pattern: "*"
          include: true
          source: true
    scripts:
      - flutter packages pub get
      - flutter test --coverage
    artifacts:
      - coverage/lcov.info
```

That’s all there is to it! We’ve named our workflow, specified the Flutter environment to use, the branch pattern to match on, the scripts to run, and the artifacts to save.

If we commit these changes and push to master, we can go over to the Codemagic builds and see a new build get triggered.

At the end, the build should be passing and it should look something like this:

![Codemagic build](https://blog.codemagic.io/codemagic-build_11463185220700816564_hu3686bdb38d0d187436744e89a386efa9_0_1280x1800_fit_linear_3.png)

You can [learn more](https://docs.codemagic.io/getting-started/yaml/) about the `codemagic.yaml` and see the [complete source code](https://github.com/felangel/codemagic_bloc_unit_tests) for this project for more details!
