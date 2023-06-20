# Домашнее задание к занятию «Система мониторинга Zabbix» Лазарев Даниил
## Задание 1
![Скриншот-1](https://github.com/n123tw/srlb-homework/blob/main/9-02/img/1.jpg)
```
wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb
dpkg -i zabbix-release_6.4-1+debian11_all.deb
apt update
apt install zabbix-server-pgsql zabbix-frontend-php php7.4-pgsql zabbix-nginx-conf zabbix-sql-scripts zabbix-agent2
sudo -u postgres createuser --pwprompt zabbix
sudo -u postgres createdb -O zabbix zabbix
zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
# Insert previously created password
nano /etc/zabbix/zabbix_server.conf
# Enable site at port 8080, set server_name
nano /etc/zabbix/nginx.conf
systemctl restart zabbix-server zabbix-agent nginx php7.4-fpm
systemctl enable zabbix-server zabbix-agent nginx php7.4-fpm
```
## Задание 2
![Скриншот-2](https://github.com/n123tw/srlb-homework/blob/main/9-02/img/2.jpg)
![Скриншот-3](https://github.com/n123tw/srlb-homework/blob/main/9-02/img/3.jpg)
![Скриншот-4](https://github.com/n123tw/srlb-homework/blob/main/9-02/img/4.jpg)
```
wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb
dpkg -i zabbix-release_6.4-1+debian11_all.deb
apt update
apt install zabbix-agent2 zabbix-agent2-plugin-*
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
# Add Server and ServerActive
nano /etc/zabbix/zabbix_agent2.conf
systemctl restart zabbix-agent2
```
