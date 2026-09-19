# 30. (Л) Статистичні функції та генерація випадкових даних у NumPy

## Зміст лекції

1. Навіщо це програмісту
2. Середнє і медіана
3. Мода: найчастіше значення
4. Розкид: `ptp`, `var`, `std`
5. `ddof`: генеральна сукупність чи вибірка
6. Перцентилі та квантилі
7. Квартилі, IQR і пошук викидів
8. Статистика по осях
9. Стандартизація: z-оцінка
10. Зважене середнє: `np.average`
11. Накопичувальні суми: `cumsum`
12. Гістограма: `np.histogram` і `np.bincount`
13. Кореляція: `np.corrcoef`
14. Пропущені значення: `nan`-функції
15. Випадкові числа: генератор `default_rng`
16. Зерно (`seed`) і відтворюваність
17. Рівномірні числа: `random`, `uniform`, `integers`
18. Нормальний розподіл
19. Вибір і перемішування: `choice`, `shuffle`, `permutation`
20. Симуляція: кидки кубика
21. Метод Монте-Карло
22. Приклад: аналіз оцінок групи
23. Старий API `np.random.*`
24. Типові помилки
25. Підсумок

## Навіщо це програмісту

Будь-які дані — час відповіді сервера, оцінки студентів, продажі, показники датчиків — рідко цікаві «поштучно». Потрібні відповіді на питання на кшталт:

- яке значення **типове**? (середнє, медіана)
- наскільки дані **розкидані**? (дисперсія, стандартне відхилення)
- яка межа, нижче якої лежать 95% значень? (перцентиль)
- чи є **аномальні** значення? (викиди)
- чи пов'язані дві величини між собою? (кореляція)

Друга половина лекції — про **випадкові дані**. Вони потрібні, щоб:

- **моделювати** процеси, які важко порахувати формулою (симуляції, метод Монте-Карло);
- **генерувати тестові дані** — наприклад, 10 000 «студентів» для перевірки програми аналізу оцінок;
- **перемішувати** й **ділити** дані (випадковий порядок питань тесту, навчальна і тестова вибірки в машинному навчанні).

У лекції 25 ми вже бачили агрегати `sum`, `min`, `max`, `mean`, `argmin`, `argmax` і параметр `axis`. Тепер додамо до них решту статистичного набору.

## Середнє і медіана

**Середнє арифметичне** — сума, поділена на кількість:

$$
\bar{x} = \frac{x_0 + x_1 + \dots + x_{n-1}}{n}
$$

**Медіана** — значення посередині відсортованого масиву. Для парної кількості елементів — середнє двох центральних.

```python
import numpy as np

# час відповіді сервера, мс
response = np.array([120, 95, 110, 105, 130, 98, 102])

print("sorted:", np.sort(response))
print("mean:  ", response.mean())
print("median:", np.median(response))
```

```text
sorted: [ 95  98 102 105 110 120 130]
mean:   108.57142857142857
median: 105.0
```

Поки дані «спокійні», середнє і медіана близькі. Тепер додамо один повільний запит:

```python
import numpy as np

response = np.array([120, 95, 110, 105, 130, 98, 102, 3000])

print("mean:  ", response.mean())
print("median:", np.median(response))
```

```text
mean:   470.0
median: 107.5
```

Одне аномальне значення підняло середнє майже вчетверо, а медіана зсунулася лише трохи. Кажуть, що медіана **стійка до викидів**.

!!! tip "Що обрати"
    Для зарплат, цін на житло, часу відповіді — усього, де бувають поодинокі дуже великі значення, — типове значення краще описує **медіана**. Середнє корисне, коли важлива сума (загальна виручка, сумарний час) або коли дані симетричні без викидів.

Зверніть увагу: `median` є лише у формі функції `np.median(...)`, методу `array.median()` не існує.

## Мода: найчастіше значення

**Мода** — значення, яке зустрічається найчастіше. Окремої функції в NumPy немає, але її легко отримати через `np.unique` з підрахунком:

```python
import numpy as np

# розміри взуття, продані за день
sizes = np.array([42, 40, 43, 42, 41, 42, 44, 40, 42, 43])

values, counts = np.unique(sizes, return_counts=True)
print("values:", values)
print("counts:", counts)
print("mode:  ", values[counts.argmax()])
```

```text
values: [40 41 42 43 44]
counts: [2 1 4 2 1]
mode:   42
```

`np.unique` повертає відсортовані унікальні значення, а з `return_counts=True` — ще й скільки разів кожне трапилося. `argmax` знаходить позицію найбільшої кількості.

Для **невід'ємних цілих** чисел є швидший спосіб — `np.bincount`: він рахує, скільки разів зустрілося кожне число від `0` до максимуму.

```python
import numpy as np

marks = np.array([4, 5, 3, 5, 4, 5, 2, 5, 4])

counts = np.bincount(marks)
print(counts)
print("mode:", counts.argmax())
```

```text
[0 0 1 1 3 4]
mode: 5
```

Індекс у результаті `bincount` — це саме значення: `counts[5]` — кількість п'ятірок.

## Розкид: `ptp`, `var`, `std`

Два набори можуть мати однакове середнє і зовсім різну поведінку:

```python
import numpy as np

# денна температура в двох містах за тиждень
city_a = np.array([20, 21, 19, 20, 21, 19, 20])
city_b = np.array([10, 30, 15, 25, 12, 28, 20])

print("mean:", city_a.mean(), city_b.mean())
print("ptp: ", np.ptp(city_a), np.ptp(city_b))
print("var: ", round(city_a.var(), 3), round(city_b.var(), 3))
print("std: ", round(city_a.std(), 3), round(city_b.std(), 3))
```

```text
mean: 20.0 20.0
ptp:  2 20
var:  0.571 54.0
std:  0.756 7.348
```

- **`np.ptp`** (peak to peak) — розмах: `max - min`. Простий, але залежить лише від двох крайніх значень.
- **`var`** — **дисперсія**: середній квадрат відхилення від середнього.
- **`std`** — **стандартне відхилення**: корінь із дисперсії.

$$
\sigma^2 = \frac{1}{n}\sum_{i=0}^{n-1} (x_i - \bar{x})^2, \qquad \sigma = \sqrt{\sigma^2}
$$

Дисперсія має «квадратні» одиниці (градуси²), тому на практиці частіше дивляться на `std` — воно в тих самих одиницях, що й дані. Грубо: «температура в місті B зазвичай відхиляється від середнього на 7 градусів».

Перевіримо формулу вручну:

```python
import numpy as np

data = np.array([10, 30, 15, 25, 12, 28, 20])

deviation = data - data.mean()
variance = np.mean(deviation ** 2)

print(round(variance, 3), round(data.var(), 3))
print(round(np.sqrt(variance), 3), round(data.std(), 3))
```

```text
54.0 54.0
7.348 7.348
```

## `ddof`: генеральна сукупність чи вибірка

За замовчуванням NumPy ділить на \(n\) — це дисперсія **генеральної сукупності** (коли у вас є *всі* дані). Якщо ж дані — лише **вибірка** з більшої сукупності (опитали 50 людей з міста), статистики ділять на \(n - 1\):

$$
s^2 = \frac{1}{n - 1}\sum_{i=0}^{n-1} (x_i - \bar{x})^2
$$

Параметр `ddof` (delta degrees of freedom) задає, що відняти від \(n\) у знаменнику:

```python
import numpy as np

sample = np.array([10, 30, 15, 25, 12, 28, 20])

print("ddof=0:", round(sample.std(), 3))
print("ddof=1:", round(sample.std(ddof=1), 3))
```

```text
ddof=0: 7.348
ddof=1: 7.937
```

!!! warning "Різні бібліотеки — різні замовчування"
    NumPy за замовчуванням використовує `ddof=0`, а pandas (наступні лекції) — `ddof=1`. Тому `std` для тих самих даних у NumPy і pandas може відрізнятися. На великих масивах різниця мізерна, на малих — помітна.

## Перцентилі та квантилі

**Перцентиль** \(p\) — значення, нижче якого лежить \(p\%\) даних. Медіана — це 50-й перцентиль.

```python
import numpy as np

response = np.array([95, 98, 102, 105, 110, 120, 130, 150, 180, 400])

print("p50:", np.percentile(response, 50))
print("p90:", round(np.percentile(response, 90), 1))
print("p95:", round(np.percentile(response, 95), 1))
print("several:", np.percentile(response, [25, 50, 75]))
```

```text
p50: 115.0
p90: 202.0
p95: 301.0
several: [102.75 115.   145.  ]
```

Перцентиль може бути значенням, якого в масиві немає: між сусідніми елементами NumPy виконує **лінійну інтерполяцію**. Спосіб можна змінити параметром `method` (наприклад, `method="nearest"` поверне найближчий наявний елемент).

**Квантиль** — те саме, але в частках від 0 до 1 замість відсотків:

```python
import numpy as np

response = np.array([95, 98, 102, 105, 110, 120, 130, 150, 180, 400])

print(round(np.quantile(response, 0.9), 1))
print(np.quantile(response, [0.25, 0.5, 0.75]))
```

```text
202.0
[102.75 115.   145.  ]
```

!!! note "Перцентилі в реальних системах"
    Моніторинг серверів майже завжди показує не середній час відповіді, а **p95** чи **p99**: «95% запитів обробляються швидше за 250 мс». Середнє ховає повільні запити, а саме на них скаржаться користувачі.

## Квартилі, IQR і пошук викидів

Три перцентилі 25, 50, 75 ділять дані на чотири рівні частини — це **квартилі** Q1, Q2 (медіана), Q3. Відстань між Q1 і Q3 — **міжквартильний розмах** (IQR, interquartile range): у ньому лежить середня половина даних.

```text
   min        Q1        Q2        Q3        max
    |---------|=========|=========|---------|
     25% даних  25% даних 25% даних  25% даних
              <------- IQR ------->
```

Класичне правило Тьюкі: значення, що лежить далі ніж \(1.5 \cdot IQR\) за межами квартилів, вважається **викидом**.

```python
import numpy as np

response = np.array([95, 98, 102, 105, 110, 120, 130, 150, 180, 400])

q1, q3 = np.percentile(response, [25, 75])
iqr = q3 - q1
low = q1 - 1.5 * iqr
high = q3 + 1.5 * iqr

print("Q1, Q3:", q1, q3)
print("IQR:", iqr)
print("bounds:", low, high)

is_outlier = (response < low) | (response > high)
print("outliers:", response[is_outlier])
print("clean mean:", response[~is_outlier].mean())
```

```text
Q1, Q3: 102.75 145.0
IQR: 42.25
bounds: 39.375 208.375
outliers: [400]
clean mean: 121.11111111111111
```

Тут працює булева індексація з лекції 27: маска `is_outlier` відбирає аномальні значення, а `~is_outlier` — усі інші.

## Статистика по осях

Усі статистичні функції приймають `axis`, як і `sum` чи `mean`. Оцінки чотирьох студентів із трьох предметів:

```python
import numpy as np

# рядок — студент, стовпець — предмет
grades = np.array([
    [78, 65, 90],
    [55, 70, 62],
    [92, 88, 95],
    [70, 48, 81],
])

print("median per subject:", np.median(grades, axis=0))
print("std per subject:   ", np.round(grades.std(axis=0), 2))
print("mean per student:  ", np.round(grades.mean(axis=1), 2))
print("p75 per subject:   ", np.percentile(grades, 75, axis=0))
```

```text
median per subject: [74.  67.5 85.5]
std per subject:    [13.39 14.25 12.59]
mean per student:   [77.67 62.33 91.67 66.33]
p75 per subject:    [81.5  74.5  91.25]
```

Правило те саме, що в лекції 25: `axis` — це вісь, яка «зникає». `axis=0` дає по одному числу на **стовпець** (предмет), `axis=1` — на **рядок** (студента).

## Стандартизація: z-оцінка

Як порівняти 78 балів з математики і 65 з фізики, якщо фізика складніша? Перевести обидва числа в **z-оцінку** — кількість стандартних відхилень від середнього:

$$
z = \frac{x - \bar{x}}{\sigma}
$$

- \(z = 0\) — рівно середній результат;
- \(z = 1\) — на одне стандартне відхилення вище за середнє;
- \(z = -2\) — значно нижче за середнє.

```python
import numpy as np

grades = np.array([
    [78, 65, 90],
    [55, 70, 62],
    [92, 88, 95],
    [70, 48, 81],
])

mean = grades.mean(axis=0)
std = grades.std(axis=0)
z = (grades - mean) / std

print(np.round(z, 2))
print("z mean:", np.round(z.mean(axis=0), 2))
print("z std: ", np.round(z.std(axis=0), 2))
```

```text
[[ 0.32 -0.19  0.64]
 [-1.4   0.16 -1.59]
 [ 1.36  1.42  1.03]
 [-0.28 -1.39 -0.08]]
z mean: [0. 0. 0.]
z std:  [1. 1. 1.]
```

Форми `(4, 3)` і `(3,)` сумісні для broadcasting (лекція 27), тож середнє кожного предмета віднімається від свого стовпця. Після стандартизації кожен стовпець має середнє 0 і стандартне відхилення 1, а значення різних предметів стають порівнянними. Студент 1 має 55 з математики і 70 з фізики: z-оцінки −1.4 і 0.16 показують, що з математики він серед найслабших у групі, а з фізики — трохи вище за середнє.

Стандартизація — обов'язковий крок підготовки даних для багатьох алгоритмів машинного навчання.

## Зважене середнє: `np.average`

Якщо значення мають різну **вагу**, звичайне середнє не підходить. Приклад: підсумкова оцінка, де екзамен важить 50%, а лабораторні і контрольна — по 25%.

$$
\bar{x}_w = \frac{\sum_i w_i x_i}{\sum_i w_i}
$$

```python
import numpy as np

# лабораторні, контрольна, екзамен
scores = np.array([90, 70, 60])
weights = np.array([0.25, 0.25, 0.5])

print("mean:    ", scores.mean())
print("weighted:", np.average(scores, weights=weights))
print("manual:  ", np.sum(scores * weights) / np.sum(weights))
```

```text
mean:     73.33333333333333
weighted: 70.0
manual:   70.0
```

`np.average` без `weights` поводиться як `mean`. Ваги не обов'язково мають давати в сумі 1 — функція сама ділить на їхню суму. Так, ваги `[1, 1, 2]` дадуть той самий результат.

## Накопичувальні суми: `cumsum`

`cumsum` повертає масив проміжних сум: кожен елемент — сума всіх попередніх разом із поточним.

```python
import numpy as np

# денні продажі за тиждень
daily = np.array([12, 15, 8, 20, 17, 25, 30])

total = np.cumsum(daily)
print(total)
print("day when 50 reached:", np.argmax(total >= 50))
```

```text
[ 12  27  35  55  72  97 127]
day when 50 reached: 3
```

`np.argmax` для булевого масиву повертає індекс першого `True` — зручний спосіб знайти момент, коли накопичена сума вперше перетнула поріг.

Схожі функції: `np.cumprod` (накопичувальний добуток — наприклад, складні відсотки) і `np.diff` (різниця сусідніх елементів — обернена до `cumsum` операція):

```python
import numpy as np

total = np.array([12, 27, 35, 55, 72, 97, 127])
print(np.diff(total))

# зростання вкладу на 10%, 5%, 20% за три роки
growth = np.array([1.10, 1.05, 1.20])
print(np.round(1000 * np.cumprod(growth), 2))
```

```text
[15  8 20 17 25 30]
[1100. 1155. 1386.]
```

## Гістограма: `np.histogram` і `np.bincount`

**Гістограма** показує, скільки значень потрапило в кожен інтервал («кошик», bin). `np.histogram` повертає дві речі: кількості та межі кошиків.

```python
import numpy as np

scores = np.array([45, 52, 58, 61, 63, 67, 70, 72, 74, 75, 78, 81, 85, 88, 93, 97])

counts, edges = np.histogram(scores, bins=[0, 60, 75, 90, 101])
print("counts:", counts)
print("edges: ", edges)

labels = ["F", "C", "B", "A"]
for label, count in zip(labels, counts):
    print(f"{label}: {'#' * count}")
```

```text
counts: [3 6 5 2]
edges:  [  0  60  75  90 101]
F: ###
C: ######
B: #####
A: ##
```

Кошики напіввідкриті: `[0, 60)`, `[60, 75)`, `[75, 90)`, а **останній** — закритий з обох боків: `[90, 101]`. Тому 75 потрапляє до «B», а не до «C».

Можна передати просто кількість кошиків — `bins=5`, і NumPy сам розіб'є діапазон від мінімуму до максимуму на рівні частини.

Для цілих значень із невеликого діапазону (оцінки 1–5, грані кубика) простіше `np.bincount`, який ми вже бачили в розділі про моду. Параметр `minlength` гарантує, що в результаті будуть усі значення, навіть якщо деяких не трапилося:

```python
import numpy as np

# п'ятірок у групі немає
marks = np.array([4, 3, 4, 2, 4, 3])

print(np.bincount(marks))
print(np.bincount(marks, minlength=6))
```

```text
[0 0 1 2 3]
[0 0 1 2 3 0]
```

## Кореляція: `np.corrcoef`

**Коефіцієнт кореляції Пірсона** \(r\) показує, наскільки дві величини пов'язані **лінійно**:

- \(r \approx 1\) — одна росте, коли росте інша;
- \(r \approx -1\) — одна росте, коли інша спадає;
- \(r \approx 0\) — лінійного зв'язку немає.

```python
import numpy as np

hours = np.array([1, 2, 3, 4, 5, 6, 7, 8])
score = np.array([52, 55, 61, 64, 70, 72, 79, 83])
absences = np.array([9, 8, 8, 6, 5, 3, 2, 1])

matrix = np.corrcoef(hours, score)
print(np.round(matrix, 3))

print("hours vs score:   ", round(np.corrcoef(hours, score)[0, 1], 3))
print("hours vs absences:", round(np.corrcoef(hours, absences)[0, 1], 3))
```

```text
[[1.    0.996]
 [0.996 1.   ]]
hours vs score:    0.996
hours vs absences: -0.988
```

`np.corrcoef` повертає **матрицю** кореляцій: на діагоналі — кореляція величини з самою собою (завжди 1), поза діагоналлю — те, що нас цікавить. Тому потрібне значення беруть як `[0, 1]`.

!!! warning "Кореляція — не причинність"
    Висока кореляція означає лише, що величини змінюються разом. Продажі морозива і кількість сонячних опіків сильно корелюють, але одне не спричиняє інше — обидва залежать від погоди.

## Пропущені значення: `nan`-функції

У реальних даних бувають пропуски: датчик не відповів, студент не здав роботу. У NumPy пропуск позначають `np.nan` (not a number). Проблема в тому, що будь-яка арифметика з `nan` дає `nan`:

```python
import numpy as np

temperatures = np.array([12.5, np.nan, 15.0, 9.5, np.nan, 18.0])

print("mean:   ", temperatures.mean())
print("nanmean:", np.nanmean(temperatures))
print("nanmax: ", np.nanmax(temperatures))
print("nanstd: ", round(np.nanstd(temperatures), 3))
print("missing:", np.isnan(temperatures).sum())
```

```text
mean:    nan
nanmean: 13.75
nanmax:  18.0
nanstd:  3.132
missing: 2
```

Для кожної статистичної функції є `nan`-версія, яка просто ігнорує пропуски: `np.nansum`, `np.nanmean`, `np.nanmedian`, `np.nanstd`, `np.nanvar`, `np.nanmin`, `np.nanmax`, `np.nanpercentile`.

`np.isnan` повертає булеву маску пропусків. Її можна використати, щоб прибрати `nan` або замінити їх:

```python
import numpy as np

temperatures = np.array([12.5, np.nan, 15.0, 9.5, np.nan, 18.0])

print(temperatures[~np.isnan(temperatures)])

filled = np.where(np.isnan(temperatures), np.nanmean(temperatures), temperatures)
print(filled)
```

```text
[12.5 15.   9.5 18. ]
[12.5  13.75 15.    9.5  13.75 18.  ]
```

!!! note "Чому не `== np.nan`"
    `nan` не дорівнює нічому, навіть самому собі: `np.nan == np.nan` дає `False`. Тому пропуски шукають лише через `np.isnan`.

## Випадкові числа: генератор `default_rng`

Комп'ютер — детермінована машина, тому «випадкові» числа насправді **псевдовипадкові**: їх видає алгоритм, який із початкового стану (**зерна**, seed) будує довгу послідовність чисел, що виглядають випадковими.

Сучасний спосіб отримати випадкові числа в NumPy — створити об'єкт **генератора** і викликати його методи:

```python
import numpy as np

rng = np.random.default_rng()

print(rng.random())
print(rng.integers(1, 7, size=5))
```

Вивід щоразу інший. Ім'я `rng` (random number generator) — загальноприйнята домовленість.

Генератор — звичайний об'єкт. Його створюють один раз на початку програми і далі передають у функції, яким потрібні випадкові числа.

!!! warning "Не для паролів"
    Генератор NumPy швидкий і статистично якісний, але **передбачуваний**: знаючи зерно, послідовність можна відтворити. Для паролів, токенів і ключів використовуйте стандартний модуль `secrets`.

## Зерно (`seed`) і відтворюваність

Якщо передати в `default_rng` ціле число, генератор щоразу видаватиме **ту саму** послідовність:

```python
import numpy as np

first = np.random.default_rng(42)
second = np.random.default_rng(42)

print(first.integers(1, 7, size=8))
print(second.integers(1, 7, size=8))
print(first.integers(1, 7, size=8))
```

```text
[1 5 4 3 3 6 1 5]
[1 5 4 3 3 6 1 5]
[2 1 4 6 5 5 5 5]
```

Два генератори з однаковим зерном видали однакові числа. Третій рядок інший — генератор `first` уже просунувся далі своєю послідовністю.

Навіщо це потрібно:

- **налагодження**: помилка, яка виникає на «випадкових» даних, відтворюється при кожному запуску;
- **тести**: результат функції з випадковістю можна порівняти з очікуваним;
- **наукові розрахунки й навчальні приклади**: інші люди отримують ті самі числа.

Усі приклади далі в лекції використовують зерно, тож у вас вивід буде таким самим.

!!! tip "Коли зерно не потрібне"
    У «бойовій» програмі (гра, генерація варіантів тесту) зерно зазвичай не задають, щоб результат щоразу був різний. Зручний прийом — зробити зерно параметром: `default_rng(seed)`, де `seed=None` означає «справді випадково».

## Рівномірні числа: `random`, `uniform`, `integers`

**Рівномірний** розподіл — усі значення в діапазоні однаково ймовірні.

```python
import numpy as np

rng = np.random.default_rng(1)

# дробові числа з [0, 1)
print(np.round(rng.random(4), 3))

# дробові числа з [low, high)
print(np.round(rng.uniform(-5, 5, size=4), 3))

# цілі числа з [low, high) — верхня межа не входить
print(rng.integers(1, 7, size=10))

# з endpoint=True верхня межа входить
print(rng.integers(1, 6, size=10, endpoint=True))
```

```text
[0.512 0.95  0.144 0.949]
[-1.882 -0.767  3.277 -0.908]
[4 4 1 1 6 5 6 4 5 2]
[3 5 1 2 1 3 6 1 3 3]
```

Параметр `size` задає форму результату — це може бути й кортеж:

```python
import numpy as np

rng = np.random.default_rng(2)

matrix = rng.integers(0, 10, size=(3, 4))
print(matrix)
print(matrix.shape)
```

```text
[[8 2 1 2]
 [4 8 4 0]
 [3 6 8 7]]
(3, 4)
```

!!! warning "Верхня межа"
    Як і в `range`, верхня межа в `integers` **не входить**: `rng.integers(1, 6)` ніколи не поверне 6. Для кубика пишіть `integers(1, 7)` або `integers(1, 6, endpoint=True)`.

Переконаємося, що числа справді рівномірні — порахуємо, скільки разів випала кожна грань за 60 000 кидків:

```python
import numpy as np

rng = np.random.default_rng(3)

rolls = rng.integers(1, 7, size=60_000)
counts = np.bincount(rolls, minlength=7)[1:]

print(counts)
print(np.round(counts / rolls.size, 3))
```

```text
[10108  9951  9981 10008  9967  9985]
[0.168 0.166 0.166 0.167 0.166 0.166]
```

Кожна грань випала приблизно 10 000 разів, тобто з частотою близькою до \(1/6 \approx 0.167\).

## Нормальний розподіл

Багато природних величин — зріст людей, похибки вимірювань, оцінки великої групи — групуються навколо середнього, а великі відхилення рідкісні. Це **нормальний** (гауссів) розподіл, графік якого — «дзвін». Його задають два параметри: середнє `loc` і стандартне відхилення `scale`.

```python
import numpy as np

rng = np.random.default_rng(4)

# зріст 10 000 людей: середнє 170 см, std 8 см
height = rng.normal(loc=170, scale=8, size=10_000)

print("mean:", round(height.mean(), 2))
print("std: ", round(height.std(), 2))
print("min, max:", round(height.min(), 1), round(height.max(), 1))
```

```text
mean: 170.13
std:  7.95
min, max: 137.5 197.3
```

Вибіркові середнє і std дуже близькі до заданих 170 і 8.

Для нормального розподілу діє **правило трьох сигм**: приблизно 68% значень лежать у межах одного стандартного відхилення від середнього, 95% — двох, 99.7% — трьох.

```python
import numpy as np

rng = np.random.default_rng(4)
height = rng.normal(loc=170, scale=8, size=10_000)

z = np.abs(height - 170) / 8
for k in (1, 2, 3):
    share = np.mean(z <= k)
    print(f"within {k} sigma: {share:.3f}")
```

```text
within 1 sigma: 0.691
within 2 sigma: 0.956
within 3 sigma: 0.997
```

`np.mean` від булевого масиву — частка `True`: `True` рахується як 1, `False` — як 0. Це зручний спосіб отримати відсоток елементів, що задовольняють умову.

Подивимося на форму розподілу за допомогою гістограми в терміналі:

```python
import numpy as np

rng = np.random.default_rng(4)
height = rng.normal(loc=170, scale=8, size=10_000)

counts, edges = np.histogram(height, bins=range(146, 196, 4))
for left, count in zip(edges, counts):
    print(f"{left:5.0f} | {'#' * (count // 50)}")
```

```text
  146 | 
  150 | ###
  154 | ########
  158 | #################
  162 | ##############################
  166 | #####################################
  170 | ######################################
  174 | ##############################
  178 | ##################
  182 | #########
  186 | ###
  190 | 
```

!!! note "Інші розподіли"
    Генератор уміє більше: `rng.binomial` (кількість успіхів у серії спроб), `rng.poisson` (кількість подій за інтервал — дзвінки в колл-центр за годину), `rng.exponential` (час між подіями). Повний список — у документації за посиланням наприкінці лекції.

## Вибір і перемішування: `choice`, `shuffle`, `permutation`

`rng.choice` обирає випадкові елементи з масиву:

```python
import numpy as np

rng = np.random.default_rng(5)

students = np.array(["Anna", "Bohdan", "Iryna", "Oleh", "Taras", "Yulia"])

# один студент до дошки
print(rng.choice(students))

# троє різних студентів (без повторень)
print(rng.choice(students, size=3, replace=False))

# 8 виборів з повтореннями
print(rng.choice(students, size=8))
```

```text
Taras
['Oleh' 'Taras' 'Anna']
['Oleh' 'Bohdan' 'Yulia' 'Anna' 'Bohdan' 'Iryna' 'Oleh' 'Iryna']
```

- `replace=True` (за замовчуванням) — вибір **з поверненням**, елементи можуть повторюватися;
- `replace=False` — **без повернення**, кожен елемент не більше одного разу; `size` тоді не може перевищувати довжину масиву.

Параметр `p` задає **ймовірності** — наприклад, «нечесна» монета або погода:

```python
import numpy as np

rng = np.random.default_rng(6)

weather = rng.choice(["sun", "cloud", "rain"], size=1000, p=[0.6, 0.3, 0.1])

values, counts = np.unique(weather, return_counts=True)
for value, count in zip(values, counts):
    print(value, count)
```

```text
cloud 302
rain 98
sun 600
```

Ймовірності в `p` мають у сумі давати рівно 1, інакше буде `ValueError`.

Для перемішування є дві функції:

```python
import numpy as np

rng = np.random.default_rng(7)

questions = np.arange(1, 11)

# permutation повертає перемішану копію
order = rng.permutation(questions)
print("copy:    ", order)
print("original:", questions)

# shuffle перемішує на місці і нічого не повертає
rng.shuffle(questions)
print("shuffled:", questions)
```

```text
copy:     [ 9  1  8  2  4  7  3  5  6 10]
original: [ 1  2  3  4  5  6  7  8  9 10]
shuffled: [ 5  3  9  4 10  2  6  7  1  8]
```

Для 2D-масиву обидві функції перемішують **рядки**, не змішуючи дані всередині рядка. Це саме те, що потрібно для таблиці, де рядок — один запис.

Типове застосування — випадковий поділ даних на дві частини (наприклад, 80% для навчання моделі і 20% для перевірки):

```python
import numpy as np

rng = np.random.default_rng(8)

data = np.arange(100, 120)

index = rng.permutation(data.size)
split = int(data.size * 0.8)

train = data[index[:split]]
test = data[index[split:]]

print("train:", train.size, "test:", test.size)
print("test:", test)
```

```text
train: 16 test: 4
test: [101 106 105 102]
```

Перемішуються не самі дані, а **індекси** — так можна однаково переставити кілька пов'язаних масивів (наприклад, ознаки й відповіді).

## Симуляція: кидки кубика

**Симуляція** — моделювання випадкового процесу багато разів, щоб оцінити ймовірності чи середні значення. З NumPy не потрібен цикл: генеруємо всі кидки одразу.

Кинемо два кубики 100 000 разів і подивимося на розподіл суми:

```python
import numpy as np

rng = np.random.default_rng(10)

rolls = rng.integers(1, 7, size=(100_000, 2))
total = rolls.sum(axis=1)

counts = np.bincount(total, minlength=13)[2:]
share = counts / total.size

for value, part in zip(range(2, 13), share):
    print(f"{value:2d} {part:.3f} {'#' * int(part * 200)}")

print("P(sum = 7):", round(np.mean(total == 7), 4))
print("exact 6/36:", round(6 / 36, 4))
```

```text
 2 0.028 #####
 3 0.057 ###########
 4 0.083 ################
 5 0.111 ######################
 6 0.140 ############################
 7 0.166 #################################
 8 0.139 ###########################
 9 0.111 ######################
10 0.083 ################
11 0.055 ##########
12 0.028 #####
P(sum = 7): 0.1659
exact 6/36: 0.1667
```

Масив `(100_000, 2)` — це 100 000 рядків по два кидки; `sum(axis=1)` дає суму для кожного рядка. Сума 7 найімовірніша, бо її можна отримати шістьма способами (1+6, 2+5, ..., 6+1), а 2 і 12 — лише одним.

**Закон великих чисел**: що більше кидків, то ближче частота до справжньої ймовірності. Перевіримо на частоті шістки:

```python
import numpy as np

rng = np.random.default_rng(11)

for n in (10, 100, 1_000, 10_000, 1_000_000):
    rolls = rng.integers(1, 7, size=n)
    share = np.mean(rolls == 6)
    print(f"{n:>9} rolls: {share:.4f}")

print(f"    exact: {1 / 6:.4f}")
```

```text
       10 rolls: 0.0000
      100 rolls: 0.2500
     1000 rolls: 0.1620
    10000 rolls: 0.1641
  1000000 rolls: 0.1667
    exact: 0.1667
```

На 10 кидках частота може бути будь-якою, на мільйоні — збігається з \(1/6\) до третього знака.

Середнє поточної частоти по ходу експерименту зручно рахувати через `cumsum`:

```python
import numpy as np

rng = np.random.default_rng(12)

rolls = rng.integers(1, 7, size=10_000)
running_mean = np.cumsum(rolls) / np.arange(1, rolls.size + 1)

for step in (1, 10, 100, 1_000, 10_000):
    print(f"after {step:>6}: {running_mean[step - 1]:.3f}")
```

```text
after      1: 4.000
after     10: 3.200
after    100: 3.710
after   1000: 3.541
after  10000: 3.509
```

Середнє значення кубика прямує до \((1 + 2 + \dots + 6) / 6 = 3.5\).

## Метод Монте-Карло

**Метод Монте-Карло** — оцінка величини через велику кількість випадкових експериментів. Класичний приклад — наближення числа \(\pi\).

Розкидаємо точки рівномірно в квадраті \([-1, 1] \times [-1, 1]\). Частка точок, що потрапили в коло радіуса 1, дорівнює відношенню площ:

$$
\frac{\text{площа кола}}{\text{площа квадрата}} = \frac{\pi \cdot 1^2}{2 \cdot 2} = \frac{\pi}{4}
$$

```python
import numpy as np

rng = np.random.default_rng(13)

n = 1_000_000
points = rng.uniform(-1, 1, size=(n, 2))

inside = np.sum(points ** 2, axis=1) <= 1
pi_estimate = 4 * inside.mean()

print("estimate:", pi_estimate)
print("error:   ", round(abs(pi_estimate - np.pi), 5))
```

```text
estimate: 3.139732
error:    0.00186
```

Той самий підхід працює для задач, які формулою не розв'язати: ризик інвестиційного портфеля, надійність системи з багатьох компонентів, час обслуговування в черзі.

Приклад прикладної задачі: у групі 25 студентів. Яка ймовірність, що принаймні двоє мають день народження в один день? (Відомий «парадокс днів народження».)

```python
import numpy as np

rng = np.random.default_rng(14)

trials = 100_000
group = 25

birthdays = rng.integers(0, 365, size=(trials, group))
birthdays.sort(axis=1)

# у відсортованому рядку однакові дні стоять поруч
has_pair = np.any(np.diff(birthdays, axis=1) == 0, axis=1)
print("probability:", round(has_pair.mean(), 3))
```

```text
probability: 0.569
```

Понад 50% — набагато більше, ніж підказує інтуїція. Кожен рядок масиву — одна «група»; `np.diff` по рядку дає нуль там, де два сусідні дні однакові, а `np.any(..., axis=1)` перевіряє, чи є такий нуль у кожній групі.

## Приклад: аналіз оцінок групи

Зберемо все разом. Згенеруємо оцінки 30 студентів з 4 предметів і проаналізуємо їх.

```python
import numpy as np

rng = np.random.default_rng(2026)

subjects = np.array(["math", "physics", "python", "english"])
students = 30

# кожен предмет має свою складність: різне середнє і розкид
means = np.array([70, 62, 78, 74])
stds = np.array([12, 15, 10, 8])

scores = rng.normal(loc=means, scale=stds, size=(students, subjects.size))
scores = np.clip(np.round(scores), 0, 100).astype(int)

print("shape:", scores.shape)
print("first 3 students:")
print(scores[:3])

print("\nsubject   mean  median   std  min  max")
for i, name in enumerate(subjects):
    column = scores[:, i]
    print(
        f"{name:8} {column.mean():5.1f} {np.median(column):7.1f} "
        f"{column.std():5.1f} {column.min():4d} {column.max():4d}"
    )

average = scores.mean(axis=1)
print("\nbest student:", average.argmax(), "avg:", round(average.max(), 2))
print("failed (avg < 60):", np.sum(average < 60))
print("top 10% threshold:", round(np.percentile(average, 90), 2))

counts, _ = np.histogram(average, bins=[0, 60, 75, 90, 101])
print("F/C/B/A:", counts)

print("corr math-physics:", round(np.corrcoef(scores[:, 0], scores[:, 1])[0, 1], 3))
```

```text
shape: (30, 4)
first 3 students:
[[60 66 59 85]
 [78 58 75 76]
 [67 59 85 78]]

subject   mean  median   std  min  max
math      68.9    67.0  12.7   47  100
physics   64.9    65.0  13.1   43  100
python    77.7    77.5   9.8   59  100
english   74.8    76.0   8.5   56   90

best student: 12 avg: 93.75
failed (avg < 60): 1
top 10% threshold: 79.25
F/C/B/A: [ 1 23  5  1]
corr math-physics: 0.222
```

Що тут нового:

- `rng.normal(loc=means, scale=stds, size=(30, 4))` — параметри-масиви форми `(4,)` поширюються (broadcasting) на кожен рядок, тож кожен стовпець отримує своє середнє і розкид;
- **`np.clip(a, 0, 100)`** обрізає значення до діапазону: усе менше за 0 стає 0, більше за 100 — 100. Нормальний розподіл теоретично може видати і 110, і −5, тому для оцінок обрізання обов'язкове;
- `astype(int)` після `round` — оцінки цілі.

Кореляція математики і фізики невелика (0.22), хоча ми генерували предмети незалежно один від одного. На 30 студентах така «випадкова» кореляція — звичайна справа: на малих вибірках статистики помітно «гуляють». У реальних даних зв'язок між математикою і фізикою був би сильнішим.

## Старий API `np.random.*`

У підручниках і на Stack Overflow часто зустрічається інший стиль:

```python
import numpy as np

np.random.seed(42)
print(np.random.rand(3))
print(np.random.randint(1, 7, size=5))
```

Це **застарілий** (legacy) інтерфейс. Він працює, але:

- використовує один **глобальний** стан на всю програму — будь-яка бібліотека, що викликала `np.random.seed`, змінює поведінку вашого коду;
- базується на старішому алгоритмі (Mersenne Twister) замість сучаснішого PCG64;
- нові функції додаються лише до `Generator`.

Відповідність старих і нових викликів:

| Старий виклик | Новий виклик |
|---|---|
| `np.random.seed(42)` | `rng = np.random.default_rng(42)` |
| `np.random.rand(3)` | `rng.random(3)` |
| `np.random.randint(1, 7, 5)` | `rng.integers(1, 7, 5)` |
| `np.random.randn(3)` | `rng.standard_normal(3)` |
| `np.random.normal(0, 1, 3)` | `rng.normal(0, 1, 3)` |
| `np.random.choice(a, 3)` | `rng.choice(a, 3)` |
| `np.random.shuffle(a)` | `rng.shuffle(a)` |

У новому коді використовуйте `default_rng`.

## Типові помилки

**`array.median()`.** Такого методу немає — лише функція `np.median(array)`. Те саме стосується `percentile` і `quantile`.

**Середнє там, де потрібна медіана.** Один викид зсуває середнє як завгодно далеко. Для «типового значення» даних із викидами — медіана.

**Різні `std` у NumPy і pandas.** NumPy за замовчуванням ділить на \(n\) (`ddof=0`), pandas — на \(n - 1\). Для вибірки явно пишіть `ddof=1`.

**Відсотки в `quantile`.** `np.quantile(a, 95)` — помилка: квантиль приймає частки від 0 до 1. Відсотки — це `np.percentile(a, 95)`.

**`nan` у даних.** `mean`, `std`, `max` повертають `nan`, якщо є хоча б один пропуск. Використовуйте `nanmean` тощо або прибирайте пропуски через `np.isnan`.

**`x == np.nan`.** Завжди `False`. Лише `np.isnan(x)`.

**`integers(1, 6)` для кубика.** Верхня межа не входить — шістки не буде ніколи.

**Новий генератор у циклі з тим самим зерном.** `default_rng(42)` усередині циклу щоразу починає послідовність спочатку — усі «випадкові» результати будуть однаковими. Генератор створюють один раз, до циклу.

**Цикл Python замість векторизації.** `[rng.integers(1, 7) for _ in range(1_000_000)]` у сотні разів повільніше за `rng.integers(1, 7, size=1_000_000)`.

**`shuffle` повертає `None`.** `a = rng.shuffle(a)` перетворить `a` на `None`. Або `rng.shuffle(a)` без присвоєння, або `a = rng.permutation(a)`.

**Нормальний розподіл без обмежень.** Згенеровані «оцінки» можуть вийти за 0–100, «зріст» — стати від'ємним. Обрізайте через `np.clip`.

**`np.random` для паролів.** Для всього, що стосується безпеки, — модуль `secrets`.

## Підсумок

- **`mean`** — середнє, **`np.median`** — медіана; медіана стійка до викидів.
- **Моду** шукають через `np.unique(..., return_counts=True)` або `np.bincount` для невід'ємних цілих.
- **`np.ptp`** — розмах, **`var`** — дисперсія, **`std`** — стандартне відхилення; для вибірки — `ddof=1`.
- **`np.percentile(a, 95)`** і **`np.quantile(a, 0.95)`** — одне й те саме в різних одиницях.
- **IQR** \(= Q3 - Q1\); значення поза \([Q1 - 1.5 \cdot IQR,\ Q3 + 1.5 \cdot IQR]\) — викиди.
- Усі статистичні функції приймають **`axis`**.
- **z-оцінка** `(x - mean) / std` робить різні шкали порівнянними.
- **`np.average(a, weights=w)`** — зважене середнє.
- **`cumsum`**, **`cumprod`**, **`diff`** — накопичувальні суми, добутки та різниці.
- **`np.histogram`** рахує значення в інтервалах, **`np.bincount`** — для цілих.
- **`np.corrcoef`** повертає матрицю кореляцій; потрібне число — `[0, 1]`.
- **`nan`-функції** (`nanmean`, `nanstd`, ...) ігнорують пропуски; пропуски шукають через **`np.isnan`**.
- Випадкові числа дає **`rng = np.random.default_rng(seed)`**; зерно робить результат відтворюваним.
- **`rng.random`**, **`rng.uniform`**, **`rng.integers`** — рівномірні числа; у `integers` верхня межа не входить.
- **`rng.normal(loc, scale, size)`** — нормальний розподіл; правило 68–95–99.7.
- **`rng.choice`** (з `replace` і `p`), **`rng.permutation`** (копія), **`rng.shuffle`** (на місці).
- Симуляції й метод Монте-Карло — це генерація всіх експериментів одним масивом і статистика по ньому, без циклів.

## Корисні посилання

- [NumPy: statistics](https://numpy.org/doc/stable/reference/routines.statistics.html)
- [NumPy: random sampling](https://numpy.org/doc/stable/reference/random/index.html)
- [`numpy.random.Generator`](https://numpy.org/doc/stable/reference/random/generator.html)
- [`np.percentile`](https://numpy.org/doc/stable/reference/generated/numpy.percentile.html)
- [`np.histogram`](https://numpy.org/doc/stable/reference/generated/numpy.histogram.html)
- [`np.corrcoef`](https://numpy.org/doc/stable/reference/generated/numpy.corrcoef.html)
- [Модуль `secrets`](https://docs.python.org/3/library/secrets.html)
