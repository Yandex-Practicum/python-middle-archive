# Авторское решение

```python
import json
import sqlite3
from collections import defaultdict
from dataclasses import dataclass

from random import choice, sample, shuffle
from typing import Dict, List
from urllib.parse import urlencode

@dataclass
class Actor:
    id: int
    name: str

@dataclass
class Writer:
    id: str
    name: str

@dataclass()
class Movie:
    id: str
    genre: List[str]
    actors: List[Actor]
    writers: List[Writer]
    directors: List[str]
    title: str
    description: str

conn = sqlite3.connect('db.sqlite')

writers: Dict[str, Writer] = dict()
movie_actors: Dict[str, List[Actor]] = defaultdict(list)
movies: List[Movie] = list()

# соберём всю необходимую информацию по фильмам
for actor_id, name, movie_id in conn.execute(
        'select id, name, movie_id from actors a inner join movie_actors ma on a.id = ma.actor_id',
):
    actor_id = int(actor_id)
    actor = Actor(actor_id, name)
    movie_actors[movie_id].append(actor)

for row in conn.execute('select id, name from writers'):
    writer = Writer(*row)
    writers[writer.id] = writer

for movie_id, genre, director, writer, writers_json, title, plot in conn.execute(
        'select id, genre, director, writer, writers, title, plot from movies',
):
    writers_list = []
    if writer:
        writers_list.append(writers[writer])
    else:
        for w in json.loads(writers_json):
            writers_list.append(writers[w['id']])

    movie = Movie(
        movie_id,
        list(map(str.strip, genre.split(','))),
        movie_actors[movie_id],
        writers_list,
        list(map(str.strip, director.split(','))),
        title,
        plot,
    )
    movies.append(movie)

skip_words = {'the', 'to', 'a', 'na', 'its', 'in', 'on', 'where', 'this', 'how', 'set', 'by', 'of', 'this'}

# удалим лишние слова из строки
def clean_str(s: str) -> str:
    s = s.lower()
    if s in skip_words:
        return ''
    return ''.join(filter(str.isalpha, s))

# список запросов
queries: List[str] = []

for m in movies:
    query = {
        'sort': choice(['asc', 'desc'])
    }
    queries.append(urlencode(query))

    # запрос на поиск по актёру
    query['search'] = choice(m.actors).name
    queries.append(urlencode(query))

    # запрос на поиск по режиссёру
    query['search'] = choice(m.directors)
    queries.append(urlencode(query))

    # запрос на поиск по сценаристу
    query['search'] = choice(m.writers).name
    queries.append(urlencode(query))

    # запрос на поиск по части названия фильма
    for title_part in filter(None, map(clean_str, m.title.split(' '))):
        query['search'] = title_part

    # создаём запрос с лимитом и страницей
    for limit in [5, 7, 10, 1, 15, 20, 50]:
        query['limit'] = limit
        queries.append(urlencode(query))
        for page in [1, 2, 3, 4, 5, 6]:
            queries.append(urlencode(query))

            # закидываем в одну кучу название фильма и имена актёра, режиссёра и сценариста — тут как повезёт.
            # рандомом вытаскиваем то, что попало в кучу
            some_search_terms = set()
            for title_part in filter(None, map(clean_str, m.title.split(' '))):
                some_search_terms.add(title_part)

            for actor in m.actors:
                some_search_terms.add(actor.name)

            for director in m.directors:
                some_search_terms.add(director)

            for writer in m.writers:
                some_search_terms.add(writer.name)

            some_search_terms = list(some_search_terms)
            if len(some_search_terms) < 3:
                shuffle(some_search_terms)
                query['search'] = ' '.join(some_search_terms)
            else:
                query['search'] = ' '.join(sample(some_search_terms, 3))
            queries.append(urlencode(query))

# смешать, но не взбалтывать
shuffle(queries)

# записать запросы в файл
with open('targets.txt', 'w+') as f:
    for q in queries:
        f.write(f'GET http://127.0.0.1:8000/api/movies?{q}\n\n')
```

