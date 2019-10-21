
```
select * from test order by a ;
select * from test order by a collate latin1_general_cs ;									//字母大小写排序，大小写敏感
select * from url order by substring_index(url,'.',-2) limit 5;								//根据分段字符串排序
select inet_aton('138.56.98.52'), inet_ntoa(2318950964) ;									//IP地址转换
select * from url order by right(url ,2) ,mid(url,4),mid(url ,5,4) ,left(url, 5) limit 10 ;	//substring_index返回根据分隔符分隔后字符串
select substring(url ,6)  from url order by substring_index(url,'.',-2) limit 10 ;			//substring返回字符串
select url  from url order by substring_index(url,'.',-1) ,substring_index(url,'.',-2)  limit 10 ;
select * from url order by if(url is null ,1 ,0)  desc ,url desc limit 30 ;					//如果列值为空，则替换成1 排序，否则为0
select * from test order by field(name,'ermao','ermao2'),id;								//当field没有匹配到的时候，返回值是0；也就是说根据插入的顺序先返回没有匹配到的值
{枚举值排序，根据定义的顺序作为数值顺序
create table weekday ( day enum('MON','SUN','TUES','WEN')) ;    
select day ,day+0 from weekday  order by day ;
select day ,day+0 from weekday  order by cast(day as char);									//char(n) , date , time , datetime , DECIMAL , signed , unsigned 
select day from weekday order by field(day,'MON','SUN','TUES','WEN') ;
```
