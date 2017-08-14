sudo docker run --detach \
    --hostname gitlab.example.com \
    --publish 443:443 --publish 80:80 \
    --name gitlab \
    --restart always \
    --volume /srv/gitlab/config:/etc/gitlab \
    --volume /srv/gitlab/logs:/var/log/gitlab \
    --volume /srv/gitlab/data:/var/opt/gitlab \
    gitlab/gitlab-ce:latest


docker exec mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /home/ubuntu/mysql/all-databases.sql

docker exec -i CONTAINER /usr/bin/mysql -u root --password=root DATABASE < backup.sql

docker -H tcp://swarm-server-ip:2375 run -tid  \ 
        --restart=always --name mysql-server-ywwd \ 
            -e MYSQL_DATABASE="ywwd" \ 
              -e MYSQL_USER="ywwd" \ 
              -e MYSQL_PASSWORD="ywwd.net" \ 
              -e MYSQL_ROOT_PASSWORD="ywwd.net" \ 
        -v /data/container/ywwd/mysql:/var/lib/mysql \ 
            -v /etc/localtime:/etc/localtime:ro \ 
           mysql \ 
     --character-set-server=utf8 --collation-server=utf8_general_ci --sql-mode="NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"

docker run -dit --name mysql --character-set-server=utf8 --collation-server=utf8_general_ci -P mysql 

ubuntu@ip-10-23-220-250:~$ docker run -e MYSQL_ROOT_PASSWORD=123456 -dit --name m1 -P mysql --character-set-server=utf8 --collation-server=utf8_general_ci

SHOW VARIABLES LIKE ‘character%’;

基础镜像:

centos

1:java
2:mysql
3:redis
4:maven

:rabbitMQ


