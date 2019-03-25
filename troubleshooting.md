# Troubleshooting

## Упал сервер elasticsearch 
В логах ошибка "Native controller process has stopped - no new native processes can be started", скорее всего ваш elasticsearch убил OOM killer, нужно добавить JAVA HEAP. 
Если запускаете в docker-контейнерах, то из за особенностей логирования в контейнерах сообщения от OOM killer может не быть.