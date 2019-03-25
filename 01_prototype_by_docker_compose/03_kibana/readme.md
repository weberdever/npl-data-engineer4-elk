# Запускаем kibana

1. Заходим в директорию ```cd 01_prototype_by_docker_compose/03_kibana```
2. Делаем ```docker-compose up```
3. Ждем примерно 1-2 минуты открываем в браузере <http://localhost:5601> 

##  Изменения в docker-compose.yaml
1. Добавлена конфигурация для kibana 
2. Только две ноды elasticsearch, исключительно в целях экономии ресурсов локальной машинки 