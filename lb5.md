# SQflite и SharedPreferences + разбор кода

## 1) Что такое и взаимодействие с SQflite

`SQflite` — это Flutter-плагин для работы с локальной SQLite-базой данных на устройстве.

Что это дает:
- хранение структурированных данных в таблицах (строки/колонки);
- SQL-запросы: `INSERT`, `SELECT`, `UPDATE`, `DELETE`;
- удобство для истории, каталогов, офлайн-данных, где важны фильтры/сортировка.

Как обычно идет взаимодействие:
1. Подключаете пакет `sqflite` (и обычно `path` для пути БД).
2. Открываете БД через `openDatabase(...)`.
3. В `onCreate` создаете таблицы.
4. Выполняете CRUD-операции SQL-запросами.
5. Преобразуете результаты (`Map<String, dynamic>`) в модели Dart.

Коротко: `SQflite` выбирают, когда данных много, нужна сложная выборка, связи и надежная структура хранения.

## 2) Что такое и взаимодействие с SharedPreferences

`SharedPreferences` — это простое локальное key-value хранилище (пары `ключ -> значение`).

Что удобно хранить:
- настройки приложения;
- флаги (`isFirstLaunch`), тему;
- небольшие списки/строки/числа.

Как обычно идет взаимодействие:
1. Получаете экземпляр: `SharedPreferences.getInstance()`.
2. Сохраняете значения: `setString`, `setInt`, `setBool`, `setStringList`.
3. Читаете значения: `getString`, `getInt`, `getBool`, `getStringList`.
4. Для сложных объектов обычно сериализуют в JSON-строку.

В этом проекте история расчетов хранится именно через `SharedPreferences`: каждый `CalculationRecord` преобразуется в JSON-строку и кладется в список строк.

---

## Файл: `lib/data/models/calculation_record.dart`

```dart
class CalculationRecord { // Объявляем модель одной записи вычисления.
  final double a; // Первое введенное число.
  final double b; // Второе введенное число.
  final double sum; // Результат суммы a + b.
  final double squaredSum; // Результат (a + b)^2.
  final DateTime createdAt; // Дата и время создания записи.

  const CalculationRecord({ // Конструктор модели, принимает все поля как обязательные.
    required this.a, // Требует передать a при создании объекта.
    required this.b, // Требует передать b при создании объекта.
    required this.sum, // Требует передать сумму.
    required this.squaredSum, // Требует передать квадрат суммы.
    required this.createdAt, // Требует передать время создания.
  }); // Завершение конструктора.

  Map<String, dynamic> toJson() { // Метод сериализует объект модели в Map для JSON.
    return { // Возвращаем map с полями объекта.
      'a': a, // Сохраняем значение a под ключом 'a'.
      'b': b, // Сохраняем значение b под ключом 'b'.
      'sum': sum, // Сохраняем сумму под ключом 'sum'.
      'squaredSum': squaredSum, // Сохраняем квадрат суммы под ключом 'squaredSum'.
      'createdAt': createdAt.toIso8601String(), // Преобразуем дату в строку ISO 8601 для безопасного хранения.
    }; // Конец map.
  } // Конец метода toJson.

  factory CalculationRecord.fromJson(Map<String, dynamic> json) { // Фабрика для создания объекта из JSON-map.
    return CalculationRecord( // Создаем и возвращаем новый объект CalculationRecord.
      a: (json['a'] as num?)?.toDouble() ?? 0, // Читаем a как число, приводим к double, при null ставим 0.
      b: (json['b'] as num?)?.toDouble() ?? 0, // Читаем b как число, приводим к double, при null ставим 0.
      sum: (json['sum'] as num?)?.toDouble() ?? 0, // Читаем sum как число, при ошибке/отсутствии ставим 0.
      squaredSum: (json['squaredSum'] as num?)?.toDouble() ?? 0, // Читаем squaredSum как число, при null ставим 0.
      createdAt: DateTime.tryParse(json['createdAt'] as String? ?? '') ?? // Пытаемся распарсить дату из строки.
          DateTime.fromMillisecondsSinceEpoch(0), // Если парсинг не удался, ставим дату-минимум (эпоха).
    ); // Конец создания объекта.
  } // Конец фабрики fromJson.
} // Конец класса модели.
```

## Файл: `lib/data/storage/calculation_history_storage.dart`

```dart
import 'dart:convert'; // Подключаем JSON-кодирование/декодирование.

import 'package:laboratory_work/data/models/calculation_record.dart'; // Импорт модели CalculationRecord.
import 'package:shared_preferences/shared_preferences.dart'; // Импорт API SharedPreferences.

class CalculationHistoryStorage { // Класс-слой хранения истории расчетов.
  static const _recordsKey = 'calculation_records'; // Ключ, под которым записи лежат в SharedPreferences.

  Future<void> saveRecord(CalculationRecord record) async { // Метод сохранения одной новой записи.
    final preferences = await SharedPreferences.getInstance(); // Получаем экземпляр локального key-value хранилища.
    final records = await loadRecords(); // Загружаем текущую историю из памяти.
    final updated = [record, ...records]; // Добавляем новую запись в начало списка (самая свежая сверху).

    final encoded = updated // Начинаем формировать список строк для сохранения.
        .map((item) => jsonEncode(item.toJson())) // Каждый объект превращаем в JSON-строку.
        .toList(growable: false); // Собираем в неизменяемый по размеру список.

    await preferences.setStringList(_recordsKey, encoded); // Сохраняем список JSON-строк под ключом _recordsKey.
  } // Конец метода saveRecord.

  Future<List<CalculationRecord>> loadRecords() async { // Метод чтения всех записей истории.
    final preferences = await SharedPreferences.getInstance(); // Получаем экземпляр хранилища.
    final encoded = preferences.getStringList(_recordsKey) ?? <String>[]; // Читаем список строк; если пусто, возвращаем пустой список.

    return encoded // Берем все сохраненные JSON-строки.
        .map(_decodeRecord) // Пытаемся декодировать каждую строку в CalculationRecord?.
        .whereType<CalculationRecord>() // Оставляем только успешно декодированные (не null) записи.
        .toList(growable: false); // Собираем финальный список моделей.
  } // Конец метода loadRecords.

  CalculationRecord? _decodeRecord(String source) { // Вспомогательный метод безопасного декодирования одной строки.
    try { // Начало блока с обработкой возможной ошибки JSON.
      final json = jsonDecode(source) as Map<String, dynamic>; // Парсим JSON-строку в map.
      return CalculationRecord.fromJson(json); // Создаем модель из map и возвращаем ее.
    } catch (_) { // Если строка повреждена/невалидна, ловим исключение.
      return null; // Возвращаем null, чтобы позже отфильтровать невалидную запись.
    } // Конец try/catch.
  } // Конец метода _decodeRecord.
} // Конец класса CalculationHistoryStorage.
```

## Файл: `lib/screens/history/history_screen.dart`

```dart
import 'package:flutter/material.dart'; // Базовые виджеты Flutter Material.
import 'package:flutter_bloc/flutter_bloc.dart'; // Инструменты BLoC (BlocBuilder и др.).
import 'package:laboratory_work/screens/history/cubit/history_cubit.dart'; // Импорт Cubit для истории.
import 'package:laboratory_work/screens/history/cubit/history_state.dart'; // Импорт состояний экрана истории.

class HistoryScreen extends StatelessWidget { // Экран истории как неизменяемый StatelessWidget.
  const HistoryScreen({super.key}); // Конструктор виджета с опциональным key.

  @override // Переопределяем метод построения UI.
  Widget build(BuildContext context) { // Метод возвращает дерево виджетов экрана.
    return Scaffold( // Каркас экрана: AppBar + body.
      appBar: AppBar(title: const Text('История расчетов')), // Верхняя панель с заголовком.
      body: BlocBuilder<HistoryCubit, HistoryState>( // Подписываемся на изменения состояния HistoryCubit.
        builder: (context, state) { // Функция перестраивает UI при каждом новом состоянии.
          if (state is! HistoryLoadedState) { // Пока данные не загружены.
            return const Center(child: CircularProgressIndicator()); // Показываем индикатор загрузки.
          } // Конец ветки загрузки.

          if (state.records.isEmpty) { // Если загрузка завершена, но записей нет.
            return const Center( // Центрируем текст по экрану.
              child: Text('История пуста'), // Сообщение о пустой истории.
            ); // Конец Center.
          } // Конец проверки пустого списка.

          return ListView.separated( // Строим список карточек истории с разделителями.
            padding: const EdgeInsets.all(12), // Внутренние отступы списка.
            itemCount: state.records.length, // Количество элементов равно количеству записей.
            separatorBuilder: (_, __) => const SizedBox(height: 8), // Вертикальный отступ между элементами.
            itemBuilder: (context, index) { // Строим каждый элемент списка по индексу.
              final record = state.records[index]; // Берем соответствующую запись.

              return Card( // Оборачиваем запись в Material-карточку.
                child: ListTile( // Используем ListTile для компактной структуры title/subtitle.
                  title: Text( // Основной заголовок карточки.
                    'a = ${record.a.toStringAsFixed(2)}, b = ${record.b.toStringAsFixed(2)}', // Показываем входные числа с 2 знаками после запятой.
                  ), // Конец title.
                  subtitle: Text( // Дополнительный текст карточки.
                    'Сумма: ${record.sum.toStringAsFixed(2)}\n' // Строка с суммой.
                    'Квадрат суммы: ${record.squaredSum.toStringAsFixed(2)}\n' // Строка с квадратом суммы.
                    'Дата: ${_formatDate(record.createdAt)}', // Строка с отформатированной датой.
                  ), // Конец subtitle.
                ), // Конец ListTile.
              ); // Конец Card.
            }, // Конец itemBuilder.
          ); // Конец ListView.separated.
        }, // Конец builder.
      ), // Конец BlocBuilder.
    ); // Конец Scaffold.
  } // Конец build.

  String _formatDate(DateTime date) { // Вспомогательный метод форматирования даты в строку.
    final day = date.day.toString().padLeft(2, '0'); // День, дополненный слева нулем до 2 символов.
    final month = date.month.toString().padLeft(2, '0'); // Месяц в формате 01..12.
    final year = date.year.toString(); // Год как строка.
    final hour = date.hour.toString().padLeft(2, '0'); // Часы в формате 00..23.
    final minute = date.minute.toString().padLeft(2, '0'); // Минуты с ведущим нулем.
    final second = date.second.toString().padLeft(2, '0'); // Секунды с ведущим нулем.
    return '$day.$month.$year $hour:$minute:$second'; // Собираем итоговую строку даты/времени.
  } // Конец _formatDate.
} // Конец класса HistoryScreen.
```

## Файл: `lib/screens/history/history_screen_provider.dart`

```dart
import 'package:flutter/material.dart'; // Импорт базовых Flutter-виджетов.
import 'package:flutter_bloc/flutter_bloc.dart'; // Импорт BlocProvider.
import 'package:laboratory_work/screens/history/cubit/history_cubit.dart'; // Импорт кубита истории.
import 'package:laboratory_work/screens/history/history_screen.dart'; // Импорт самого UI-экрана истории.

class HistoryScreenProvider extends StatelessWidget { // Обертка, которая предоставляет Cubit экрану.
  const HistoryScreenProvider({super.key}); // Конструктор виджета-провайдера.

  @override // Переопределяем метод build.
  Widget build(BuildContext context) { // Строим дерево виджетов.
    return BlocProvider( // Создаем провайдер состояния для дочерних виджетов.
      create: (_) => HistoryCubit()..loadHistory(), // Создаем кубит и сразу запускаем загрузку истории.
      child: const HistoryScreen(), // Передаем экран истории как дочерний виджет.
    ); // Конец BlocProvider.
  } // Конец build.
} // Конец класса HistoryScreenProvider.
```

## Файл: `lib/screens/history/cubit/history_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart'; // Импорт базового класса Cubit.
import 'package:laboratory_work/data/storage/calculation_history_storage.dart'; // Импорт слоя хранения истории.
import 'package:laboratory_work/screens/history/cubit/history_state.dart'; // Импорт состояний для Cubit.

class HistoryCubit extends Cubit<HistoryState> { // Cubit управляет состоянием экрана истории.
  final CalculationHistoryStorage _storage; // Зависимость для чтения данных из локального хранилища.

  HistoryCubit({CalculationHistoryStorage? storage}) // Конструктор, позволяет подменить storage (удобно для тестов).
      : _storage = storage ?? CalculationHistoryStorage(), // Если storage не передан, создаем реальный.
        super(const HistoryInitialState()); // Начальное состояние: история еще не загружена.

  Future<void> loadHistory() async { // Публичный метод загрузки истории.
    final records = await _storage.loadRecords(); // Асинхронно читаем записи из storage.
    emit(HistoryLoadedState(records: records)); // Публикуем новое состояние с загруженными данными.
  } // Конец метода loadHistory.
} // Конец класса HistoryCubit.
```

## Файл: `lib/screens/history/cubit/history_state.dart`

```dart
import 'package:laboratory_work/data/models/calculation_record.dart'; // Импорт модели записи расчета.

abstract class HistoryState { // Базовый абстрактный тип всех состояний истории.
  const HistoryState(); // Константный конструктор базового состояния.
} // Конец абстрактного класса.

class HistoryInitialState extends HistoryState { // Состояние до загрузки данных.
  const HistoryInitialState(); // Константный конструктор initial-состояния.
} // Конец класса HistoryInitialState.

class HistoryLoadedState extends HistoryState { // Состояние, когда данные успешно загружены.
  final List<CalculationRecord> records; // Список записей истории для отображения на экране.

  const HistoryLoadedState({required this.records}); // Конструктор, требующий передать список записей.
} // Конец класса HistoryLoadedState.
```
