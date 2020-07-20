- ONLY_FULL_GROUP_BY，对于GROUP BY聚合操作，如果在SELECT中的列，没有在GROUP BY中出现，那么这个SQL是不合法的，因为列不在GROUP BY从句中

- NO_AUTO_VALUE_ON_ZERO，该值影响自增长列的插入。默认设置下，插入0或NULL代表生成下一个自增长值。如果用户希望插入的值为0，而该列又是自增长的，那么这个选项就有用了

- NO_AUTO_CREATE_USER：不能直接通过grant授权来添加用户，而需要grant……identified by

  