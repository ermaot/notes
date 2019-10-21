## load data

```
> load data infile "mytbl.txt" into table mytbl;
```

```
mysql > load data local infile 'infile.txt' [REPLACE|IGNORE] into table test fields terminated by ':'  escaped by '' lines terminated by '\r' ignore 1 lines ;							//以制表符分隔
mysql > show warnings	;			//load data完毕之后执行
mysql > load data local infile 'infile.txt' into table test ignore 1 lines (@date, @time,@name,@weight,@state) set dt = concat(@date,' ',@time)
```
## mysqlimport

```
% mysqlimport --user root --host 127.0.0.1 --fields-terminated-by=":" --fields-encloses-by='"' --fields-escaped-by='' --lines-terminated-by="\r"--local cookbook mytbl.txt	
//mytbl.txt文件导入到mytbl表中，必须同名
//--ignore-lines=n		--columns=b,c,a
```

## select into outfile

```
SELECT * FROM passwd INTO OUTFILE '/tmp/passwd.txt'
FIELDS TERMINATED BY 1,1ENCLOSEDBY1"'
LINES TERMINATED BY ' \r\n' ;
```
## mysql 客户端导入到本地

```
mysq1 -e " SELECT account, she11 FROM passwd" --skip-column-names cookbook > shells.txt
```
