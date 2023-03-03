# **Проектная работа на тему:**
## **Создание и тестирование высоконагруженного отказоустойчивого кластера PostgreSQL на базе Патрони** ##

### **Описание стенда**
---

Тестовый стенд состоит из 7 виртуальных машин расположенных в яндекс облаке с предустановленной на них ОС Ubuntu:
+ 3 ВМ с ETCD
+ 3 ВМ с Patroni и PostgreSQL
+ ВМ с Haproxy и клиентом Postgresql

![1](https://user-images.githubusercontent.com/97864676/222652169-f263cd39-dcbf-424e-b295-08c70df6ec6d.png)


Настроенная схема взаимодействия:

![2](https://user-images.githubusercontent.com/97864676/222652199-1f51b66e-c410-4a51-9652-7fe1601fc810.jpg)

### Установка и конфигурация ETCD на примере одной из трех нод(config файлы лежат в соответвующих папка проекта)
---
1. Заходим на ноду: 	
    ```
    ssh ubuntu@158.160.55.22
    ```

2. Запускаем установку etcd:
    ```
	ubuntu@pg-instance-etcd1:~$ sudo apt -y install etcd
     ```

3. Редактируем конфигурационный файл(все настройки вставил в конец файла):	 
    ```
    ubuntu@pg-instance-etcd1:~$ sudo nano /etc/default/etcd

    ETCD_NAME="pg-instance-etcd1"
    ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
    ETCD_HEARTBEAT_INTERVAL="1000"
    ETCD_ELECTION_TIMEOUT="5000"
    ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
    ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"

    ETCD_ENABLE_V2=true
    ETCD_INITIAL_ADVERTISE_PEER_URLS="http://158.160.55.22:2380"
    ETCD_INITIAL_CLUSTER="pg-instance-etcd1=http://158.160.55.22:2380,pg-instance-etcd2=http://158.160.38.83:2380,pg-instance-etcd3=http://158.160.61.32:2380"
    ETCD_INITIAL_CLUSTER_STATE="new"
    #ETCD_INITIAL_CLUSTER_STATE="existing"
    ETCD_INITIAL_CLUSTER_TOKEN="etcd"
    ETCD_ADVERTISE_CLIENT_URLS="http://158.160.55.22:2379"
    ```
4. Проделываем данную операцию на трех нодах.
5. Включаем в автозагрузку:
    ```
	ubuntu@pg-instance-etcd1:~$ sudo systemctl enable etcd
    ```

6. Запускаем etcd на всех трех нодах:
    ```
	ubuntu@pg-instance-etcd1:~$ sudo service etcd start
    ```
7. Команда проверки статуса:
    ```
	ubuntu@pg-instance-etcd1:~$ sudo systemctl status etcd
    ```
9. Команда проверки сердцебиения кластера:
    ```
	ubuntu@pg-instance-etcd3:~$ etcdctl cluster-health
    ``` 
10. Полученный результат:
    
 ![3](https://user-images.githubusercontent.com/97864676/222652248-cd75beb0-e91b-4626-a4e6-eb76ac16a10f.png)   

### Установка и конфигурация Patroni на примере одной из трех нод(config файлы лежат в соответвующих папка проекта)
---
1. Заходим на ноду:
    ``` 	
    ssh ubuntu@62.84.119.251
    ```
2. Запускаем установку postgresql 14:
    ```
	sudo apt -y install postgresql-14
    ```
3. Делаем линки:
    ```
	sudo ln -s /usr/local/bin/patroni /bin/patroni
    ```

4. Устанавливаем питон и все что с ним связано:	
    ```
    sudo apt-get update
	sudo apt install python3-pip
	pip install setuptools
	sudo apt -y install libpq-dev
	pip install psycopg2
	pip install psycopg2-binary
	pip install patroni
	pip install python-etcd
    ``` 
5. Устанавливаем patroni
    ```
	sudo apt install patroni
    ```
6. Создаем каталог для конфига и дает пользователю postgres на него права:	 
    ```
    sudo mkdir /etc/patroni
	sudo chown postgres:postgres /etc/patroni
	sudo chmod 700 /etc/patroni
    ```
7. Настраиаем файл patroni.service
    ```
	sudo nano /etc/systemd/system/patroni.service


    [Unit]
	Description=Runners to orchestrate a high-availability PostgreSQL
	After=syslog.target network.target
	[Service]
	Type=simple
	User=postgres
	Group=postgres
	ExecStart=/usr/local/bin/patroni /etc/patroni/patroni.yml
	ExecReload=/bin/kill -s HUP $MAINPID
	KillMode=process
	TimeoutSec=30
	Restart=no
	[Install]
	WantedBy=multi-user.target

    ```
8. Останавливаем postgresql
    ```
	sudo  systemctl stop postgresql
    ```

9. Удаляем postgresql
    ```
    sudo -u postgres pg_dropcluster 14 main
    ```

10. Перезагружаем демона
    ```
    sudo systemctl daemon-reload
    ```

11. Настраиваем patroni.yml
    ```
	sudo nano /etc/patroni/patroni.yml


        scope: pgsql 
        namespace: /cluster/ 
        name: postgres1 

        restapi:
            listen: 0.0.0.0:8008 # адрес той ноды, в которой находится этот файл
            connect_address: 62.84.119.251:8008 # адрес той ноды, в которой находится этот файл

        etcd:
            hosts: 158.160.55.22:2379,158.160.38.83:2379,158.160.61.32:2379

        bootstrap:
            dcs:
                ttl: 100
                loop_wait: 10
                retry_timeout: 10
                maximum_lag_on_failover: 1048576
                postgresql:
                    use_pg_rewind: true
                    use_slots: true
                    parameters:
                            wal_level: replica
                            hot_standby: "on"
                            wal_keep_segments: 5120
                            max_wal_senders: 5
                            max_replication_slots: 5
                            checkpoint_timeout: 30

            initdb:
            - encoding: UTF8
            # init pg_hba.conf должен содержать адреса ВСЕХ машин, используемых в кластере
            pg_hba:
            - host replication postgres ::1/128 md5
            - host replication postgres 127.0.0.1/8 md5
            - host replication postgres 62.84.119.251/24 md5
            - host replication postgres 158.160.51.102/24 md5
            - host replication postgres 158.160.48.61/24 md5
            - host all all 0.0.0.0/0 md5

            users:
                admin:
                    password: '123'
                    options:
                        - createrole
                        - createdb

        postgresql:
            listen: 0.0.0.0:5432 # адрес той ноды, в которой находится этот файл
            connect_address: 62.84.119.251:5432
            data_dir: /var/lib/postgresql/14/main
            bin_dir: /usr/lib/postgresql/14/bin 	
            pgpass: /tmp/pgpass
            authentication:
                replication:
                    username: postgres
                    password: '123'
                superuser:
                    username: postgres
                    password: '123'
            create_replica_methods:
                basebackup:
                    checkpoint: 'fast'
            parameters:
                unix_socket_directories: '.'

        tags:
            nofailover: false
            noloadbalance: false
            clonefrom: false
            nosync: false
    ```
12. Заходим под пользователем postgres
    ```
    sudo -iu postgres
    ```

13. Заходим в созданную директорию
    ```
	cd /etc/patroni
    ```

14. Стартуем
    ```
	patroni patroni.yml
    ```
15. Полученный результат:
    
    ![4](https://user-images.githubusercontent.com/97864676/222652289-198a0eba-cd0f-4ece-889f-6a944b356de1.png)

### Установка и конфигурация Haproxi(config файл лежит в соответвующей папке проекта)
---
1. Заходим на ноду предназначенную для Haproxy:
    ```
	ssh ubuntu@158.160.48.172
    ```
2. Запускаем установку:
    ```
	ubuntu@pg-instance-haproxy:~$ sudo apt-get install haproxy 
    ```

3. Правим конфигурационный файл:
    ```
    ubuntu@pg-instance-haproxy:~$ sudo nano /etc/haproxy/haproxy.cfg
    ```
4. Устанавливаем клиента постгрес:
    ```
    sudo apt-get install postgresql-client-14
    ```
5. Стартуем:
    ```
	ubuntu@pg-instance-haproxy:~$ sudo systemctl restart haproxy 
    ```


