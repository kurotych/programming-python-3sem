# 27. (Л) Індексація, slicing та broadcasting у NumPy

## Зміст лекції

1. Індексація одновимірного масиву
2. Індексація багатовимірного масиву
3. Зрізи одновимірного масиву
4. Зрізи багатовимірного масиву
5. Зріз — це перегляд, а не копія
6. Запис через індекс і зріз
7. Булева індексація
8. Комбінування умов
9. `np.where`
10. Індексація масивом індексів
11. Перегляд чи копія: підсумок
12. Broadcasting: ідея
13. Правила broadcasting
14. Додавання осі: `np.newaxis`
15. Broadcasting і пам'ять
16. Приклад: рейтинг студентів
17. Типові помилки
18. Підсумок

У попередній лекції масив був єдиним цілим: створили, порахували, вивели. Зазвичай же потрібна частина даних — один рядок таблиці, останні сім вимірів, тільки ті значення, що перевищують поріг. Цим займається **індексація**.

Друга тема лекції — **broadcasting**: правило, за яким NumPy поєднує масиви різних форм. Ми вже користувалися ним, коли писали `prices * 1.2`; тепер розберемо загальний механізм.

## Індексація одновимірного масиву

Одновимірний масив індексується так само, як список: з нуля, з підтримкою від'ємних індексів.

```python
import numpy as np

temperatures = np.array([12.5, 15.0, 9.5, 18.0, 21.5, 17.0, 11.5])

print(temperatures[0])
print(temperatures[3])
print(temperatures[-1])
print(temperatures[-2])
print(temperatures.size)

try:
    print(temperatures[7])
except IndexError as error:
    print("IndexError:", error)
```

```text
12.5
18.0
11.5
17.0
7
IndexError: index 7 is out of bounds for axis 0 with size 7
```

```text
індекс:    0     1     2     3     4     5     6
        +-----+-----+-----+-----+-----+-----+-----+
        |12.5 |15.0 | 9.5 |18.0 |21.5 |17.0 |11.5 |
        +-----+-----+-----+-----+-----+-----+-----+
від'ємний: -7    -6    -5    -4    -3    -2    -1
```

Результат індексації одним числом — **не масив**, а скалярне значення типу NumPy:

```python
import numpy as np

temperatures = np.array([12.5, 15.0, 9.5, 18.0, 21.5, 17.0, 11.5])

value = temperatures[2]
print(value, type(value))

temperatures[2] = 10.0
print(temperatures)

temperatures[0] = 7
print(temperatures)

counts = np.array([1, 2, 3])
counts[0] = 9.87
print(counts)
```

```text
9.5 <class 'numpy.float64'>
[12.5 15.  10.  18.  21.5 17.  11.5]
[ 7.  15.  10.  18.  21.5 17.  11.5]
[9 2 3]
```

`np.float64` поводиться як звичайний `float`: його можна додавати, виводити, порівнювати.

Два моменти при записі:

- **тип масиву не змінюється**. Цілому масиву присвоїли `9.87` — записалося `9`, дробова частина відкинулася мовчки;
- масив має фіксований розмір: `append`, `insert`, `pop` у нього немає. Додавання елемента — це створення нового масиву (`np.append`, `np.concatenate`), і в циклі так робити не варто.

## Індексація багатовимірного масиву

Для 2D-масиву індекси пишуть **через кому в одних дужках**: спершу рядок, потім стовпець.

```python
import numpy as np

# рядок — магазин, стовпець — день
sales = np.array([
    [12, 15, 11, 20],
    [8, 9, 14, 10],
    [20, 18, 25, 22],
])

print(sales[0, 0])
print(sales[2, 3])
print(sales[1, -1])
print(sales[0][0])
```

```text
12
22
10
12
```

Запис `sales[0][0]` теж працює, але робить зайву роботу: спочатку дістає весь рядок `sales[0]`, потім з нього — елемент. Правильна форма — `sales[0, 0]`.

Якщо індексів менше, ніж вимірів, решта осей беруться цілком:

```python
import numpy as np

sales = np.array([
    [12, 15, 11, 20],
    [8, 9, 14, 10],
    [20, 18, 25, 22],
])

row = sales[1]
print(row, row.shape)

print(sales[1, :])
print(sales[:, 2])
print(sales[:, 2].shape)
```

```text
[ 8  9 14 10] (4,)
[ 8  9 14 10]
[11 14 25]
(3,)
```

`sales[1]` і `sales[1, :]` — те саме: другий рядок. Двокрапка означає «уся вісь».

**Стовпець дістається тільки через кому**: `sales[:, 2]`. Записати `sales[2]` і отримати стовпець не вийде — це буде рядок.

Зверніть увагу: і рядок, і стовпець повертаються як **одновимірні** масиви. Форма `(3, 4)` після вибору одного рядка стає `(4,)` — вісь, по якій узяли один елемент, зникає.

## Зрізи одновимірного масиву

Синтаксис зрізу той самий, що у списків: `array[start:stop:step]`. Кінець не входить, будь-яка частина може бути пропущена.

```python
import numpy as np

letters = np.array(["a", "b", "c", "d", "e", "f", "g"])

print(letters[1:4])
print(letters[:3])
print(letters[4:])
print(letters[:])
print(letters[1:6:2])
print(letters[::2])
print(letters[::-1])
print(letters[-3:])
```

```text
['b' 'c' 'd']
['a' 'b' 'c']
['e' 'f' 'g']
['a' 'b' 'c' 'd' 'e' 'f' 'g']
['b' 'd' 'f']
['a' 'c' 'e' 'g']
['g' 'f' 'e' 'd' 'c' 'b' 'a']
['e' 'f' 'g']
```

| Зріз | Що означає |
|---|---|
| `a[1:4]` | елементи 1, 2, 3 |
| `a[:3]` | перші три |
| `a[4:]` | від четвертого до кінця |
| `a[::2]` | кожен другий |
| `a[::-1]` | у зворотному порядку |
| `a[-3:]` | останні три |

На відміну від індексації, зріз **не виходить за межі** — він просто обрізається:

```python
import numpy as np

data = np.arange(5)

print(data[2:10])
print(data[10:20])
print(data[10:20].shape)
```

```text
[2 3 4]
[]
(0,)
```

Порожній масив — не помилка. Це частий результат фільтрації, і його треба перевіряти: `array.size == 0` перед тим, як брати `min()` чи `mean()`.

## Зрізи багатовимірного масиву

Для кожної осі пишеться свій зріз, осі розділяються комами.

```python
import numpy as np

table = np.arange(1, 21).reshape(4, 5)
print(table)

print("rows 1-2:")
print(table[1:3])

print("cols 1-3:")
print(table[:, 1:4])

print("block:")
print(table[1:3, 2:5])

print("every second row, every second column:")
print(table[::2, ::2])

print("last row reversed:")
print(table[-1, ::-1])
```

```text
[[ 1  2  3  4  5]
 [ 6  7  8  9 10]
 [11 12 13 14 15]
 [16 17 18 19 20]]
rows 1-2:
[[ 6  7  8  9 10]
 [11 12 13 14 15]]
cols 1-3:
[[ 2  3  4]
 [ 7  8  9]
 [12 13 14]
 [17 18 19]]
block:
[[ 8  9 10]
 [13 14 15]]
every second row, every second column:
[[ 1  3  5]
 [11 13 15]]
last row reversed:
[20 19 18 17 16]
```

```text
        table[1:3, 2:5]

          0    1    2    3    4
        +----+----+----+----+----+
      0 |  1 |  2 |  3 |  4 |  5 |
        +----+----+====+====+====+
      1 |  6 |  7 |  8 |  9 | 10 |   <- рядки 1..2
        +----+----+----+----+----+
      2 | 11 | 12 | 13 | 14 | 15 |
        +----+----+====+====+====+
      3 | 16 | 17 | 18 | 19 | 20 |
        +----+----+----+----+----+
                    ^^^^^^^^^^^^
                    стовпці 2..4
```

Ось важлива відмінність між **числом** і **зрізом довжиною один**: число прибирає вісь, зріз її зберігає.

```python
import numpy as np

table = np.arange(1, 21).reshape(4, 5)

print(table[1, 2])
print(table[1:2, 2:3])
print(table[1:2, 2:3].shape)
print(table[1].shape)
print(table[1:2].shape)
```

```text
8
[[8]]
(1, 1)
(5,)
(1, 5)
```

Якщо функція далі очікує двовимірний масив, зріз `table[1:2]` безпечніший за `table[1]`.

## Зріз — це перегляд, а не копія

Найважливіша відмінність масивів від списків. Зріз списку — новий список; зріз масиву — **перегляд** (view): інший об'єкт, але **ті самі дані в пам'яті**.

```python
import numpy as np

original = np.arange(10)
part = original[2:5]

print("original:", original)
print("part:    ", part)

part[0] = 999

print("after part[0] = 999")
print("original:", original)
print("part:    ", part)
```

```text
original: [0 1 2 3 4 5 6 7 8 9]
part:     [2 3 4]
after part[0] = 999
original: [  0   1 999   3   4   5   6   7   8   9]
part:     [999   3   4]
```

Змінили зріз — змінився оригінал. Порівняйте зі списком:

```python
import numpy as np

numbers = np.arange(10)

data_list = list(range(10))
list_part = data_list[2:5]
list_part[0] = 999

print("list:      ", data_list)
print("list slice:", list_part)

array_part = numbers[2:5]
array_part[0] = 999

print("array:      ", numbers)
print("array slice:", array_part)
```

```text
list:       [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
list slice: [999, 3, 4]
array:       [  0   1 999   3   4   5   6   7   8   9]
array slice: [999   3   4]
```

Це зроблено заради швидкості: зріз масиву на мільйон елементів не копіює жодного байта, а лише запам'ятовує, звідки починати й з яким кроком іти.

Перевірити, чи є масив переглядом, можна через атрибут `base`. У перегляду він вказує на масив-власник даних, у самостійного масиву дорівнює `None`.

```python
import numpy as np

original = np.arange(10)
part = original[2:5]
copy = original[2:5].copy()

print(part.base is original)
print(copy.base is None)

part[0] = 999
copy[1] = -1

print("original:", original)
print("copy:    ", copy)
```

```text
True
True
original: [  0   1 999   3   4   5   6   7   8   9]
copy:     [ 2 -1  4]
```

!!! warning "Коли потрібна копія"
    Якщо ви збираєтеся **змінювати** результат зрізу, а оригінал має лишитися недоторканим — беріть `.copy()` явно. Помилка «дані десь змінилися самі» майже завжди означає забутий `.copy()`.

## Запис через індекс і зріз

Зріз можна не тільки читати, а й **присвоювати** йому значення. Праворуч може стояти число — тоді воно розійдеться по всіх елементах зрізу:

```python
import numpy as np

table = np.zeros((3, 4), dtype=int)

table[0] = 5
table[:, 1] = 7
table[1:3, 2:4] = 1
table[2, 0] = -1

print(table)
```

```text
[[ 5  7  5  5]
 [ 0  7  1  1]
 [-1  7  1  1]]
```

Праворуч може стояти й список чи масив — тоді розміри мають збігатися:

```python
import numpy as np

row = np.arange(6)
print(row)

row[1:4] = [10, 20, 30]
print(row)

try:
    row[1:4] = [10, 20]
except ValueError as error:
    print("ValueError:", error)
```

```text
[0 1 2 3 4 5]
[ 0 10 20 30  4  5]
ValueError: could not broadcast input array from shape (2,) into shape (3,)
```

Слово `broadcast` у повідомленні не випадкове: присвоєння одного числа цілому зрізу — це той самий broadcasting, до якого ми дійдемо в другій половині лекції.

## Булева індексація

Порівняння масиву з числом дає масив `bool` тієї ж форми — **маску**. Якщо передати маску в квадратні дужки, отримаємо лише ті елементи, навпроти яких стоїть `True`.

```python
import numpy as np

scores = np.array([55, 91, 74, 60, 88, 42, 100])

mask = scores >= 60
print(mask)
print(scores[mask])
print(scores[scores >= 60])
print(scores[scores >= 60].shape)
```

```text
[False  True  True  True  True False  True]
[ 91  74  60  88 100]
[ 91  74  60  88 100]
(5,)
```

```text
scores:  55     91     74     60     88     42    100
mask:  False   True   True   True   True  False  True
          |      |      |      |      |      |     |
          x      v      v      v      v      x     v
result:         91     74     60     88          100
```

Маска має бути тієї ж довжини, що й масив, а результат — завжди одновимірний, і його довжина заздалегідь невідома: скільки `True`, стільки й елементів.

Маску зазвичай не зберігають в окрему змінну, а пишуть прямо в дужках: `scores[scores >= 60]`. Це й називається **фільтрацією без циклу**.

```python
import numpy as np

scores = np.array([55, 91, 74, 60, 88, 42, 100])

print("count:", (scores >= 60).sum())
print("mean of passed:", scores[scores >= 60].mean())
print("any failed:", (scores < 60).any())
print("all passed:", (scores >= 60).all())
```

```text
count: 5
mean of passed: 82.6
any failed: True
all passed: False
```

Для двовимірного масиву маска теж двовимірна, а результат усе одно одновимірний — інакше й бути не може, бо `True` можуть стояти будь-де:

```python
import numpy as np

sales = np.array([
    [12, 15, 11, 20],
    [8, 9, 14, 10],
    [20, 18, 25, 22],
])

mask = sales > 15
print(mask)
print(sales[mask])
print(sales[mask].shape)
```

```text
[[False False False  True]
 [False False False False]
 [ True  True  True  True]]
[20 20 18 25 22]
(5,)
```

Маску можна використати й **ліворуч від знака рівності** — щоб змінити тільки відібрані елементи:

```python
import numpy as np

scores = np.array([55, 91, 74, 60, 88, 42, 100])

scores[scores < 60] = 60
print(scores)

scores[scores > 95] -= 5
print(scores)
```

```text
[ 60  91  74  60  88  60 100]
[60 91 74 60 88 60 95]
```

## Комбінування умов

Кілька умов поєднуються операторами `&` (і), `|` (або), `~` (не). **Кожна умова обов'язково в дужках** — у Python `&` має вищий пріоритет, ніж `>=`.

```python
import numpy as np

scores = np.array([55, 91, 74, 60, 88, 42, 100])

good = (scores >= 60) & (scores < 90)
print(good)
print(scores[good])

edge = (scores < 60) | (scores == 100)
print(scores[edge])

print(scores[~(scores >= 60)])
```

```text
[False False  True  True  True False False]
[74 60 88]
[ 55  42 100]
[55 42]
```

!!! warning "`and` / `or` з масивами не працюють"
    Ключові слова `and`, `or`, `not` намагаються перетворити весь масив на одне значення `True` або `False` — і не можуть:

    ```python
    import numpy as np

    scores = np.array([55, 91, 74])

    try:
        print(scores[(scores >= 60) and (scores < 90)])
    except ValueError as error:
        print("ValueError:", error)
    ```

    ```text
    ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()
    ```

    Це найчастіша помилка новачків у NumPy. Для масивів — тільки `&`, `|`, `~` і дужки.

| Умова | Python зі списками | NumPy з масивами |
|---|---|---|
| і | `and` | `&` |
| або | `or` | `\|` |
| не | `not` | `~` |

## `np.where`

`np.where(condition, value_if_true, value_if_false)` — векторизований аналог тернарного оператора: будує новий масив, беручи значення з другого аргументу там, де умова істинна, і з третього — там, де хибна.

```python
import numpy as np

temperatures = np.array([12.5, 15.0, 9.5, 18.0, 21.5])

labels = np.where(temperatures > 15, "warm", "cold")
print(labels)

adjusted = np.where(temperatures < 10, 10.0, temperatures)
print(adjusted)

indices = np.where(temperatures > 15)
print(indices)
print(indices[0])
print(temperatures[indices])
```

```text
['cold' 'cold' 'cold' 'warm' 'warm']
[12.5 15.  10.  18.  21.5]
(array([3, 4]),)
[3 4]
[18.  21.5]
```

Два різні режими:

- **з трьома аргументами** — заміна значень; результат тієї ж форми, що й умова;
- **з одним аргументом** — пошук позицій; результат — кортеж масивів індексів (по одному на вісь). Для 1D-масиву потрібний масив лежить у `indices[0]`.

Чим `np.where(cond, a, b)` кращий за маску: він не викидає елементи, а **замінює** їх, тож форма масиву зберігається.

## Індексація масивом індексів

Замість одного числа в дужки можна передати **список або масив індексів** — тоді NumPy збере елементи саме в тому порядку, у якому їх перелічено. Цей прийом англійською називають *fancy indexing*.

```python
import numpy as np

prices = np.array([100.0, 250.0, 75.0, 500.0, 320.0])
names = np.array(["pen", "book", "cup", "chair", "lamp"])

print(prices[[0, 3, 4]])
print(names[[0, 3, 4]])

order = np.array([4, 4, 0, 1])
print(prices[order])

print(prices[np.array([-1, -2])])
```

```text
[100. 500. 320.]
['pen' 'chair' 'lamp']
[320. 320. 100. 250.]
[320. 500.]
```

Індекси можуть повторюватися й іти в довільному порядку, а форма результату дорівнює формі списку індексів, а не вихідного масиву.

Найкорисніше застосування — **сортування одного масиву за значеннями іншого**. Метод `argsort` повертає не відсортовані значення, а порядок індексів, у якому їх треба взяти:

```python
import numpy as np

prices = np.array([100.0, 250.0, 75.0, 500.0, 320.0])
names = np.array(["pen", "book", "cup", "chair", "lamp"])

order = prices.argsort()
print(order)
print(prices[order])
print(names[order])

top2 = prices.argsort()[::-1][:2]
print(names[top2], prices[top2])
```

```text
[2 0 1 4 3]
[ 75. 100. 250. 320. 500.]
['cup' 'pen' 'book' 'lamp' 'chair']
['chair' 'lamp'] [500. 320.]
```

Для двовимірного масиву список індексів в одній осі вибирає рядки або стовпці, а два списки разом — окремі елементи:

```python
import numpy as np

sales = np.array([
    [12, 15, 11, 20],
    [8, 9, 14, 10],
    [20, 18, 25, 22],
])

print(sales[[0, 2]])
print(sales[[0, 2], :])
print(sales[:, [0, 3]])
print(sales[[0, 2], [1, 3]])
```

```text
[[12 15 11 20]
 [20 18 25 22]]
[[12 15 11 20]
 [20 18 25 22]]
[[12 20]
 [ 8 10]
 [20 22]]
[15 22]
```

Останній рядок читається парами: `[0, 2]` і `[1, 3]` дають елементи `sales[0, 1]` та `sales[2, 3]`, тобто `15` і `22`. Це **не** підтаблиця 2×2 — щоб отримати підтаблицю, потрібен `sales[[0, 2], :][:, [1, 3]]`.

## Перегляд чи копія: підсумок

```python
import numpy as np

numbers = np.arange(10)

view = numbers[2:5]
fancy = numbers[[2, 3, 4]]
masked = numbers[numbers > 6]

view[0] = -1
fancy[1] = -2
masked[0] = -3

print("numbers:", numbers)
print("view.base is numbers:", view.base is numbers)
print("fancy.base:", fancy.base)
print("masked.base:", masked.base)
```

```text
numbers: [ 0  1 -1  3  4  5  6  7  8  9]
view.base is numbers: True
fancy.base: None
masked.base: None
```

Змінилося тільки те, що чіпав `view`. Правило просте:

| Спосіб | Результат | Зміна результату впливає на оригінал |
|---|---|---|
| `a[2]` | скаляр | — |
| `a[2:5]`, `a[:, 1]`, `a.reshape(...)` | **перегляд** | так |
| `a[[2, 3, 4]]` | копія | ні |
| `a[a > 6]` | копія | ні |
| `a.copy()`, `a.astype(...)` | копія | ні |

Причина відмінності — у тому, чи можна описати вибірку правилом «почати звідси, іти з таким кроком». Зріз можна, довільний список індексів — ні, тому його доводиться копіювати.

Важливий виняток: **присвоєння працює завжди**. `a[a > 6] = 0` змінює саме `a`, бо тут копія не створюється — NumPy одразу записує значення на місце.

## Broadcasting: ідея

Ми вже багато разів писали `prices * 2` — множення масиву на число:

```python
import numpy as np

prices = np.array([100.0, 250.0, 75.0])

print(prices * 2)
print(prices + 10)
```

```text
[200. 500. 150.]
[110. 260.  85.]
```

Формально це операція над масивами форм `(3,)` і `()` — різних. NumPy «розтягує» число до потрібної форми й виконує поелементну операцію. Цей механізм називається **broadcasting** (трансляція), і працює він не тільки з числами.

Задача: таблиця продажів (рядок — магазин, стовпець — товар) і прайс (ціна кожного товару). Потрібна виручка по кожній комірці.

```python
import numpy as np

# рядок — магазин, стовпець — товар
sales = np.array([
    [2, 1, 4],
    [0, 3, 1],
    [5, 2, 0],
])
prices = np.array([100.0, 250.0, 75.0])

revenue = sales * prices
print(revenue)
print(revenue.sum(axis=1))
```

```text
[[200. 250. 300.]
 [  0. 750.  75.]
 [500. 500.   0.]]
[ 750.  825. 1000.]
```

Масив `(3, 3)` помножився на масив `(3,)`: рядок цін застосувався до **кожного** рядка таблиці. Без broadcasting довелося б писати цикл по магазинах.

```text
   sales (3, 3)        prices (3,)          результат (3, 3)

  [[2  1  4]                              [[2*100  1*250  4*75]
   [0  3  1]     *    [100 250 75]   =     [0*100  3*250  1*75]
   [5  2  0]]                              [5*100  2*250  0*75]]

                  «розтягується» на
                  кожен рядок
```

## Правила broadcasting

NumPy порівнює форми **справа наліво**. Осі сумісні, якщо виконано одну з умов:

1. розміри збігаються;
2. один із розмірів дорівнює `1` — тоді ця вісь «розтягується» до розміру іншої;
3. в одного з масивів осі просто немає — вважається, що там `1`.

Якщо хоч на одній позиції умови не виконано, операція неможлива.

```text
Приклад 1: сумісні

  A      (3, 4)
  B         (4,)   ->  вважаємо (1, 4)
  ---------------
  результат (3, 4)

Приклад 2: сумісні

  A      (3, 1)
  B      (1, 4)
  ---------------
  результат (3, 4)

Приклад 3: НЕсумісні

  A      (3, 4)
  B         (3,)   ->  вважаємо (1, 3)
  ---------------
           4 != 3  ->  ValueError
```

```python
import numpy as np

a = np.ones((3, 4))
b = np.ones((4,))
c = np.ones((3, 1))
d = np.ones((3,))

print((a + b).shape)
print((a + c).shape)

try:
    a + d
except ValueError as error:
    print("ValueError:", error)
```

```text
(3, 4)
(3, 4)
ValueError: operands could not be broadcast together with shapes (3,4) (3,) 
```

Останній випадок вартий уваги: таблиця `(3, 4)` і вектор `(3,)` **не** складаються, хоча чисел у векторі рівно стільки, скільки рядків. Вирівнювання йде справа: `3` порівнюється з `4`, а не з `3`.

Обидва масиви можуть розтягуватися одночасно:

```python
import numpy as np

column = np.array([[1], [2], [3]])
row = np.array([10, 20, 30, 40])

print(column.shape, row.shape)
print(column + row)
print((column + row).shape)
```

```text
(3, 1) (4,)
[[11 21 31 41]
 [12 22 32 42]
 [13 23 33 43]]
(3, 4)
```

Стовпець `(3, 1)` розтягнувся на 4 стовпці, рядок `(4,)` — на 3 рядки; результат `(3, 4)` містить усі пари сум.

Перевірити сумісність форм, не створюючи масивів, допомагає `np.broadcast_shapes`:

```python
import numpy as np

print(np.broadcast_shapes((3, 4), (4,)))
print(np.broadcast_shapes((3, 1), (1, 4)))
print(np.broadcast_shapes((2, 1, 3), (5, 3)))

try:
    np.broadcast_shapes((3, 4), (3,))
except ValueError as error:
    print("ValueError:", error)
```

```text
(3, 4)
(3, 4)
(2, 5, 3)
ValueError: shape mismatch: objects cannot be broadcast to a single shape.  Mismatch is between arg 0 with shape (3, 4) and arg 1 with shape (3,).
```

## Додавання осі: `np.newaxis`

Коли форми не сходяться, вектор треба перетворити на стовпець — тобто з `(3,)` зробити `(3, 1)`. Для цього є `reshape(-1, 1)` і коротший запис `np.newaxis` (це просто `None`, вставлений у дужки).

```python
import numpy as np

numbers = np.array([1, 2, 3, 4])
print(numbers.shape)

as_column = numbers.reshape(-1, 1)
print(as_column.shape)
print(as_column)

print(numbers[:, np.newaxis].shape)
print(numbers[np.newaxis, :].shape)
```

```text
(4,)
(4, 1)
[[1]
 [2]
 [3]
 [4]]
(4, 1)
(1, 4)
```

`np.newaxis` стоїть на тому місці, де треба вставити нову вісь довжини 1: `[:, np.newaxis]` — вісь у кінець (виходить стовпець), `[np.newaxis, :]` — на початок (рядок).

Класичний приклад — таблиця множення, побудована без жодного циклу:

```python
import numpy as np

sizes = np.array([1, 2, 3, 4, 5])

table = sizes[:, np.newaxis] * sizes[np.newaxis, :]
print(table)
```

```text
[[ 1  2  3  4  5]
 [ 2  4  6  8 10]
 [ 3  6  9 12 15]
 [ 4  8 12 16 20]
 [ 5 10 15 20 25]]
```


Тепер повернемося до типової задачі — відняти від таблиці середнє. Якщо середнє рахується **по стовпцях**, форма збігається сама собою:

```python
import numpy as np

# рядок — студент, стовпець — предмет
grades = np.array([
    [80, 60, 90],
    [55, 70, 65],
    [95, 88, 100],
    [72, 64, 81],
])

subject_mean = grades.mean(axis=0)
print("subject mean:", subject_mean, subject_mean.shape)

deviation = grades - subject_mean
print(np.round(deviation, 2))
```

```text
subject mean: [75.5 70.5 84. ] (3,)
[[  4.5 -10.5   6. ]
 [-20.5  -0.5 -19. ]
 [ 19.5  17.5  16. ]
 [ -3.5  -6.5  -3. ]]
```

Форми `(4, 3)` і `(3,)` вирівнюються справа: `3` проти `3` — збіг, зліва в другого масиву осі немає. Усе працює саме собою.

А от середнє **по рядках** дає форму `(4,)`, і вирівнювання справа зіставить `3` із `4`:

```python
import numpy as np

grades = np.array([
    [80, 60, 90],
    [55, 70, 65],
    [95, 88, 100],
    [72, 64, 81],
])

student_mean = grades.mean(axis=1)
print("student mean:", np.round(student_mean, 2), student_mean.shape)

try:
    grades - student_mean
except ValueError as error:
    print("ValueError:", error)

print(np.round(grades - student_mean[:, np.newaxis], 2))
```

```text
student mean: [76.67 63.33 94.33 72.33] (4,)
ValueError: operands could not be broadcast together with shapes (4,3) (4,) 
[[  3.33 -16.67  13.33]
 [ -8.33   6.67   1.67]
 [  0.67  -6.33   5.67]
 [ -0.33  -8.33   8.67]]
```

`student_mean[:, np.newaxis]` перетворює `(4,)` на `(4, 1)`, і тепер кожне середнє відповідає своєму рядку.

!!! danger "Квадратна таблиця приховує помилку"
    Якби студентів було рівно троє, форми `(3, 3)` і `(3,)` **зійшлися б** — і NumPy мовчки відняв би середнє по рядках від стовпців. Помилки немає, результат неправильний. Тому на квадратних даних цю помилку не видно, а на реальних вона вилазить.

Надійніший спосіб — попросити агрегат **зберегти вісь** параметром `keepdims=True`:

```python
import numpy as np

grades = np.array([
    [80, 60, 90],
    [55, 70, 65],
    [95, 88, 100],
    [72, 64, 81],
])

student_mean = grades.mean(axis=1, keepdims=True)
print(np.round(student_mean, 2))
print(student_mean.shape)
print(np.round(grades - student_mean, 2))
```

```text
[[76.67]
 [63.33]
 [94.33]
 [72.33]]
(4, 1)
[[  3.33 -16.67  13.33]
 [ -8.33   6.67   1.67]
 [  0.67  -6.33   5.67]
 [ -0.33  -8.33   8.67]]
```

!!! tip "`keepdims=True`"
    Якщо результат агрегації буде брати участь в операції з вихідним масивом — одразу пишіть `keepdims=True`. Форма збережеться, broadcasting спрацює правильно, а `np.newaxis` не знадобиться.

## Broadcasting і пам'ять

Слово «розтягується» не означає, що NumPy справді створює копії рядків. Розтягнутий масив існує лише концептуально: обчислення читає той самий елемент кілька разів.

```python
import numpy as np

rows = np.arange(1000)[:, np.newaxis]
cols = np.arange(1000)[np.newaxis, :]

table = rows * cols

print("rows: ", rows.nbytes, "bytes")
print("cols: ", cols.nbytes, "bytes")
print("table:", table.nbytes, "bytes")
```

```text
rows:  8000 bytes
cols:  8000 bytes
table: 8000000 bytes
```

Вхідні масиви займають по 8 КБ, проміжних копій `(1000, 1000)` не створюється — пам'ять витрачається лише на результат. Саме тому broadcasting не просто зручний, а й швидкий.

Але пам'ятайте про **результат**: `(10000, 1)` разом із `(1, 10000)` дадуть масив на 800 МБ. Перед такою операцією корисно порахувати `shape` у голові.

## Приклад: рейтинг студентів

Зберемо все разом. Є бали за три контрольні, треба перевести їх у відсотки від максимуму, знайти середнє по студенту, відібрати кращих і впорядкувати рейтинг. Жодного циклу.

```python
import numpy as np

# рядок — студент, стовпець — контрольна робота
points = np.array([
    [12.0, 18.0, 25.0],
    [20.0, 9.0, 30.0],
    [15.0, 15.0, 15.0],
    [8.0, 20.0, 22.0],
])
names = np.array(["petrenko", "kovalenko", "shevchuk", "bondar"])

# 1. Максимум по кожній контрольній (axis=0 -> форма (3,))
max_points = points.max(axis=0)
print("max per test:", max_points)

# 2. Broadcasting: (4, 3) / (3,) — кожен стовпець ділиться на свій максимум
percent = points / max_points * 100
print(np.round(percent, 1))

# 3. Середній відсоток кожного студента
average = percent.mean(axis=1)
print("average:", np.round(average, 1))

# 4. Булева маска: хто набрав понад 70%
good = average > 70
print("good students:", names[good])

# 5. Сортування за середнім, від кращого до гіршого
order = average.argsort()[::-1]
print(names[order])
print(np.round(average[order], 1))
```

```text
max per test: [20. 20. 30.]
[[ 60.   90.   83.3]
 [100.   45.  100. ]
 [ 75.   75.   50. ]
 [ 40.  100.   73.3]]
average: [77.8 81.7 66.7 71.1]
good students: ['petrenko' 'kovalenko' 'bondar']
['kovalenko' 'petrenko' 'bondar' 'shevchuk']
[81.7 77.8 71.1 66.7]
```

П'ять рядків обчислень замість трьох вкладених циклів — і це типовий вигляд коду на NumPy.

## Типові помилки

**`and` замість `&`.** `(a > 0) and (a < 10)` кидає `ValueError: The truth value of an array ... is ambiguous`. Правильно: `(a > 0) & (a < 10)`.

**Забуті дужки навколо умов.** `a > 0 & a < 10` через пріоритет операторів обчислиться як `a > (0 & a) < 10` і дасть незрозумілу помилку.

**Змінили зріз — зіпсували оригінал.** Зріз — перегляд. Якщо результат буде змінюватися, беріть `.copy()`.

**Очікували копію від `reshape`.** `reshape` теж повертає перегляд; запис у результат змінює вихідний масив.

**Маска іншої довжини.** Маска має точно відповідати формі масиву:

```python
import numpy as np

values = np.array([10.0, 20.0, 30.0, 40.0])
mask = np.array([True, False, True])

try:
    values[mask]
except IndexError as error:
    print("IndexError:", error)
```

```text
IndexError: boolean index did not match indexed array along axis 0; size of axis is 4 but size of corresponding boolean axis is 3
```

**`a[0][1]` замість `a[0, 1]`.** Працює, але створює зайвий проміжний масив.

**Стовпець через `a[i]`.** `a[1]` — це рядок. Стовпець — `a[:, 1]`.

**Вектор `(n,)` і таблиця `(m, n)` переплутані місцями.** Форми вирівнюються справа, тому `(3, 4) - (3,)` — помилка. Для віднімання по рядках потрібен `(3, 1)`: `keepdims=True` або `[:, np.newaxis]`.

**Порожній результат фільтрації.** `array[array > 100].mean()` на порожньому масиві дає `nan` і `RuntimeWarning`. Перевіряйте `size` перед агрегацією.

**Цикл там, де достатньо маски.** Замість `for` з `if` — одна умова в дужках. Якщо в коді з'явився `for i in range(len(array))`, майже завжди його можна прибрати.

## Підсумок

- **Індексація одним числом** прибирає вісь і повертає скаляр (для 1D) або масив меншої розмірності; від'ємні індекси рахуються з кінця.
- У багатовимірному масиві індекси пишуться **через кому**: `a[рядок, стовпець]`. Стовпець дістається лише як `a[:, j]`.
- **Зріз** `start:stop:step` не виходить за межі й зберігає вісь: `a[1:2]` має форму `(1, n)`, а `a[1]` — `(n,)`.
- **Зріз і `reshape` повертають перегляд** (view) — спільні дані з оригіналом. Незалежний масив дає `.copy()`.
- **Булева маска** фільтрує без циклу: `a[a > 0]`. Умови поєднуються через `&`, `|`, `~` і обов'язкові дужки; `and`/`or` з масивами не працюють.
- **`np.where`** замінює значення за умовою (три аргументи) або знаходить позиції (один аргумент).
- **Список індексів** вибирає елементи в довільному порядку й повертає копію; разом з `argsort` дозволяє сортувати один масив за іншим.
- **Broadcasting** узгоджує форми справа наліво: розміри або збігаються, або один із них дорівнює `1`, або осі немає.
- **`np.newaxis` і `keepdims=True`** додають вісь довжини 1, коли форми не сходяться — типово при відніманні середнього по рядках.
- Розтягування **не копіює даних**: пам'ять витрачається тільки на результат.

## Корисні посилання

- [NumPy: indexing on ndarrays](https://numpy.org/doc/stable/user/basics.indexing.html)
- [NumPy: broadcasting](https://numpy.org/doc/stable/user/basics.broadcasting.html)
- [NumPy: copies and views](https://numpy.org/doc/stable/user/basics.copies.html)
- [`np.where`](https://numpy.org/doc/stable/reference/generated/numpy.where.html)
- [`np.argsort`](https://numpy.org/doc/stable/reference/generated/numpy.argsort.html)
- [`np.broadcast_shapes`](https://numpy.org/doc/stable/reference/generated/numpy.broadcast_shapes.html)
