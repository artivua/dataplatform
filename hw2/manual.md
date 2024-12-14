# Развертывание YARN

## Шаг 0. Требования

- HDFS Запущен через пользователя hadoop. Все дальнейшие инструкции должны быть выполнены из под данного пользователя.
- В переменную окружения PATH добавлены пути ```/home/hadoop/hadoop-3.4.0/sbin```, ```/home/hadoop/hadoop-3.4.0/bin```
- Опубликованы веб интерфейсы NameNode и Secondary NameNode как в этой инструкции (ДЗ1): https://github.com/szarema/Hadoop_hse  

## Шаг 1: Настройка названий нод, чтобы не вводить каждый раз ip адрес. 
В файл /etc/hosts на team@team-5-jn, team@team-5-nn, team@team-5-dn-00, team@team-5-dn-01 вставить следующие строчки в конец файла:
```
192.168.1.22 team-5-jn
192.168.1.23 team-5-nn
192.168.1.24 team-5-dn-0 
192.168.1.25 team-5-dn-1
```

## Шаг 2. Конфигурация
Все конфигурационные файлы лежат в директории
```bash
~/hadoop-3.4.0/etc/hadoop
```

1. На team-5-nn, team-5-dn-0, team-5-dn-1
в конфигурационный файл mapred-site.xml нужно добавить следующую информацию в блок ```<configuration></configuration>```, который изначально пустой.

```xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
	<property>
		<name>mapreduce.application.classpath</name>
		<value>$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
	</property>
</configuration>
```

2. На team-5-nn, team-5-dn-0, team-5-dn-1
в конфигурационный файл yarn-site.xml нужно добавить следующую информацию в блок ```<configuration></configuration>```, который изначально пустой.

```xml
<configuration>
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>
	<property>
		<name>yarn.nodemanager.env-whitelist</name>
		<value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME,PATH,LANG,TZ,HADOOP_MAPRED_HOME</value>
	</property>
</configuration>

```

## Шаг 3. Запуск YARN
На team-5-nn из директории
```bash
~/hadoop-3.4.0
```
выполнить следующую команду
```bash
/sbin/start-yarn.sh
```

## Шаг 5. Запуск historyserver
На team-5-nn из директории
```bash
~/hadoop-3.4.0
```
выполнить следующую команду
```bash
mapred --daemon start historyserver
```
## Шаг 6: Настройка Nginx для YARN и History Server

```bash
# Создаем файл для yarn
touch /etc/nginx/sites-available/yarn

# Заполняем файл и прописываем аутентификацию по паролю
sudo vim /etc/nginx/sites-available/yarn
```

Содержимое файла:

```nginx
server {
    listen 8088 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;

    location / {
        proxy_pass http://team-5-nn:8088;
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

```bash
# Добавляем UI namenode к доступным сайтам
sudo ln -s /etc/nginx/sites-available/yarn /etc/nginx/sites-enabled/yarn
```

Повторите аналогичные шаги для (в каждом файле в двух местах меняем порты):
- History Server (порт 19888)

### Доступ к веб-интерфейсам

После настройки вы можете получить доступ к следующим веб-интерфейсам:
- NameNode UI: http://176.109.91.7:9870
- Secondary NameNode UI: http://176.109.91.7:9868
- YARN ResourceManager: http://176.109.91.7:8088
- History Server: http://176.109.91.7:19888

## Шаг 5. Остановка всех сервисов

- Остановка History Server. На team-5-nn выполнить команду ```mapred --daemon stop historyserver``` Из директории ```~/hadoop-3.4.0```
- Остановка YARN. На team-5-nn выполнить команду ```./sbin/stop-yarn.sh``` Из директории ```~/hadoop-3.4.0```
- Остановка HDFS. На team-5-nn выполнить команду ```./sbin/stop-dfs.sh``` Из директории ```~/hadoop-3.4.0```
