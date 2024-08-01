
Learning PostgreSQL
===================

[Severalnines: Understanding the PostgreSQL](https://severalnines.com/blog/understanding-postgresql-architecture/)
[The Internals of PostgreSQL](https://www.interdb.jp/pg/)
[WAL в PostgreSQL: 1. Буферный кеш](https://habr.com/ru/companies/postgrespro/articles/458186/)
[WAL в PostgreSQL: 2. Журнал предзаписи](https://habr.com/ru/companies/postgrespro/articles/459250/)
[WAL в PostgreSQL: 3. Контрольная точка](https://habr.com/ru/companies/postgrespro/articles/460423/)
[WAL в PostgreSQL: 4. Настройка журнала](https://habr.com/ru/companies/postgrespro/articles/461523/)
https://pgtune.leopard.in.ua/

su - postgres
```sql
psql [OPTION]... [DBNAME [USERNAME]]
    psql -h <hostname> -p 5432 -d <dbname> -U <username> -W
```
Из psql выполнить команду bash:
\! <command>
Из bash выпонить команду psql:
echo '<command>' | psql

```sql
\set AUTOCOMMIT off
\echo :AUTOCOMMIT
\conninfo - information about the current database connection
\password - сброс пароля
\d - (describe) выводит список всех объектов базы данных
```
```sql
select version();
```
```sql
show transaction_isolation;
set transaction_isolation TO <level>
DEFAULT | "read committed" | "read uncommitted" | "repeatable read" | serializable
```
```shell
netstat -a | grep postgres
netstat -na | grep 5432
```
```shell
pg_lsclusters  # show information about all PostgreSQL clusters
```
```shell
pg_ctlcluster  # start/stop/restart/reload a PostgreSQL cluster
    pg_ctlcluster [options] <cluster-version> <cluster-name action>
    where action = start|stop|restart|reload|promote
pg_dropcluster  # completely delete a PostgreSQL cluster
    pg_dropcluster [--stop] <cluster-version> <cluster-name>
pg_createcluster  # create a new PostgreSQL cluster
    pg_createcluster [options] <version> <name> [-- initdb options]
```
```sql
postgres=# select pg_backend_pid();
 pg_backend_pid
----------------
         217802
```
Разделяемая память - находится в IPC shared memory сегменте:

    - buffer pool - хранит страницы таблиц и индексов;
    - WAL buffer - лог транзакций;
    - commit log (clog) - кэш состояние транзакций.
```sql
postgres=# select oid, datname from pg_database order by oid;
  oid  |  datname
-------+-----------
     1 | template1
 13760 | template0
 13761 | postgres
```

Табличные пространства - отдельный каталог с точки зрения файловой системы.
По умолчанию:

    - pg_default - $PGDATA/base
    - pg_global - $PGDATA/global
Новые табличные пространства создаются в:
    - $PGDATA/tblspc
```sql
postgres=# select * from pg_tablespace;
 oid  |  spcname   | spcowner | spcacl | spcoptions
------+------------+----------+--------+------------
 1663 | pg_default |       10 |        |
 1664 | pg_global  |       10 |        |
```
Настройка удаленного соединения:
    - postgresql.conf:
        listen_addresses = 'localhost' > '*'
    - pg_hba.conf:
        IPv4 local connections 127.0.0.1/32 > 0.0.0.0/0

Конфигурационные файлы:
    - config_file = '/etc/postgresql/<version>/main/postgresql.conf'
      show config_file;
    - hba_file = '/etc/postgresql/<version>/main/pg_hba.conf'
    - ident_file = '/etc/postgresql/<version>/main/pg_ident.conf'

View pg_settings:
    источник информации о параметрах и их значениях, а так же откуда эти значения берутся и как могут изменяться и применяться.
    select count(*) from pg_settings;
    \d+ pg_settings
View pg_file_settings:
    сводное содержимое файлов конфигурации сервера.
    select count(*) from pg_file_settings;
    \d+ pg_file_settings

Получить текущее значение параметров:
    - select setting||' x '||coalesce(unit,'units') from pg_settings where name='shared_buffers';
    - select setting||' x '||coalesce(unit,'units') from pg_settings where name='max_connections';
    - show max_connections;
    - select current_setting('max_connections');

Применить текущие значения параметров:
    - pg_ctl restart
    - pg_ctl reload
    - kill -HUP postmaster
    - pg_reload_conf()

Изменить текущее значение:
    с сохранением значения:
        - в файле с параметрами
        - alter system set <name>=’<value>’
            записывает параметры в postgresql.auto.conf но не применяет их
    для текущей сессии:
        - set <parameter> to '<value>';
        - select set_config('<name>', '<value>', false);

Сбросить значения параметров:
    - alter system reset <name>
    - alter system reset all

Нагрузочное тестирование:
https://github.com/Percona-Lab/sysbench-tpcc (требует установки https://github.com/akopytov/sysbench)
pgbench

"relation" (отношение) используется для обозначения объектов, таких как таблицы, индексы, представления и секвенции.
Типы relation:
r = ordinary table,
i = index,
S = sequence,
v = view,
m = materialized view,
c = composite type,
t = TOAST table,
f = foreign table

## База данных (Database)
По умолчанию в кластере PostgreSQL создаются БД:
    - template0 (для восстановления из резервной копии, по умолчанию даже нет прав на connect)
    - template1 (используется как шаблон для создания БД, в нем имеет смысл делать некие действия, которые не хочется делать каждый раз при создании новых БД)
    - postgres (первая БД для регулярной работы, создается по умолчанию, хорошая практика - так же не использовать)

https://www.postgresql.org/docs/current/sql-createdatabase.html
```sql
CREATE DATABASE name
    [ WITH ] [ OWNER [=] user_name ]
           [ TEMPLATE [=] template ]
           [ ENCODING [=] encoding ]
           [ STRATEGY [=] strategy ]
           [ LOCALE [=] locale ]
           [ LC_COLLATE [=] lc_collate ]
           [ LC_CTYPE [=] lc_ctype ]
           [ ICU_LOCALE [=] icu_locale ]
           [ ICU_RULES [=] icu_rules ]
           [ LOCALE_PROVIDER [=] locale_provider ]
           [ COLLATION_VERSION = collation_version ]
           [ TABLESPACE [=] tablespace_name ]
           [ ALLOW_CONNECTIONS [=] allowconn ]
           [ CONNECTION LIMIT [=] connlimit ]
           [ IS_TEMPLATE [=] istemplate ]
           [ OID [=] oid ]
```
psql -d <database>  # подключение к определенной БД
# \l  # список БД
# \c  # к какой базе данных и под каким пользователем подключена текущая сессия
# \c <database>  # подключиться к другой БД
```sql
select oid, datname, datistemplate, datallowconn
from pg_database;
```
## Схема (Schema)
Иногда называется namespace
По умолчанию в каждой базе данных есть схемы:
    - pg_catalog
    - information_schema
    - public
    - pg_temp_<n>

Полное имя объекта: <schema>.<object>
Текущая схема: select current_schema();
Строка поиска определяется параметром search_path. pg_catalog всегда неявно первый.

\dn - список схем
```sql
CREATE SCHEMA schema_name [ AUTHORIZATION role_specification ] [ schema_element [ ... ] ]
CREATE SCHEMA AUTHORIZATION role_specification [ schema_element [ ... ] ]
CREATE SCHEMA IF NOT EXISTS schema_name [ AUTHORIZATION role_specification ]
CREATE SCHEMA IF NOT EXISTS AUTHORIZATION role_specification

where role_specification can be:
    user_name
  | CURRENT_ROLE
  | CURRENT_USER
  | SESSION_USER
```
```sql
select * from pg_namespace;
```
## Пользователь (User)

`\h  #create user`

`\d pg_user`
```sql
select * from pg_user;
```
```sql
CREATE USER name [ [ WITH ] option [ ... ] ]

where option can be:

      SUPERUSER | NOSUPERUSER
    | CREATEDB | NOCREATEDB
    | CREATEROLE | NOCREATEROLE
    | INHERIT | NOINHERIT
    | LOGIN | NOLOGIN
    | REPLICATION | NOREPLICATION
    | BYPASSRLS | NOBYPASSRLS
    | CONNECTION LIMIT connlimit
    | [ ENCRYPTED ] PASSWORD 'password' | PASSWORD NULL
    | VALID UNTIL 'timestamp'
    | IN ROLE role_name [, ...]
    | IN GROUP role_name [, ...]
    | ROLE role_name [, ...]
    | ADMIN role_name [, ...]
    | USER role_name [, ...]
    | SYSID uid
```
URL: https://www.postgresql.org/docs/14/sql-createuser.html


## Права (Privileges)

grant/revoke на объект:
    - relation [attribute, …]
    - schema
    - database


# Резервное копирование и восстановление
## Логическое копирование
https://www.postgresql.org/docs/14/sql-copy.html

COPY — copy data between a file and a table
```sql
COPY table_name [ ( column_name [, ...] ) ]
    FROM { 'filename' | PROGRAM 'command' | STDIN }
    [ [ WITH ] ( option [, ...] ) ]
    [ WHERE condition ]

COPY { table_name [ ( column_name [, ...] ) ] | ( query ) }
    TO { 'filename' | PROGRAM 'command' | STDOUT }
    [ [ WITH ] ( option [, ...] ) ]

where option can be one of:

    FORMAT format_name
    FREEZE [ boolean ]
    DELIMITER 'delimiter_character'
    NULL 'null_string'
    HEADER [ boolean ]
    QUOTE 'quote_character'
    ESCAPE 'escape_character'
    FORCE_QUOTE { ( column_name [, ...] ) | * }
    FORCE_NOT_NULL ( column_name [, ...] )
    FORCE_NULL ( column_name [, ...] )
    ENCODING 'encoding_name'
```

Архивирование - утилита PG_DUMP
выдает на консоль или в файл либо SQL-скрипт,
либо архив в специальном формате с оглавлением
поддерживает параллельное выполнение
позволяет ограничить набор выгружаемых объектов
(таблицы --table, схемы --schema-only, данные --data-only и т.п.)
!!!по умолчанию не создает tablespace и юзеров
$ pg_dump -d backup --create
$ pg_dump -d backup --create | gzip > backup.gz
$ pg_dump -d backup -Fc >1.gz - для pg_restore

Восстановление - psql, так как это простой SQL скрипт
$psql < 1.sql
• заранее должны быть созданы роли и табличные пространства
или если архив с оглавлением — pg_restore
• (позволяет ограничить набор объектов при восстановлении)
• поддерживает параллельное выполнение
• заранее должны быть созданы роли, табличные пространства и БД!!!
• после восстановления имеет смысл выполнить сбор статистики (ANALIZE)
$echo "create database backup;" | psql
$pg_restore 2.gz

PG_DUMPALL
сохраняет весь кластер, включая роли и табличные пространства
выдает на консоль или в файл SQL-скрипт
параллельное выполнение не поддерживается, но можно выгрузить
только глобальные объекты и воспользоваться pg_dump
$pg_dumpall >backup.sql
$pg_dumpall --clean --globals-only >globals.sql
$pg_dumpall --clean --schema-only >schema.sql
Восстановление
$psql < backup.sql

## Физическое резервирование

Используется механизм восстановления после сбоя:
копия данных и журналы предзаписи
+ скорость восстановления
+ можно восстановить кластер на определенный момент времени
− нельзя восстановить отдельную базу данных, только весь кластер
− восстановление только на той же основной версии и архитектуре

Холодное - когда БД остановлена
• сервер корректно остановлен (необходимы только файлы
данных)
• некорректно выключенный (файлы данных и wal сегменты)
Горячее - на работающем экземпляре
• необходимы как файлы данных, так и wal сегменты, причем
нужно проконтролировать, чтобы сервер сохранил все wal файлы
на время копирования основных данных

Создание автономной копии

Автономная копия содержит и файлы данных, и WAL
Резервное копирование — утилита pg_basebackup
• подключается к серверу по протоколу репликации
• выполняет контрольную точку
• переключается на следующий сегмент WAL
• копирует файловую систему в указанный каталог
• переключается на следующий сегмент WAL
• сохраняет все сегменты WAL, сгенерированные за время копирования
Восстановление
• разворачиваем созданную автономную копию
• запускаем сервер

Протокол репликации
• получение потока журнальных записей
• команды управления резервным копированием и репликацией
Обслуживается процессом wal_sender
Параметр wal_level = replica
Слот репликации
• серверный объект для получения журнальных записей
• помнит, какая запись была считана последней
• сегмент WAL не удаляется, пока он полностью не прочитан через слот
SELECT name, setting FROM pg_settings WHERE name IN ('wal_level','max_wal_senders');
Необходимо настроить файервол в файле pg_hba.conf
SELECT type, database, user_name, address, auth_method
FROM pg_hba_file_rules() WHERE database = '{replication}';

Создадим 2 кластер
$pg_createcluster -d /var/lib/postgresql/10/main2 10 main2
Удалим оттуда файлы
$rm -rf /var/lib/postgresql/10/main2
Сделаем бэкап нашей БД
$pg_basebackup -p 5432 -D /var/lib/postgresql/10/main2
Зададим другой порт
$echo 'port = 5433' >> /var/lib/postgresql/10/main2/postgresql.auto.conf
Стуртуем кластер
$pg_ctlcluster 10 main2 start
Смотрим как стартовал
$pg_lsclusters

##Архив журналов

Файловый архив
• сегменты WAL копируются в архив по мере заполнения
• механизм работает под управлением сервера
• неизбежны задержки попадания данных в архив
Потоковый архив
• в архив постоянно записывается поток журнальных записей
• требуются внешние средства
• задержки минимальны

Файловый архив журналов

Процесс archiver
Параметры
SELECT name, setting FROM pg_settings WHERE name IN
('archive_mode','archive_command','archive_timeout');
• ALTER SYSTEM SET archive_mode = on
• ALTER SYSTEM SET archive_command - команда shell для копирования сегмента WAL в
отдельное хранилище
• ALTER SYSTEM SET archive_timeout - максимальное время для переключения на новый
сегмент WAL
• требуется рестарт сервера
Алгоритм
• при заполнении сегмента WAL вызывается команда archive_command
• если команда завершается со статусом 0, сегмент удаляется
• если команда возвращает не 0 (в частности, если команда не задана),сегмент остается до
тех пор, пока попытка не будет успешной

Потоковый архив журналов

Утилита pg_receivewal
• подключается по протоколу репликации (можно использовать слот)
• и направляет поток записей WAL в файлы-сегменты
• стартовая позиция — начало сегмента, следующего за последним заполненным
сегментом, найденным в каталоге,
• или начало текущего сегмента сервера, если каталог пустой
• в отличие от файлового архива, записи пишутся постоянно
• при переходе на новый сервер надо перенастраивать параметры

Варианты бэкапа:
• [Разгоняем бэкап. Лекция Яндекса / Блог компании Яндекс / Хабр](https://habr.com/ru/company/yandex/blog/415817/)
• [Многоярусный бэкап PostgreSQL с помощью Barman и синхронного переноса журналов транзакций / Блог компании Яндекс.Деньги / Хабр](https://habr.com/ru/company/yamoney/blog/333844/)
• barman
• wal-e
• wal-g
• BART
• pg_probackup

WAL-G
https://github.com/wal-g/wal-g
позволяет хранить wal как в файлах, так и в облачных бакетах
• Amazon S3
• Google Cloud Storage
• Azure Storage
Данные хранятся в облаке, поэтому мониторить ничего не нужно, это уже проблема облачного сервиса, как обеспечить  доступность ваших данных, когда они вам нужны.

• бэкап с реплики
не нужно нагружать ваш мастер БД читающей нагрузкой, вы можете снять резервную копию с реплики
• инкрементный бэкап (дельта копии)
При настройке WAL-G вы указываете количество шагов на которые максимально отстоит от
base-бэкапа дельта-бэкап, и указываете политику дельта-копии. Либо вы делаете копию с
последней существующей дельты, либо делаете дельту от изначального полного бэкапа. Это
нужно на тот случай, когда у вас в базе данных всегда меняется одна и та же составляющая
БД, одни и те же данные постоянно изменяются.


## Репликация
Требуется, чтобы обеспечить высокую доступность и масштабируемость.

Виды репликации:
    - физическая
    - логическая

Физическая репликация
- мастер-слейв: поток данных только в одну сторону
- трансляция потока журнальных записей или файлов
журнала
- требуется двоичная совместимость серверов
- возможна репликация только всего кластера

базовая резервная копия — pg_basebackup
• разворачиваем резервную копию,
• создаем управляющий файл recovery.conf
(standby_mode = on)
• и запускаем сервер
• сервер восстанавливает согласованность
• и продолжает применять поступающие журналы
• доставка — поток по протоколу репликации или архив
WAL
• подключения (только для чтения) разрешаются
• сразу после восстановления согласованности

Допускаются на реплике
• запросы на чтение данных (select, copy to, курсоры)
• установка параметров сервера (set, reset)
• управление транзакциями (begin, commit, rollback...)
• создание резервной копии (pg_basebackup)
Не допускаются
• любые изменения (insert, update, delete, truncate, nextval...)
• блокировки, предполагающие изменение (select for update...)
• команды DDL (create, drop...), в том числе создание временных таблиц
• команды сопровождения (vacuum, analyze, reindex...)
• управление доступом (grant, revoke...)
• не срабатывают триггеры и пользовательские (advisory) блокировки
Vacuum?
• параметр hot_standby_feedback

Перевод реплики в состояние мастера
$ pg_ctl -w -D /var/lib/postgresql/10/main2 promote
• waiting for server to promote.... done
• server promoted
Получим 2 разных независимых сервера %)

Простое подключение бывшего мастера — не работает
• проблема потери записей WAL, не попавших на реплику из-за задержки
Восстановление «с нуля» из резервной копии
• на месте бывшего мастера разворачивается абсолютно новая реплика
• процесс занимает много времени (отчасти можно ускорить rsync)
Утилита pg_rewind
• «откатывает» потерянные записи WAL, заменяя соответствующие
• страницы на диске страницами с нового мастера
• есть ряд ограничений, ограничивающий применение
https://habr.com/ru/post/269889/

Начиная с версии 10, все необходимые настройки уже присутствуют по умолчанию:
● wal_level = replica;
● max_wal_senders
● разрешение на подключение в pg_hba.conf.
Создадим 2 кластер
$pg_createcluster -d /var/lib/postgresql/10/main2 10 main2
Удалим оттуда файлы
$rm -rf /var/lib/postgresql/10/main2
Сделаем бэкап нашей БД. Ключ -R создаст заготовку управляющего файла recovery.conf.
$pg_basebackup -p 5432 -R -D /var/lib/postgresql/10/main2
Зададим другой порт
$echo 'port = 5433' >> /var/lib/postgresql/10/main2/postgresql.auto.conf
Добавим параметр горячего резерва, чтобы реплика принимала запросы на чтение
$echo 'hot_standby = on' >> /var/lib/postgresql/10/main2/postgresql.auto.conf
Стартуем кластер
$pg_ctlcluster 10 main2 start
Смотрим как стартовал
$pg_lsclusters

Проверить состояние репликации:
SELECT * FROM pg_stat_replication \gx

Теперь переведем реплику из режима восстановления в обычный режим. Таким образом,
получим два самостоятельных, никак не связанных друг с другом сервера.
$pg_ctlcluster 10 main2 promote

## Логическая репликация
• поставщик-подписчик: поток данных возможен в обе
стороны
• информация о строках (уровень журнала logical)
• требуется совместимость на уровне протокола
• возможна выборочная репликация отдельных таблиц

• Встроенная логическая репликация доступна в версиях PostgreSQL, начиная с 10. Для
более ранних версий аналогичный функционал доступен в расширении pg_logical.
• Для передачи логических изменений (на уровне строк) используется протокол
репликации. Для работы такой репликации требуется установка уровня журнала logical.
• Другой способ организации логической репликации состоит в использовании триггеров
для перехвата изменений, помещения этой информации в очередь событий и передача
ее на другой сервер. Такой способ, однако, менее эффективен, и уходит в прошлое
(Slony-I).
• При логической репликации у сервера нет выделенной роли мастера или реплики, что
позволяет организовать в том числе и двунаправленную репликацию.

Публикующий сервер
• выдает изменения данных построчно в порядке их фиксации
• (реплицируются команды INSERT, UPDATE, DELETE), в 11 версии добавили TRANCATE
• возможна начальная синхронизация
• всегда используется слот логической репликации
• DDL не передаются, то есть таблицы-приемники на стороне подписчика надо создавать
вручную.
• применение изменений происходит без выполнения команд SQL и связанных с этим
накладных расходов на разбор и планирование, что уменьшает нагрузку на подписчика.
• параметр wal_level = logical
Подписчики
• получают и применяют изменения
• без разбора, трансформаций и планирования — сразу выполнение
• возможны конфликты с локальными данными

Режимы идентификации для изменения и удаления
• столбцы первичного ключа (по умолчанию)
• столбцы указанного уникального индекса с ограничением NOT NULL
• все столбцы
• без идентификации (по умолчанию для системного каталога)
Конфликты — нарушение ограничений целостности
• репликация приостанавливается до устранения конфликта вручную
• либо исправление данных,
• либо пропуск конфликтующей транзакции

Используем два сервера, полученные на предыдущей практике и настроим логическую
репликацию. Для этого нам понадобится дополнительная информация в журнале.
ALTER SYSTEM SET wal_level = logical;
Рестартуем кластер
$sudo pg_ctlcluster 10 main restart
На первом сервере создаем публикацию:
\c replica
CREATE TABLE test(i int);
CREATE PUBLICATION test_pub FOR TABLE test;
\dRp+

создадим подписку на втором экземпляре
\c replica
CREATE TABLE test(i int);
CREATE SUBSCRIPTION test_sub
CONNECTION 'host=localhost port=5432 user=postgres password=test dbname=replica'
PUBLICATION test_pub WITH (copy_data = false);
\dRs

состояние подписки
SELECT * FROM pg_stat_subscription \gx

вернем настройку
ALTER SYSTEM SET wal_level = replica;

Механизм репликации основан на передаче журнальных записей на
реплику и их применении
• трансляция потока записей или файлов WAL
Физическая репликация создает точную копию всего кластера
• однонаправленная
• требует двоичной совместимости
Логическая репликация передает изменения строк отдельных таблиц
• разнонаправленная
• совместимость на уровне протокола