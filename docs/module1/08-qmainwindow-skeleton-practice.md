# 8. (П4) Каркас десктоп-застосунку: менеджер нотаток

## Передумови

- Прочитана [Лекція 7 — Головне вікно застосунку (QMainWindow). Структура головного вікна](/ua/courses/programming-3sem/module1/07-qmainwindow-lecture/)
- Успішно запущені приклади `MenuDemo`, `StatusBarDemo`, `DockDemo`, `PersistentWindow` та `TextEditor` з лекції
- Виконана [Практична 3 — панель нотаток](/ua/courses/programming-3sem/module1/06-adaptive-layout-practice/): база SQLite і таблиця `notes` беруться звідти

!!! warning "Мова в коді"
    Усі рядкові літерали, написи на віджетах і в меню, заголовок вікна, SQL-запити та імена змінних — **лише латиницею**. Своє ім'я, прізвище та групу пишіть у транслітерації: `Ivan Petrenko`, `KI-31`. Кирилиця допускається тільки в коментарях.

## Завдання

Створити файл `notes_app.py` — той самий менеджер нотаток, але вже як **повноцінний десктоп-застосунок** на `QMainWindow`: з меню, панеллю інструментів, бічною панеллю, рядком стану й запам'ятовуванням вигляду вікна між запусками.

## Персоналізація

На початку файлу оголосити чотири константи і використовувати **лише їх** (не дублювати ці значення в коді):

```python
STUDENT_NAME = "Ivan Petrenko"                # ваше ім'я та прізвище транслітерацією
STUDENT_GROUP = "KI-31"                       # ваша група
DB_FILE = "workspace_ivan_petrenko.db"        # файл бази з практичної 3
SETTINGS_APP = "NotesAppIvanPetrenko"         # ім'я застосунку для QSettings
```

## База даних

Та сама таблиця, що й у практичній 3 (створюється при старті, якщо її ще немає):

```sql
CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    text TEXT NOT NULL,
    created_at TEXT NOT NULL
)
```

`created_at` заповнюється в Python при створенні нотатки: `datetime.now().strftime("%Y-%m-%d %H:%M")`.

## Схема головного вікна

Підпис усередині кожної зони — **ім'я змінної**, під яким її треба створити:

```text
+--------------------------------------------------------------------+
|  NotesWindow (QMainWindow)         Notes - Ivan Petrenko, KI-31     |
+--------------------------------------------------------------------+
|  File      View      Help                       <- menuBar()        |
+--------------------------------------------------------------------+
|  [ New ] [ Save ] [ Delete ]                    <- main_toolbar     |
+--------------------------------------------------------------------+
|                        |                                           |
|  +------------------+  |  +-------------------------------------+  |
|  |                  |  |  |                                     |  |
|  |    notes_list    |  |  |            note_editor              |  |
|  |                  |  |  |         (centralWidget)             |  |
|  |  (у notes_dock)  |  |  |                                     |  |
|  |                  |  |  |                                     |  |
|  +------------------+  |  +-------------------------------------+  |
|                        |                                           |
+--------------------------------------------------------------------+
|  Ready                                          [ counter_label ]  |
+--------------------------------------------------------------------+
                          ^- statusBar()
```

## Складові вікна

### Дії (`QAction`)

Усі дії створюються **один раз** у методі `create_actions()`. Кожна має текст, гарячу клавішу, `setStatusTip()` і підключений `triggered`.

| Ім'я | Текст | Гаряча клавіша | Що робить |
|---|---|---|---|
| `new_action` | `New` | `QKeySequence.StandardKey.New` | починає нову нотатку |
| `save_action` | `Save` | `QKeySequence.StandardKey.Save` | зберігає поточну нотатку в базу |
| `delete_action` | `Delete` | `QKeySequence("Ctrl+D")` | видаляє поточну нотатку з бази |
| `quit_action` | `Quit` | `QKeySequence.StandardKey.Quit` | викликає `self.close()` |
| `about_action` | `About` | — | показує `QMessageBox` з даними студента |

### Меню (`menuBar()`)

| Меню | Пункти |
|---|---|
| `File` | `new_action`, `save_action`, `delete_action`, роздільник (`addSeparator()`), `quit_action` |
| `View` | `notes_dock.toggleViewAction()`, `main_toolbar.toggleViewAction()` |
| `Help` | `about_action` |

### Панель інструментів (`main_toolbar`)

Створюється через `self.addToolBar("Main")`, обов'язково `setObjectName("main_toolbar")`. Містить `new_action`, `save_action`, `delete_action` — **ті самі об'єкти**, що вже додані в меню `File`, а не їхні копії.

### Бічна панель (`notes_dock`)

`QDockWidget` із заголовком `Notes`, `setObjectName("notes_dock")`, прикріплений ліворуч (`Qt.DockWidgetArea.LeftDockWidgetArea`). Дозволені зони — лише ліва та права. Усередині — `notes_list` (`QListWidget`).

### Центральний віджет

`note_editor` — `QTextEdit`, встановлений через `setCentralWidget()`.

!!! danger "`setLayout()` на `QMainWindow` не працює"
    Не намагайтесь викликати `self.setLayout(...)` — головне вікно вже має власний компонувальник. Вміст ставиться лише через `setCentralWidget()`.

### Рядок стану

- `counter_label` — `QLabel`, доданий через `addPermanentWidget()`, показує `Notes: 4`.
- Тимчасові повідомлення через `showMessage(text, 2000)` після кожної операції з базою.

## Поведінка

**Старт.** Підключитись до бази, створити таблицю, побудувати дії, меню, панелі та рядок стану, відновити налаштування вікна, після чого заповнити список нотаток.

**Стан документа.** Тримати два атрибути:

- `self._current_id` — `id` нотатки, яку зараз редагують, або `None`, якщо це нова нотатка;
- `self._modified` — `True`, якщо текст у `note_editor` змінено після останнього збереження (виставляється в обробнику сигналу `textChanged`).

**Заголовок вікна.** Метод `update_title()` формує заголовок `Notes - Ivan Petrenko, KI-31` з констант і додає `*` на початку, якщо `self._modified` — `True`. Викликається після кожної зміни стану.

**`refresh_list()`.** Читає `SELECT id, created_at, text FROM notes ORDER BY id DESC` і наповнює `notes_list`. Для кожної нотатки:

- текст елемента — `<created_at>  <перший рядок тексту, обрізаний до 40 символів>`;
- `id` зберігається в самому елементі: `item.setData(Qt.ItemDataRole.UserRole, note_id)`.

Тут же оновлюється `counter_label` на `Notes: <кількість>`. Окремої копії нотаток у пам'яті класу тримати не треба — джерело істини завжди база.

!!! tip "Як не спіймати зайвий сигнал"
    Заповнення `notes_list` саме по собі змінює поточний елемент і викликає ваш обробник вибору. Щоб при оновленні списку не перезаписати текст, який користувач зараз друкує, обгортайте наповнення в `notes_list.blockSignals(True)` / `blockSignals(False)`.

**Вибір нотатки в `notes_list`.** Сигнал `currentItemChanged`. Обробник дістає `id` з елемента (`item.data(Qt.ItemDataRole.UserRole)`), читає `SELECT text FROM notes WHERE id = ?`, кладе текст у `note_editor`, записує `self._current_id`, скидає `self._modified` у `False` і оновлює заголовок.

**`new_action`.** Очищає `note_editor`, ставить `self._current_id = None`, знімає виділення в списку (`notes_list.setCurrentRow(-1)`), скидає `self._modified`, переводить фокус у редактор (`setFocus()`).

**`save_action`.** Бере текст `note_editor.toPlainText()`. Якщо після `.strip()` він порожній — `showMessage("Nothing to save", 2000)` і вихід. Інакше:

- якщо `self._current_id is None` — `INSERT` нового рядка з поточним часом, а новий `id` взяти з `cursor.lastrowid` і записати в `self._current_id`;
- інакше — `UPDATE notes SET text = ? WHERE id = ?`.

Далі `commit()`, `refresh_list()`, `self._modified = False`, `update_title()` і `showMessage("Note saved", 2000)`.

**`delete_action`.** Якщо `self._current_id is None` — `showMessage("Nothing to delete", 2000)`. Інакше запитати підтвердження через `QMessageBox.question()` з кнопками `Yes` / `No`; при `Yes` — `DELETE FROM notes WHERE id = ?`, `commit()`, після чого поводитись як `new_action` (порожній редактор, `_current_id = None`) і оновити список.

**`about_action`.** `QMessageBox.information()` із заголовком `About` і текстом із трьох рядків: ім'я та прізвище, група, `Python Programming, Semester 3`.

**Налаштування вікна.** Через `QSettings("KTBP", SETTINGS_APP)`:

- у `closeEvent()` зберегти `saveGeometry()` під ключем `geometry` і `saveState()` під ключем `window_state`;
- при старті відновити обидва; якщо збереженої геометрії ще немає — `self.resize(800, 500)`.

**Закриття.** Перевизначити `closeEvent()`:

1. Якщо `self._modified` — показати `QMessageBox.question()` з кнопками `Save`, `Discard`, `Cancel`. `Save` — зберегти й закривати далі, `Discard` — закривати без збереження, `Cancel` — `event.ignore()` і вихід із методу (нічого більше не робити).
2. Зберегти геометрію та стан вікна в `QSettings`.
3. Закрити підключення до бази: `self.connection.close()`.
4. `event.accept()`.

## Перевірка

Перед здачею переконайтесь, що:

- `Ctrl+N`, `Ctrl+S`, `Ctrl+D`, `Ctrl+Q` працюють, коли фокус у редакторі;
- наведення миші на пункт меню показує текст із `setStatusTip()` у рядку стану;
- панель `Notes` перетягується до правого краю, ховається через меню `View` і повертається назад;
- після закриття й повторного запуску розмір вікна, положення панелей та всі нотатки на місці;
- при спробі закрити вікно з незбереженим текстом з'являється діалог, а `Cancel` справді скасовує закриття.

!!! danger "Дві помилки, за які знімаються бали"
    **SQL-параметри.** Текст нотатки та `id` підставляти **тільки** через плейсхолдер: `cursor.execute("UPDATE notes SET text = ? WHERE id = ?", (text, note_id))`. Склеювання SQL через f-рядок — помилка.

    **`objectName`.** Без `setObjectName()` у `main_toolbar` і `notes_dock` виклик `saveState()` виведе попередження, а розташування панелей не відновиться після перезапуску.
