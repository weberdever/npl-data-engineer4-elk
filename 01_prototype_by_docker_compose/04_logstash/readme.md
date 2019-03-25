# Запускаем logstash с тестовым pipeline

1. Заходим в директорию ```cd 01_prototype_by_docker_compose/04_logstash```
2. Делаем ```docker-compose up```
3. Стартует почти всегда долго. Ждем примерно 2-4 минуты открываем в браузере <http://localhost:9600/_node/stats/pipelines?pretty>. По ссылке видем статистику по нашему pipeline если обновить с интервалом 5-10 секунд увидем динамику по увеличению счетчиков.

## Логика pipeline
Наш демо pipeline реализует логику: 
1. Раз в 5-ть секунд logstash создает сообщение "Hello from Logstash" и пропускается по своему pipeline
2. Полученное сообщене выводим в консоль в читаемом виде. Чтобы посмотреть только логи нашего logstash нужно в текущей директории выполнить команду ```docker-compose logs --tail 0 -f logstash```. Подождав 5-ть секунд увидим примерно след. вывод и сообщения будут добавляться:
    ```
    $ docker-compose logs --tail 0 -f logstash
    logstash_1   | {
    logstash_1   |       "@version" => "1",
    logstash_1   |           "host" => "144db88146e7",
    logstash_1   |        "message" => "Hello from Logstash",
    logstash_1   |     "@timestamp" => 2019-03-24T18:45:43.894Z
    logstash_1   | }
    logstash_1   | {
    logstash_1   |       "@version" => "1",
    logstash_1   |           "host" => "144db88146e7",
    logstash_1   |        "message" => "Hello from Logstash",
    logstash_1   |     "@timestamp" => 2019-03-24T18:45:48.894Z
    logstash_1   | }
    ```
3. Кроме вывода в stdout,  pipeline пишет данные в elasticsearch в индекс newprolab-heartbeat. Посмотреть его содержимое мы можем в kibana или выполнив команду ```curl "http://localhost:9200/newprolab-heartbeat/_search?pretty"```. Результат будет примерно такой:
	```
	$ curl "http://localhost:9200/newprolab-heartbeat/_search?pretty"
	{
      "took" : 3,
      "timed_out" : false,
      "_shards" : {
        "total" : 1,
        "successful" : 1,
        "skipped" : 0,
        "failed" : 0
      },
      "hits" : {
        "total" : {
          "value" : 2,
          "relation" : "eq"
        },
        "max_score" : 1.0,
        "hits" : [
          {
            "_index" : "newprolab-heartbeat",
            "_type" : "_doc",
            "_id" : "zPUNsWkBCTG2l6hRKzSA",
            "_score" : 1.0,
            "_source" : {
              "@version" : "1",
              "host" : "144db88146e7",
              "message" : "Hello from Logstash",
              "@timestamp" : "2019-03-24T18:53:42.238Z"
            }
          },
          {
            "_index" : "newprolab-heartbeat",
            "_type" : "_doc",
            "_id" : "zfUNsWkBCTG2l6hRNzSe",
            "_score" : 1.0,
            "_source" : {
              "@version" : "1",
              "host" : "144db88146e7",
              "message" : "Hello from Logstash",
              "@timestamp" : "2019-03-24T18:53:47.237Z"
            }
          }
        ]
      }
    }
    
	```

## Изменения в docker-compose.yaml
1. Изменены имена сервисов для elasticsearch. Были elastic_1 и elastic_2, стали elastic-1 и elastic-2. Заменили "_" на "-", т.к. logstash считает что URL с нижним подчеркиванием является не валидным и стартовать не будет. 
2. Добавлена конфигурация для logstash. Подробнее про конфигурацию текущего демо-pipline расписано в директории текущего репозитория 05_logstash/01_intro 
3. В файле docker-compose.yaml секция определения сервиса для logstash выглядит иначе в отличии от elasticsearch и kibana, используем не конкретный docker-образ, а docker-файл:
    ```
    build:
        context: .
        dockerfile: Dockerfile-logstash
    ```
    При старте сервиса logstash собирается новый образ из файла Dockerfile-logstash, т.к. нам необходим плагин которого нет в исходном образе. В результате мы плучили образ такой же как ```docker.elastic.co/logstash/logstash:7.0.0-beta1``` + плагин prune.
4. В параметре ```volumes``` перечислены все файлы и папки которые мы монтируем внутрь контейнера, чтобы запущенный logstash мог прочитать те конфиги которые мы написали. Абсолютные пути куда монтируем файлы могут отличаться если у вас будет другой базовый образ отличный от ```docker.elastic.co/logstash/logstash:7.0.0-beta1```.
