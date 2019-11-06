MySQL的explain是各种执行计划选择的结果，如果想看整个执行计划以及对于多种索引方案之间是如何选择的？optimizer_trace

## 简介optimizer_trace参数

版本mysql 8.0.17-debug Source distribution

- 默认设置

```
> show variables like "OPTIMIZER_TRACE";
+-----------------+--------------------------+
| Variable_name   | Value                    |
+-----------------+--------------------------+
| optimizer_trace | enabled=off,one_line=off |
+-----------------+--------------------------+
1 row in set (0.01 sec)
```

- 修改为on

  ```
  > set optimizer_trace="enabled=on";
  Query OK, 0 rows affected (0.00 sec)
  
  > show variables like "%optimizer_trace%";
  +------------------------------+----------------------------------------------------------------------------+
  | Variable_name                | Value                                                                      |
  +------------------------------+----------------------------------------------------------------------------+
  | optimizer_trace              | enabled=on,one_line=off                                                    |
  | optimizer_trace_features     | greedy_search=on,range_optimizer=on,dynamic_range=on,repeated_subselect=on |
  | optimizer_trace_limit        | 1                                                                          |
  | optimizer_trace_max_mem_size | 1048576                                                                    |
  | optimizer_trace_offset       | -1                                                                         |
  +------------------------------+----------------------------------------------------------------------------+
  5 rows in set (0.01 sec)
  
  ```

  ## 查看trace

  ```
  > select * from information_schema.optimizer_trace\G
  Empty set (0.10 sec)
  
  > explain select * from test0906;
  +----+-------------+----------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
  | id | select_type | table    | partitions | type  | possible_keys | key        | key_len | ref  | rows | filtered | Extra       |
  +----+-------------+----------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
  |  1 | SIMPLE      | test0906 | NULL       | index | NULL          | test0906_b | 5       | NULL |    2 |   100.00 | Using index |
  +----+-------------+----------+------------+-------+---------------+------------+---------+------+------+----------+-------------+
  1 row in set, 1 warning (0.00 sec)
  
  > select * from information_schema.optimizer_trace\G
  *************************** 1. row ***************************
                              QUERY: explain select * from test0906
                              TRACE: {
    "steps": [
      {
        "join_preparation": {
          "select#": 1,
          "steps": [
            {
              "expanded_query": "/* select#1 */ select `test0906`.`a` AS `a`,`test0906`.`b` AS `b` from `test0906`"
            }
          ]
        }
      },
      {
        "join_optimization": {
          "select#": 1,
          "steps": [
            {
              "table_dependencies": [
                {
                  "table": "`test0906`",
                  "row_may_be_null": false,
                  "map_bit": 0,
                  "depends_on_map_bits": [
                  ]
                }
              ]
            },
            {
              "rows_estimation": [
                {
                  "table": "`test0906`",
                  "table_scan": {
                    "rows": 2,
                    "cost": 1
                  }
                }
              ]
            },
            {
              "considered_execution_plans": [
                {
                  "plan_prefix": [
                  ],
                  "table": "`test0906`",
                  "best_access_path": {
                    "considered_access_paths": [
                      {
                        "rows_to_scan": 2,
                        "access_type": "scan",
                        "resulting_rows": 2,
                        "cost": 1.2,
                        "chosen": true
                      }
                    ]
                  },
                  "condition_filtering_pct": 100,
                  "rows_for_plan": 2,
                  "cost_for_plan": 1.2,
                  "chosen": true
                }
              ]
            },
            {
              "attaching_conditions_to_tables": {
                "original_condition": null,
                "attached_conditions_computation": [
                ],
                "attached_conditions_summary": [
                  {
                    "table": "`test0906`",
                    "attached": null
                  }
                ]
              }
            },
            {
              "finalizing_table_conditions": [
              ]
            },
            {
              "refine_plan": [
                {
                  "table": "`test0906`"
                }
              ]
            }
          ]
        }
      },
      {
        "join_explain": {
          "select#": 1,
          "steps": [
          ]
        }
      }
    ]
  }
  MISSING_BYTES_BEYOND_MAX_MEM_SIZE: 0
            INSUFFICIENT_PRIVILEGES: 0
  1 row in set (0.00 sec)
  
  ```

  

