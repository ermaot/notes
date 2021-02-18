## 一、bcc的tplist

```
# tplist | wc -l
1330
```



## 二、perf的perf list
```
# perf list | grep "Tracepoint" | wc -l
1317

```

## 三、 /sys/kernel/debug/tracing/available_events 
```
# # cat /sys/kernel/debug/tracing/available_events | wc -l
1315

```

三者内容基本一致，只是稍有不同

tplist比available_events 多了ftrace

```
ftrace:bprint
ftrace:bputs
ftrace:branch
ftrace:context_switch
ftrace:funcgraph_entry
ftrace:funcgraph_exit
ftrace:function
ftrace:hwlat
ftrace:kernel_stack
ftrace:mmiotrace_map
ftrace:mmiotrace_rw
ftrace:print
ftrace:raw_data
ftrace:user_stack
ftrace:wakeup

```

而perf list比available_events 多了
```
ftrace:function                                                       
ftrace:print
```
