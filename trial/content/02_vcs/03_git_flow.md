## Ветвление версий

Все работы из примера проводились в master**-**ветке. В ней хранится последняя стабильная версия приложения. Чтобы случайно не сломать мастер-ветку, для разработки новой функциональности или исправления ошибок создаются отдельные ветки. Такой подход называется [git-flow](https://habr.com/ru/company/flant/blog/491320/). На курсе мы рассмотрим его облегчённую версию.

Для разработки новой функциональности в название новой ветки добавляют префикс `feature/`. Он нужен, чтобы отличать её от тех, где исправляют ошибки — к названию таких веток добавляется префикс `fix/`.

Доработаем функциональность программы в отдельной feature-ветке:

```bash
# ключ -b нужен для создания новой ветки 
# если ветка с таким именем уже существует, то появится ошибка
git checkout -b feature/add_args_to_hello_function
```

Внесём изменения в программу:

```python
def hello(name: str):
    print(f'Hello {name}!')

if __name__ == '__main__':
    hello('git')
```

Выполните команду `git status`. После этого отобразится список файлов с изменениями, которые не были зафиксированы с помощью git.

В результате git покажет, что файл main.py был изменён. Выполните команду `git diff main.py`, чтобы просмотреть изменения: 

```bash
diff --git a/main.py b/main.py
index 43f608b..c5d453b 100644
--- a/main.py
+++ b/main.py
@@ -1,9 +1,9 @@

# знак "-" напротив строки означает, что её удалили
# знак "+" означает, что строку добавили 
# если строчка была просто изменена, то её старая версия отобразится со знаком "-", а новая — со знаком "+"
# этот подход нужен когда под рукой нет IDE или merge-инструмента

-def hello_git():
-    print('Hello git!')
+def hello(name: str):
+    print(f'Hello {name}!')

 if __name__ == '__main__':
-    hello_git()
+    hello('git')
```

Коммитим изменения:

```bash
git add main.py
# проверяем, не осталось ли файлов, изменения которых хотим закоммитить
git status 
# коммитим изменения
git commit -m 'replace hello_git func by hello'
# отправляем изменения на сервер
git push origin feature/add_args_to_hello_function
```

Вернёмся в мастер-ветку и убедимся, что её не затронули изменения:

```bash
# обратите внимание: в уже существующую ветку переходим без ключа "-b"
git checkout master
# видим, что содержание файла в первозданном виде
cat main.py 
```

Теперь нужно слить изменения ветки `feature/add_args_to_hello_function` с основной веткой разработки:

```bash
# сливаем изменения ветки с master-веткой
git merge feature/add_args_to_hello_function
# проверяем, что файл поменялся
cat main.py
# отправляем изменения на сервер
git push origin master
```