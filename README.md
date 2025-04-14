# Лабораторная работа №12

## Задание

Разработать графический интерфейс для просмотра картотеки Spotify

_Вы можете отходить от дизайна, но должен присутствовать обязательный функционал_

### Обязательный функционал

- Поиск по названию песни, исполнителю, альбому
- Сортировка по дате добавления (дефолт), длительности, популярности
- Информация должна забираться с пути `http://omsktec-playgrounds.ru/algos/lab13`

_Если у вас сайт заблокирован Ростелекомом, то забирайте с IP адреса (`http://147.45.158.220/algos/lab13`)_

- Для запросов должна быть использована библиотека `requests` (не забудьте установить её)

## Референс

<img src="./.repo/finished.png" />

## Работа с таблицами

Компонент Treeview в Tkinter используется для отображения табличных данных, а также для создания иерархических структур (деревьев).

### Создание таблицы

```python
import tkinter as tk
from tkinter import ttk

# Создание окна
root = tk.Tk()
root.title("Пример таблицы")

# Создание таблицы с указанием колонок
columns = ('id', 'name', 'age')
table = ttk.Treeview(root, columns=columns, show='headings')

# Размещение таблицы
table.pack(fill=tk.BOTH, expand=True)
```

Параметр `show='headings'` означает, что будут отображаться только заголовки колонок без дополнительной колонки для структуры дерева.

### Настройка колонок

```python
# Настройка заголовков
table.heading('id', text='ID')
table.heading('name', text='Имя')
table.heading('age', text='Возраст')

# Настройка ширины колонок
table.column('id', width=50, anchor=tk.CENTER)
table.column('name', width=150)
table.column('age', width=70, anchor=tk.CENTER)
```

### Добавление данных

```python
# Добавление данных в таблицу
data = [
    (1, 'Иван', 25),
    (2, 'Мария', 22),
    (3, 'Алексей', 30)
]

for item in data:
    table.insert('', tk.END, values=item)
```

Первый параметр метода `insert()` - это родительский элемент (пустая строка для корневого уровня), второй параметр - индекс (tk.END для добавления в конец), а `values` - кортеж со значениями для колонок.

### Обработка выделения строк

```python
def on_item_select(event):
    # Получение выделенных элементов
    selected_items = table.selection()
    if selected_items:  # Проверка, что что-то выбрано
        item = selected_items[0]  # Берем первый выбранный элемент
        values = table.item(item, 'values')
        print(f"Выбран: {values}")

# Привязка события выбора элемента
table.bind('<<TreeviewSelect>>', on_item_select)
```

### Сортировка и фильтрация

Treeview не имеет встроенных функций сортировки и фильтрации, но вы можете их реализовать самостоятельно:

```python
def sort_by_column(tree, col, reverse):
    # Получение данных из таблицы
    data = [(tree.item(item, 'values'), item) for item in tree.get_children('')]
    
    # Сортировка данных
    data.sort(key=lambda x: x[0][columns.index(col)], reverse=reverse)
    
    # Перестановка элементов
    for index, (values, item) in enumerate(data):
        tree.move(item, '', index)
    
    # Следующий вызов метода изменит направление сортировки
    tree.heading(col, command=lambda: sort_by_column(tree, col, not reverse))

# Привязка сортировки к заголовкам колонок
for col in columns:
    table.heading(col, command=lambda c=col: sort_by_column(table, c, False))
```

## Работа с картинками

Для работы с изображениями в Tkinter рекомендуется использовать библиотеку PIL/Pillow.

### Установка Pillow

```
pip install pillow
```

### Загрузка изображений

```python
from PIL import Image, ImageTk
import requests
from io import BytesIO

# Загрузка изображения из файла
image = Image.open('image.jpg')

# Загрузка изображения из URL
def load_image_from_url(url):
    response = requests.get(url)
    if response.status_code == 200:
        return Image.open(BytesIO(response.content))
    return None
```

### Изменение размера изображений

```python
# Изменение размера с сохранением пропорций
def resize_image(image, max_width, max_height):
    # Получаем исходные размеры
    width, height = image.size
    
    # Находим коэффициент масштабирования
    ratio = min(max_width / width, max_height / height)
    
    # Новые размеры
    new_width = int(width * ratio)
    new_height = int(height * ratio)
    
    # Изменяем размер
    return image.resize((new_width, new_height), Image.LANCZOS)
```

### Отображение изображений в Tkinter

```python
# Преобразование изображения PIL в формат Tkinter
photo_image = ImageTk.PhotoImage(image)

# Создание метки для отображения
image_label = ttk.Label(root)
image_label.pack()

# Отображение изображения
image_label.config(image=photo_image)

# Сохраняем ссылку на изображение, чтобы избежать проблем со сборщиком мусора
image_label.image = photo_image
```

### Асинхронная загрузка изображений

Чтобы избежать "зависания" интерфейса при загрузке изображений, особенно из сети, используйте потоки:

```python
import threading

def download_and_display_image(url, label):
    def download():
        try:
            # Загрузка в отдельном потоке
            response = requests.get(url)
            if response.status_code == 200:
                image = Image.open(BytesIO(response.content))
                image = resize_image(image, 300, 300)
                photo = ImageTk.PhotoImage(image)
                
                # Обновление UI нужно делать в главном потоке
                label.after(0, lambda: update_label(label, photo))
        except Exception as e:
            print(f"Ошибка загрузки изображения: {e}")
    
    def update_label(label, photo):
        label.config(image=photo)
        label.image = photo  # Сохраняем ссылку
    
    # Запуск загрузки в отдельном потоке
    threading.Thread(target=download).start()
```
