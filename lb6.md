# Работа с сетью и разбор кода Unsplash

## 1. Вложенные модели

Вложенные модели это модели данных, в которых один объект содержит другой объект или список объектов.

Пример: ответ API может содержать объект `photo`, внутри которого есть объект `urls`, а внутри него поля `regular`, `small`.

Как с ними работать:
1. Создать отдельные классы под каждую вложенность.
2. В `fromJson` сначала читать внутренний map, потом собирать дочернюю модель.
3. Для nullable-полей задавать безопасные значения по умолчанию.

В вашем коде есть упрощенный вариант: вместо отдельной модели `Urls` из JSON берутся нужные поля `urls['regular']` и `urls['small']` сразу в `UnsplashPhoto`.

## 2. Библиотека http

`http` это пакет Dart/Flutter для выполнения HTTP-запросов (`GET`, `POST`, и т.д.).

Что делает в проекте:
- отправляет запрос к `api.unsplash.com`;
- получает `Response` со статус-кодом и телом;
- используется вместе с `jsonDecode` для преобразования тела ответа в Dart-структуры.

Типичный поток:
1. Сформировать `Uri`.
2. Выполнить `await http.get(uri)`.
3. Проверить `statusCode`.
4. Распарсить `response.body`.

## 3. Виды HTTP запросов

Основные методы:
- `GET` - получить данные (без изменения на сервере).
- `POST` - создать ресурс или отправить данные.
- `PUT` - полностью обновить ресурс.
- `PATCH` - частично обновить ресурс.
- `DELETE` - удалить ресурс.

В этом проекте используется `GET`, потому что нужно только искать и читать фотографии из Unsplash API.

## 4. Что такое виджеты компоновки. Использование в задачах сети

Виджеты компоновки это виджеты, которые управляют расположением и размером дочерних элементов (`ListView`, `Padding`, `GridView`, `Center`, `SafeArea`).

В сетевых задачах они помогают:
- разместить поле поиска и кнопку в едином прокручиваемом контейнере (`ListView`);
- показывать состояния загрузки/ошибки/пустого результата (`Center`, `Padding`);
- отображать данные из API списком или сеткой (`ListView`, `GridView`).

В вашем `UnsplashScreen` это реализовано через:
- `ListView` (форма поиска и контент в одном прокручиваемом списке);
- `GridView.builder` (сетка загруженных изображений);
- переключение UI по состояниям Cubit.

## 5. Обработка ошибок

Обработка ошибок это перехват исключений и показ пользователю понятного состояния вместо падения приложения.

В проекте:
1. В API-слое проверяется `statusCode`; при ошибке выбрасывается `Exception`.
2. В Cubit запрос оборачивается в `try/catch`.
3. При ошибке эмитится `UnsplashErrorState` с текстом ошибки.
4. Экран показывает сообщение в UI.

Также есть отдельная защита от проблем картинки: `Image.network(..., errorBuilder: ...)`.

## 6. Методы создания моделей

На практике чаще всего применяются:
1. Обычный конструктор - создание вручную из кода.
2. `factory fromJson(...)` - создание из JSON.
3. `toJson()` - обратная сериализация в JSON.
4. `copyWith(...)` (необязательно) - копия модели с изменением отдельных полей.

В вашем проекте для `UnsplashPhoto` используется `factory fromJson`, который безопасно достает поля и формирует объект модели.

---

## Файл: `lib/data/requests/unsplash_api.dart`

```dart
import 'dart:convert'; // Импортируем функции для преобразования JSON-строк в Dart-объекты.

import 'package:flutter/foundation.dart'; // Импортируем debugPrint для логирования в консоль.
import 'package:http/http.dart' as http; // Подключаем пакет http и даем ему алиас http.
import 'package:laboratory_work/config/unsplash_keys.dart'; // Импортируем конфиг, где хранится ключ доступа Unsplash.
import 'package:laboratory_work/data/models/unsplash_photo.dart'; // Импортируем модель фото Unsplash.

class UnsplashApi { // Класс для работы с удаленным API Unsplash.
  Future<List<UnsplashPhoto>> searchPhotos(String query) async { // Асинхронный метод поиска фото по строке запроса.
    if (UnsplashKeys.accessKey == '') { // Проверяем, заполнен ли API-ключ.
      throw Exception('Укажите Access Key в lib/config/unsplash_keys.dart'); // Если ключ пустой, выбрасываем понятную ошибку.
    } // Закрываем условие проверки ключа.
    if (query.trim().isEmpty) { // Проверяем, что запрос не пустой после удаления пробелов.
      return <UnsplashPhoto>[]; // Для пустого запроса возвращаем пустой список без сетевого вызова.
    } // Закрываем условие пустого запроса.

    final uri = Uri.https( // Создаем корректный HTTPS-URI для запроса к API.
      'api.unsplash.com', // Хост сервиса Unsplash.
      '/search/photos', // Путь к endpoint поиска фотографий.
      { // Query-параметры URL.
        'client_id': UnsplashKeys.accessKey, // Передаем access key в параметре client_id.
        'query': query.trim(), // Передаем строку поиска (очищенную от лишних пробелов).
        'per_page': '30', // Ограничиваем размер страницы 30 результатами.
      }, // Завершаем карту query-параметров.
    ); // Завершаем создание URI.
    debugPrint('[UnsplashApi] GET $uri'); // Пишем в лог финальный адрес GET-запроса.

    final response = await http.get(uri).timeout(const Duration(seconds: 30)); // Выполняем GET-запрос и задаем таймаут 30 секунд.
    debugPrint('[UnsplashApi] statusCode=${response.statusCode}'); // Логируем HTTP-статус ответа.

    if (response.statusCode != 200) { // Проверяем, что сервер вернул успешный код 200.
      throw Exception('Ошибка API Unsplash: ${response.statusCode}'); // При любом другом коде выбрасываем ошибку.
    } // Закрываем проверку статуса.

    final json = jsonDecode(response.body) as Map<String, dynamic>; // Декодируем JSON-тело ответа в Map.
    final results = json['results'] as List<dynamic>? ?? <dynamic>[]; // Берем список результатов или пустой список при отсутствии поля.
    debugPrint('[UnsplashApi] results=${results.length}'); // Логируем количество найденных элементов.

    return results // Возвращаем преобразованный список моделей.
        .map((item) => UnsplashPhoto.fromJson(item as Map<String, dynamic>)) // Каждый элемент JSON превращаем в UnsplashPhoto.
        .where((photo) => photo.imageUrl.isNotEmpty) // Оставляем только записи, где есть непустая ссылка на изображение.
        .toList(growable: false); // Собираем окончательный список фиксированного размера.
  } // Завершаем метод searchPhotos.
} // Завершаем класс UnsplashApi.
```

## Файл: `lib/data/models/unsplash_photo.dart`

```dart
class UnsplashPhoto { // Модель одного фото из ответа Unsplash API.
  final String id; // Уникальный идентификатор фотографии.
  final String imageUrl; // URL изображения для показа в интерфейсе.

  const UnsplashPhoto({ // Константный конструктор модели.
    required this.id, // Требуем передать id при создании модели.
    required this.imageUrl, // Требуем передать ссылку изображения.
  }); // Завершаем конструктор.

  factory UnsplashPhoto.fromJson(Map<String, dynamic> json) { // Фабричный конструктор для создания модели из JSON.
    final urls = json['urls'] as Map<String, dynamic>? ?? {}; // Достаем вложенный объект urls или подставляем пустую map.
    final regular = urls['regular'] as String? ?? ''; // Пытаемся взять ссылку regular, иначе пустую строку.
    final small = urls['small'] as String? ?? ''; // Пытаемся взять ссылку small, иначе пустую строку.

    return UnsplashPhoto( // Возвращаем новый объект модели.
      id: json['id'] as String? ?? '', // Берем id из JSON или пустую строку, если поле отсутствует.
      imageUrl: regular.isNotEmpty ? regular : small, // Выбираем regular, а если он пустой, берем small.
    ); // Завершаем создание модели.
  } // Завершаем fromJson.
} // Завершаем класс UnsplashPhoto.
```

## Файл: `lib/screens/unsplash/unsplash_screen.dart`

```dart
import 'package:flutter/foundation.dart'; // Импорт debugPrint для логирования действий экрана.
import 'package:flutter/material.dart'; // Импорт material-виджетов Flutter.
import 'package:flutter_bloc/flutter_bloc.dart'; // Импорт BlocConsumer и context.read для работы с Cubit.
import 'package:laboratory_work/screens/unsplash/cubit/unsplash_cubit.dart'; // Импорт Cubit для загрузки фото.
import 'package:laboratory_work/screens/unsplash/cubit/unsplash_state.dart'; // Импорт набора состояний экрана Unsplash.

class UnsplashScreen extends StatefulWidget { // StatefulWidget, так как есть контроллер поля ввода и локальные методы.
  const UnsplashScreen({super.key}); // Конструктор экрана с опциональным key.

  @override // Переопределяем метод создания состояния для StatefulWidget.
  State<UnsplashScreen> createState() => _UnsplashScreenState(); // Возвращаем экземпляр приватного state-класса.
} // Завершаем класс виджета UnsplashScreen.

class _UnsplashScreenState extends State<UnsplashScreen> { // Класс состояния экрана Unsplash.
  final _queryController = TextEditingController(); // Контроллер для чтения и управления текстом в TextField.

  @override // Переопределяем dispose для корректной очистки ресурсов.
  void dispose() { // Метод вызывается перед уничтожением state.
    _queryController.dispose(); // Освобождаем ресурсы контроллера текстового поля.
    super.dispose(); // Вызываем dispose родительского класса.
  } // Завершаем dispose.

  void _search() { // Вспомогательный метод запуска поиска.
    final query = _queryController.text.trim(); // Берем текст из поля и убираем пробелы по краям.
    debugPrint('[UnsplashScreen] search query="$query"'); // Логируем, какой запрос отправляем.
    context.read<UnsplashCubit>().loadPhotos(query); // Вызываем у Cubit загрузку фотографий по запросу.
  } // Завершаем _search.

  @override // Переопределяем build для построения UI.
  Widget build(BuildContext context) { // Главный метод, возвращающий дерево виджетов.
    return Scaffold( // Базовый каркас экрана Material.
      appBar: AppBar(title: const Text('Unsplash Search')), // Верхняя панель с заголовком.
      body: SafeArea( // Защищаем контент от вырезов и системных областей.
        child: BlocConsumer<UnsplashCubit, UnsplashState>( // Слушаем изменения состояния и строим UI.
          listener: (_, state) { // Listener для побочных действий при смене состояния.
            debugPrint('[UnsplashScreen] state=${state.runtimeType}'); // Логируем тип текущего состояния.
          }, // Завершаем listener.
          builder: (context, state) { // Builder возвращает UI под конкретное состояние.
            return ListView( // Единый прокручиваемый контейнер для формы и результата.
              padding: const EdgeInsets.all(12), // Отступы контента от краев экрана.
              children: [ // Список элементов внутри ListView.
                TextField( // Поле ввода поискового запроса.
                  controller: _queryController, // Подключаем контроллер, чтобы читать текст.
                  textInputAction: TextInputAction.search, // На клавиатуре показываем действие "Поиск".
                  onSubmitted: (_) => _search(), // При нажатии Enter/Поиск запускаем _search.
                  decoration: const InputDecoration( // Оформление поля ввода.
                    hintText: 'Введите запрос', // Подсказка внутри поля.
                    border: OutlineInputBorder(), // Рисуем рамку вокруг поля.
                    isDense: true, // Делаем поле более компактным по высоте.
                  ), // Завершаем декорацию поля.
                ), // Завершаем TextField.
                const SizedBox(height: 8), // Отступ 8 пикселей между полем и кнопкой.
                ElevatedButton( // Кнопка запуска поиска.
                  onPressed: _search, // По нажатию вызываем метод _search.
                  child: const Text('Найти'), // Текст на кнопке.
                ), // Завершаем кнопку.
                const SizedBox(height: 12), // Отступ между формой поиска и блоком состояния.
                if (state is UnsplashInitialState) // Если еще не выполняли поиск.
                  const Padding( // Добавляем вертикальные отступы для текста-подсказки.
                    padding: EdgeInsets.symmetric(vertical: 40), // Отступы сверху и снизу.
                    child: Center( // Центрируем подсказку.
                      child: Text('Введите текст и нажмите "Найти"'), // Сообщение для пользователя.
                    ), // Завершаем Center.
                  ) // Завершаем блок initial.
                else if (state is UnsplashLoadingState) // Если идет загрузка.
                  const Padding( // Добавляем вертикальные отступы для индикатора.
                    padding: EdgeInsets.symmetric(vertical: 40), // Отступы сверху и снизу.
                    child: Center(child: CircularProgressIndicator()), // Показываем индикатор загрузки.
                  ) // Завершаем блок loading.
                else if (state is UnsplashErrorState) // Если произошла ошибка.
                  Padding( // Добавляем отступы для текста ошибки.
                    padding: const EdgeInsets.all(16), // Равномерные отступы 16 пикселей.
                    child: Text( // Текст ошибки.
                      state.message, // Сообщение берется из состояния ошибки.
                      textAlign: TextAlign.center, // Выравниваем текст по центру.
                    ), // Завершаем Text.
                  ) // Завершаем блок error.
                else if (state is UnsplashLoadedState && state.photos.isEmpty) // Если запрос успешен, но результатов нет.
                  const Padding( // Добавляем вертикальные отступы для сообщения.
                    padding: EdgeInsets.symmetric(vertical: 40), // Отступы сверху и снизу.
                    child: Center(child: Text('Ничего не найдено')), // Сообщение о пустом результате.
                  ) // Завершаем блок пустого результата.
                else if (state is UnsplashLoadedState) // Если фото успешно загружены.
                  GridView.builder( // Строим сетку изображений по данным из состояния.
                    shrinkWrap: true, // Включаем расчет высоты по содержимому для вложения в ListView.
                    physics: const NeverScrollableScrollPhysics(), // Отключаем внутренний скролл GridView.
                    itemCount: state.photos.length, // Количество карточек равно количеству фото.
                    gridDelegate: // Делегат описывает параметры сетки.
                        const SliverGridDelegateWithFixedCrossAxisCount( // Фиксированное число колонок.
                      crossAxisCount: 2, // Две колонки в сетке.
                      crossAxisSpacing: 8, // Горизонтальный промежуток между ячейками.
                      mainAxisSpacing: 8, // Вертикальный промежуток между ячейками.
                      childAspectRatio: 1, // Квадратные ячейки (отношение сторон 1:1).
                    ), // Завершаем делегат сетки.
                    itemBuilder: (context, index) { // Функция построения конкретной ячейки по индексу.
                      final photo = state.photos[index]; // Берем объект фото из списка по индексу.

                      return ClipRRect( // Обрезаем содержимое по скругленной рамке.
                        borderRadius: BorderRadius.circular(8), // Радиус скругления углов 8 пикселей.
                        child: Image.network( // Загружаем и отображаем изображение по URL.
                          photo.imageUrl, // URL картинки из модели.
                          fit: BoxFit.cover, // Масштабируем так, чтобы заполнить ячейку.
                          errorBuilder: (_, __, ___) => const ColoredBox( // Если картинка не загрузилась, рисуем fallback.
                            color: Color(0xFFE0E0E0), // Серый фон fallback-блока.
                            child: Center( // Центрируем иконку ошибки.
                              child: Icon(Icons.broken_image_outlined), // Иконка "битого" изображения.
                            ), // Завершаем Center fallback.
                          ), // Завершаем ColoredBox fallback.
                        ), // Завершаем Image.network.
                      ); // Завершаем ClipRRect.
                    }, // Завершаем itemBuilder.
                  ), // Завершаем GridView.builder.
              ], // Завершаем children у ListView.
            ); // Завершаем ListView.
          }, // Завершаем builder BlocConsumer.
        ), // Завершаем BlocConsumer.
      ), // Завершаем SafeArea.
    ); // Завершаем Scaffold.
  } // Завершаем build.
} // Завершаем класс _UnsplashScreenState.
```

## Файл: `lib/screens/unsplash/unsplash_screen_provider.dart`

```dart
import 'package:flutter/material.dart'; // Импорт базовых material-виджетов.
import 'package:flutter_bloc/flutter_bloc.dart'; // Импорт BlocProvider для внедрения Cubit в дерево.
import 'package:laboratory_work/screens/unsplash/cubit/unsplash_cubit.dart'; // Импорт UnsplashCubit.
import 'package:laboratory_work/screens/unsplash/unsplash_screen.dart'; // Импорт экрана Unsplash.

class UnsplashScreenProvider extends StatelessWidget { // Виджет-обертка, который предоставляет Cubit экрану.
  const UnsplashScreenProvider({super.key}); // Конструктор StatelessWidget.

  @override // Переопределяем метод build.
  Widget build(BuildContext context) { // Строим дерево виджетов.
    return BlocProvider( // Создаем провайдер состояния BLoC/Cubit.
      create: (_) => UnsplashCubit(), // Создаем экземпляр UnsplashCubit.
      child: const UnsplashScreen(), // Отдаем экран как дочерний виджет внутри провайдера.
    ); // Завершаем BlocProvider.
  } // Завершаем build.
} // Завершаем класс UnsplashScreenProvider.
```

## Файл: `lib/screens/unsplash/cubit/unsplash_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart'; // Импорт базового класса Cubit.
import 'package:flutter/foundation.dart'; // Импорт debugPrint для логов.
import 'package:laboratory_work/data/requests/unsplash_api.dart'; // Импорт API-класса для сетевых запросов.
import 'package:laboratory_work/screens/unsplash/cubit/unsplash_state.dart'; // Импорт состояний Unsplash-экрана.

class UnsplashCubit extends Cubit<UnsplashState> { // Cubit, управляющий состояниями поиска фото.
  final UnsplashApi _api; // Зависимость для обращения к Unsplash API.

  UnsplashCubit({UnsplashApi? api}) // Конструктор с возможностью передать API извне (удобно для тестов).
      : _api = api ?? UnsplashApi(), // Если api не передан, используем реальный UnsplashApi.
        super(const UnsplashInitialState()); // Начальное состояние экрана.

  Future<void> loadPhotos(String query) async { // Асинхронный метод загрузки фото по запросу.
    final normalizedQuery = query.trim(); // Нормализуем строку поиска, убирая лишние пробелы.
    debugPrint('[UnsplashCubit] loadPhotos query="$normalizedQuery"'); // Логируем запрос.

    if (normalizedQuery.isEmpty) { // Если после trim строка пустая.
      emit(const UnsplashLoadedState(photos: [])); // Отдаем пустой результат без вызова API.
      return; // Завершаем метод раньше.
    } // Завершаем проверку пустого запроса.

    emit(const UnsplashLoadingState()); // Переводим экран в состояние загрузки.

    try { // Начинаем блок безопасного выполнения запроса.
      final photos = await _api.searchPhotos(normalizedQuery); // Запрашиваем фото через API-слой.
      debugPrint('[UnsplashCubit] loaded photos=${photos.length}'); // Логируем число загруженных фото.
      emit(UnsplashLoadedState(photos: photos)); // Эмитим состояние с результатами.
    } catch (error) { // Ловим любые исключения во время запроса или парсинга.
      debugPrint('[UnsplashCubit] error=$error'); // Пишем ошибку в лог.
      emit(UnsplashErrorState(message: error.toString())); // Переводим экран в состояние ошибки с сообщением.
    } // Завершаем try/catch.
  } // Завершаем метод loadPhotos.
} // Завершаем класс UnsplashCubit.
```

## Файл: `lib/screens/unsplash/cubit/unsplash_state.dart`

```dart
import 'package:laboratory_work/data/models/unsplash_photo.dart'; // Импорт модели фото для loaded-состояния.

abstract class UnsplashState { // Базовый абстрактный класс для всех состояний экрана Unsplash.
  const UnsplashState(); // Константный конструктор базового состояния.
} // Завершаем базовый класс состояния.

class UnsplashInitialState extends UnsplashState { // Начальное состояние: пользователь еще не выполнял поиск.
  const UnsplashInitialState(); // Конструктор initial-состояния.
} // Завершаем класс UnsplashInitialState.

class UnsplashLoadingState extends UnsplashState { // Состояние активной загрузки данных из сети.
  const UnsplashLoadingState(); // Конструктор loading-состояния.
} // Завершаем класс UnsplashLoadingState.

class UnsplashLoadedState extends UnsplashState { // Состояние успешной загрузки фото.
  final List<UnsplashPhoto> photos; // Список полученных фотографий.

  const UnsplashLoadedState({required this.photos}); // Конструктор, требующий передать список фото.
} // Завершаем класс UnsplashLoadedState.

class UnsplashErrorState extends UnsplashState { // Состояние ошибки при запросе или обработке данных.
  final String message; // Текст ошибки для отображения пользователю.

  const UnsplashErrorState({required this.message}); // Конструктор состояния ошибки.
} // Завершаем класс UnsplashErrorState.
```
