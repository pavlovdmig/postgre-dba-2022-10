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
    ![1](https://user-images.githubusercontent.com/97864676/200368311-bd62a044-1e7a-451c-8313-672511f7366e.png)


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
    ![2](https://user-images.githubusercontent.com/97864676/200368344-dfbe08de-2aee-4b83-a63d-b5d462edc1fb.png) 
6. Создадим докер образ и смонтируем его в ранее созданный каталог:
    ```
    sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:14
    ```
7. Проверим наличие ранее созданного образа командой:
    ```
    sudo docker ps -a
    ```
    ![3](https://user-images.githubusercontent.com/97864676/200368370-8eccc8ec-3cea-4ddd-8fc8-82ee4f42ea03.png)
8. Создадим БД otus, создадим таблицу otus_table и запоним ее данными. Получим в результате следующее:
    ![Без имени](https://user-images.githubusercontent.com/97864676/200368436-95aa0ce9-a90e-49b1-a62a-9e79fc271af2.png)
9. Подключимся к контейнеру с сервером с ноутбука извне инстансов ЯО.
    ```
    psql -p 5432 -U postgres -h <public ip> -d postgres -W
    ```

10. Проверим нашу ранее созданную бд и таблицу

    ![6](https://user-images.githubusercontent.com/97864676/200368501-185eef70-8326-480e-9063-a534198a17b5.png)

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
    ![7](https://user-images.githubusercontent.com/97864676/200369463-d03ab426-6312-4d49-bbd4-c13d8227b268.png)


13. Подключимся повторно и проверим наличие нашей бд с таблицей:
    ![8](https://user-images.githubusercontent.com/97864676/200369556-9ac45a18-6bd7-42bb-b8e5-a247628a2744.png)

    
    

