# Домашнее задание к занятию "`Работа с данными (DDL/DML)`" - `Симаров Дмитрий`


### Задание 1

1.1. Поднимите чистый инстанс MySQL версии 8.0+. Можно использовать локальный сервер или контейнер Docker.

```
docker run --name mysql80 -e MYSQL_ROOT_PASSWORD=rootpass -p 3306:3306 -d mysql:8.0
```

1.2. Создайте учётную запись sys_temp.

```
docker exec -it mysql80 mysql -uroot -prootpass
CREATE USER 'sys_temp'@'localhost' IDENTIFIED BY 'pass1';
```

1.3. Выполните запрос на получение списка пользователей в базе данных. (скриншот)

```
SELECT user, host FROM mysql.user;
```
![DDL-DML_hw_13](https://github.com/arccos2x-lwr/DDL_DML-hw/blob/main/img/13.png)

1.4. Дайте все права для пользователя sys_temp.

```
GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

1.5. Выполните запрос на получение списка прав для пользователя sys_temp. (скриншот)

```
SHOW GRANTS FOR 'sys_temp'@'localhost';
```
![DDL-DML_hw_15](https://github.com/arccos2x-lwr/DDL_DML-hw/blob/main/img/15.png)

1.6. Переподключитесь к базе данных от имени sys_temp.

```
exit;
docker exec -it mysql80 mysql -usys_temp -ppass1
```

Для смены типа аутентификации с sha2 используйте запрос:

`ALTER USER 'sys_test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

```
ALTER USER 'sys_temp'@'localhost' IDENTIFIED WITH mysql_native_password BY 'pass1';
FLUSH PRIVILEGES;
```

1.6. По ссылке https://downloads.mysql.com/docs/sakila-db.zip скачайте дамп базы данных.

```
wget https://downloads.mysql.com/docs/sakila-db.zip
unzip sakila-db.zip
```

1.7. Восстановите дамп в базу данных.

```
docker cp sakila-db/sakila-schema.sql mysql80:/
docker cp sakila-db/sakila-data.sql mysql80:/

CREATE DATABASE sakila;
USE sakila;
SOURCE /sakila-schema.sql;
SOURCE /sakila-data.sql;
```

1.8. При работе в IDE сформируйте ER-диаграмму получившейся базы данных. При работе в командной строке используйте команду для получения всех таблиц базы данных. (скриншот)

```
SHOW TABLES;
```
![DDL-DML_hw_18](https://github.com/arccos2x-lwr/DDL_DML-hw/blob/main/img/18.png)

Результатом работы должны быть скриншоты обозначенных заданий, а также простыня со всеми запросами.

```
1.1
docker run --name mysql80 -e MYSQL_ROOT_PASSWORD=rootpass -p 3306:3306 -d mysql:8.0

1.2
docker exec -it mysql80 mysql -uroot -prootpass
CREATE USER 'sys_temp'@'localhost' IDENTIFIED BY 'pass1';

1.3
SELECT user, host FROM mysql.user;

1.3
GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;

1.4
SHOW GRANTS FOR 'sys_temp'@'localhost';

1.5
exit;
docker exec -it mysql80 mysql -usys_temp -ppass1
ALTER USER 'sys_temp'@'localhost' IDENTIFIED WITH mysql_native_password BY 'pass1';
FLUSH PRIVILEGES;

1.6
wget https://downloads.mysql.com/docs/sakila-db.zip
unzip sakila-db.zip

1.7
(второй терминал)
docker cp sakila-db/sakila-schema.sql mysql80:/
docker cp sakila-db/sakila-data.sql mysql80:/

(первый терминал)
CREATE DATABASE sakila;
USE sakila;
SOURCE /sakila-schema.sql;
SOURCE /sakila-data.sql;

1.8
SHOW TABLES;
```

### Задание 2

Составьте таблицу, используя любой текстовый редактор или Excel, в которой должно быть два столбца: в первом должны быть названия таблиц восстановленной базы, во втором названия первичных ключей этих таблиц.

```
SELECT 
    tab.table_name AS 'Название таблицы',
    COALESCE(
        GROUP_CONCAT(kcu.column_name ORDER BY kcu.ordinal_position SEPARATOR ', '),
        '-'
    ) AS 'Название первичного ключа'
FROM 
    information_schema.tables tab
LEFT JOIN 
    information_schema.table_constraints tco
    ON tab.table_schema = tco.table_schema
    AND tab.table_name = tco.table_name
    AND tco.constraint_type = 'PRIMARY KEY'
LEFT JOIN 
    information_schema.key_column_usage kcu
    ON tco.constraint_schema = kcu.constraint_schema
    AND tco.constraint_name = kcu.constraint_name
    AND tco.table_name = kcu.table_name
WHERE 
    tab.table_schema = 'sakila'
    -- Убираем фильтр по BASE TABLE, чтобы включить представления
GROUP BY 
    tab.table_name
ORDER BY 
    tab.table_name;
```

![DDL-DML_hw_21](https://github.com/arccos2x-lwr/DDL_DML-hw/blob/main/img/21.png)