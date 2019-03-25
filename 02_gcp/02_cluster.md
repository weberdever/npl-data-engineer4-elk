# Собираем кластер elasticsearch
У elasticsearch есть упрощающие поддержку плагины для обнаружения нод  кластера в рамках одного аккаунта в облаках и GCP не исключение. 

## Настраиваем созданую ранее ноду

Важно начать настройку кластера с ноды созданной ранее, в противном случае могут начаться проблемы со сборкой кластера и потребуются доп. действия.

1. Меняем настройки разрешений для доступа к API GCP. Для того чтобы изменить настройки доступа к API нужно остановить ВМ и после остановки зайти в редактирование ВМ, в разделе "Область действия доступа", выбрать "Настроить доступ к каждому API", далее в списке разрешений выставить для "Compute Engine" разрешения на чтение и запись.
2. Установим [плагин обнаружения нод для GCP](https://www.elastic.co/guide/en/elasticsearch/plugins/7.0/discovery-gce.html)
   ```/usr/share/elasticsearch/bin/elasticsearch-plugin install discovery-gce```
3. Вносим правки в конфиг /etc/elasticsearch/elasticsearch.yml:
   * ```node.name: es-1``` - уникальное имя нод в кластере 
   * Заменяем параметр ```discovery.type=single-node``` на ```discovery.seed_providers: gce``` - т.к. теперь у нас будет две ноды в кластере
   * ```cluster.initial_master_nodes=es-1``` - имя мастер ноды (значение ```node.name```) которая будет в ручную выбрана при старте кластера, информация из этой ноды используется только один раз при запуске кластера, когда нет данных о актуальном состоянии кластера
   * ```cloud.gce.project_id: [ваш ID проекта в GCP]``` - ID проекта в котором у вас будут обе ноды elasticsearch
   * ```cloud.gce.zone: ["europe-west3-c","europe-north1-a"]``` - имена зон где размещены ноды вашего кластера, в моем случае это europe-west3-c и europe-north1-a. Зон может быть несколько или одна.
4. Перезапускаем elasticsearch  ```service elasticsearch restart```
5. Проверяем доступность ноды запросив через браузер http://[внешний IP ВМ]:9200 
   Мастер нода - готова!

## Настраиваем второую ноду

5. Создаем вторую ВМ с аналогичным характеристиками и параметрами как на предыдущем шаге с небольшими изменениями. 
   Плагину elasticsearch для gcp требуются особые разрешения для обнаружения нод. При создании ВМ, в разделе "Профиль и API-доступ" нужно выбрать "Настроить доступ к каждому API", далее в списке разрешений выставить для "Compute Engine" разрешения на чтение и запись.
   Как создать ВМ для elasticsearch с помощью консольных инструментов GCP описано на странице https://www.elastic.co/guide/en/elasticsearch/plugins/7.0/discovery-gce-usage-long.html
6. Повторяем все действия из первой части
7. Вносим правки в конфиг /etc/elasticsearch/elasticsearch.yml:
	- ```node.name: es-2```
    - ```discovery.seed_providers: gce```
    - ```cluster.initial_master_nodes=es-1```
    - ```cloud.gce.project_id: [ваш ID проекта в GCP]```
    - ```cloud.gce.zone: ["europe-west3-c","europe-north1-a"]```
  8. Перезапускаем elasticsearch  ```service elasticsearch restart```
  9. Через минуту(с запасом) кластер должен будет собраться. Проверить что кластер собрался можно отправив запрос на один из IP elasticsearch
     ```
     $ curl "http://[IP_elasticsearch]:9200/_cat/nodes?v"
     ip          heap.percent ram.percent cpu load_1m load_5m load_15m node.role master name
     10.166.0.2             7          96   0    0.04    0.01     0.00 mdi       -      es-2
     10.156.0.62           31          51   0    0.00    0.00     0.00 mdi       *      es-1
     ```
Если по какой-то причине результат отличается можно воспользываться разделом troubleshooting.md или обратиться за поддержкой в slack
     

