# Задания для самостоятельного изучения

## Сбор логов
Нужно реализовать сбор логов с настроенного стека ELK или других приложений через [FileBeat](https://www.elastic.co/guide/en/beats/filebeat/7.0/index.html). 

## Настройка очистки старых индексов
Нужно настроить livecycle policy для индексов в elasticsearch, чтобы автоматически удалялись данные при устаревании. Попробывать настроить политику по максимальному размеру индекса и по времени жизни. 

## Реализовать прием событий по http
Используя [input-плагин http](https://www.elastic.co/guide/en/logstash/current/plugins-inputs-http.html) для logstash реализовать логику принятия события в JSON формате, парсинг сообщения и запись в индекс elasticsearch в структурированном виде.

Пример события:
```
{
    "timestamp": 1553467261411,
    "name": "click_button",
    "color": "red",
    "price": 885.50,
    "uuid": "0a518dfc-2cb9-48d6-97e1-f6eda20d777e"
}
```

Пример отправки запроса к logstash:
```
curl -XPOST "http://[IP_адрес_logstash]:81" \
	-H 'Content-Type:application/json'
	-d '{"timestamp": 1553467261411, "name": "click_button","color": "red","price": 885.50,"uuid": "0a518dfc-2cb9-48d6-97e1-f6eda20d777e"}'
```
В качестве событий которые будут присылаться в logstash можно использовать пушку событий которую использовали для других практических работ. Конфиг logstash будет не сильно отличаться в зависимости от структуры событий.