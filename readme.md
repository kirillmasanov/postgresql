https://severalnines.com/blog/understanding-postgresql-architecture/
https://www.interdb.jp/pg/
https://habr.com/ru/companies/postgrespro/articles/458186/
https://habr.com/ru/companies/postgrespro/articles/459250/
https://habr.com/ru/companies/postgrespro/articles/460423/
https://habr.com/ru/companies/postgrespro/articles/461523/

su - postgres

\set AUTOCOMMIT off
\echo :AUTOCOMMIT

show transaction_isolation;
set transaction_isolation TO <level>
DEFAULT | "read committed" | "read uncommitted" | "repeatable read" | serializable

netstat -a | grep postgres

postgres=# select pg_backend_pid();
 pg_backend_pid
----------------
         217802

Разделяемая память - находится в IPC shared memory сегменте:
    - buffer pool - хранит страницы таблиц и индексов;
    - WAL buffer - лог транзакций;
    - commit log (clog) - кэш состояние транзакций.

По умолчанию в кластере PostgreSQL создаются БД:
    - template0
    - template1
    - postgres

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

