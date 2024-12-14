# Руководство по установке и настройке Hadoop

## Предварительные требования

- Установленная Java (OpenJDK 11 или новее)
- SSH доступ ко всем узлам
- Привилегии sudo
- Все файлы редактируем либо через vim либо через nano как вам удобно

## Шаг 1: Доступ к удаленному хосту

1. Подключитесь к jump-node используя PuTTY.
2. Оттуда перейдите на namenode:

   ```
   ssh 192.168.1.23
   ```

## Шаг 2: Создание пользователя Hadoop

На каждом узле (192.168.1.23, 192.168.1.24, 192.168.1.25):

```bash
sudo adduser hadoop  # Используйте сложный пароль, в нашем случае поставлен 'h@DooP$' в качестве пароля
# и дальше все действия продолжаем через пользователь hadoop
sudo su hadoop
```

## Шаг 3: Генерация и распространение SSH ключей

На всех узлах генерируем ssh ключи. Снизу пример для namenode (192.168.1.23):

```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
ssh-copy-id hadoop@192.168.1.23
ssh-copy-id hadoop@192.168.1.24
ssh-copy-id hadoop@192.168.1.25
```

Проверка беспарольного SSH:

```bash
ssh hadoop@192.168.1.24
ssh hadoop@192.168.1.25
```

## Шаг 4: Установка Hadoop

На всех узлах (устанавливаем в ~ то есть в домашней директории):

```bash
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.4.0/hadoop-3.4.0.tar.gz
tar -xzvf hadoop-3.4.0.tar.gz
```

## Шаг 5: Настройка переменных окружения

На всех узлах добавьте в `~/.bashrc`:

```bash
export JAVA_HOME=$(readlink -f /usr/bin/java | sed 's:bin/java::')
export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_INSTALL=/home/hadoop/hadoop-3.4.0
export HADOOP_COMMON_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_HDFS_HOME=/home/hadoop/hadoop-3.4.0
export HADOOP_CONF_DIR=/home/hadoop/hadoop-3.4.0/etc/hadoop
export PATH=$PATH:/home/hadoop/hadoop-3.4.0/sbin:/home/hadoop/hadoop-3.4.0/bin

source ~/.bashrc
```

## Шаг 6: Настройка Hadoop

### Все конфиг файлы лежат в `/home/hadoop/hadoop-3.4.0/etc/hadoop`

1. Установите JAVA_HOME в `hadoop-env.sh`:

   ```bash
   echo "export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64" >> $HADOOP_CONF_DIR/hadoop-env.sh
   ```

2. Настройте `core-site.xml` на всех узлах:

   ```xml
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://192.168.1.23:9000</value>
     </property>
   </configuration>
   ```

3. Настройте `hdfs-site.xml` на NameNode:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.namenode.name.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/namenode</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

4. Настройте `hdfs-site.xml` на DataNodes:

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.datanode.data.dir</name>
       <value>file:///home/hadoop/hadoop-3.4.0/hdfs/datanode</value>
     </property>
   </configuration>
   ```

5. Настройте файл `workers` (/home/hadoop/hadoop-3.4.0/etc/hadoop/workers), нужно добавить остальные датаноды:

   ```
   team-5-dn-0
   team-5-dn-1
   ```

## Шаг 7: Создание директорий для HDFS

Только в NameNode:

```bash
mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/namenode
```

Во всех нодах (NameNode и DataNodes):

```bash
mkdir -p /home/hadoop/hadoop-3.4.0/hdfs/datanode
```

## Шаг 8: Форматирование NameNode

На NameNode:

```bash
hdfs namenode -format
```

## Шаг 9: Запуск сервисов Hadoop

На NameNode:

```bash
start-dfs.sh
```

## Шаг 10: Проверка установки

Проверка запущенных сервисов:

```bash
jps
```

Для Namenode должны увидеть
- 21922 Jps
- 21603 NameNode
- 21787 SecondaryNameNode
- 19728 DataNode

Для остальных Datanode
- 19728 DataNode
- 19819 Jps


Проверка версии Hadoop:

```bash
hadoop version
```

## Шаг 11: Остановка сервисов Hadoop

На NameNode:

```bash
stop-dfs.sh
```

## Шаг 12: Настройка аутентификации и прокси для Hadoop UI

## Установка htpasswd (все действия воспроизводятся на jump-node)

```bash
sudo apt update
sudo apt install apache2-utils
```

## Создание файла паролей

```bash
# Создаем файл паролей и добавляем пользователя
sudo htpasswd -c /etc/nginx/.htpasswd hadoop
# Вам будет предложено ввести пароль для hadoop
# пароль: h@DooP$

# Выдаем нужные права
sudo chmod 644 /etc/nginx/.htpasswd
```

Примечание: Флаг `-c` создаёт новый файл. Если файл уже существует и вам нужно добавить ещё одного пользователя, используйте команду без `-c`:

```bash
sudo htpasswd /etc/nginx/.htpasswd new_user
```

## Настройка Nginx для NameNode UI

```bash
# Создаем файл для namenode
touch /etc/nginx/sites-available/nn

# Заполняем файл и прописываем аутентификацию по паролю
sudo vim /etc/nginx/sites-available/nn
```

Содержимое файла:

```nginx
server {
    listen 9870 default_server;
    root /var/www/html;
    index index.html index.htm index.nginx-debian.html;
    server_name _;

    location / {
        proxy_pass http://team-5-nn:9870;
        auth_basic "Restricted Access";
        auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

```bash
# Добавляем UI namenode к доступным сайтам
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn
```

## Настройка для Secondary NameNode (порт 9868)


### Secondary NameNode

```bash
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/sn
sudo vim /etc/nginx/sites-available/sn # В двух местах меняем порты из 9870 на 9868
sudo ln -s /etc/nginx/sites-available/sn /etc/nginx/sites-enabled/sn
```
## Доступ к веб-интерфейсам

После настройки вы можете получить доступ к следующим веб-интерфейсам:

- NameNode UI: http://176.109.91.7:9870
- Secondary NameNode UI: http://176.109.91.7:9868
```
