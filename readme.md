https://severalnines.com/blog/understanding-postgresql-architecture/
https://www.interdb.jp/pg/
https://habr.com/ru/companies/postgrespro/articles/458186/
https://habr.com/ru/companies/postgrespro/articles/459250/
https://habr.com/ru/companies/postgrespro/articles/460423/
https://habr.com/ru/companies/postgrespro/articles/461523/
https://pgtune.leopard.in.ua/

su - postgres
psql [OPTION]... [DBNAME [USERNAME]]
    psql -h <hostname> -p 5432 -d <dbname> -U <username> -W

\set AUTOCOMMIT off
\echo :AUTOCOMMIT
\conninfo - information about the current database connection
\password - сброс пароля
\d - (describe) выводит список всех объектов базы данных

select version();

show transaction_isolation;
set transaction_isolation TO <level>
DEFAULT | "read committed" | "read uncommitted" | "repeatable read" | serializable

netstat -a | grep postgres
netstat -na | grep 5432

pg_lsclusters - show information about all PostgreSQL clusters
pg_ctlcluster - start/stop/restart/reload a PostgreSQL cluster
    pg_ctlcluster [options] <cluster-version> <cluster-name action>
    where action = start|stop|restart|reload|promote
pg_dropcluster - completely delete a PostgreSQL cluster
    pg_dropcluster [--stop] <cluster-version> <cluster-name>
pg_createcluster - create a new PostgreSQL cluster
    pg_createcluster [options] <version> <name> [-- initdb options]

postgres=# select pg_backend_pid();
 pg_backend_pid
----------------
         217802

Разделяемая память - находится в IPC shared memory сегменте:
    - buffer pool - хранит страницы таблиц и индексов;
    - WAL buffer - лог транзакций;
    - commit log (clog) - кэш состояние транзакций.

postgres=# select oid, datname from pg_database order by oid;
  oid  |  datname
-------+-----------
     1 | template1
 13760 | template0
 13761 | postgres


Табличные пространства - отдельный каталог с точки зрения файловой системы.
По умолчанию:
    - pg_default - $PGDATA/base
    - pg_global - $PGDATA/global
Новые табличные пространства создаются в:
    - $PGDATA/tblspc

postgres=# select * from pg_tablespace;
 oid  |  spcname   | spcowner | spcacl | spcoptions
------+------------+----------+--------+------------
 1663 | pg_default |       10 |        |
 1664 | pg_global  |       10 |        |

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

### База данных (Database)
По умолчанию в кластере PostgreSQL создаются БД:
    - template0 (для восстановления из резервной копии, по умолчанию даже нет прав на connect)
    - template1 (используется как шаблон для создания БД, в нем имеет смысл делать некие действия, которые не хочется делать каждый раз при создании новых БД)
    - postgres (первая БД для регулярной работы, создается по умолчанию, хорошая практика - так же не использовать)

https://www.postgresql.org/docs/current/sql-createdatabase.html
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

psql -d <database>  # подключение к определенной БД
# \l  # список БД
# \c  # к какой базе данных и под каким пользователем подключена текущая сессия
# \c <database>  # подключиться к другой БД

select oid, datname, datistemplate, datallowconn
from pg_database;

### Схема (Schema)
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

CREATE SCHEMA schema_name [ AUTHORIZATION role_specification ] [ schema_element [ ... ] ]
CREATE SCHEMA AUTHORIZATION role_specification [ schema_element [ ... ] ]
CREATE SCHEMA IF NOT EXISTS schema_name [ AUTHORIZATION role_specification ]
CREATE SCHEMA IF NOT EXISTS AUTHORIZATION role_specification

where role_specification can be:
    user_name
  | CURRENT_ROLE
  | CURRENT_USER
  | SESSION_USER

select * from pg_namespace;

### Пользователь (User)

\h create user
\d pg_user
select * from pg_user;

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

URL: https://www.postgresql.org/docs/14/sql-createuser.html


### Права (Privileges)

grant/revoke на объект:
    - relation [attribute, …]
    - schema
    - database