# Представление вещественных чисел

Существует два способа представления вещественных чисел: с фиксированным количеством разрядов (fixed-point) под дробную часть, и с переменным числом разрядов (floating-point).

Представление чисел с фиксированной точкой часто используется там, где требуется гарантированная точность до определенного разряда, например, в финансовой сфере.

Представление в формате с плавающей точкой является более универсальным, и все современные архитектуры процессоров работают именно в этом формате.


## Числа с плавающей точкой в формате IEE754

Два основных типа вещественных с плавающей точкой, которые определены стандартом языка Си, - это `float` (используется 4 байта для хранения) и `double` (используется 8 байт).

Самый старший бит в представлении числа - это признак отрицательного значения. Далее, по старшинству бит, хранится значения *смещенной экспоненциальной* части (8 бит для `float` или 11 бит для `double`), а затем - значение *мантиссы* (23 или 52 бит).

Смещение экспоненциальной части необходимо для того, чтобы можно было в таком представлении хранить значения с отрицательной экспонентой. Смещение для типа `float` равно `127`, для типа `double` - `1023`.

Таким образом, итоговое значение может быть получено как:

```
Value = (-1)^S * 2^(E-B) * ( 1 + M / (2^M_bits - 1) )
```
где `S` - бит знака, `E` - значение смещенной экспоненты, `B` - смещение (127 или 1023), а `M` - значение мантиссы, `M_bits` - количество бит в экспоненте.


## Как получить отдельные биты вещественного числа

Поразрядные операции относятся к целочисленной арифметике, и не предусмотрены для типов `float` и `double`. Таким образом, нужно сохранить вещественное число в памяти, и затем прочитать его, интерпретируя как целое число. В случае с языком C++ для этого предназначен оператор `reinterpret_cast`. Для языка Си есть два способа: использовать аналог `reinterpret_cast` - приведение указателей, либо использовать тип `union`.

### Приведение указателей
```
// У нас есть некоторое целое вещественное число, которое хранится в памяти
double a = 3.14159;

// Получаем указатель на это число
double* a_ptr_as_double = &a;

// Теряем информацию о типе, приведением его к типу void*
void* a_ptr_as_void = a_ptr_as_void;

// Указатель void* в языке Си можно присваивать любому указателю
uint64_t* a_ptr_as_uint = a_ptr_as_void;

// Ну а дальше просто разыменовываем указатель
uint64_t b = *a_as_uint;
```

### Использование типа `union`

Тип `union` - это тип данных, который синтаксически очень похож на тип `struct`, то есть там можно перечислить несколько именованных полей, но концептуально - это совершенно разные типы данных! Если в структуре или классе, для хранения каждого поля для предусмотрено отдельное место в памяти, то для `union` этого не происходит, и все поля накладываются друг на друга при размещении в памяти.

Обычно тип `union` используется в качестве вариантного типа данных (в С++ начиная с 17-го стандарта для этого предусмотрен `std::variant`), но в качестве побочного эффекта - его удобно использовать приведения типов в стиле `reinterpret_cast`, не используя при этом указатели.

```
// У нас есть некоторое целое вещественное число, которое хранится в памяти
double a = 3.14159;

// Используем тип union
typedef union {
    double     real_value;
    uint64_t   uint_value;
} real_or_uint;

real_or_uint u;
u.real_value = a;
uint64_t b = u.uint_value;
```

## Специальные значения в формате IEEE754

 * Бесконечность: `E=0xFF...FF`, `M=0`
 * Минус ноль (результат деления 1 на минус бесконечность): `S=1`, `E=0`, `M=0`
 * NaN (сигнальный): `S=0`, `E=0xFF...FF`, `M <> 0`
 * NaN (тихий): `S=0`, `E=0xFF...FF`, `M <> 0`

Некоторые процессоры, например архитектуры x86, поддерживают расширение стандарта, позволяющее более эффективно представлять множество чисел, значения которых близко к нулю. Такие числа называются *денормализованными*.

Признаком денормализованного числа является значение смещенной экспоненты `E=0`. В этом случае, численное значение получается следующим образом:

```
Value = (-1)^S * ( M / (2^M_bits - 1) )
```
