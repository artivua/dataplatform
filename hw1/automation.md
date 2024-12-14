# Инструкция по установке Ansible и применению Playbook

## Что делает данный playbook:
1. Создает пользователя hadoop на всех нодах с указанным паролем.
2. Генерирует SSH ключи на namenode и разворачивает их на все ноды для беспарольного доступа.
3. Устанавливает OpenJDK 11.
4. Скачивает, распаковывает и настраивает Hadoop (core-site.xml, hdfs-site.xml, workers, переменные окружения).
5. Форматирует namenode.
6. Запускает и останавливает сервисы Hadoop (по желанию).
7. На jump-node устанавливает htpasswd, создает файлы конфигурации nginx для аутентификации и проксирования UI NameNode и SecondaryNameNode, затем рестартует nginx.

## Предварительные требования

- Операционная система: Linux (Debian/Ubuntu/CentOS и т.д.)
- Доступ к интернету для установки необходимых пакетов
- Доступ по SSH к управляемым хостам

## Использование
1. Установка Ansible (На примере Ubuntu/Debian):
```bash
sudo apt update
sudo apt install ansible -y
```

2. Запуск playbook
```bash
ansible-playbook -i hosts.ini automation.yml
```
