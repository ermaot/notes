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
performance-schema-digests-size                              -1
performance-schema-error-size                                4486
performance-schema-events-stages-history-long-size           -1
performance-schema-events-stages-history-size                -1
performance-schema-events-statements-history-long-size       -1
performance-schema-events-statements-history-size            -1
performance-schema-events-transactions-history-long-size     -1
performance-schema-events-transactions-history-size          -1
performance-schema-events-waits-history-long-size            -1
performance-schema-events-waits-history-size                 -1
performance-schema-hosts-size                                -1
performance-schema-instrument                                
performance-schema-max-cond-classes                          100
performance-schema-max-cond-instances                        -1
performance-schema-max-digest-length                         1024
performance-schema-max-digest-sample-age                     60
performance-schema-max-file-classes                          80
performance-schema-max-file-handles                          32768
performance-schema-max-file-instances                        -1
performance-schema-max-index-stat                            -1
performance-schema-max-memory-classes                        450
performance-schema-max-metadata-locks                        -1
performance-schema-max-mutex-classes                         300
performance-schema-max-mutex-instances                       -1
performance-schema-max-prepared-statements-instances         -1
performance-schema-max-program-instances                     -1
performance-schema-max-rwlock-classes                        60
performance-schema-max-rwlock-instances                      -1
performance-schema-max-socket-classes                        10
performance-schema-max-socket-instances                      -1
performance-schema-max-sql-text-length                       1024
performance-schema-max-stage-classes                         175
performance-schema-max-statement-classes                     218
performance-schema-max-statement-stack                       10
performance-schema-max-table-handles                         -1
performance-schema-max-table-instances                       -1
performance-schema-max-table-lock-stat                       -1
performance-schema-max-thread-classes                        100
performance-schema-max-thread-instances                      -1
performance-schema-session-connect-attrs-size                -1
performance-schema-setup-actors-size                         -1
performance-schema-setup-objects-size                        -1
```

