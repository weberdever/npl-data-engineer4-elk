# Запуск одной ноды elasticsearch

1. Заходим в директорию ```cd 01_prototype_by_docker_compose/01_elasticsearch_single_node```
2. Делаем ```docker-compose up```
3. Ждем примерно 30 секунд открываем в браузере <http://localhost:9200/> или делаем в консоли ```curl localhost:9200```

    Результат должен быть таким:
    ```
    $ curl localhost:9200
    {
      "name" : "80141924116e",
      "cluster_name" : "es_newprolab",
      "cluster_uuid" : "RtW3G2Q5SWyPrrMO4Pbjsg",
      "version" : {
        "number" : "7.0.0-beta1",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "15bb494",
        "build_date" : "2019-02-13T12:30:14.432234Z",
        "build_snapshot" : false,
        "lucene_version" : "8.0.0",
        "minimum_wire_compatibility_version" : "6.7.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
      },
      "tagline" : "You Know, for Search"
    }
    ```

Такой ответ означает что наш elasticsearch запустился успешно и готов принимать запросы

## Параметры которые указаны в docker-compose.yaml
По сути это параметры elasticsearch которые мы можем указать в файле настроек elasticsearch.yml 
* ```http.cors.enabled=true``` и ```http.cors.allow-origin="*"```- настройки разрешающие делать запросы к REST API из некоторых клиентов, для большенства REST API клиентов будет работать и без этой настройки
* ```http.port=9200``` - порт который будет слушать elasticsearch внутри контейнера
* ```cluster.name=es_newprolab``` - имя кластера, по сути произвольная строка
* ```network.host=_site_``` - IP адрес интерфейса с которого elasticsearch будет принимать запросы. Константа _site_ - означает принимать запросы со всех интерфейсов которые не смотрят в интернет. Подробнее [тут](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/network.host.html)
* ```discovery.type=single-node``` - параметр сообщает серверу elasticsearch что в кластере только одна нода и не нужно пытаться искать другие ноды. Без этого параметра сервер не запустится без доп. нод
* ```bootstrap.memory_lock:true``` - запрещает уносить в swap память выделенную по elasticsearch
* Раздел ```ulimits``` в docker-compose.yaml необходим для коорректной работы с параметром ```bootstrap.memory_lock```
* ```ES_JAVA_OPTS=-Xms256m -Xmx256m``` - Ограничиваем HEAP elasticsearch, т.е. память потребляемая JVM для работы нашей ноды elasticsearch, будет не более 256mb. Для работы с тестовыми данными этого лимита вполне достаточно, но в случае появления ошибок "Cannot allocate memory" можно крутить этот параметр.
