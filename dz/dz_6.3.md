### Задача №1

docker-compose.yml 

``` yml
version: "3"

networks:
  my_net:
    driver: bridge

services:

  mysql_01: 
    image: mysql:8
    container_name: mysql_host_01

    ports:
     - 3306:3306
    
    volumes:
      - ./mysql:/var/lib/mysql
    
    environment:
      MYSQL_ROOT_PASSWORD: "P@ssw0rd"

    networks: 
      - my_net

    restart: always
```

Поскольку в бэкапе только таблица из бд test_db, то сделал так:
``` commanline
vvk@bubuntu:~/dz/6.3$ docker exec -it mysql_host_01 bash
bash-4.4# mysql -p
Enter password:
```
``` mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 8.0.29 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> cretate database test_db;
Query OK, 1 row affected (0.01 sec)
mysql> use test_db;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

mysql> source /var/lib/mysql/test_dump.sql
...

Query OK, 0 rows affected (0.00 sec)

```

``` mysql
mysql> \h

...
status    (\s) Get status information from the server.
...
mysql> status
--------------
mysql  Ver 8.0.29 for Linux on x86_64 (MySQL Community Server - GPL)

mysql> show tables;
+-------------------+
| Tables_in_test_db |
+-------------------+
| orders            |
+-------------------+
1 row in set (0.00 sec)

mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)
```

### Задача №2

``` mysql
mysql> create user 'test@localhost' identified with mysql_native_password by 'test-pass' with max_queries_per_hour 100 password expire interval 180 day \
failed_login_attempts 3 attribute '{"FirstName":"James","LastName":"Pretty"}';
Query OK, 0 rows affected (0.02 sec)

mysql> grant select on test_db.* to test@localhost;
Query OK, 0 rows affected, 1 warning (0.05 sec)

mysql> select * from INFORMATION_SCHEMA.USER_ATTRIBUTES where user='test';
+------+-----------+----------------------------------------------+
| USER | HOST      | ATTRIBUTE                                    |
+------+-----------+----------------------------------------------+
| test | localhost | {"LastName": "Pretty", "FirstName": "James"} |
+------+-----------+----------------------------------------------+
1 row in set (0.00 sec)
```

### Задача №3

```mysql
mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)

mysql> select * from orders;
+----+-----------------------+-------+
| id | title                 | price |
+----+-----------------------+-------+
|  1 | War and Peace         |   100 |
|  2 | My little pony        |   500 |
|  3 | Adventure mysql times |   300 |
|  4 | Server gravity falls  |   300 |
|  5 | Log gossips           |   123 |
+----+-----------------------+-------+
5 rows in set (0.00 sec)

mysql> show profiles;
+----------+------------+----------------------------------------+
| Query_ID | Duration   | Query                                  |
+----------+------------+----------------------------------------+
|        1 | 0.00019075 | select * from orders where price > 300 |
|        2 | 0.00031125 | select * from orders                   |
+----------+------------+----------------------------------------+
2 rows in set, 1 warning (0.00 sec)
```
```mysql
mysql> show table status where name = 'orders';
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| Name   | Engine | Version | Row_format | Rows | Avg_row_length | Data_length | Max_data_length | Index_length | Data_free | Auto_increment | Create_time         | Update_time         | Check_time | Collation          | Checksum | Create_options | Comment |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
| orders | InnoDB |      10 | Dynamic    |    5 |           3276 |       16384 |               0 |            0 |         0 |              6 | 2022-07-21 19:12:07 | 2022-07-21 19:12:07 | NULL       | utf8mb4_0900_ai_ci |     NULL |                |         |
+--------+--------+---------+------------+------+----------------+-------------+-----------------+--------------+-----------+----------------+---------------------+---------------------+------------+--------------------+----------+----------------+---------+
1 row in set (0.00 sec)

mysql> alter table orders engine=myisam;
Query OK, 5 rows affected (0.03 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from orders;
+----+-----------------------+-------+
| id | title                 | price |
+----+-----------------------+-------+
|  1 | War and Peace         |   100 |
|  2 | My little pony        |   500 |
|  3 | Adventure mysql times |   300 |
|  4 | Server gravity falls  |   300 |
|  5 | Log gossips           |   123 |
+----+-----------------------+-------+
5 rows in set (0.00 sec)

mysql> select * from orders where price > 300;
+----+----------------+-------+
| id | title          | price |
+----+----------------+-------+
|  2 | My little pony |   500 |
+----+----------------+-------+
1 row in set (0.00 sec)

mysql> show profiles;
+----------+------------+----------------------------------------+
| Query_ID | Duration   | Query                                  |
+----------+------------+----------------------------------------+
|        1 | 0.00019075 | select * from orders where price > 300 |
|        2 | 0.00031125 | select * from orders                   |
|        3 | 0.02198800 | alter table orders engine=myisam       |
|        4 | 0.00022075 | select * from orders                   |
|        5 | 0.00020850 | select * from orders where price > 300 |
+----------+------------+----------------------------------------+
5 rows in set, 1 warning (0.00 sec)

```

### Задача №4

Добавил в docker-compose.yml в секцию volunes:
```
- ./conf.d:/etc/mysql/conf.d
```
``` commandline
vvk@bubuntu:~/dz/6.3$ cat conf.d/my_custom.cnf
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql

pid-file=/var/run/mysqld/mysqld.pid

innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1
innodb_log_buffer_size = 1M
innodb_buffer_pool_size = 5G
innodb_log_file_size = 100M


[client]
socket=/var/run/mysqld/mysqld.sock

```
  ```mysql
  mysql> SHOW GLOBAL VARIABLES like 'innodb%';
  ```

<details>

```mysql
+------------------------------------------+------------------------+
| Variable_name                            | Value                  |
+------------------------------------------+------------------------+
| innodb_adaptive_flushing                 | ON                     |
| innodb_adaptive_flushing_lwm             | 10                     |
| innodb_adaptive_hash_index               | ON                     |
| innodb_adaptive_hash_index_parts         | 8                      |
| innodb_adaptive_max_sleep_delay          | 150000                 |
| innodb_api_bk_commit_interval            | 5                      |
| innodb_api_disable_rowlock               | OFF                    |
| innodb_api_enable_binlog                 | OFF                    |
| innodb_api_enable_mdl                    | OFF                    |
| innodb_api_trx_level                     | 0                      |
| innodb_autoextend_increment              | 64                     |
| innodb_autoinc_lock_mode                 | 2                      |
| innodb_buffer_pool_chunk_size            | 134217728              |
| innodb_buffer_pool_dump_at_shutdown      | ON                     |
| innodb_buffer_pool_dump_now              | OFF                    |
| innodb_buffer_pool_dump_pct              | 25                     |
| innodb_buffer_pool_filename              | ib_buffer_pool         |
| innodb_buffer_pool_in_core_file          | ON                     |
| innodb_buffer_pool_instances             | 8                      |
| innodb_buffer_pool_load_abort            | OFF                    |
| innodb_buffer_pool_load_at_startup       | ON                     |
| innodb_buffer_pool_load_now              | OFF                    |
| innodb_buffer_pool_size                  | 5368709120             |
| innodb_change_buffer_max_size            | 25                     |
| innodb_change_buffering                  | all                    |
| innodb_checksum_algorithm                | crc32                  |
| innodb_cmp_per_index_enabled             | OFF                    |
| innodb_commit_concurrency                | 0                      |
| innodb_compression_failure_threshold_pct | 5                      |
| innodb_compression_level                 | 6                      |
| innodb_compression_pad_pct_max           | 50                     |
| innodb_concurrency_tickets               | 5000                   |
| innodb_data_file_path                    | ibdata1:12M:autoextend |
| innodb_data_home_dir                     |                        |
| innodb_ddl_buffer_size                   | 1048576                |
| innodb_ddl_threads                       | 4                      |
| innodb_deadlock_detect                   | ON                     |
| innodb_dedicated_server                  | OFF                    |
| innodb_default_row_format                | dynamic                |
| innodb_directories                       |                        |
| innodb_disable_sort_file_cache           | OFF                    |
| innodb_doublewrite                       | ON                     |
| innodb_doublewrite_batch_size            | 0                      |
| innodb_doublewrite_dir                   |                        |
| innodb_doublewrite_files                 | 2                      |
| innodb_doublewrite_pages                 | 4                      |
| innodb_extend_and_initialize             | ON                     |
| innodb_fast_shutdown                     | 1                      |
| innodb_file_per_table                    | ON                     |
| innodb_fill_factor                       | 100                    |
| innodb_flush_log_at_timeout              | 1                      |
| innodb_flush_log_at_trx_commit           | 2                      |
| innodb_flush_method                      | fsync                  |
| innodb_flush_neighbors                   | 0                      |
| innodb_flush_sync                        | ON                     |
| innodb_flushing_avg_loops                | 30                     |
| innodb_force_load_corrupted              | OFF                    |
| innodb_force_recovery                    | 0                      |
| innodb_fsync_threshold                   | 0                      |
| innodb_ft_aux_table                      |                        |
| innodb_ft_cache_size                     | 8000000                |
| innodb_ft_enable_diag_print              | OFF                    |
| innodb_ft_enable_stopword                | ON                     |
| innodb_ft_max_token_size                 | 84                     |
| innodb_ft_min_token_size                 | 3                      |
| innodb_ft_num_word_optimize              | 2000                   |
| innodb_ft_result_cache_limit             | 2000000000             |
| innodb_ft_server_stopword_table          |                        |
| innodb_ft_sort_pll_degree                | 2                      |
| innodb_ft_total_cache_size               | 640000000              |
| innodb_ft_user_stopword_table            |                        |
| innodb_idle_flush_pct                    | 100                    |
| innodb_io_capacity                       | 200                    |
| innodb_io_capacity_max                   | 2000                   |
| innodb_lock_wait_timeout                 | 50                     |
| innodb_log_buffer_size                   | 1048576                |
| innodb_log_checksums                     | ON                     |
| innodb_log_compressed_pages              | ON                     |
| innodb_log_file_size                     | 104857600              |
| innodb_log_files_in_group                | 2                      |
| innodb_log_group_home_dir                | ./                     |
| innodb_log_spin_cpu_abs_lwm              | 80                     |
| innodb_log_spin_cpu_pct_hwm              | 50                     |
| innodb_log_wait_for_flush_spin_hwm       | 400                    |
| innodb_log_write_ahead_size              | 8192                   |
| innodb_log_writer_threads                | ON                     |
| innodb_lru_scan_depth                    | 1024                   |
| innodb_max_dirty_pages_pct               | 90.000000              |
| innodb_max_dirty_pages_pct_lwm           | 10.000000              |
| innodb_max_purge_lag                     | 0                      |
| innodb_max_purge_lag_delay               | 0                      |
| innodb_max_undo_log_size                 | 1073741824             |
| innodb_monitor_disable                   |                        |
| innodb_monitor_enable                    |                        |
| innodb_monitor_reset                     |                        |
| innodb_monitor_reset_all                 |                        |
| innodb_old_blocks_pct                    | 37                     |
| innodb_old_blocks_time                   | 1000                   |
| innodb_online_alter_log_max_size         | 134217728              |
| innodb_open_files                        | 4000                   |
| innodb_optimize_fulltext_only            | OFF                    |
| innodb_page_cleaners                     | 4                      |
| innodb_page_size                         | 16384                  |
| innodb_parallel_read_threads             | 4                      |
| innodb_print_all_deadlocks               | OFF                    |
| innodb_print_ddl_logs                    | OFF                    |
| innodb_purge_batch_size                  | 300                    |
| innodb_purge_rseg_truncate_frequency     | 128                    |
| innodb_purge_threads                     | 4                      |
| innodb_random_read_ahead                 | OFF                    |
| innodb_read_ahead_threshold              | 56                     |
| innodb_read_io_threads                   | 4                      |
| innodb_read_only                         | OFF                    |
| innodb_redo_log_archive_dirs             |                        |
| innodb_redo_log_encrypt                  | OFF                    |
| innodb_replication_delay                 | 0                      |
| innodb_rollback_on_timeout               | OFF                    |
| innodb_rollback_segments                 | 128                    |
| innodb_segment_reserve_factor            | 12.500000              |
| innodb_sort_buffer_size                  | 1048576                |
| innodb_spin_wait_delay                   | 6                      |
| innodb_spin_wait_pause_multiplier        | 50                     |
| innodb_stats_auto_recalc                 | ON                     |
| innodb_stats_include_delete_marked       | OFF                    |
| innodb_stats_method                      | nulls_equal            |
| innodb_stats_on_metadata                 | OFF                    |
| innodb_stats_persistent                  | ON                     |
| innodb_stats_persistent_sample_pages     | 20                     |
| innodb_stats_transient_sample_pages      | 8                      |
| innodb_status_output                     | OFF                    |
| innodb_status_output_locks               | OFF                    |
| innodb_strict_mode                       | ON                     |
| innodb_sync_array_size                   | 1                      |
| innodb_sync_spin_loops                   | 30                     |
| innodb_table_locks                       | ON                     |
| innodb_temp_data_file_path               | ibtmp1:12M:autoextend  |
| innodb_temp_tablespaces_dir              | ./#innodb_temp/        |
| innodb_thread_concurrency                | 0                      |
| innodb_thread_sleep_delay                | 10000                  |
| innodb_tmpdir                            |                        |
| innodb_undo_directory                    | ./                     |
| innodb_undo_log_encrypt                  | OFF                    |
| innodb_undo_log_truncate                 | ON                     |
| innodb_undo_tablespaces                  | 2                      |
| innodb_use_fdatasync                     | OFF                    |
| innodb_use_native_aio                    | ON                     |
| innodb_validate_tablespace_paths         | ON                     |
| innodb_version                           | 8.0.29                 |
| innodb_write_io_threads                  | 4                      |
+------------------------------------------+------------------------+
149 rows in set (0.00 sec)
```
</details>
