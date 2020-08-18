## Нагрузочное тестирование

Нагрузочное тестирование нужно для выявления неоптимизированных методов сервиса или для определения количества запросов, которые сервис может обработать за определённый промежуток времени. С помощью нагрузочных тестов можно узнать, не упадёт ли производительность, если вы решили переписать приложение с Flask на FastAPI. 

Для проведения нагрузочных тестов используйте консольную утилиту [vegeta](https://github.com/tsenart/vegeta).

### Установка vegeta

- OSX `brew install vegeta`;
- Из исходников с помощью Go `go get -u [github.com/tsenart/vegeta](http://github.com/tsenart/vegeta)`;
- [Скачать нужную версию для своей ОС](https://github.com/tsenart/vegeta/releases):
    - распаковать архив,
    - добавить `vegeta` в одну из папок внутри `$PATH`, например в `/usr/local/bin`.

### Тестирование с помощью vegeta

Перед началом теста создадим файл `targets.txt`. В него запишем URL, которые будем обстреливать. После них пропишем три цели — для каждой свой User-Agent:

```python
GET http://localhost:8000/client/info
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1.1 Safari/605.1.15675309

GET http://localhost:8000/client/info
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 Firefox/77.0

GET http://localhost:8000/client/info
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:77.0) Gecko/20100101 firefox/77.0
```

Формат файла: 

```python
{HTTP_METHOD} {HTTP_SCHEMA}://{HOST}[:{HTTP_PORT}]{URL}
# опционально указываем нужные заголовки
{HEADER_NAME}: {HEADER_VALUE}
# опционально тело запроса для POST-, PUT-, PATCH-запросов. Если хотите отправить на сервер JSON, укажите полный путь до файла, в котором он записан
@{PATH_TO_FILE}
# надо оставить пустую строку в качестве разделителя между целями, которые надо обстрелять
```

Обстреляем наш сервер

```bash
# vegeta — название утилиты, с помощью которой нагружаем сервис
# attack — команда, которая запускает нагрузку
# -targets — файл, в котором записаны цели для нагрузки
# -timeout — максимальное время ответа от сервера
# -duration — длительность теста
# -rate — с какой переодичностью бомбить сервис в единицу времени. Например, 700/1m или 700 запросов в минуту
# через pipe "|" передаём данные тестирования в команду, которая выводит итоговый результат тестирования
vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 30/1s > load_test_result.bin | vegeta report
```

Вывод будет отличаться от каждого прогона, потому что на производительность сервера влияют текущие программы. Например, вы запустили тестирование на своём рабочем компьютере, а в этот момент PyCharm решил переиндексировать файлы. Следовательно, он заберёт на себя часть ресурсов компьютера, которые нужны были вашему сервису в процессе тестирования.

```bash
# total — сколько всего было сделано запросов
# rate — интенсивность запросов
# throughput — пропускная способность сервера
Requests      [total, rate, throughput]         300, 30.10, 30.09
# total — сколько времени длился тест
# attack — сколько времени потрачено непосредственно на обстрел сервера
# wait — сколько времени было потрачено на ожидание ответа на последний запрос
Duration      [total, attack, wait]             9.97s, 9.966s, 3.9ms
# min — 1.807ms самое быстрое время выполнение запроса
# mean — 3.424ms среднее время выполнение запроса
# 50 персентиль — 50% запросов выполнилось до 3.426ms
# 90 персентиль — 90% запросов выполнилось до 4.02ms
# 95 персентиль — 95% запросов выполнилось до 4.09ms
# 99 персентиль — 99% запросов выполнилось до 4.897ms
# max — 8.605ms максимальное время запрос
Latencies     [min, mean, 50, 90, 95, 99, max]  1.807ms, 3.424ms, 3.426ms, 4.02ms, 4.09ms, 4.897ms, 8.605ms
# total — сколько байт было получено от сервера
# mean — среднее количество получено байт в ответе на запрос
Bytes In      [total, mean]                     39800, 132.67
# total — сколько байт было передано серверу
# mean — среднее количество переданных данных в запросе
Bytes Out     [total, mean]                     0, 0.00
# процент ответов без ошибок от сервера
Success       [ratio]                           100.00%
# какие HTTP-статусы были получены от сервера и в каком количестве
Status Codes  [code:count]                      200:300
# перечень полученных ошибок
Error Set:
```

Увеличим нагрузку на сервер, чтобы спровоцировать ошибку. Увеличим параметр  `-rate` до 700/1s и запустим тесты несколько раз. Если ошибки не появятся, увеличивайте до победного:

```bash
bash-3.2$ vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.11, 181.00
Duration      [total, attack, wait]             11.784s, 9.998s, 1.786s
Latencies     [min, mean, 50, 90, 95, 99, max]  34.87µs, 184.627ms, 117.732µs, 372.555ms, 1.96s, 2s, 2.002s
Bytes In      [total, mean]                     282494, 40.36
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           30.47%
Status Codes  [code:count]                      0:4867  200:2133
Error Set:
Get "http://localhost:8000/client/info": read tcp 127.0.0.1:55979->127.0.0.1:8000: read: connection reset by peer
Get "http://localhost:8000/client/info": read tcp 127.0.0.1:55981->127.0.0.1:8000: read: connection reset by peer
Get "http://localhost:8000/client/info": read tcp 127.0.0.1:55990->127.0.0.1:8000: read: connection reset by peer
Get "http://localhost:8000/client/info": read tcp 127.0.0.1:56008->127.0.0.1:8000: read: connection reset by peer
Get "http://localhost:8000/client/info": read tcp 127.0.0.1:56048->127.0.0.1:8000: read: connection reset by peer
Get "http://localhost:8000/client/info": dial tcp: lookup localhost: no such host
Get "http://localhost:8000/client/info": dial tcp 0.0.0.0:0->[::1]:8000: connect: connection refused
Get "http://localhost:8000/client/info": context deadline exceeded (Client.Timeout exceeded while awaiting headers)

bash-3.2$ vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.14, 329.56
Duration      [total, attack, wait]             11.855s, 9.998s, 1.857s
Latencies     [min, mean, 50, 90, 95, 99, max]  27.398µs, 259.369ms, 216.485ms, 308.366ms, 1.991s, 2s, 2.002s
Bytes In      [total, mean]                     518314, 74.04
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           55.81%
Status Codes  [code:count]                      0:3093  200:3907
Error Set:
Get "http://localhost:8000/client/info": dial tcp: lookup localhost: no such host
Get "http://localhost:8000/client/info": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
Get "http://localhost:8000/client/info": dial tcp 0.0.0.0:0->[::1]:8000: socket: too many open files
Get "http://localhost:8000/client/info": dial tcp 0.0.0.0:0->[::1]:8000: connect: connection refused

bash-3.2$ vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.10, 226.46
Duration      [total, attack, wait]             10.302s, 9.999s, 303.501ms
Latencies     [min, mean, 50, 90, 95, 99, max]  36.696µs, 196.132ms, 135.124µs, 336.513ms, 1.857s, 2s, 2.002s
Bytes In      [total, mean]                     309526, 44.22
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           33.33%
Status Codes  [code:count]                      0:4667  200:2333
Error Set:
Get "http://localhost:8000/client/info": dial tcp: lookup localhost: no such host
Get "http://localhost:8000/client/info": dial tcp 0.0.0.0:0->[::1]:8000: socket: too many open files
Get "http://localhost:8000/client/info": dial tcp 0.0.0.0:0->[::1]:8000: connect: connection refused
Get "http://localhost:8000/client/info": context deadline exceeded (Client.Timeout exceeded while awaiting headers)
```

По результату видна пропускная способность варьируется от 181 до 329 запросов в секунду

При этом в `Error Set` появились сетевые ошибки:

- `connection refused` — в какой то момент `vegeta` не смогла подключиться к серверу, потому что сервер перестал слушать 8000 порт или был переполнен бэклог сокета
- `lookup [localhost](http://localhost): no such host` — проблемы с DNS на хосте, где запускается нагрузочный тест. Из-за того, что в targets мы указали localhost вместо IP, нагрузочный инструмент будет резолвить IP на каждый запрос.
- `context deadline exceeded (Client.Timeout exceeded while awaiting headers)` — время ожидания ответа от сервиса превысило заданный таймаут. В нашем случае это 2 секунды.

Поправим targets.txt и заменим [`localhost`](http://localhost) на `127.0.0.1` и снова прогоним тесты, что бы исключить DNS резолвер из тестов.

```bash
bash-3.2$ vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.11, 699.94
Duration      [total, attack, wait]             10.001s, 9.998s, 2.516ms
Latencies     [min, mean, 50, 90, 95, 99, max]  652.715µs, 3.101ms, 1.142ms, 2.834ms, 5.049ms, 67.808ms, 148.061ms
Bytes In      [total, mean]                     928696, 132.67
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           100.00%
Status Codes  [code:count]                      200:7000
Error Set:

bash-3.2$ vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.11, 695.00
Duration      [total, attack, wait]             10.072s, 9.998s, 73.584ms
Latencies     [min, mean, 50, 90, 95, 99, max]  889.85µs, 32.573ms, 15.713ms, 85.677ms, 124.493ms, 177.087ms, 205.517ms
Bytes In      [total, mean]                     928696, 132.67
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           100.00%
Status Codes  [code:count]                      200:7000
Error Set:

bash-3.2 vegeta attack -targets ./targets.txt -timeout 2s -duration 10s -rate 700/1s | vegeta report

Requests      [total, rate, throughput]         7000, 700.02, 695.00
Duration      [total, attack, wait]             10.066s, 10s, 66.471ms
Latencies     [min, mean, 50, 90, 95, 99, max]  787.254µs, 25.855ms, 3.958ms, 113.163ms, 147.38ms, 192.133ms, 243.09ms
Bytes In      [total, mean]                     928180, 132.60
Bytes Out     [total, mean]                     0, 0.00
Success       [ratio]                           99.94%
Status Codes  [code:count]                      0:4  200:6996
Error Set:
Get "http://127.0.0.1:8000/client/info": read tcp 127.0.0.1:64451->127.0.0.1:8000: read: connection reset by peer
Get "http://127.0.0.1:8000/client/info": read tcp 127.0.0.1:64452->127.0.0.1:8000: read: connection reset by peer
Get "http://127.0.0.1:8000/client/info": read tcp 127.0.0.1:64455->127.0.0.1:8000: read: connection reset by peer
Get "http://127.0.0.1:8000/client/info": read tcp 127.0.0.1:64480->127.0.0.1:8000: read: connection reset by peer
```

После изменений в targets.txt исчезли сетевые ошибки и пропускная способность значительно выросла и в целом держится в районе 700 запросов в секунду. Таким образом мы исключили из тестов часть инфраструктуры и смогли более точно протестировать сервис. Я хочу донести идею, что тормозом может являться не только код вашего сервиса но и инфраструктура где он работает. Это может быть балансировщик, база данных, сервер самого сервиса, операционная система или проблемы с нагрузочным инструментом. Что бы точнее понимать, что тормозит надо получать как больше метрик со всех компонентов системы, что бы на графиках увидеть где именно у нас проблемы.

Вы разобрались, как можно протестировать один URL. Но такой тест сложно применить для эмуляции нагрузки настоящих пользователей, ведь они обращаются к разным URL. Разработчику приходится одновременно обстреливать разные URL, потому что бизнес-логика одних URL-обработчиков может влиять на производительность других. Например, обработчик, который выгружает статистику по просмотрам фильмов, делает «тяжёлые» запросы к SQL-базе и использует много ресурсов сервера. Их будет не хватать для обработчиков, которые выдают список фильмов. В то же время, дёргать все URL подряд — не самая лучшая идея. Это не отразит поведение пользователей. 

Один из вариантов решения проблемы: выгрузить логи сервера за сутки и на их основе сгенерировать цели для обстрела.