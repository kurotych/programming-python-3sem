# 29. (Л) Базова лінійна алгебра у NumPy. Матричні операції, системи лінійних рівнянь

## Зміст лекції

1. Навіщо це програмісту
2. Вектор і скалярний добуток
3. Норма вектора та відстань
4. Матриця: форма і транспонування
5. `*` і `@` — дві різні операції
6. Матричний добуток: правило розмірів
7. Матриця на вектор
8. Одинична матриця
9. Визначник
10. Обернена матриця
11. Системи лінійних рівнянь
12. `solve` проти `inv`
13. Коли розв'язку немає: вироджена матриця
14. Ранг матриці
15. Приклад: розрахунок суміші
16. Перевизначені системи та `lstsq`
17. Власні числа: коротко
18. Типові помилки
19. Підсумок

## Навіщо це програмісту

Лінійна алгебра — не окрема «математична» тема, а мова, якою описують багато прикладних задач:

- **комп'ютерна графіка**: поворот, масштабування, зсув — усе це множення координат на матрицю;
- **обробка зображень**: фільтри, перетворення кольорових просторів;
- **машинне навчання**: один шар нейромережі — це `inputs @ weights + bias`;
- **економіка й інженерія**: система рівнянь на кшталт «скільки чого змішати, щоб отримати заданий склад»;
- **геометрія й фізика**: відстані, кути, проєкції.

NumPy покриває цю тему модулем `numpy.linalg` (від *linear algebra*) плюс оператором `@` для множення матриць. Далі — базовий набір, якого вистачає для більшості прикладних задач.

## Вектор і скалярний добуток

Вектор у NumPy — звичайний одновимірний масив. **Скалярний добуток** (dot product) двох векторів однакової довжини — це сума попарних добутків:

$$
a \cdot b = a_0 b_0 + a_1 b_1 + \dots + a_{n-1} b_{n-1}
$$

```python
import numpy as np

prices = np.array([25.0, 40.0, 15.0, 8.0])
amounts = np.array([2.0, 1.0, 3.0, 10.0])

# три записи однієї операції
print(np.sum(prices * amounts))
print(np.dot(prices, amounts))
print(prices @ amounts)
```

```text
215.0
215.0
215.0
```

Усі три рядки дають те саме: загальну вартість покупки. Різниця в тому, що `prices * amounts` спершу створює проміжний масив із чотирьох чисел, а `@` і `np.dot` рахують суму одразу, без зайвої пам'яті.

!!! note "Оператор `@`"
    `@` — це окремий оператор Python для матричного множення (PEP 465). Для двох одновимірних масивів він дає число, для двовимірних — добуток матриць. Саме він, а не `*`, відповідає множенню з підручника лінійної алгебри.

Довжини мають збігатися:

```python
import numpy as np

first = np.array([1.0, 2.0, 3.0])
second = np.array([1.0, 2.0])

try:
    first @ second
except ValueError as error:
    print("ValueError:", error)
```

```text
ValueError: matmul: Input operand 1 has a mismatch in its core dimension 0, with gufunc signature (n?,k),(k,m?)->(n?,m?) (size 2 is different from 3)
```

## Норма вектора та відстань

**Норма** — довжина вектора, корінь із суми квадратів координат. Рахує її `np.linalg.norm`:

$$
\|a\| = \sqrt{a_0^2 + a_1^2 + \dots + a_{n-1}^2}
$$

```python
import numpy as np

point = np.array([3.0, 4.0])

print(np.linalg.norm(point))
print(np.sqrt(np.sum(point ** 2)))

first = np.array([1.0, 2.0, 3.0])
second = np.array([4.0, 6.0, 3.0])

# евклідова відстань між двома точками
print(np.linalg.norm(first - second))
```

```text
5.0
5.0
5.0
```

Норма працює і з набором точок одразу — треба лише вказати вісь:

```python
import numpy as np

# рядок — точка, стовпці — координати x, y
points = np.array([
    [0.0, 0.0],
    [3.0, 4.0],
    [6.0, 8.0],
    [1.0, 1.0],
])
target = np.array([3.0, 4.0])

distances = np.linalg.norm(points - target, axis=1)
print(np.round(distances, 3))
print("nearest index:", distances.argsort()[1])
```

```text
[5.    0.    5.    3.606]
nearest index: 3
```

Тут працює broadcasting із минулої лекції: `(4, 2) - (2,)` віднімає `target` від кожного рядка, а `axis=1` каже рахувати норму по кожному рядку окремо. Одна з найчастіших задач — «знайти найближчий об'єкт» — вирішується двома рядками.

!!! tip "Кут між векторами"
    Через скалярний добуток і норми рахується косинус кута: `a @ b / (norm(a) * norm(b))`. Ця формула — основа «схожості» текстів і рекомендаційних систем.

## Матриця: форма і транспонування

Матриця — двовимірний масив. Її форма записується як `(рядки, стовпці)`.

**Транспонування** міняє рядки і стовпці місцями: елемент `a[i, j]` стає `a[j, i]`. Атрибут `.T`:

```python
import numpy as np

matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])

print(matrix)
print(matrix.shape)
print(matrix.T)
print(matrix.T.shape)
```

```text
[[1 2 3]
 [4 5 6]]
(2, 3)
[[1 4]
 [2 5]
 [3 6]]
(3, 2)
```

```text
       матриця (2, 3)              транспонована (3, 2)
      +---+---+---+                    +---+---+
      | 1 | 2 | 3 |                    | 1 | 4 |
      +---+---+---+       .T           +---+---+
      | 4 | 5 | 6 |       ->           | 2 | 5 |
      +---+---+---+                    +---+---+
                                       | 3 | 6 |
                                       +---+---+
```

`.T` повертає **перегляд** (view), а не копію — даних у пам'яті не додається:

```python
import numpy as np

matrix = np.array([
    [1, 2, 3],
    [4, 5, 6],
])
transposed = matrix.T

transposed[0, 0] = 99
print(matrix)
print(transposed.base is matrix)
```

```text
[[99  2  3]
 [ 4  5  6]]
True
```

!!! warning "Транспонування одновимірного масиву нічого не робить"
    У масиву форми `(3,)` немає другої осі, тому `.T` повертає його ж. Щоб отримати «стовпець», потрібен `reshape(-1, 1)` або `[:, np.newaxis]`.

```python
import numpy as np

vector = np.array([1, 2, 3])

print(vector.shape, vector.T.shape)
print(vector[:, np.newaxis])
print(vector[:, np.newaxis].shape)
```

```text
(3,) (3,)
[[1]
 [2]
 [3]]
(3, 1)
```

## `*` і `@` — дві різні операції

Це найчастіше джерело помилок у новачків. `*` — **поелементне** множення (те саме, що `+` чи `-`), `@` — **матричне**.

```python
import numpy as np

a = np.array([
    [1, 2],
    [3, 4],
])
b = np.array([
    [5, 6],
    [7, 8],
])

print(a * b)
print(a @ b)
```

```text
[[ 5 12]
 [21 32]]
[[19 22]
 [43 50]]
```

Поелементно: `1*5 = 5`, `2*6 = 12`, і так далі. Матрично — кожен елемент результату є скалярним добутком **рядка** першої матриці на **стовпець** другої:

$$
c_{ij} = \sum_{k} a_{ik} b_{kj}
$$


```text
c[0, 0] = 1*5 + 2*7 = 19
c[0, 1] = 1*6 + 2*8 = 22
c[1, 0] = 3*5 + 4*7 = 43
c[1, 1] = 3*6 + 4*8 = 50
```

!!! danger "Квадратні матриці приховують помилку"
    Для матриць `2x2` обидві операції проходять без помилки — просто дають різні числа. Якщо переплутати `*` і `@` на квадратних даних, програма не впаде, а тихо порахує не те.

## Матричний добуток: правило розмірів

Щоб перемножити `A @ B`, **внутрішні розміри мають збігатися**:

```text
   A          B          A @ B
(m, k)  @  (k, n)   ->   (m, n)
    ^        ^
    +--------+
   мають збігатися
```

```python
import numpy as np

a = np.arange(6).reshape(2, 3)
b = np.arange(12).reshape(3, 4)

result = a @ b
print(a.shape, "@", b.shape, "->", result.shape)
print(result)

try:
    b @ a
except ValueError as error:
    print("ValueError:", error)
```

```text
(2, 3) @ (3, 4) -> (2, 4)
[[20 23 26 29]
 [56 68 80 92]]
ValueError: matmul: Input operand 1 has a mismatch in its core dimension 0, with gufunc signature (n?,k),(k,m?)->(n?,m?) (size 2 is different from 4)
```

Звідси головна властивість: **матричний добуток не комутативний**, `A @ B` і `B @ A` — різні речі (а часто друге взагалі не існує).

```python
import numpy as np

a = np.array([
    [1, 2],
    [0, 1],
])
b = np.array([
    [1, 0],
    [3, 1],
])

print(a @ b)
print(b @ a)
print(np.array_equal(a @ b, b @ a))
```

```text
[[7 2]
 [3 1]]
[[1 2]
 [3 7]]
False
```

Ще одна корисна властивість — транспонування добутку перевертає порядок: \((AB)^T = B^T A^T\).

```python
import numpy as np

a = np.arange(6).reshape(2, 3)
b = np.arange(12).reshape(3, 4)

print(np.array_equal((a @ b).T, b.T @ a.T))
```

```text
True
```

## Матриця на вектор

Найчастіший випадок на практиці: матриця `(m, n)` множиться на вектор `(n,)` і дає вектор `(m,)`. NumPy розуміє одновимірний масив як стовпець і повертає результат теж одновимірним.

```python
import numpy as np

# рядок — рецепт напою, стовпець — інгредієнт
recipes = np.array([
    [50.0, 30.0, 20.0],
    [0.0, 80.0, 20.0],
    [70.0, 0.0, 30.0],
])
# ціна за одиницю кожного інгредієнта
prices = np.array([0.4, 0.25, 0.1])

cost = recipes @ prices
print(np.round(cost, 2))
print(recipes.shape, "@", prices.shape, "->", cost.shape)
```

```text
[29.5 22.  31. ]
(3, 3) @ (3,) -> (3,)
```

Той самий результат через явні цикли зайняв би вісім рядків і працював би в десятки разів повільніше.

Класичний приклад із графіки — поворот точки навколо початку координат на кут \(\theta\):

$$
R = \begin{pmatrix}
\cos\theta & -\sin\theta \\
\sin\theta & \cos\theta
\end{pmatrix}
\qquad
R \begin{pmatrix} x \\ y \end{pmatrix} =
\begin{pmatrix} x\cos\theta - y\sin\theta \\ x\sin\theta + y\cos\theta \end{pmatrix}
$$

```python
import numpy as np

angle = np.pi / 2  # 90 градусів у радіанах
rotation = np.array([
    [np.cos(angle), -np.sin(angle)],
    [np.sin(angle), np.cos(angle)],
])

point = np.array([1.0, 0.0])
print(np.round(rotation @ point, 6))

# кілька точок одразу: (n, 2) @ (2, 2).T
points = np.array([
    [1.0, 0.0],
    [0.0, 1.0],
    [2.0, 3.0],
])
rotated = points @ rotation.T
print(np.round(rotated, 6))
```

```text
[0. 1.]
[[ 0.  1.]
 [-1.  0.]
 [-3.  2.]]
```

!!! note "Чому `points @ rotation.T`"
    Коли точки лежать у **рядках** масиву `(n, 2)`, формула повороту перетворюється на `points @ R.T`. Якби точки були у стовпцях (`(2, n)`), писали б `R @ points`. Обидва варіанти зустрічаються в коді — дивіться на `shape`.

## Одинична матриця

Одинична матриця має одиниці на головній діагоналі й нулі поза нею. Вона грає роль числа 1: `A @ I == A`.

```python
import numpy as np

identity = np.eye(3)
print(identity)

a = np.array([
    [2.0, 1.0, 0.0],
    [0.0, 3.0, 5.0],
    [1.0, 1.0, 1.0],
])

print(np.array_equal(a @ identity, a))
print(np.array_equal(identity @ a, a))
```

```text
[[1. 0. 0.]
 [0. 1. 0.]
 [0. 0. 1.]]
True
True
```

Поряд корисні `np.diag` (дістати діагональ або побудувати діагональну матрицю) і `np.trace` (сума діагоналі):

```python
import numpy as np

a = np.array([
    [2.0, 1.0, 0.0],
    [0.0, 3.0, 5.0],
    [1.0, 1.0, 1.0],
])

print(np.diag(a))
print(np.trace(a))
print(np.diag(np.array([1.0, 2.0, 3.0])))
```

```text
[2. 3. 1.]
6.0
[[1. 0. 0.]
 [0. 2. 0.]
 [0. 0. 3.]]
```

## Визначник

**Визначник** (determinant) — число, яке показує, чи «вироджена» квадратна матриця. Для розміру `2x2` формула проста:

$$
\det \begin{pmatrix} a & b \\ c & d \end{pmatrix} = ad - bc
$$
 Якщо він дорівнює нулю, матриця необоротна: рядки лінійно залежні, і перетворення «склеює» простір.

```python
import numpy as np

good = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])
# другий рядок = перший, помножений на 2
bad = np.array([
    [2.0, 1.0],
    [4.0, 2.0],
])

print(np.linalg.det(good))
print(np.linalg.det(bad))
```

```text
5.000000000000001
0.0
```

Зверніть увагу на `5.000000000000001`: визначник рахується чисельно, тому **порівнювати його з нулем через `==` не можна**. Для перевірки беруть невеликий поріг або, надійніше, `np.linalg.matrix_rank` (про неї нижче).

```python
import numpy as np

matrix = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [7.0, 8.0, 9.0],
])

determinant = np.linalg.det(matrix)
print(determinant)
print(determinant == 0)
print(abs(determinant) < 1e-10)
```

```text
6.66133814775094e-16
False
True
```

Математично визначник цієї матриці дорівнює нулю (третій рядок = другий плюс різниця рядків), але чисельно вийшло `6.7e-16`. Перевірка `== 0` дала **`False`** — тобто «матриця нормальна», хоча вона вироджена. Порівняння з порогом дає правильну відповідь.

## Обернена матриця

Обернена матриця \(A^{-1}\) — це така, що \(A A^{-1} = A^{-1} A = I\). Рахує її `np.linalg.inv`.

```python
import numpy as np

a = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])

inverse = np.linalg.inv(a)
print(np.round(inverse, 4))
print(np.round(a @ inverse, 10))
print(np.allclose(a @ inverse, np.eye(2)))
```

```text
[[ 0.6 -0.2]
 [-0.2  0.4]]
[[ 1.  0.]
 [-0.  1.]]
True
```

!!! tip "`np.allclose` замість `==`"
    Обчислення з плаваючою комою майже ніколи не дають точний результат. `np.allclose(x, y)` порівнює з допуском і є стандартним способом перевірки в лінійній алгебрі.

Для виродженої матриці `inv` кидає помилку:

```python
import numpy as np

bad = np.array([
    [2.0, 1.0],
    [4.0, 2.0],
])

try:
    np.linalg.inv(bad)
except np.linalg.LinAlgError as error:
    print("LinAlgError:", error)
```

```text
LinAlgError: Singular matrix
```

`np.linalg.LinAlgError` — окремий клас винятку для всього модуля `linalg`. Його й ловлять, коли дані можуть виявитися виродженими.

## Системи лінійних рівнянь

Ось навіщо все попереднє. Система

$$
\begin{cases}
2x + y = 11 \\
x + 3y = 18
\end{cases}
$$

у матричному вигляді — це \(Ax = b\), де

$$
A = \begin{pmatrix} 2 & 1 \\ 1 & 3 \end{pmatrix}
\qquad
b = \begin{pmatrix} 11 \\ 18 \end{pmatrix}
\qquad
x = \begin{pmatrix} x \\ y \end{pmatrix}
$$

Розв'язок дає `np.linalg.solve(A, b)`:

```python
import numpy as np

a = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])
b = np.array([11.0, 18.0])

solution = np.linalg.solve(a, b)
print(solution)

# перевірка: підставляємо розв'язок назад
print(a @ solution)
print(np.allclose(a @ solution, b))
```

```text
[3. 5.]
[11. 18.]
True
```

Відповідь: \(x = 3\), \(y = 5\). Перевірка підстановкою — обов'язковий крок, він коштує один рядок.

Розмір системи не обмежений двома рівняннями. Три невідомі:

$$
\begin{cases}
x + 2y + 3z = 14 \\
2x - y + z = 3 \\
3x + y - z = 2
\end{cases}
$$

```python
import numpy as np

a = np.array([
    [1.0, 2.0, 3.0],
    [2.0, -1.0, 1.0],
    [3.0, 1.0, -1.0],
])
b = np.array([14.0, 3.0, 2.0])

solution = np.linalg.solve(a, b)
print(np.round(solution, 6))
print(np.allclose(a @ solution, b))
```

```text
[1. 2. 3.]
True
```

`solve` вимагає, щоб `A` була **квадратною** і невиродженою, а довжина `b` дорівнювала кількості рядків `A`.

## `solve` проти `inv`

Математично \(x = A^{-1} b\), тому виникає спокуса написати `np.linalg.inv(a) @ b`. Так робити не варто: `solve` точніший і швидший, бо не рахує обернену матрицю повністю.

```python
import numpy as np

rng = np.random.default_rng(0)
size = 500
a = rng.random((size, size)) + np.eye(size) * size
b = rng.random(size)

by_solve = np.linalg.solve(a, b)
by_inverse = np.linalg.inv(a) @ b

print(np.allclose(by_solve, by_inverse))
print("max difference:", np.max(np.abs(by_solve - by_inverse)))
```

```text
True
max difference: 8.673617379884035e-18
```

На цій системі різниця мікроскопічна, але на погано обумовлених матрицях `inv` втрачає точність помітно. Порівняймо швидкість (абсолютні числа залежать від комп'ютера, важливе співвідношення):

```python
import time

import numpy as np

rng = np.random.default_rng(0)
size = 800
a = rng.random((size, size)) + np.eye(size) * size
b = rng.random(size)

start = time.perf_counter()
for _ in range(10):
    np.linalg.solve(a, b)
solve_time = time.perf_counter() - start

start = time.perf_counter()
for _ in range(10):
    np.linalg.inv(a) @ b
inverse_time = time.perf_counter() - start

print(f"solve: {solve_time:.4f} s")
print(f"inv:   {inverse_time:.4f} s")
print(f"ratio: {inverse_time / solve_time:.1f}x")
```

```text
solve: 0.4295 s
inv:   1.4893 s
ratio: 3.5x
```

!!! tip "Правило"
    Потрібен розв'язок системи — `np.linalg.solve`. Обернена матриця потрібна рідко й сама по собі: наприклад, коли її треба показати або застосовувати багато разів до різних правих частин (та й тоді краще передати `solve` одразу матрицю правих частин).

`solve` вміє розв'язувати кілька систем із тією самою матрицею за один виклик — треба лише передати `b` як матрицю `(n, k)`:

```python
import numpy as np

a = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])
# два різних набори правих частин у стовпцях
b = np.array([
    [11.0, 5.0],
    [18.0, 10.0],
])

solution = np.linalg.solve(a, b)
print(solution)
```

```text
[[3. 1.]
 [5. 3.]]
```

Кожен стовпець результату — розв'язок для відповідного стовпця `b`.

## Коли розв'язку немає: вироджена матриця

Якщо рівняння лінійно залежні (одне є комбінацією інших), система або не має розв'язку, або має їх безліч. `solve` у такому разі кидає `LinAlgError`:

```python
import numpy as np

# друге рівняння — перше, помножене на 2
a = np.array([
    [1.0, 2.0],
    [2.0, 4.0],
])
b = np.array([5.0, 11.0])

try:
    np.linalg.solve(a, b)
except np.linalg.LinAlgError as error:
    print("LinAlgError:", error)

print("det:", np.linalg.det(a))
print("rank:", np.linalg.matrix_rank(a))
```

```text
LinAlgError: Singular matrix
det: 0.0
rank: 1
```

У реальній програмі це обробляють так:

```python
import numpy as np


def solve_system(matrix, right_side):
    """Повертає розв'язок системи або None, якщо матриця вироджена."""
    try:
        return np.linalg.solve(matrix, right_side)
    except np.linalg.LinAlgError:
        return None


good = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])
bad = np.array([
    [1.0, 2.0],
    [2.0, 4.0],
])
right_side = np.array([11.0, 18.0])

print(solve_system(good, right_side))
print(solve_system(bad, right_side))
```

```text
[3. 5.]
None
```

!!! warning "Майже вироджена матриця небезпечніша за вироджену"
    Якщо визначник не нуль, а `1e-15`, `solve` не кине помилку — він поверне числа, які можуть не мати сенсу. Перевірити стійкість системи допомагає **число обумовленості** `np.linalg.cond`: чим воно більше, тим менше довіри до результату.

```python
import numpy as np

stable = np.array([
    [2.0, 1.0],
    [1.0, 3.0],
])
shaky = np.array([
    [1.0, 2.0],
    [2.0, 4.000001],
])

print(f"stable cond: {np.linalg.cond(stable):.2f}")
print(f"shaky cond:  {np.linalg.cond(shaky):.2e}")
```

```text
stable cond: 2.62
shaky cond:  2.50e+07
```

Орієнтир: `cond` порядку одиниць чи десятків — добре; `1e10` і більше — результату довіряти не можна, дані треба переглянути.

## Ранг матриці

**Ранг** — кількість лінійно незалежних рядків. Для квадратної матриці «ранг дорівнює розміру» означає те саме, що «визначник не нуль», але `matrix_rank` стійкіша до похибок округлення і працює з будь-якою формою.

```python
import numpy as np

full = np.array([
    [1.0, 2.0],
    [3.0, 5.0],
])
# третій рядок = перший + другий
dependent = np.array([
    [1.0, 2.0, 3.0],
    [4.0, 5.0, 6.0],
    [5.0, 7.0, 9.0],
])

print(np.linalg.matrix_rank(full), full.shape)
print(np.linalg.matrix_rank(dependent), dependent.shape)
```

```text
2 (2, 2)
2 (3, 3)
```

Ранг `2` при розмірі `3` означає: одне рівняння зайве, незалежної інформації лише на два. Перевірка рангу перед `solve` — спосіб зрозуміти, чи взагалі має сенс розв'язувати систему.

## Приклад: розрахунок суміші

Типова прикладна задача. Є три сировини з відомим вмістом трьох компонентів, треба отримати 100 кг суміші із заданим складом.

| Сировина | Білок, % | Жир, % | Волокно, % |
|---|---|---|---|
| A | 40 | 10 | 20 |
| B | 20 | 30 | 30 |
| C | 10 | 5 | 60 |

Ціль: 100 кг суміші, у яких 21.5 кг білка, 12.75 кг жиру і 40.5 кг волокна.

Якщо взяти \(x_A\), \(x_B\), \(x_C\) кілограмів сировини, то для кожного компонента отримуємо своє рівняння:

$$
\begin{cases}
0.40 x_A + 0.20 x_B + 0.10 x_C = 21.5 \\
0.10 x_A + 0.30 x_B + 0.05 x_C = 12.75 \\
0.20 x_A + 0.30 x_B + 0.60 x_C = 40.5
\end{cases}
$$
 Отримуємо систему з трьох рівнянь — матриця складів множиться на вектор кількостей.

```python
import numpy as np

# рядок — компонент, стовпець — сировина
composition = np.array([
    [0.40, 0.20, 0.10],   # білок
    [0.10, 0.30, 0.05],   # жир
    [0.20, 0.30, 0.60],   # волокно
])
target = np.array([21.5, 12.75, 40.5])
materials = np.array(["A", "B", "C"])

amounts = np.linalg.solve(composition, target)

for name, value in zip(materials, amounts):
    print(f"{name}: {value:.2f} kg")

print("total:", round(amounts.sum(), 2), "kg")
print("check:", np.allclose(composition @ amounts, target))
```

```text
A: 30.00 kg
B: 25.00 kg
C: 45.00 kg
total: 100.0 kg
check: True
```

Окремо варто перевіряти, чи має відповідь фізичний сенс: від'ємна кількість сировини математично коректна, а практично — ні.

```python
import numpy as np

composition = np.array([
    [0.40, 0.20, 0.10],
    [0.10, 0.30, 0.05],
    [0.20, 0.30, 0.60],
])
# недосяжний склад: забагато жиру
target = np.array([15.0, 29.5, 44.0])

amounts = np.linalg.solve(composition, target)
print(np.round(amounts, 2))

if np.any(amounts < 0):
    print("mixture is not feasible: negative amount")
```

```text
[-20. 100.  30.]
mixture is not feasible: negative amount
```

## Перевизначені системи та `lstsq`

Часто рівнянь **більше**, ніж невідомих: наприклад, є 10 вимірювань і 2 параметри прямої. Точного розв'язку зазвичай не існує, але можна знайти такий, що дає найменшу суму квадратів відхилень. Це робить `np.linalg.lstsq` (least squares — метод найменших квадратів).

Підберемо пряму \(y = kx + c\) до набору точок, мінімізуючи \(\sum_i (y_i - kx_i - c)^2\):

```python
import numpy as np

hours = np.array([1.0, 2.0, 3.0, 4.0, 5.0, 6.0])
score = np.array([52.0, 55.0, 61.0, 64.0, 70.0, 72.0])

# матриця плану: стовпець x і стовпець одиниць
design = np.column_stack([hours, np.ones(hours.size)])
print(design)

result = np.linalg.lstsq(design, score, rcond=None)
slope, intercept = result[0]

print(f"slope: {slope:.3f}")
print(f"intercept: {intercept:.3f}")
print("predicted for 7 hours:", round(slope * 7 + intercept, 2))
```

```text
[[1. 1.]
 [2. 1.]
 [3. 1.]
 [4. 1.]
 [5. 1.]
 [6. 1.]]
slope: 4.229
intercept: 47.533
predicted for 7 hours: 77.13
```

`lstsq` повертає кортеж із чотирьох елементів: розв'язок, суму квадратів залишків, ранг матриці та її сингулярні числа. У більшості випадків потрібен лише перший.

`lstsq` не падає на вироджених даних — на відміну від `solve`, він завжди щось повертає, тому корисний у прикладних задачах апроксимації.

## Власні числа: коротко

`np.linalg.eig` знаходить **власні числа** і **власні вектори**: такі вектори, напрям яких матриця не змінює, а лише розтягує в \(\lambda\) разів: \(Av = \lambda v\). Тема виходить за межі курсу, але знати про функцію варто — вона лежить в основі методу головних компонент (PCA) та аналізу стійкості систем.

```python
import numpy as np

matrix = np.array([
    [2.0, 0.0],
    [0.0, 3.0],
])

values, vectors = np.linalg.eig(matrix)
print(values)
print(vectors)
```

```text
[2. 3.]
[[1. 0.]
 [0. 1.]]
```

Для діагональної матриці власні числа — це і є діагональ, а власні вектори — осі координат.

## Типові помилки

**`*` замість `@`.** Найчастіша помилка. `a * b` множить поелементно; для квадратних матриць помилки не буде, буде неправильний результат.

**Переплутаний порядок множників.** `A @ B` ≠ `B @ A`. Перед множенням подумайте, які розміри мають зійтися.

**Не збігаються внутрішні розміри.** `(2, 3) @ (2, 3)` — помилка; потрібне `(2, 3) @ (3, k)`. Часто лікується транспонуванням: `a @ b.T`.

**`.T` на одновимірному масиві.** Нічого не змінює. Для стовпця — `reshape(-1, 1)`.

**Порівняння визначника з нулем через `==`.** Через похибки округлення нуль майже ніколи не буває точним. Беріть поріг або `matrix_rank`.

**`inv(a) @ b` замість `solve(a, b)`.** Повільніше і менш точно.

**Не перевірений розв'язок.** `np.allclose(a @ x, b)` — один рядок, який рятує від довгого налагодження.

**Цілі типи там, де потрібні дробові.** Якщо матриця має `dtype=int`, `solve` сам переведе результат у `float`, але всі попередні обчислення (наприклад, ділення) можуть мовчки відкинути дробову частину. Для лінійної алгебри одразу створюйте масиви з `float`.

**Незловлений `LinAlgError`.** Якщо матриця приходить ззовні (введення користувача, файл), виклик `solve` має бути в `try/except`.

**Ігнорування `cond`.** Формально коректний розв'язок погано обумовленої системи може не мати жодного сенсу.

## Підсумок

- **`@`** — матричне множення, **`*`** — поелементне. Плутанина між ними не викликає помилки, але дає неправильний результат.
- Для векторів `a @ b` — **скалярний добуток**, число.
- **`np.linalg.norm`** рахує довжину вектора; з `axis=1` — відстані для набору точок одразу.
- **`.T`** транспонує матрицю і повертає перегляд; на одновимірному масиві не робить нічого.
- Множення можливе, коли **внутрішні розміри збігаються**: `(m, k) @ (k, n) -> (m, n)`.
- **`np.eye`** створює одиничну матрицю, **`np.diag`** — діагональну, **`np.trace`** рахує слід.
- **`np.linalg.det`** — визначник; нуль означає вироджену матрицю. Порівнювати з нулем треба через поріг.
- **`np.linalg.inv`** — обернена матриця, кидає `LinAlgError` на виродженій.
- **`np.linalg.solve(a, b)`** розв'язує систему \(Ax = b\) — точніше й швидше за `inv(a) @ b`.
- Результат завжди перевіряють через **`np.allclose(a @ x, b)`**.
- **`np.linalg.matrix_rank`** і **`np.linalg.cond`** показують, чи має система надійний розв'язок.
- **`np.linalg.lstsq`** знаходить наближений розв'язок, коли рівнянь більше, ніж невідомих — основа лінійної регресії.

## Корисні посилання

- [NumPy: linear algebra](https://numpy.org/doc/stable/reference/routines.linalg.html)
- [`np.linalg.solve`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.solve.html)
- [`np.linalg.inv`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.inv.html)
- [`np.linalg.det`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.det.html)
- [`np.linalg.norm`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.norm.html)
- [`np.linalg.lstsq`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.lstsq.html)
- [`np.linalg.matrix_rank`](https://numpy.org/doc/stable/reference/generated/numpy.linalg.matrix_rank.html)
- [PEP 465 — оператор `@`](https://peps.python.org/pep-0465/)
