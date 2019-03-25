# Запуск ELK с использованием docker-compose

В этой части мы научимся настраивать окружение для работы с ELK используя dokcer + docker-compose. А так же рассмотрим основные параметры конфигурации ELK.

## Необходимо установить на рабочий компьютер или на ВМ
* [Docker](https://docs.docker.com/install/#general-availability)
* [docker-compose](https://docs.docker.com/compose/install/)

## Команды которым будем пользоваться

1. Запуск контейнеров
    ```
    docker-compose up
    ```

2. Остановка контейнеров
    ```
    docker-compose down
    ```
    Предполагается что нужно выполнить после каждого этапа, иначе запущенные ранее контейнеры или набор контейнеров могут конфликтовать с новыми контейнерами. 

3. Просмотр запущенных контейнеров можно выполнить командой
    ```
    docker ps
    ```
    
## 01_elasticsearch_single_node
Настройка одной ноды elasticsearch

## 02_elasticsearch_cluster
Настройка кластера elasticsearch

## 03_kibana
Настройка kibana

## 04_logstash
Настройка logstash с тестовым pipeline
