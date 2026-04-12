Архитектура:

* точку входа приложения
* provider, который создаёт cubit
* сам cubit с логикой состояний
* экран, который отображает UI
* state, который описывает состояние экрана

---

# Вопросы

## 1. Cubit

**Cubit** — это простой способ управления состоянием (state) приложения.

👉 Это упрощённая версия BLoC (без событий).

### Суть:

* хранит состояние (`state`)
* изменяет его через методы
* уведомляет UI об изменениях

### Пример:

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0); // начальное состояние

  void increment() {
    emit(state + 1);
  }
}
```

📌 Здесь:

* `state` — текущее значение
* `emit()` — обновляет состояние

---

## 2. Виджет BlocProvider

**BlocProvider** — это виджет, который **создаёт и предоставляет Cubit (или Bloc)** всему дереву виджетов.

### Зачем нужен:

Чтобы не передавать Cubit вручную через конструкторы

### Пример:

```dart
BlocProvider(
  create: (_) => CounterCubit(),
  child: MyScreen(),
)
```

📌 Что происходит:

* создаётся `CounterCubit`
* он доступен во всех дочерних виджетах

---

## 3. Виджет BlocBuilder

**BlocBuilder** — это виджет, который **перестраивает UI при изменении состояния**.

### Пример:

```dart
BlocBuilder<CounterCubit, int>(
  builder: (context, state) {
    return Text('$state');
  },
)
```

📌 Здесь:

* `state` — текущее состояние Cubit
* при `emit()` → UI автоматически обновляется

---

## 4. Класс Cubit

Это базовый класс из библиотеки `flutter_bloc`.

```dart
abstract class Cubit<State> extends BlocBase<State>
```

### Что он даёт:

* хранение состояния (`state`)
* поток изменений
* метод `emit()`

📌 Ты создаёшь свой Cubit так:

```dart
class MyCubit extends Cubit<MyState>
```

---

## 5. Вызов метода Cubit

Чтобы вызвать метод Cubit из UI, используется `context.read()` или `context.watch()`.

### Пример:

```dart
context.read<CounterCubit>().increment();
```

📌 Разница:

* `read` — просто вызвать метод (без подписки)
* `watch` — следить за изменениями

---

## 6. Функция emit

**emit()** — это главный механизм изменения состояния.

### Что делает:

* отправляет новое состояние
* триггерит обновление UI

### Пример:

```dart
emit(state + 1);
```

📌 Важно:

* нельзя менять state напрямую!
* только через `emit()`

---

## Как всё работает вместе (логика)

1. `BlocProvider` создаёт Cubit
2. UI вызывает метод Cubit
3. Cubit вызывает `emit()`
4. `BlocBuilder` ловит изменение
5. UI обновляется



# 1. `main.dart`

```dart
import 'package:flutter/material.dart'; 
// Подключаем библиотеку Flutter Material.
// В ней находятся основные Material-виджеты:
// runApp, StatelessWidget, MaterialApp, Scaffold, Text, Button и т.д.

import 'package:laboratory_work/screens/main_screen_provider.dart'; 
// Подключаем файл, в котором находится MainScreenProvider.
// Именно через него будет создаваться Cubit и передаваться экрану.

void main() {
  runApp(const AppRoot());
}

class AppRoot extends StatelessWidget {
// Создаём корневой виджет приложения.
// Наследование от StatelessWidget означает,
// что сам AppRoot не хранит внутреннего изменяемого состояния.

  const AppRoot({super.key});
  // Конструктор виджета.
  // super.key передаёт ключ родительскому классу Widget.
  // Key нужен Flutter для правильного сравнения и обновления виджетов в дереве.

  @override
  Widget build(BuildContext context) {
    // build(...) — главный метод любого виджета.
    // Он должен вернуть другой виджет, который будет показан на экране.
    //
    // BuildContext — это специальный объект, который содержит информацию
    // о положении виджета в дереве Flutter.

    return const MaterialApp(
      // MaterialApp — это корневой виджет Material-приложения.
      // Он настраивает базовые вещи:
      // навигацию, тему, локализацию, направление текста и т.д.

      debugShowCheckedModeBanner: false,
      // Убираем красный баннер DEBUG в правом верхнем углу экрана.

      home: MainScreenProvider(),
      // home — главный экран приложения.
      // Здесь мы запускаем не сразу MainScreen,
      // а MainScreenProvider, потому что именно он создаёт Cubit
      // и передаёт его внутрь MainScreen.
    );
  }
}
```

---

# 2. `main_screen_provider.dart`

```dart
import 'package:flutter/material.dart';
// Подключаем базовые Material-виджеты Flutter.

import 'package:flutter_bloc/flutter_bloc.dart';
// Подключаем библиотеку flutter_bloc.
// В ней есть BlocProvider, BlocBuilder, context.read() и другие инструменты
// для работы с Cubit/BLoC.

import 'package:laboratory_work/screens/cubit/main_screen_cubit.dart';
// Подключаем класс MainScreenCubit.
// Это класс, который будет хранить состояние экрана и изменять его.

import 'package:laboratory_work/screens/main_screen.dart';
// Подключаем сам экран MainScreen,
// который будет отображать интерфейс приложения.

class MainScreenProvider extends StatelessWidget {
// Создаём отдельный виджет-провайдер.
// Его задача — не рисовать сложный интерфейс,
// а создать Cubit и передать его экрану.

  const MainScreenProvider({super.key});
  // Конструктор виджета.
  // super.key передаёт ключ базовому классу.

  @override
  Widget build(BuildContext context) {
    // build(...) возвращает виджет,
    // который будет находиться в дереве приложения.

    return BlocProvider(
      // BlocProvider — специальный виджет из flutter_bloc.
      // Он создаёт Cubit/BLoC и делает его доступным
      // всем дочерним виджетам ниже по дереву.

      create: (_) => MainScreenCubit(),
      // create — функция, которая создаёт экземпляр Cubit.
      //
      // (_) означает, что BuildContext нам здесь не нужен,
      // поэтому параметр мы не используем.
      //
      // MainScreenCubit() — создаём объект кубита.
      // Именно он будет хранить текущее состояние экрана
      // и менять его через emit(...).

      child: const MainScreen(),
      // child — дочерний виджет, которому будет доступен этот Cubit.
      // Здесь это главный экран приложения MainScreen.
      //
      // Теперь внутри MainScreen можно обращаться к кубиту так:
      // context.read<MainScreenCubit>()
      // или слушать его через BlocBuilder.
    );
  }
}
```

---

# 3. `main_screen_cubit.dart`

```dart
import 'package:flutter_bloc/flutter_bloc.dart';
// Подключаем библиотеку flutter_bloc.
// В ней находится базовый класс Cubit.

import 'package:laboratory_work/screens/cubit/main_screen_state.dart';
// Подключаем классы состояний экрана:
// MainScreenState, MainScreenInputState, MainScreenResultState.

class MainScreenCubit extends Cubit<MainScreenState> {
// Создаём свой cubit.
// Он наследуется от Cubit<MainScreenState>,
// то есть этот cubit будет хранить состояние типа MainScreenState
// и его наследников.

  MainScreenCubit() : super(const MainScreenInputState(isAgree: false));
  // Конструктор cubit.
  //
  // После двоеточия идёт вызов конструктора родительского класса Cubit.
  // super(...) задаёт начальное состояние cubit.
  //
  // Здесь начальное состояние — MainScreenInputState(isAgree: false),
  // то есть при запуске приложения мы показываем форму ввода,
  // а чекбокс по умолчанию не отмечен.

  void setAgreement(bool value) {
    // Метод для изменения состояния согласия на обработку данных.
    // value — новое значение чекбокса: true или false.

    final current = state;
    // Сохраняем текущее состояние cubit в локальную переменную current.
    // state — встроенное свойство Cubit, которое хранит текущее состояние.

    if (current is MainScreenInputState) {
      // Проверяем: если текущее состояние — экран ввода данных.

      emit(MainScreenInputState(isAgree: value));
      // emit(...) отправляет новое состояние.
      //
      // Здесь мы создаём новое состояние ввода,
      // но уже с новым значением isAgree.
      //
      // То есть экран останется экраном ввода,
      // но чекбокс обновится.

      return;
      // Завершаем выполнение метода,
      // потому что нужное состояние уже отправлено.
    }

    if (current is MainScreenResultState) {
      // Проверяем: если текущее состояние — экран результата.

      emit(
        MainScreenResultState(
          isAgree: value,
          // Обновляем только флаг согласия.

          a: current.a,
          // Сохраняем старое значение a из текущего состояния.

          b: current.b,
          // Сохраняем старое значение b.

          sum: current.sum,
          // Сохраняем уже посчитанную сумму.

          squaredSum: current.squaredSum,
          // Сохраняем уже посчитанный квадрат суммы.
        ),
      );
      // Здесь мы пересоздаём состояние результата,
      // но меняем только isAgree, а остальные данные не теряем.
    }
  }

  void calculate(double a, double b) {
    // Метод для вычисления результата.
    // Получает два числа: a и b.

    final sum = a + b;
    // Считаем сумму чисел.

    final squaredSum = sum * sum;
    // Считаем квадрат суммы.

    emit(
      MainScreenResultState(
        isAgree: state.isAgree,
        // Берём текущее значение согласия из состояния cubit.
        // Это позволяет не потерять флаг чекбокса при переходе к результату.

        a: a,
        // Сохраняем введённое значение a в состояние результата.

        b: b,
        // Сохраняем введённое значение b.

        sum: sum,
        // Сохраняем вычисленную сумму.

        squaredSum: squaredSum,
        // Сохраняем вычисленный квадрат суммы.
      ),
    );
    // Отправляем новое состояние — теперь экран должен перейти
    // из режима ввода в режим отображения результата.
  }

  void resetToInput() {
    // Метод для возврата обратно к экрану ввода.

    emit(MainScreenInputState(isAgree: state.isAgree));
    // Отправляем состояние ввода.
    // При этом сохраняем текущее значение isAgree,
    // чтобы чекбокс не сбрасывался без необходимости.
  }
}
```

---

# 4. `main_screen.dart`

Разберу тоже подробно.

```dart
import 'package:flutter/material.dart';
// Подключаем Material-виджеты Flutter:
// Scaffold, AppBar, TextFormField, CheckboxListTile, ElevatedButton и т.д.

import 'package:flutter_bloc/flutter_bloc.dart';
// Подключаем flutter_bloc,
// чтобы использовать BlocBuilder и context.read().

import 'package:laboratory_work/screens/cubit/main_screen_cubit.dart';
// Подключаем MainScreenCubit,
// чтобы вызывать его методы из UI.

import 'package:laboratory_work/screens/cubit/main_screen_state.dart';
// Подключаем классы состояний,
// чтобы проверять, какое сейчас состояние экрана.

class MainScreen extends StatefulWidget {
// MainScreen делаем StatefulWidget,
// потому что внутри экрана есть объекты,
// жизненным циклом которых нужно управлять вручную:
// FormKey и TextEditingController.

  const MainScreen({super.key});
  // Конструктор виджета.

  @override
  State<MainScreen> createState() => _MainScreenState();
  // createState() создаёт объект состояния для StatefulWidget.
  // Вся логика, связанная с жизненным циклом этого экрана,
  // будет находиться в _MainScreenState.
}

class _MainScreenState extends State<MainScreen> {
// Класс состояния для MainScreen.
// Здесь хранятся контроллеры, ключ формы и методы обработки UI.

  final _formKey = GlobalKey<FormState>();
  // GlobalKey<FormState> — ключ для формы.
  // С его помощью можно получить доступ к состоянию Form:
  // например, вызвать validate() и проверить все поля.

  final _aController = TextEditingController();
  // Контроллер для поля ввода числа a.
  // Позволяет читать и изменять текст в TextFormField.

  final _bController = TextEditingController();
  // Контроллер для поля ввода числа b.

  @override
  void dispose() {
    // dispose() вызывается, когда виджет удаляется из дерева.
    // Здесь нужно освобождать ресурсы.

    _aController.dispose();
    // Освобождаем ресурсы контроллера поля a.

    _bController.dispose();
    // Освобождаем ресурсы контроллера поля b.

    super.dispose();
    // Вызываем dispose() родительского класса.
    // Это обязательная хорошая практика.
  }

  String? _validateNumber(String? value, String fieldName) {
    // Метод валидации текстового поля.
    // Возвращает:
    // - строку с ошибкой, если введено неверное значение
    // - null, если всё корректно

    if (value == null || value.trim().isEmpty) {
      // Если значение null или пустое после удаления пробелов,
      // значит пользователь ничего не ввёл.

      return 'Введите число $fieldName';
      // Возвращаем текст ошибки.
      // Form покажет этот текст под полем.
    }

    final normalized = value.replaceAll(',', '.');
    // Заменяем запятые на точки.
    // Это удобно, потому что пользователь может ввести 12,5,
    // а double.parse обычно ожидает формат 12.5.

    final number = double.tryParse(normalized);
    // Пробуем преобразовать строку в число типа double.
    // tryParse не выбрасывает ошибку,
    // а просто возвращает null, если строка некорректна.

    if (number == null) {
      // Если преобразование не удалось,
      // значит введено не число.

      return 'Введите корректное число';
      // Возвращаем сообщение об ошибке.
    }

    return null;
    // Если ошибок нет, возвращаем null.
    // Это означает, что поле валидно.
  }

  void _onCalculatePressed() {
    // Метод, который вызывается при нажатии кнопки "Рассчитать".

    final cubit = context.read<MainScreenCubit>();
    // Получаем экземпляр MainScreenCubit из контекста.
    // read() используется, когда нужно просто вызвать метод cubit,
    // без подписки на изменения состояния.

    final isFormValid = _formKey.currentState?.validate() ?? false;
    // Вызываем validate() у формы.
    // Это запускает validator у всех TextFormField внутри Form.
    //
    // Если currentState == null, то подстраховываемся и получаем false.

    if (!cubit.state.isAgree) {
      // Проверяем, установил ли пользователь галочку согласия.
      // Если isAgree == false, значит согласие не дано.

      ScaffoldMessenger.of(context).showSnackBar(
        // Показываем временное всплывающее сообщение снизу экрана.

        const SnackBar(
          content: Text('Необходимо дать согласие на обработку данных'),
          // Текст сообщения внутри SnackBar.
        ),
      );

      return;
      // Прерываем дальнейшее выполнение метода,
      // потому что без согласия расчёт выполнять нельзя.
    }

    if (!isFormValid) {
      // Если форма невалидна,
      // например, поля пустые или введён неверный формат числа.

      return;
      // Просто выходим из метода.
      // Ошибки уже будут показаны под полями.
    }

    final a = double.parse(_aController.text.replaceAll(',', '.'));
    // Берём текст из первого поля,
    // заменяем запятые на точки
    // и преобразуем строку в число.

    final b = double.parse(_bController.text.replaceAll(',', '.'));
    // Аналогично читаем и преобразуем значение второго поля.

    cubit.calculate(a, b);
    // Вызываем метод calculate(...) у cubit.
    // Cubit выполнит вычисления,
    // сформирует состояние результата
    // и через emit(...) сообщит UI о смене состояния.
  }

  @override
  Widget build(BuildContext context) {
    // build(...) возвращает всё содержимое экрана.

    return Scaffold(
      // Scaffold — базовый каркас страницы Material Design.
      // Обычно содержит appBar, body, floatingActionButton и т.д.

      appBar: AppBar(
        // Верхняя панель приложения.

        title: const Text('Лабораторная работа 4'),
        // Заголовок в AppBar.

        centerTitle: true,
        // Размещаем заголовок по центру.
      ),

      body: BlocBuilder<MainScreenCubit, MainScreenState>(
        // BlocBuilder слушает изменения состояния MainScreenCubit.
        // Когда cubit делает emit(newState),
        // builder(...) вызывается заново и UI перестраивается.

        builder: (context, state) {
          // state — текущее состояние cubit.
          // Здесь мы решаем, что показать на экране
          // в зависимости от типа состояния.

          if (state is MainScreenResultState) {
            // Если состояние результата,
            // значит нужно показывать блок с вычислениями.

            return _ResultBody(
              // Возвращаем отдельный виджет для отображения результата.

              resultState: state,
              // Передаём туда текущее состояние результата,
              // в котором уже лежат a, b, sum, squaredSum.

              onNewCalculation: () {
                // Передаём callback,
                // который будет вызван по кнопке "Новый расчет".

                context.read<MainScreenCubit>().resetToInput();
                // Возвращаем cubit в состояние ввода.
                // После emit(...) BlocBuilder перестроится
                // и снова покажет форму.
              },
            );
          }

          return SingleChildScrollView(
            // Если состояние не результат,
            // значит показываем форму ввода.
            //
            // SingleChildScrollView позволяет экрану прокручиваться,
            // если контента станет больше или откроется клавиатура.

            padding: const EdgeInsets.all(16),
            // Внутренние отступы по 16 пикселей со всех сторон.

            child: Form(
              // Form объединяет несколько полей ввода
              // и позволяет валидировать их вместе.

              key: _formKey,
              // Передаём ключ формы,
              // чтобы потом вызвать validate().

              child: Column(
                // Column размещает дочерние элементы вертикально.

                children: [
                  TextFormField(
                    // Поле ввода числа a с поддержкой валидации.

                    controller: _aController,
                    // Контроллер, через который можно читать введённый текст.

                    keyboardType: const TextInputType.numberWithOptions(
                      decimal: true,
                      // Разрешаем десятичные числа.

                      signed: true,
                      // Разрешаем ввод отрицательных чисел.
                    ),

                    decoration: const InputDecoration(
                      labelText: 'Число a',
                      // Подпись поля.

                      border: OutlineInputBorder(),
                      // Рамка вокруг поля ввода.
                    ),

                    validator: (value) => _validateNumber(value, 'a'),
                    // Функция валидации поля.
                    // Проверяет, что значение введено и является числом.
                  ),

                  const SizedBox(height: 16),
                  // Вертикальный отступ между полями.

                  TextFormField(
                    // Второе поле ввода — для числа b.

                    controller: _bController,
                    // Контроллер второго поля.

                    keyboardType: const TextInputType.numberWithOptions(
                      decimal: true,
                      signed: true,
                    ),
                    // Настройка клавиатуры:
                    // разрешаем десятичные и отрицательные числа.

                    decoration: const InputDecoration(
                      labelText: 'Число b',
                      border: OutlineInputBorder(),
                    ),
                    // Оформление второго поля.

                    validator: (value) => _validateNumber(value, 'b'),
                    // Валидация второго поля.
                  ),

                  const SizedBox(height: 12),
                  // Отступ перед чекбоксом.

                  CheckboxListTile(
                    // Виджет чекбокса с текстом.

                    value: state.isAgree,
                    // Текущее значение чекбокса берём из состояния cubit.
                    // Это важно: UI не хранит этот флаг локально,
                    // а читает из единого источника истины — состояния.

                    onChanged: (value) {
                      // Вызывается при изменении чекбокса пользователем.

                      context
                          .read<MainScreenCubit>()
                          .setAgreement(value ?? false);
                      // Передаём новое значение в cubit.
                      // Если value == null, подставляем false.
                    },

                    title: const Text('Согласен на обработку данных'),
                    // Текст рядом с чекбоксом.

                    controlAffinity: ListTileControlAffinity.leading,
                    // Показываем сам чекбокс слева от текста.

                    contentPadding: EdgeInsets.zero,
                    // Убираем стандартные внутренние отступы.
                  ),

                  const SizedBox(height: 20),
                  // Отступ перед кнопкой.

                  SizedBox(
                    // SizedBox используется здесь,
                    // чтобы задать размер кнопки по ширине.

                    width: double.infinity,
                    // Делаем кнопку шириной на всю доступную ширину.

                    child: ElevatedButton(
                      // Кнопка Material Design.

                      onPressed: _onCalculatePressed,
                      // При нажатии вызывается метод обработки расчёта.

                      child: const Text('Рассчитать'),
                      // Текст на кнопке.
                    ),
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}
```

---

# 5. `_ResultBody`

```dart
class _ResultBody extends StatelessWidget {
// Отдельный приватный виджет для отображения результата.
// Нижнее подчёркивание в имени означает,
// что класс приватный внутри этого файла.

  final MainScreenResultState resultState;
  // Поле, в котором хранится состояние результата.
  // Сюда передаются a, b, sum и squaredSum.

  final VoidCallback onNewCalculation;
  // Поле для функции без параметров и без возвращаемого значения.
  // Этот callback будет вызван при нажатии кнопки "Новый расчет".

  const _ResultBody({
    required this.resultState,
    // required означает, что передать resultState обязательно.

    required this.onNewCalculation,
    // required означает, что callback тоже обязателен.
  });

  @override
  Widget build(BuildContext context) {
    // build(...) возвращает интерфейс блока результата.

    return Padding(
      padding: const EdgeInsets.all(16),
      // Добавляем внутренние отступы по 16 пикселей.

      child: Column(
        // Размещаем все элементы вертикально.

        crossAxisAlignment: CrossAxisAlignment.start,
        // Выравниваем элементы по левому краю.

        children: [
          Text('Число a: ${resultState.a.toStringAsFixed(2)}'),
          // Показываем число a.
          // toStringAsFixed(2) форматирует число до 2 знаков после запятой.

          const SizedBox(height: 8),
          // Небольшой вертикальный отступ.

          Text('Число b: ${resultState.b.toStringAsFixed(2)}'),
          // Показываем число b.

          const SizedBox(height: 16),
          // Отступ перед суммой.

          Text(
            'Сумма чисел: ${resultState.sum.toStringAsFixed(2)}',
            // Показываем сумму чисел.

            style: const TextStyle(fontSize: 18),
            // Делаем шрифт немного крупнее.
          ),

          const SizedBox(height: 12),
          // Отступ перед главным результатом.

          Text(
            'Квадрат суммы: ${resultState.squaredSum.toStringAsFixed(2)}',
            // Показываем квадрат суммы.

            style: const TextStyle(
              fontSize: 20,
              // Размер шрифта больше.

              fontWeight: FontWeight.bold,
              // Делаем текст жирным.

              color: Colors.blue,
              // Делаем текст синим.
            ),
          ),

          const Spacer(),
          // Spacer занимает всё оставшееся свободное место.
          // Благодаря этому кнопка "Новый расчет" прижимается вниз.

          SizedBox(
            width: double.infinity,
            // Кнопка будет на всю ширину.

            child: OutlinedButton(
              // Кнопка с контурным стилем.

              onPressed: onNewCalculation,
              // При нажатии вызывается переданный callback.

              child: const Text('Новый расчет'),
              // Текст кнопки.
            ),
          ),
        ],
      ),
    );
  }
}
```

