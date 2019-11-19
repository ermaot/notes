## performance_schema配置

#### 1.启动时配置

```
# mysqld --verbose --help  | grep performance-schema | grep -v "\-\-"
performance-schema                                           TRUE
performance-schema-accounts-size                             -1
performance-schema-consumer-events-stages-current            FALSE
performance-schema-consumer-events-stages-history            FALSE
performance-schema-consumer-events-stages-history-long       FALSE
performance-schema-consumer-events-statements-current        TRUE
performance-schema-consumer-events-statements-history        TRUE
performance-schema-consumer-events-statements-history-long   FALSE
performance-schema-consumer-events-transactions-current      TRUE
performance-schema-consumer-events-transactions-history      TRUE
performance-schema-consumer-events-transactions-history-long FALSE
performance-schema-consumer-events-waits-current             FALSE
performance-schema-consumer-events-waits-history             FALSE
performance-schema-consumer-events-waits-history-long        FALSE
performance-schema-consumer-global-instrumentation           TRUE
performance-schema-consumer-statements-digest                TRUE
performance-schema-consumer-thread-instrumentation           TRUE
……………………
```
- 启动选项，启动mysqld时通过命令指定或者my.cnf中指定，但在show variables中查看不到
- 参数说明：
1. performance_schema_consumer_events_statements_current :是否在server启动时就开启events_statements_current（记录当前语句事件信息）；启动后可以在setup_consumers更新该选项
2. 
