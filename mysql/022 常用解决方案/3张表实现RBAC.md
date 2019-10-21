管理员表
```
CREATE TABLE cqh_admin
(
    id smallint unsigned not null auto_increment comment 'id',
    username varchar(30) not null comment '用户名',
    password char(32) not null comment '密码',
    salt char(6) not null comment '随机生成6位字符',
    role_id mediumint unsigned not null default '0' comment '角色的id',
    primary key (id),
    key (role_id)
) engine=MyISAM default charset=utf8 comment '管理员';
INSERT INTO cqh_admin VALUES(1,'admin','69eef864983c4f9af8fada82c0a9d89a','cqh123',1);
```
权限表
```
CREATE TABLE cqh_privilege
(
    id mediumint unsigned not null auto_increment comment 'id',
    pri_name varchar(30) not null comment '权限名称',
    module_name varchar(30) not null comment '对应的模块名',
    controller_name varchar(30) not null comment '对应的控制器名',
    action_name varchar(30) not null comment'对应的方法名',
    parent_id mediumint unsigned not null default '0' comment '上级权限的id',
    primary key (id)
)engine=MyISAM default charset=utf8 comment '权限';
```
角色表


```
CREATE TABLE cqh_role
(
    id mediumint unsigned not null auto_increment comment 'id',
    role_name varchar(30) not null comment '角色名称',
    pri_id varchar(1500) not null default '' comment '权限的ID，如果有多个权限就用,隔开，如1,3,4',
    primary key (id)
)engine=MyISAM default charset=utf8 comment '角色';
INSERT INTO cqh_role VALUES(1,'超级管理员','*');
```
