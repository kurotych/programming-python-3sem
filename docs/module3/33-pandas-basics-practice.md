# 33. (П15) Практичні завдання з pandas: базові операції, фільтрація

## Передумови

- Прочитана [Лекція 32 — Вступ до pandas. Series, DataFrame, читання CSV](/ua/courses/programming-3sem/module3/32-pandas-intro-lecture/)
- Встановлений pandas: `pip install pandas`

!!! warning "Мова в коді"
    Усі рядкові літерали, вивід `print()`, імена файлів та змінних — **лише латиницею**. Кирилиця допускається тільки в коментарях.

## Завдання

Написати скрипт `pandas_basics.py`, у якому кожне завдання нижче виводить свій результат. Перед кожним завданням вивести заголовок, наприклад `--- Task 3 ---`, а в завданні 5 — заголовок на кожну частину: `--- Task 5.1 ---`, `--- Task 5.2 ---` і так далі.

Цикли `for` / `while` та `iterrows()` для обчислень **не використовувати** — лише операції над стовпцями, маски та методи pandas.

## Персоналізація

```python
STUDENT_NAME = "Ivan Petrenko"   # ваше ім'я та прізвище транслітерацією
STUDENT_GROUP = "KI-31"          # ваша група
N = 7                            # ваш номер у списку групи + 5
```

Перший рядок виводу: `Ivan Petrenko, KI-31, N = 7`.

Усі приклади виводу нижче наведені для `N = 7`. У вас частина чисел буде іншою — це нормально.

## Завдання 1. `Series`: створення і доступ

Створити `Series` з назвою `pages` зі словника (кількість сторінок у книгах):

```python
pages = pd.Series(
    {"Dune": 412, "Neuromancer": 271, "Hyperion": 482, "Solaris": 204, "Foundation": 255},
    name="pages",
)
```

Вивести, кожне з нового рядка:

1. саму серію;
2. значення за міткою `"Hyperion"`;
3. перший елемент **за позицією** (`.iloc`);
4. зріз за мітками від `"Neuromancer"` до `"Solaris"` — як список (`.tolist()`);
5. два елементи за списком міток `["Dune", "Foundation"]` — як список;
6. в одному рядку: середнє, максимум і **мітку** максимуму (`idxmax`);
7. маску «сторінок більше за `50 * N`»;
8. самі значення, що потрапили в маску — як список.

```text
Dune           412
Neuromancer    271
Hyperion       482
Solaris        204
Foundation     255
Name: pages, dtype: int64
482
412
[271, 482, 204]
[412, 255]
324.8 482 Hyperion
Dune            True
Neuromancer    False
Hyperion        True
Solaris        False
Foundation     False
Name: pages, dtype: bool
[412, 482]
```

!!! tip "Зріз за мітками включає праву межу"
    `pages.loc["Neuromancer":"Solaris"]` дає **три** елементи, бо `"Solaris"` входить у результат.

## Завдання 2. Вирівнювання за мітками і `NaN`

Створити дві серії — продажі за березень і за квітень. Зверніть увагу: набори міток **різні**, і порядок ключів теж.

```python
march = pd.Series({"Dune": 12.0, "Hyperion": 5.0, "Solaris": 8.0})
april = pd.Series({"Hyperion": 7.0, "Dune": 10.0, "Ubik": 4.0})
```

Вивести:

1. `march + april` — і в коментарі одним реченням пояснити, звідки взялися `NaN`;
2. **кількість** пропусків у сумі (`isna().sum()`);
3. суму без пропусків (`dropna()`);
4. суму, у якій відсутня мітка вважається нулем (`add` з `fill_value`).

```text
Dune        22.0
Hyperion    12.0
Solaris      NaN
Ubik         NaN
dtype: float64
2
Dune        22.0
Hyperion    12.0
dtype: float64
Dune        22.0
Hyperion    12.0
Solaris      8.0
Ubik         4.0
dtype: float64
```

## Завдання 3. `DataFrame` і огляд таблиці

Створити таблицю `books` зі словника:

```python
books = pd.DataFrame({
    "title": ["Dune", "Neuromancer", "Hyperion", "Solaris", "Foundation", "Ubik"],
    "genre": ["sci-fi", "cyberpunk", "sci-fi", "classic", "classic", "cyberpunk"],
    "year": [1965, 1984, 1989, 1961, 1951, 1969],
    "pages": [412, 271, 482, 204, 255, 224],
    "price": [320.0, 275.5, 410.0, 189.9, 240.0, 295.0],
})
```

Вивести:

1. усю таблицю;
2. `shape` і список назв стовпців;
3. `dtypes`;
4. результат `info()` (пам'ятайте: `info()` друкує сам, `print(df.info())` писати не треба);
5. `describe()`;
6. перші 2 рядки та останні 2 рядки.

```text
         title      genre  year  pages  price
0         Dune     sci-fi  1965    412  320.0
1  Neuromancer  cyberpunk  1984    271  275.5
2     Hyperion     sci-fi  1989    482  410.0
3      Solaris    classic  1961    204  189.9
4   Foundation    classic  1951    255  240.0
5         Ubik  cyberpunk  1969    224  295.0
shape: (6, 5)
columns: ['title', 'genre', 'year', 'pages', 'price']
title        str
genre        str
year       int64
pages      int64
price    float64
dtype: object
```

## Завдання 4. Вибір стовпців і рядків

Працюємо з тією самою таблицею `books`. Вивести:

1. `type(books["price"])` і `type(books[["price"]])` — двома рядками;
2. таблицю з двох стовпців: `title` і `price`;
3. рядок з міткою `2` (через `.loc`);
4. **перший** рядок за позицією (через `.iloc`);
5. рядки з мітками від `1` до `3` і лише стовпці `title`, `pages`;
6. перші 3 рядки та перші 2 стовпці **за позиціями**;
7. в одному рядку: ціну книги з міткою `4` — спочатку через `.loc`, потім через `.iloc`.

```text
<class 'pandas.Series'>
<class 'pandas.DataFrame'>
         title  price
0         Dune  320.0
1  Neuromancer  275.5
2     Hyperion  410.0
3      Solaris  189.9
4   Foundation  240.0
5         Ubik  295.0
title    Hyperion
genre      sci-fi
year         1989
pages         482
price       410.0
Name: 2, dtype: object
```

!!! warning "`.loc` і `.iloc` дають різну кількість рядків"
    `books.loc[1:3]` поверне **три** рядки (1, 2, 3), а `books.iloc[1:3]` — **два**.

## Завдання 5. Замовлення книгарні

Підсумкове завдання на готових даних: **прочитати → оглянути → почистити → порахувати → зберегти**.

Створіть поруч зі скриптом файл `orders.csv` із таким вмістом:

```text
# export from bookshop-export v3
date;code;title;genre;copies;price
2026-03-01;0042;Dune;sci-fi;3;320,00
2026-03-01;0007;Ubik;cyberpunk;;295,00
2026-03-02;0113;Hyperion;sci-fi;2;410,00
2026-03-02;0042;Dune;sci-fi;5;320,00
2026-03-03;0085;Solaris;classic;4;189,90
2026-03-04;0007;Ubik;cyberpunk;6;295,00
2026-03-05;0261;Neuromancer;cyberpunk;-;275,50
2026-03-05;0330;Foundation;classic;7;240,00
2026-03-06;0113;Hyperion;sci-fi;4;410,00
2026-03-07;0042;Dune;sci-fi;1;320,00
```

Файл навмисно «брудний»: службовий рядок на початку, роздільник `;`, десяткова кома, артикули з провідними нулями, порожня клітинка та `-` замість кількості.

Далі — сім кроків. Перед кожним вивести заголовок `--- Task 5.1 ---` і так далі.

**5.1.** Прочитати файл **одним** викликом `read_csv`, врахувавши все одразу: службовий рядок (`comment="#"`), роздільник, десяткову кому, позначку пропуску `-`, стовпець `code` як **рядок** (провідні нулі!) та `date` як **дату**. Вивести таблицю і `dtypes`.

```text
        date  code        title      genre  copies  price
0 2026-03-01  0042         Dune     sci-fi     3.0  320.0
1 2026-03-01  0007         Ubik  cyberpunk     NaN  295.0
2 2026-03-02  0113     Hyperion     sci-fi     2.0  410.0
3 2026-03-02  0042         Dune     sci-fi     5.0  320.0
4 2026-03-03  0085      Solaris    classic     4.0  189.9
5 2026-03-04  0007         Ubik  cyberpunk     6.0  295.0
6 2026-03-05  0261  Neuromancer  cyberpunk     NaN  275.5
7 2026-03-05  0330   Foundation    classic     7.0  240.0
8 2026-03-06  0113     Hyperion     sci-fi     4.0  410.0
9 2026-03-07  0042         Dune     sci-fi     1.0  320.0
date      datetime64[us]
code                 str
title                str
genre                str
copies           float64
price            float64
dtype: object
```

!!! tip "Чому `code` треба читати як `str`"
    Без `dtype={"code": "str"}` артикул `0042` стане числом `42` — провідні нулі зникнуть назавжди.

**5.2.** Вивести `shape` і кількість пропусків у кожному стовпці (`isna().sum()`).

```text
shape: (10, 6)
date      0
code      0
title     0
genre     0
copies    2
price     0
dtype: int64
```

**5.3.** Замінити пропуски в `copies` на `0` і повернути стовпцю **цілий** тип (`fillna` + `astype("int64")`). Додати стовпець `revenue` = `copies * price`. Вивести таблицю.

```text
        date  code        title      genre  copies  price  revenue
0 2026-03-01  0042         Dune     sci-fi       3  320.0    960.0
1 2026-03-01  0007         Ubik  cyberpunk       0  295.0      0.0
2 2026-03-02  0113     Hyperion     sci-fi       2  410.0    820.0
3 2026-03-02  0042         Dune     sci-fi       5  320.0   1600.0
4 2026-03-03  0085      Solaris    classic       4  189.9    759.6
5 2026-03-04  0007         Ubik  cyberpunk       6  295.0   1770.0
6 2026-03-05  0261  Neuromancer  cyberpunk       0  275.5      0.0
7 2026-03-05  0330   Foundation    classic       7  240.0   1680.0
8 2026-03-06  0113     Hyperion     sci-fi       4  410.0   1640.0
9 2026-03-07  0042         Dune     sci-fi       1  320.0    320.0
```

!!! warning "`fillna` повертає копію"
    `orders["copies"].fillna(0)` **не змінює** таблицю — результат треба присвоїти назад: `orders["copies"] = orders["copies"].fillna(0)`.

**5.4.** Вивести загальний дохід і топ-3 замовлення за доходом — лише стовпці `date`, `title`, `revenue`.

```text
total revenue: 9549.6
        date       title  revenue
5 2026-03-04        Ubik   1770.0
7 2026-03-05  Foundation   1680.0
8 2026-03-06    Hyperion   1640.0
```

**5.5.** Вивести замовлення жанру `sci-fi` з доходом більшим за `100 * N` — дві умови через `&` і дужки. Спочатку вивести **кількість** таких замовлень (через `.sum()` від маски, без циклу), потім самі рядки — стовпці `date`, `title`, `revenue`.

```text
4
        date     title  revenue
0 2026-03-01      Dune    960.0
2 2026-03-02  Hyperion    820.0
3 2026-03-02      Dune   1600.0
8 2026-03-06  Hyperion   1640.0
```


**5.6.** Вивести, скільки замовлень припало на кожен жанр (`value_counts`).

```text
genre
sci-fi       5
cyberpunk    3
classic      2
Name: count, dtype: int64
```

