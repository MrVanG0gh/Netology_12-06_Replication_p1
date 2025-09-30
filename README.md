# Домашнее задание к занятию "Репликация и масштабирование. Часть 1" - `Иншаков Владимир`

---

## Задание 1
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

Ответить в свободной форме.

## Решение 1

Основные отличия режимов репликации Master-Slave и Master-Master:

- чтение данных в БД возможно в Master и в Slave, а запись/изменение/удаление данных - только в Master;
- Master-Master сложнее в настройке;
- в Master-Master возможны конфликты при одновременной записи данных, в Master-Slave они исключены;

---

## Задание 2
Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.

## Решение 2

Разворачивать инстансы БД MySQL буду на базе Docker контейнеров при помощи файла docker-compose.yml:

```
services:
  mysql-master:
    image: 'mysql:8.0'
    container_name: mysql-master
    env_file: .env
    volumes:
      - ./master/my.cnf:/etc/my.cnf
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3306:3306

  mysql-slave:
    image: 'mysql:8.0'
    container_name: mysql-slave
    env_file: .env
    volumes:
      - ./slave/my.cnf:/etc/my.cnf
    environment:
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      - 3307:3306
    depends_on:
      - mysql-master

```

my_cnf for Master
```
[mysqld]
server-id=1
binlog_format=ROW
log-bin
```

my_cnf for Slave
```
[mysqld]
server-id=2
```

![Screen_1](https://github.com/MrVanG0gh/Netology_12-06_Replication_p1/blob/main/Screenshots/Screenshot_1.png)

Контейнеры запущены

![Screen_3](https://github.com/MrVanG0gh/Netology_12-06_Replication_p1/blob/main/Screenshots/Screenshot_3.png)
![Screen_4](https://github.com/MrVanG0gh/Netology_12-06_Replication_p1/blob/main/Screenshots/Screenshot_4.png)

Dbeaver успешно подключился к обоим инстансам. Дополнительно произведены донастройки пользователей и режимов репликации. Для Slave были указаны master_log_pos и master_log_file. Были проверены режимы работы серверов.

SQL-script for Master
```
CREATE USER 'repl'@'%' IDENTIFIED WITH mysql_native_password BY 'slaverepl';

GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';

SHOW GRANTS FOR repl@'%';

SHOW MASTER STATUS;
```

SQL-script for Slave
```
CHANGE MASTER TO MASTER_HOST='mysql-master', MASTER_USER='repl', MASTER_PASSWORD='slaverepl', MASTER_LOG_FILE='5f6f48d8e585-bin.000003', MASTER_LOG_POS=652;

START SLAVE;

SHOW SLAVE STATUS;
```

---