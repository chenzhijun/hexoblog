任务定时处理cron, at

at now + 30 minutes
at> /sbin/shutdown -h now
at> <EOT> (ctrl+D组合键)

查询：atq

删除：atrm 1


/etc/passwd;/etc/shadow;

新增用户：useradd
设置密码：passwd
修改用户：usermod
删除用户: userdel

增加用户组: groupadd 
删除用户组: groupdel


cat /etc/passwd | grep test
