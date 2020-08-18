## Авторское решение

```python
from itertools import zip_longest
from typing import Optional

def grouper(n, iterable, fillvalue=None):
    """
    Метод делит одномерный массив на двумерный с фиксированным числом элементов (n) в каждом подмассиве.
    Если значений не хватает, то идет заполнение значением fillvalue
    """
    args = [iter(iterable)] * n
    return zip_longest(fillvalue=fillvalue, *args)

class Matrix:
    def __init__(self):
        self.size = 1  # размер матрицы
        self.last_position = 0  # позиция первого None-элемента
        self.matrix = [None]

    def matrix_scale_up(self):
        """ Расширение матрицы через добавление недостающих None-элементов """
        self.size += 1
        self.matrix.extend([None] * (self.size ** 2 - len(self.matrix)))

    def matrix_scale_down(self):
        """ Удаление ненужных элементов матрицы через срез данных """
        self.size -= 1
        self.matrix = self.matrix[:self.size ** 2]

    def add_item(self, element: Optional = None):
        """
        Добавляем новый элемент в матрицу.
        Если элемент не умещается в (size - 1) ** 2, то расширить матрицу.
        """
        if element is None:
            raise ValueError('Попытка добавить элемент со значением None')

        if self.last_position >= (self.size - 1) ** 2:
            self.matrix_scale_up()

        self.matrix[self.last_position] = element
        self.last_position += 1

    def pop(self):
        """
        Удалить последний значащий элемент из массива.
        Если значащих элементов меньше (size - 1) * (size - 2), уменьшить матрицу.
        """
        if self.size == 1:
            raise IndexError()

        self.last_position -= 1
        value = self.matrix[self.last_position]
        self.matrix[self.last_position] = None

        if self.last_position <= (self.size - 1) * (self.size - 2):
            self.matrix_scale_down()

        return value

    def __repr__(self):
        """
        Метод должен выводить матрицу в виде:
        1 2 3\nNone None None\nNone None None
        То ест между элементами строки должны быть пробелы, а строки отделены \n
        """
        return '\n'.join(' '.join(str(x) for x in y) for y in grouper(self.size, self.matrix))
```