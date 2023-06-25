# Домашнее задание к занятию "10.1 Keepalived/vrrp" Лазарев Даниил
## Задание 1
### Master-node:
```
vrrp_instance main {
state MASTER
interface ens33
virtual_router_id 11
priority 110
advert_int 4
authentication {
auth_type AH
auth_pass 12345678
}
unicast_peer{
192.168.100.21
}
virtual_ipaddress{
192.168.100.200 dev ens33 label MainNew:vip
}
}
```
![Скриншот-1](https://github.com/n123tw/srlb-homework/blob/main/10-1/img/1.jpg)

### Backup-node:
```
vrrp_instance main {
state BACKUP
interface ens33
virtual_router_id 11
priority 50
advert_int 4
authentication {
auth_type AH
auth_pass 12345678
}
unicast_peer{
192.168.100.26
}
virtual_ipaddress{
192.168.100.200 dev ens33 label MainNew
}
}
```
![Скриншот-2](https://github.com/n123tw/srlb-homework/blob/main/10-1/img/2.jpg)