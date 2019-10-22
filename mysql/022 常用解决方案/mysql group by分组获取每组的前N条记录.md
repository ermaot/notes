http://www.manongjc.com/article/1081.html

```
create table test(student_id int,student_name varchar(20),class_id int,total_score int);

insert into test values(1,"Jason",1,298),(2,"yayuan",1,295),(3,"Martin",3,300),(4,"Alison",4,289),(5,"Mathews",2,250),(6,"Celia",2,240),(7,"Rice",1,275),(8,"David",3,257),(9,"Larry",2,243),(10,"zhang",3,250),(11,"wu",4,255),(12,"ge",1,260),(13,"li",3,265),(14,"meng",4,279),(15,"qiu",1,272),(16,"liu",4,283),(17,"tian",3,299),(18,"huang",2,201),(19,"nie",1,228),(20,"wang",3,230);



SELECT * FROM test a WHERE 3> ( SELECT COUNT(*) FROM test WHERE class_id = a.class_id AND total_score > a.total_score ) ORDER BY a.class_id, a.total_score DESC ;
+------------+--------------+----------+-------------+
| student_id | student_name | class_id | total_score |
+------------+--------------+----------+-------------+
|          1 | Jason        |        1 |         298 |
|          2 | yayuan       |        1 |         295 |
|          7 | Rice         |        1 |         275 |
|          5 | Mathews      |        2 |         250 |
|          9 | Larry        |        2 |         243 |
|          6 | Celia        |        2 |         240 |
|          3 | Martin       |        3 |         300 |
|         17 | tian         |        3 |         299 |
|         13 | li           |        3 |         265 |
|          4 | Alison       |        4 |         289 |
|         16 | liu          |        4 |         283 |
|         14 | meng         |        4 |         279 |
+------------+--------------+----------+-------------+

```