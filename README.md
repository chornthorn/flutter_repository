<!-- 
This README describes the package. If you publish this package to pub.dev,
this README's contents appear on the landing page for your package.

For information about how to write a good package README, see the guide for
[writing package pages](https://dart.dev/guides/libraries/writing-package-pages). 

For general information about developing packages, see the Dart guide for
[creating packages](https://dart.dev/guides/libraries/create-library-packages)
and the Flutter guide for
[developing packages and plugins](https://flutter.dev/developing-packages). 
-->

## RepositoryProvider

RepositoryProvider is a Flutter widget which provides a repository to its children via
RepositoryProvider.of<T>(context). It is used as a dependency injection (DI) widget so that a single
instance of a repository can be provided to multiple widgets within a subtree. RepositoryProvider should only be used for repositories.

```dart
  RepositoryProvider(
    create: (context) => RepositoryA(),
    child: ChildA(),
  );
```
then from ChildA we can retrieve the Repository instance with:

```dart
  // with extensions
  context.read<RepositoryA>();

  // without extensions
  RepositoryProvider.of<RepositoryA>(context)
```
## MultiRepositoryProvider

MultiRepositoryProvider is a Flutter widget that merges multiple RepositoryProvider widgets into one. MultiRepositoryProvider improves the readability and eliminates the need to nest multiple RepositoryProvider. By using MultiRepositoryProvider we can go from:

```dart
  RepositoryProvider<RepositoryA>(
    create: (context) => RepositoryA(),
    child: RepositoryProvider<RepositoryB>(
      create: (context) => RepositoryB(),
      child: RepositoryProvider<RepositoryC>(
        create: (context) => RepositoryC(),
        child: ChildA(),
      )
    )
  )
```
  
to:

```dart
  MultiRepositoryProvider(
    providers: [
      RepositoryProvider<RepositoryA>(
        create: (context) => RepositoryA(),
      ),
      RepositoryProvider<RepositoryB>(
        create: (context) => RepositoryB(),
      ),
      RepositoryProvider<RepositoryC>(
        create: (context) => RepositoryC(),
      ),
    ],
    child: ChildA(),
  )
```

## RepositoryProvider Usage

We are going to take a look at how to use RepositoryProvider within the context of the flutter_weather example.
  
### weather_repository.dart
  
``` dart
    class WeatherRepository {
    WeatherRepository({
      MetaWeatherApiClient? weatherApiClient
    }) : _weatherApiClient = weatherApiClient ?? MetaWeatherApiClient();

    final MetaWeatherApiClient _weatherApiClient;

    Future<Weather> getWeather(String city) async {
      final location = await _weatherApiClient.locationSearch(city);
      final woeid = location.woeid;
      final weather = await _weatherApiClient.getWeather(woeid);
      return Weather(
        temperature: weather.theTemp,
        location: location.title,
        condition: weather.weatherStateAbbr.toCondition,
      );
    }
  }
```
Since the app has an explicit dependency on the WeatherRepository we inject an instance via constructor. This allows us to inject different instances of WeatherRepository based on the build flavor or environment.

### main.dart

``` dart
  import 'package:flutter/material.dart';
  import 'package:flutter_weather/app.dart';
  import 'package:weather_repository/weather_repository.dart';

  void main() {
    runApp(WeatherApp(weatherRepository: WeatherRepository()));
  }
```
  
### app.dart

``` dart
  import 'package:flutter/material.dart';
  import 'package:flutter_repository/flutter_repository.dart';
  import 'package:weather_repository/weather_repository.dart';

  class WeatherApp extends StatelessWidget {
    const WeatherApp({Key? key, required WeatherRepository weatherRepository})
        : _weatherRepository = weatherRepository,
          super(key: key);

    final WeatherRepository _weatherRepository;

    @override
    Widget build(BuildContext context) {
      return RepositoryProvider.value(
        value: _weatherRepository,
        child: WeatherAppView(),
      );
    }
  }
```
  
Now we can access the instance of a repository via context.read and inject the repository.
  
## Credit
  All code and document they are come from  `` flutter_bloc `` package.
