# Практическая работа №13; Батов Даниил ЭФБО-10-23
## Приложение «Geo & Sensors App»

---



## ИСПОЛЬЗУЕМЫЕ ПАКЕТЫ

Для реализации функционала приложения были использованы следующие пакеты из репозитория pub.dev:

### Геолокация
- **geolocator: ^12.0.0** — пакет для определения местоположения устройства. Предоставляет API для получения координат, проверки разрешений и статуса служб геолокации.

### Геокодинг
- **geocoding: ^3.0.0** — пакет для преобразования координат в адреса (прямой геокодинг) и адресов в координаты (обратный геокодинг). Используется для получения адреса по координатам.

### Датчики движения
- **sensors_plus: ^7.0.0** — пакет для работы с акселерометром и гироскопом устройства. Предоставляет потоки данных (streams) для получения значений ускорения и угловой скорости в реальном времени.

### Компас
- **flutter_compass: ^0.7.0** — пакет для получения данных компаса устройства. Позволяет определить направление на север и азимут.

### Разрешения
- **permission_handler: ^11.3.0** — пакет для управления разрешениями приложения. Используется для запроса разрешений на доступ к геолокации.

### Базовые зависимости
- **flutter** — SDK Flutter для разработки кроссплатформенных приложений
- **material** — библиотека Material Design для создания пользовательского интерфейса

---

## ОСНОВНЫЕ ФРАГМЕНТЫ КОДА

### Структура приложения

Приложение состоит из одного основного экрана `GeoSensorsPage`, который является StatefulWidget и управляет состоянием всех данных.

```dart
class GeoSensorsPage extends StatefulWidget {
  const GeoSensorsPage({super.key});
  @override
  State<GeoSensorsPage> createState() => _GeoSensorsPageState();
}
```

### Переменные состояния

В классе `_GeoSensorsPageState` определены переменные для хранения данных:

```dart
Position? _position;                    // Текущая позиция
String _address = '–';                  // Адрес по координатам
double? _compass;                       // Направление компаса
List<double>? _accelerometerValues;     // Значения акселерометра [X, Y, Z]
List<double>? _gyroscopeValues;         // Значения гироскопа [X, Y, Z]
```

### Получение местоположения

Метод `_getLocation()` реализует получение координат и адреса:

```dart
Future<void> _getLocation() async {
  // Проверка доступности службы геолокации
  bool serviceEnabled = await Geolocator.isLocationServiceEnabled();
  if (!serviceEnabled) return;

  // Запрос разрешения
  LocationPermission permission = await Geolocator.requestPermission();
  if (permission == LocationPermission.denied) return;

  // Получение текущей позиции
  final pos = await Geolocator.getCurrentPosition(
    desiredAccuracy: LocationAccuracy.high,
  );

  // Преобразование координат в адрес
  List<Placemark> placemarks =
      await placemarkFromCoordinates(pos.latitude, pos.longitude);

  setState(() {
    _position = pos;
    _address = '${placemarks.first.locality}, ${placemarks.first.street ?? ''}';
  });
}
```

### Инициализация датчиков

В методе `initState()` настраиваются подписки на потоки данных датчиков:

```dart
@override
void initState() {
  super.initState();

  // Подписка на события акселерометра
  accelerometerEvents.listen((event) {
    setState(() => _accelerometerValues = [event.x, event.y, event.z]);
  });

  // Подписка на события гироскопа
  gyroscopeEvents.listen((event) {
    setState(() => _gyroscopeValues = [event.x, event.y, event.z]);
  });

  // Подписка на события компаса
  if (FlutterCompass.events != null) {
    FlutterCompass.events!.listen((event) {
      setState(() => _compass = event.heading);
    });
  }
}
```

### Пользовательский интерфейс

Интерфейс приложения реализован с использованием Material Design:

```dart
@override
Widget build(BuildContext context) {
  return Scaffold(
    appBar: AppBar(title: const Text('Geo & Sensors Demo')),
    body: Padding(
      padding: const EdgeInsets.all(16),
      child: ListView(
        children: [
          // Кнопка для получения местоположения
          ElevatedButton(
            onPressed: _getLocation,
            child: const Text('Определить местоположение'),
          ),
          // Отображение координат и адреса
          Text('Координаты: ${_position?.latitude}, ${_position?.longitude}'),
          Text('Адрес: $_address'),
          // Отображение данных компаса
          Text('Компас: ${_compass?.toStringAsFixed(2) ?? '–'}°'),
          // Отображение данных акселерометра и гироскопа
          Text('Акселерометр: ${_accelerometerValues?.map((e) => e.toStringAsFixed(2)).join(', ') ?? '–'}'),
          Text('Гироскоп: ${_gyroscopeValues?.map((e) => e.toStringAsFixed(2)).join(', ') ?? '–'}'),
        ],
      ),
    ),
  );
}
```

### Настройка разрешений

Для Android в файле `AndroidManifest.xml` добавлены разрешения:

```xml
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
```

Для iOS в файле `Info.plist` добавлено описание:

```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Приложению необходимо ваше местоположение</string>
```

---

## РЕЗУЛЬТАТЫ И ВЫВОДЫ

### Результаты работы приложения
<img width="576" height="1280" alt="image" src="https://github.com/user-attachments/assets/f3a21183-5164-42af-a5f6-d4c242b98d32" />

<img width="576" height="1280" alt="image" src="https://github.com/user-attachments/assets/3521ae34-eb05-4f54-8118-eafd6711cfbe" />


