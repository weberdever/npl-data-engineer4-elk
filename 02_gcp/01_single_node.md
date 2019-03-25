# Настройка одной ноды elasticsearch
1. Создаем ВМ типа g1-small или n1-standart-1, более мощные не имеет смысла создавать для этого теста. 
    Не забываем добавить разрешение в брандмауэр или  добавляем тег сети "elasticsearch". В случае с тегом нужно будет настроить разрешающее правило для доступа из внешних сетей к вашим нодам elasticsearch к порту 9200, либо реализовать более сложные схемы с VPN обзор которых выходит за рамки текущего руководства.
2. Заходим на ВМ по ssh. Дальше все команды будут выполнятся на текущей ВМ.
3. Переходим под root вводим команду ```sudo -s```
4. Устанавливаем java и сразу же обновляем ее до 11. 
    ```apt-get install default-jdk openjdk-11-jre```
    Часто замечаем что elasticsearch часто быстро начинает использовать новые возможности java и иногда дает прирост по производительности или стабильности работы GC. 
5. Проверяем что java установилась успешно
    ```java --version```
6. Добавляем в сприсок репозиториев системы репозиторий с elasticsearch 7.
    Берем из документации команды https://www.elastic.co/guide/en/elasticsearch/reference/7.x/install-elasticsearch.html (для debian)

    ```
    wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
    echo "deb https://artifacts.elastic.co/packages/7.x-prerelease/apt stable main" |  tee -a /etc/apt/sources.list.d/elastic-7.x.list
    apt-get install apt-transport-https
    ```
    От версии к версии могут отличаться.
7. Устанавливаем elasticsearch
  ```
   apt-get update
   apt-get install elasticsearch
  ```
8. Запускаем elasticsearch 
     ```service elasticsearch start```

9. Проверяем что elasticsearch успешно запущен 
    ```curl http://localhost:9200```
    Пример ответа:
    ```
    $ curl http://localhost:9200
    {
        "name" : "elasticsearch-npl-1",
        "cluster_name" : "elasticsearch",
        "cluster_uuid" : "3CPwfR7oTKudzorDcKrVAQ",
        "version" : {
            "number" : "7.0.0-beta1",
            "build_flavor" : "default",
            "build_type" : "deb",
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

10. Снимаем ограничения по работе с памятью для elasticsearch
    ```systemctl edit elasticsearch```
    Вписываем две строки, сохраняем и выходим:

    ```
    [Service]
    LimitMEMLOCK=infinity
    ```
    Выполняем ```systemctl daemon-reload```
    Подробнее про эту операцию по [ссылке](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/setting-system-settings.html#systemd)

11. Сейчас пока наш elasticsearch недоступен из внешней сети, давайте исправим это. 
      Правим файл ```/etc/elasticsearch/elasticsearch.yml``` добавляем/изменняем след. параметры:

      - ```http.cors.enabled: true``` и ```http.cors.allow-origin: "*"```- настройки разрешающие делать запросы к REST API из некоторых клиентов, для большенства REST API клиентов будет работать и без этой настройки
      - ```http.port: 9200``` - порт который будет слушать elasticsearch внутри контейнера
      - ```cluster.name: es_newprolab``` - имя кластера, по сути произвольная строка
      - ```network.host: _site_``` - IP адрес интерфейса с которого elasticsearch будет принимать запросы. Константа _site_ - означает принимать запросы со всех интерфейсов которые не смотрят в интернет. Подробнее [тут](https://www.elastic.co/guide/en/elasticsearch/reference/7.0/network.host.html)
      - ```discovery.type: single-node``` - параметр сообщает серверу elasticsearch что в кластере только одна нода и не нужно пытаться искать другие ноды. Без этого параметра сервер не запустится без доп. нод
      - ```bootstrap.memory_lock: true``` - запрещает уносить в swap память выделенную по elasticsearch

12. Можно ограничить HEAP меньшими разрмерами в файле /etc/elasticsearch/jvm.options.
       ```ES_JAVA_OPTS=-Xms512m -Xmx512m``` -  для работы с тестовыми данными этого лимита вполне достаточно, но в случае появления ошибок "Cannot allocate memory" можно крутить этот параметр

13. Перезагружаем elasticsearch ```service elasticsearch restart```

Elasticsearch готов к тестовой эксплуатации!