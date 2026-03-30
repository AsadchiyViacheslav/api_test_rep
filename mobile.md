LB1
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'LB-1',
      home: const MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Асадчий Вячеслав Александрович'),
        backgroundColor: Colors.green,
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            const Text('Мемчик:'),
            const SizedBox(height: 20),
            Image.asset('assets/images/meme.jpg'),
          ],
        ),
      ),
    );
  }
}
```

LB2
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(
      title: 'LB-1',
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Асадчий Вячеслав Александрович'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Text(
                'ФИО: Асадчий Вячеслав Александрович',
                textAlign: TextAlign.center,
                style: TextStyle(
                  fontSize: 18,
                  fontWeight: FontWeight.bold,
                  color: Colors.blue,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Text(
                'Год рождения: 2005',
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.black,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Text(
                'Группа: 423-1',
                style: TextStyle(
                  fontSize: 16,
                  color: Colors.black87,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(8.0),
              child: Text(
                'Любимый спорт: Баскетбол',
                style: TextStyle(
                  fontSize: 14,
                  fontStyle: FontStyle.italic,
                  color: Colors.green,
                ),
              ),
            ),
            Padding(
              padding: const EdgeInsets.all(12.0),
              child: Image.asset(
                'assets/images/photo.png',
                width: 150,
                height: 150,
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```

LB3
```dart
import 'package:flutter/material.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'LB-3',
      debugShowCheckedModeBanner: false,
      home: const InputPage(),
    );
  }
}

class InputPage extends StatefulWidget {
  const InputPage({super.key});

  @override
  State<InputPage> createState() => _InputPageState();
}

class _InputPageState extends State<InputPage> {
  final _formKey = GlobalKey<FormState>();

  final TextEditingController _aController = TextEditingController();
  final TextEditingController _bController = TextEditingController();

  bool _isAgree = false;

  @override
  void dispose() {
    _aController.dispose();
    _bController.dispose();
    super.dispose();
  }

  String? _validateNumber(String? value, String fieldName) {
    if (value == null || value.trim().isEmpty) {
      return 'Введите число $fieldName';
    }

    final normalized = value.replaceAll(',', '.');
    final number = double.tryParse(normalized);

    if (number == null) {
      return 'Введите корректное число';
    }

    return null;
  }

  void _goToResultPage() {
    final isFormValid = _formKey.currentState!.validate();

    if (!_isAgree) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(
          content: Text('Необходимо дать согласие на обработку данных'),
        ),
      );
      return;
    }

    if (!isFormValid) {
      return;
    }

    final double a = double.parse(_aController.text.replaceAll(',', '.'));
    final double b = double.parse(_bController.text.replaceAll(',', '.'));

    Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => ResultPage(a: a, b: b),
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Асадчий Вячеслав Александрович'),
        centerTitle: true,
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16),
        child: Form(
          key: _formKey,
          child: Column(
            children: [
              const SizedBox(height: 12),
              TextFormField(
                controller: _aController,
                keyboardType: const TextInputType.numberWithOptions(
                  decimal: true,
                  signed: true,
                ),
                decoration: const InputDecoration(
                  labelText: 'Число a',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.looks_one),
                ),
                validator: (value) => _validateNumber(value, 'a'),
              ),
              const SizedBox(height: 16),
              TextFormField(
                controller: _bController,
                keyboardType: const TextInputType.numberWithOptions(
                  decimal: true,
                  signed: true,
                ),
                decoration: const InputDecoration(
                  labelText: 'Число b',
                  border: OutlineInputBorder(),
                  prefixIcon: Icon(Icons.looks_two),
                ),
                validator: (value) => _validateNumber(value, 'b'),
              ),
              const SizedBox(height: 12),
              CheckboxListTile(
                value: _isAgree,
                onChanged: (value) {
                  setState(() {
                    _isAgree = value ?? false;
                  });
                },
                title: const Text('Согласен на обработку данных'),
                controlAffinity: ListTileControlAffinity.leading,
                contentPadding: EdgeInsets.zero,
              ),
              const SizedBox(height: 20),
              SizedBox(
                width: double.infinity,
                child: ElevatedButton(
                  onPressed: _goToResultPage,
                  child: const Text('Рассчитать'),
                ),
              ),
            ],
          ),
        ),
      ),
    );
  }
}

class ResultPage extends StatelessWidget {
  final double a;
  final double b;

  const ResultPage({
    super.key,
    required this.a,
    required this.b,
  });

  @override
  Widget build(BuildContext context) {
    final double sum = a + b;
    final double result = sum * sum;

    return Scaffold(
      appBar: AppBar(
        title: const Text('Результат расчёта'),
        centerTitle: true,
      ),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            Text('Число a: ${a.toStringAsFixed(2)}'),
            const SizedBox(height: 8),
            Text('Число b: ${b.toStringAsFixed(2)}'),
            const SizedBox(height: 16),
            Text(
              'Сумма чисел: ${sum.toStringAsFixed(2)}',
              style: const TextStyle(fontSize: 18),
            ),
            const SizedBox(height: 12),
            Text(
              'Квадрат суммы: ${result.toStringAsFixed(2)}',
              style: const TextStyle(
                fontSize: 20,
                fontWeight: FontWeight.bold,
                color: Colors.blue,
              ),
            ),
            const SizedBox(height: 12),
            const Text(
              'Формула: (a + b)²',
              style: TextStyle(
                fontSize: 16,
                fontStyle: FontStyle.italic,
              ),
            ),
            const Spacer(),
            SizedBox(
              width: double.infinity,
              child: OutlinedButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                child: const Text('Назад'),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
```