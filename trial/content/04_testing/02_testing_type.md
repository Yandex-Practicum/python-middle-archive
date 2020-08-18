# Виды тестирования

## Юнит-тесты

Юнит-тесты пишут для тестирования конкретного класса или функции в вашей программе. Они должны проверить все возможные сценарии использования тестируемой функции, включая маловероятные сценарии. Юнит-тесты не должны быть завязаны на реализацию зависимостей тестируемой функции. 

Рассмотрим небольшой пример:

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class Movie:
    id: int
    title: Optional[str]

class MovieRepo:
    def get_movie_by_id(self, id: int) -> Optional[Movie]:
        raise NotImplemented

class MovieRepoImpl(MovieRepo):
    def __init__(self):
        self.db = SomeDB()

    def get_movie_by_id(self, id: int) -> Optional[Movie]:
        data = self.db.query(table='movies', id=id)
        return Movie(data['id'], data['title'])

class MoviesHandler:
    # класс зависит от интерфейса MovieRepo
    # поэтому в тестах можно передать тестовую реализацию MovieRepo,
    # чтобы проверить работу класса не зависимо от реализации MovieRepo
    def __init__(self, movie_repo: MovieRepo):
        self.movie_repo = movie_repo

    def get_movie_by_id(self, id: int) -> Optional[dict]:
        movie = self.movie_repo.get_movie_by_id(id)
        if movie is None:
            return None

        return {
            'id': movie.id,
            'title': movie.title
        }

# создаём тестовую реализацию MovieRepo,
# чтобы не зависеть от боевой реализации, которая зависит от БД
# таким образом, можно легко тестировать поведение MoviesHandler,
# возвращая разнообразные данные из метода get_movie_by_id
class MovieRepoMock(MovieRepo):
    movies = {
        42: Movie(42, 'some_title'),
        37: Movie(37, None),
    }

    def get_movie_by_id(self, id: int) -> Optional[Movie]:
        return self.movies.get(id)

def test_get_movie_by_id():
    movie_repo = MovieRepoMock()
    handler = MoviesHandler(movie_repo)
    expected = {
        'id': 42,
        'title': 'some_title',
    }
    # проверим, что метод работает без ошибок, если MovieRepo вернёт все данные
    actual = handler.get_movie_by_id(42)
    assert expected == actual

    expected = {
        'id': 37,
        'title': None,
    }
    # проверим, что метод работает без ошибок, если MovieRepo вернёт часть данных
    actual = handler.get_movie_by_id(37)
    assert expected == actual

    # проверим, что метод работает без ошибок, если MovieRepo ничего не вернул
    actual = handler.get_movie_by_id(9999)
    assert actual is None
```

## Интеграционные тесты

Этот вид тестов проверяет взаимодействие с другими компонентами программы, например, с базой данных. Если говорить про пример из юнит-тестов, то интеграционный тест должен протестировать `MovieRepoIm`, чтобы проверить корректную интеграцию с БД. Интеграционный тест проходит в несколько этапов:

1. создаём базу данных, 
2. заполняем её тестовыми данными,
3. подключаем тестируемый компонент к базе,
4. вызываем API компонента,
5. проверяем возвращаемый результат,
6. если компонент записывает данные в БД, то тест должен проверить их корректность.

Пример:

```python
import sqlite3
from dataclasses import dataclass
from typing import Optional

@dataclass
class Movie:
    id: int
    title: Optional[str]

class MovieRepo:
    def get_movie_by_id(self, id: int) -> Optional[Movie]:
        raise NotImplemented

class MovieRepoImpl(MovieRepo):
    def __init__(self, conn: sqlite3.Connection):
        self.db = conn

    def get_movie_by_id(self, id: int) -> Optional[Movie]:
        data = self.db.execute('select id, title from movies where id = ?', (id, )).fetchone()
        id, title = data
        return Movie(id, title)

def test_get_movie_by_id():
		# создаём соединение с БД
    conn = sqlite3.connect(":memory:")
    # создаём схему
    conn.execute(
        '''
            create table movies
            (
                id int primary key,
                title text
            );
        '''
    )
    # вставляем тестовые данные
    conn.execute(
        'insert into movies(id, title) values (?, ?)',
        (1, 'star wars')
    )

    movie_repo = MovieRepoImpl(conn)
    movie = movie_repo.get_movie_by_id(1)
    assert movie == Movie(1, 'star wars')
```

Если компонент интегрируется с внешним API, например с «Кинопоиском», то можно запустить собственную версию сервиса, которая будет имитировать API. Другой вариант — найти готовые решения, которые мокируют HTTP-сервисы.

## Функциональные тесты

Функциональные тесты — это набор тестовых сценариев, которые проверяют работоспособность системы с помощью отправки запросов к методам API-системы. Так мы определяем их в этом курсе.

Тестовый сценарий — один тест, который проверяет один конкретный сценарий использования клиентом заданной системы. В роли клиента выступает любой пользователь или система, которые посылают запросы по заданному API.

В бесплатной части курса вы будете использовать не полноценные функциональные тесты, а их подвид — **смоук-тестирование **или **smoke tests. Чаще всего вам будет встречаться именно английский термин. 

Основное отличие смоук-тестирования от функционального в том, что в первом случае разработчик не влияет на состояние системы во время теста. Он смотрит на результат выполнения своего запроса. Такие тесты просто писать, и они дают быстрое понимание, насколько правильно работает основная функциональность системы.

Другой вид быстрого тестирования системы — контрактное тестирование*.* Эти тесты проверяют, выдерживает ли система контракт с клиентом. Контракт — это договорённость об используемых методах, полях и ограничениях. Для вашей системы таким контрактом выступает swagger- или open api-спецификация. Такой вид тестов помогает быстро понять, не появилось ли каких-либо проблем в реализации спецификации в системе. Это особенно полезно, если версий API достаточно много.

OpenAPI спецификация, которая ранее называлась Swagger, — один из популярных способов записи API, в частности — REST API. Этот вид описания спецификации пришёл на замену форматам RAML и API Blueprint. 

Основное назначение OpenAPI — документирование API для передачи в виде JSON или YAML-файлов разработчикам, аналитикам и QA. Она показывает, какие методы есть у API, состав полей каждого метода с указанием типов и формат ответа. Помимо самих полей в запросе у методов можно указывать необходимые заголовки и HTTP-коды ответов на разные ситуации. Такое текстовое описание отрисовывается в достаточно [удобный интерфейс](https://editor.swagger.io/), в котором даже можно посылать запросы. 

Важная ремарка: бывают случаи, когда люди думают, что OpenAPI неразрывно связан с форматом данных JSON. Это убеждение ошибочно. OpenAPI не описывает формат данных, он описывает только их схему . Подробная документация по Swagger находится на [официальном сайте](https://swagger.io/specification/) на английском.