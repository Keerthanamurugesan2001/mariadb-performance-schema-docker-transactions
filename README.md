# mariadb-performance-schema-docker-transactions
To get the transaction list in mariadb with performance schema in docker container

## Performance_schema
The Performance Schema is a feature for monitoring MySQL server performance. It collects performance data and provides a way to inspect and analyze the server's performance. The data collected can include information about queries, memory usage, IO operations, and more.

### Key Components
The Performance Schema includes several key components, such as consumers, instruments, and setup tables:

**Consumers**: These are mechanisms that store collected performance data. Examples include events_statements_current, events_statements_history, and events_statements_history_long.

**Instruments**: These are points within the server code where performance data is collected. Instruments can be enabled or disabled.

**Setup Tables**: These tables allow you to configure which instruments and consumers are active.


### To enable performance_schema 

```
filename: /etc/mysql/my.cnf
add:
[mysqld]
performance_schema = ON
```

If not able to add in exist file add in include file
Since the my.cnf file does not contain a [mysqld] section and is primarily used to include other configuration files, you should instead add the configuration to one of the included configuration files.

```
docker exec -it <container_id_or_name> /bin/bash
```

```
cat <<EOT >> /etc/mysql/conf.d/performance.cnf
[mysqld]
performance_schema = ON
EOT
```

```
cat /etc/mysql/conf.d/performance.cnf
```

```
exit
```

```
docker restart <container_id_or_name>
```

### Commands to add the transaction logs:

```
UPDATE performance_schema.setup_consumers
SET ENABLED = 'YES';
```

This command enables all consumers in the Performance Schema. This ensures that any other consumers that may not have been specifically mentioned are also enabled.

```
UPDATE performance_schema.setup_instruments
SET ENABLED = 'YES', TIMED = 'YES';
```

This command enables and sets timing for all instruments in the Performance Schema. This ensures comprehensive monitoring across all types of events.

```
SELECT event_id, event_name, sql_text, timer_start, timer_end
FROM performance_schema.events_statements_history
ORDER BY timer_end DESC;
```

This query retrieves historical statement events from the events_statements_history table, ordered by timer_end in descending order. This shows recent events first.


```
SELECT event_id, event_name, sql_text, timer_start, timer_end
FROM performance_schema.events_statements_history_long
ORDER BY timer_end ASC ;
```

This query retrieves historical statement events from the events_statements_history_long table, ordered by timer_end in ascending order. This shows older events first.

```
SELECT event_id, event_name, sql_text, timer_start, timer_end
FROM performance_schema.events_statements_current;
```

This query retrieves currently executing statement events from the events_statements_current table.


```
SELECT event_id, event_name, sql_text, timer_start, timer_end
FROM performance_schema.events_statements_history
UNION ALL
SELECT event_id, event_name, sql_text, timer_start, timer_end
FROM performance_schema.events_statements_history_long
ORDER
BY timer_end DESC;
```

This query combines the results from both events_statements_history and events_statements_history_long tables using a UNION ALL to include all historical statement events, ordered by timer_end in descending order.
