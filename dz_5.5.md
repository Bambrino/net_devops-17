### Задание №1

- В чём отличие режимов работы сервисов в Docker Swarm кластере: replication и global? 
  Режим replication - запуск экземпляров происходит в указанном количестве, replication 3 - запустит 3 экземпляра на \
автоматически выбранных worker`а, может быть как единичном количестве, так и и несколько экземпляров на одной ноде 
  Рехим global - экземпляр будет запущен на кажождой ноде в количестве 1 шт


- Какой алгоритм выбора лидера используется в Docker Swarm кластере? 
  Используется Raft - позволяет согласовывать выборы лидера и гарантировать согласованность данных всех менеджеров
 

- Что такое Overlay Network?
  Это виртуальная сеть для внутреннего взаимодействия всех Docker engine внутри кластера

### Задание №2

![Screenshot](https://github.com/Bambrino/net_devops-17/blob/main/dz/dz_5.5_2.png)

### Задание №3
![Screenshot](https://github.com/Bambrino/net_devops-17/blob/main/dz/dz_5.5_3.png)

### Задание №4
docker swarm update --autolock=true - это служит для обеспечения безопасности на случай доступа злоумышленника к Raft (для взаимодействия 
используеются шифрованные данные), теперь (после ввода этой команды) будет сгенерирован ключ безопаности, который необходимо будет вводить при при 
перезагрузке ноды, чтобы она могла "вернуться", прочитав все данные из кластера


```
[centos@node02 ~]$ sudo docker node ls
Error response from daemon: Swarm is encrypted and needs to be unlocked before it can be used.Please use "docker swarm unlock" to unlock it.
```

```
[centos@node02 ~]$ sudo docker swarm unlock
Please enter unlock key:
```

```
[centos@node02 ~]$ sudo docker node ls
ID                            HOSTNAME             STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
pyg82alf08uuv8179a40st6e4     node01.netology.yc   Ready     Active         Leader           20.10.17
ifqe7rhs39khiyq462e48n5lr *   node02.netology.yc   Ready     Active         Reachable        20.10.17
vrvrv5wc0nisfx6h0i741b2yr     node03.netology.yc   Ready     Active         Reachable        20.10.17
t144env5sgz82voqicgecj5lc     node04.netology.yc   Ready     Active                          20.10.17
as5yljxnmb4mxjnovvqzde3bx     node05.netology.yc   Ready     Active                          20.10.17
ltnvbt056aw9g1rh2of209s3w     node06.netology.yc   Ready     Active                          20.10.17

```
