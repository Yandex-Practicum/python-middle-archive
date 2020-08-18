## Отладка с помощью дебаггера

### ipdb

ipdb **—** это консольный отладчик для Python. Он полезен, если под рукой нет полноценной IDE, или когда вы ведёте разработку в текстовых редакторах — Emacs, Vim и им подобных.

Для установки ipdb выполните команду `pip install ipdb`.

С помощью ipdb можно пошагово выполнить отладку кода и при этом видеть весь контекст выполнения программы. Например, в программе выполняется код**:**

```python
In [47]: from typing import List
    ...:
    ...:
    ...: class Printer:
    ...:     def print(self, *args):
    ...:         raise NotImplemented
    ...:
    ...:
    ...: class PrinterImpl:
    ...:     def print(self, *args):
    ...:         print('PrinterImpl.print', *args)
    ...:
    ...:
    ...: class Baamboozle:
    ...:     def print_func_with_naming_error(self, *args):
    ...:         print('Baamboozle.print', *args)
    ...:
    ...:
    ...: def amazing_print(printers: List[Printer], *args):
    ...:     for p in printers:
    ...:         p.print(*args)
    ...:
    ...:
    ...: printers = [PrinterImpl(), Baamboozle()]
    ...: amazing_print(printers, 'Hello', 'world!')
    ...:
PrinterImpl.print Hello world!
---------------------------------------------------------------------------
AttributeError                            Traceback (most recent call last)
<ipython-input-47-afb5cddd3a88> in <module>
     23
     24 printers = [PrinterImpl(), Baamboozle()]
---> 25 amazing_print(printers, 'Hello', 'world!')

<ipython-input-47-afb5cddd3a88> in amazing_print(printers, *args)
     19 def amazing_print(printers: List[Printer], *args):
     20     for p in printers:
---> 21         p.print(*args)
     22
     23

AttributeError: 'Baamboozle' object has no attribute 'print'
```

Программа выдаёт ошибку: один из переданных в функцию принтеров `amazing_print` не реализует интерфейс `Printer`.

Отладим код с помощью ipdb. Установите breakpoint ****— специальный маркер, на котором отладчик остановит процесс выполнения программы. Иногда эти маркеры на русском языке называются «точки останова». 

После установки брейкпоинта вы: 

- увидите содержимое аргументов, переданных в функцию;
- отследите, откуда пришли неправильные объекты;
- найдёте и исправите ошибку.

Добавим брейкпоинт в начале функции. Для этого импортируем модуль `ipdb`, а затем вызовем функцию `set_trace()`:

```python
In [48]: def amazing_print(printers: List[Printer], *args):
    ...:     import ipdb
    ...:     ipdb.set_trace()
    ...:     for p in printers:
    ...:         p.print(*args)
```

Теперь вызовем функцию `amazing_print`**.** Программа остановится там, где вы вызвали `ipdb.set_trace()`. После этого она перейдёт в режим отладки, где нужно вводить команды в консоли. 

[Полный перечень команд для самостоятельного изучения на английском](https://wangchuan.github.io/coding/2017/07/12/ipdb-cheat-sheet.html) 

```python
    ...: printers = [PrinterImpl(), Baamboozle()]
    ...: amazing_print(printers, 'Hello', 'world!')
> <ipython-input-48-d217a6be535f>(4)amazing_print()
      3     ipdb.set_trace()
----> 4     for p in printers:
      5         p.print(*args)

ipdb> # здесь нужно вводить команды для отладки
```

Укажите команду **`n`.** Отладчик исполнит текущую строку программы и перейдёт к следующей.

На каждой строке можно исполнять любой код на Python. Таким образом можно узнать список методов и атрибутов любого объекта или изменить значение любой переменной.

Для поиска ошибки из примера, нужно исполнить программу командой `n(ext)` ****и вывести список атрибутов элементов списка `printers`**:**

```python
ipdb> n
> <ipython-input-48-d217a6be535f>(5)amazing_print()
      4     for p in printers:
----> 5         p.print(*args)
      6

ipdb> dir(p)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'print']
ipdb> n
PrinterImpl.print Hello world!
> <ipython-input-48-d217a6be535f>(4)amazing_print()
      3     ipdb.set_trace()
----> 4     for p in printers:
      5         p.print(*args)

ipdb> n
> <ipython-input-48-d217a6be535f>(5)amazing_print()
      4     for p in printers:
----> 5         p.print(*args)
      6

ipdb> dir(p)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'print_func_with_naming_error']
```

Отладчик показал, что у второго элемента нет метода `print`, но есть метод с  «грамматической ошибкой» — `print_func_with_naming_error`. Это значит, что разработчик ошибся в названии метода: нужно переименовать его в `print`. Тогда программа заработает без ошибок.