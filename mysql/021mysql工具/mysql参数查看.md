## MySQL配置工作的原理

- mysql配置文件一般在/etc/my.cnf和/etc/mysql/my.cnf(类unix系统)
- 使用操作mysqld --verbose --help | grep -A 1  "Default options"查看配置文件

```
# mysqld --verbose --help | grep -A 1  "Default options"
Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/etc/my.cnf ~/.my.cnf 
```

- 配置文件分为多个部分，服务器读取[mysqld]的部分，客户端读取[client]的部分



--no-defaults 可以禁止读取配置文件

--defaults-file  --defaults-extra-file



```
 # mysql --print-defaults
 或者
 # my_print_defaults client mysql 
```

#### my_print_defaults

```
# my_print_defaults
my_print_defaults  Ver 1.6 for Linux at x86_64
This software comes with ABSOLUTELY NO WARRANTY. This is free software,
and you are welcome to modify and redistribute it under the GPL license

Prints all arguments that is give to some program using the default files
Usage: my_print_defaults [OPTIONS] [groups]
  -c, --config-file=name 
                      Deprecated, please use --defaults-file instead. Name of
                      config file to read; if no extension is given, default
                      extension (e.g., .ini or .cnf) will be added
  -#, --debug[=#]     This is a non-debug version. Catch this and exit
  -c, --defaults-file=name 
                      Like --config-file, except: if first option, then read
                      this file only, do not read global or per-user config
                      files; should be the first option
  -e, --defaults-extra-file=name 
                      Read this file after the global config file and before
                      the config file in the users home directory; should be
                      the first option
  -g, --defaults-group-suffix=name 
                      In addition to the given groups, read also groups with
                      this suffix
  -e, --extra-file=name 
                      Deprecated. Synonym for --defaults-extra-file.
  --mysqld            Read the same set of groups that the mysqld binary does.
  -n, --no-defaults   Return an empty string (useful for scripts).
  -?, --help          Display this help message and exit.
  -v, --verbose       Increase the output level
  -V, --version       Output version information and exit.

Default options are read from the following files in the given order:
/etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf 

Variables (--variable-name=value)
and boolean options {FALSE|TRUE}  Value (after reading options)
--------------------------------- ----------------------------------------
config-file                       my
defaults-file                     my
defaults-extra-file               (No default value)
defaults-group-suffix             (No default value)
extra-file                        (No default value)
mysqld                            FALSE

Example usage:
my_print_defaults --defaults-file=example.cnf client client-server mysql
```





可以看到mysql读取顺序Default options are read from the following files in the given order:
/etc/mysql/my.cnf /etc/my.cnf ~/.my.cnf  

从程序表现上看，的确是这个顺序，源码中是如何定义的？

./mysys/my_default.cc

```
char buffer[FN_REFLEN] = "~/";
  unpack_filename(buffer, buffer);
  default_paths["/etc/my.cnf"] = enum_variable_source::GLOBAL;
  default_paths["/etc/mysql/my.cnf"] = enum_variable_source::GLOBAL;
  default_paths[string(buffer) + ".my.cnf"] = enum_variable_source::MYSQL_USER;
  default_paths[string(buffer) + ".mylogin.cnf"] = enum_variable_source::LOGIN;
```



#### 