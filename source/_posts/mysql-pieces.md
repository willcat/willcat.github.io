---
title: mysql_pieces
date: 2018-08-16 10:37:50
tags: [mysql,数据库,pieces]
categories: [mysql]
---

1. mysql/mariadb服务端导出到文件命令
```sql
SELECT ... FROM TABLE_A
INTO OUTFILE "/path/to/file"
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```
要注意mysql/mariadb所属用户是否用权限写到目录或者文件

2. mysql/mariadb导出到本地文件命令
上述命令是导出到服务器端的，本命令是导出到客户端的
`mysql -h my.db.com -u usrname--password=pass db_name -e 'SELECT foo FROM bar' > /tmp/myfile.txt `
