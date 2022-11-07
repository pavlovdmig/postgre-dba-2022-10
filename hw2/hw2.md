## **Домашнее задание №2:**
**Установка и настройка PostgteSQL в контейнере Docker**

1.  Создадим VM pg-instance-hw2
    ```
    yc compute instance create --name pg-instance-hw2 --hostname pg-instance-hw2 --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:\Users\Дмитрий\.ssh\id_rsa.txt
    ```

2. Если ранее были установлены инстансы postgres-а, проверим их наличие, если такие имеются - остановим их и затем удалим:
    ```
    pg_lsclusters
    sudo pg_ctlcluster 15 main stop
    sudo pg_dropcluster 15 main
    pg_lsclusters
    ```
    имэдж1

3. Установим Docker следующей командой:
    ```
    curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER
    ```
4. После успешной установки создадим каталог(Volume) /var/lib/postgres командой:
    ```
    sudo mkdir -p /var/lib/postgres 
    ```
5. Создадим сеть командой:
    ```
    sudo docker network create pg-net
    ```
    имэдж 2 
6. Создадим докер образ и смонтируем его в ранее созданный каталог:
    ```
    sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
    ```
7. Проверим наличие ранее созданного образа командой:
    ```
    sudo docker ps -a
    ```
    имэдж 3
8. Создадим БД otus, создадим таблицу otus_table и запоним ее данными. Получим в результате:  следующее:
БЕЗЫМЯННЫЙ
9. Подключимся к контейнеру с сервером с ноутбука извне инстансов ЯО.
    ```
    psql -p 5432 -U postgres -h <public ip> -d postgres -W
    ```

10. Проверим нашу ранее созданную бд и таблицу

    6
11. Удалим наш контейнер с сервером, для этого нам нужно выполнить команды:
    ```
    sudo docker ps -a
    sudo docker stop 5dbf0a1bd40b  
    sudo docker rm 5dbf0a1bd40b
    ```
12. Создадим контейнер с сервером повторно:
     ```
    sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
    ```
    7
13. Подключимся повторно и проверим наличие нашей бд с таблицей:

8
    

