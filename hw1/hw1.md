## **Домашнее задание №1:**
**Работа с уровнями изоляции транзакции в PostgreSQL**

**Цель**:
* научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)ЯндексОБлаке на уровне Compute Cloud
* научиться управлять уровнем изоляции транзакций в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

**Установка Yandex Cloud и подключение по ssh**
1) Для выполнения дз выбрал YC и создал в нем новый проект:

    ![image](https://user-images.githubusercontent.com/97864676/199059383-d16f9d12-137f-476b-99bd-03613b35ddf4.png)

2) Установи Yandex Cloud (CLI) для работы с командной строкой.
3) Проверим настройки профиля командой:
    
    ```
    yc config list
    ```
    получаем следующее:
    
    ![image](https://user-images.githubusercontent.com/97864676/199059537-d66dcb18-d6a8-474c-abbb-3e45b8ed8fab.png)


4) Создаем SSh ключ командой

    ``` 
    ssh-keygen
    ```
    получаем следующее:
    ![image](https://user-images.githubusercontent.com/97864676/199059631-50084ec6-6e59-4040-afec-b9b9500abb99.png)


5) Для успешного подключения к VM, необходимо создать текстовый файл и поместить в него публичный ssh ключ, следуюшим образом <Пользователь>:<Ключ>
6) Создаем нашу VM

    ```
    yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:\Users\dipavlov\.ssh\id_rsa.txt
    ```
    после успешного создания получаем следующее:
    ![image](https://user-images.githubusercontent.com/97864676/199059687-452c3152-cf10-4fcb-91bd-ca1f7f87f1f3.png)


7) Для того, чтобы успешно подключиться к нашей VM используем следующую команду, указав имя пользователя ubuntu и публичный ip адрес VM
    ```
    ssh ubuntu@178.154.255.24
    ```
    Мы успешно подключились:
    ![image](https://user-images.githubusercontent.com/97864676/199059831-ef77237e-937d-41d0-8db6-1b834a603c2f.png)


**Установка PostgreSQL**

1) Установим PostgreSQL 15 следующей командой
    ```
    sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" >      /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y   install postgresql-15
    ```
    после успешной установки получаем следующее:
    ![image](https://user-images.githubusercontent.com/97864676/199059896-db53f473-4f9a-4e35-9bd3-18caee2be427.png)

2) Добавляем пароль для пользователя postgres, открываем доступ - listener, указываем откуда слушаем подключения
    ```
    sudo -u postgres psql
    sudo pg_conftool 15 main set listen_addresses '*'
    sudo nano /etc/postgresql/15/main/pg_hba.conf
    sudo pg_ctlcluster 15 main restart
    ```
3) Зайдем под пользователем postgres и cоздадим базу данных для теста
    
    ```
    CREATE DATABASE iso;
    ```
    после создания получаем следующее:
    ![image](https://user-images.githubusercontent.com/97864676/199059978-634f1a0c-7bfd-4801-8a16-db948eef963c.png)


4) Зайдем вторым SSH
    ![image](https://user-images.githubusercontent.com/97864676/199060270-5cc71ecc-9f9e-4610-b291-cf71f8656beb.png)

5) Выключаем автокомит
    ```
    \set AUTOCOMMIT OFF
    ```
    ![image](https://user-images.githubusercontent.com/97864676/199060392-954f404b-9549-4cab-8959-c2ff6601556b.png)

6) Создаем таблицу и вставляем данные
    ```
    create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
    ```
7) Проверяем, что получилось
    ```
    select * from persons
    ```
    ![image](https://user-images.githubusercontent.com/97864676/199060546-24413f9a-2f41-46b2-ad87-48fa085043a8.png)

8) Смотрим на текущий уровень изоляции:
    ```
    show transaction isolation level
    ```
    ![image](https://user-images.githubusercontent.com/97864676/199060797-1a23cb4f-1dfb-47d8-a93c-13cc2fd278f4.png)

9) Начинаю новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем  изоляции
        ```
        BEGIN;
        ```
10) В первой сессии добавляю новую запись 
    ```
    insert into persons(first_name, second_name) values('sergey', 'sergeev');  
    ```      
11) Во второй сессии 
    ```
    select * from persons;
    ```
12) Результат:
    ![image](https://user-images.githubusercontent.com/97864676/199061009-f8bfe868-664f-4410-9e1e-e490df7b4100.png)

13) Новой записи во второй сессии не видно, так как в попродившей строку транзакции, не был выполнен commit(autocommit выключен)

14) Завершаю первую транзакцию 
    ```
    commit;
    ```
15) Во второй сессиии
    ```
    select * from persons;
    ```
16) Смотрим на результат:
    ![image](https://user-images.githubusercontent.com/97864676/199061072-1469afdd-96bc-4e45-866b-6deb979f401a.png)

17) Запись появилась, так как была заффиксирована транзакция в первой сессии
18) Начинаю новые но уже repeatable read транзации 
    ```
    set transaction isolation level repeatable read;
    ```
19) В первой сессии добавить новую запись: 
    ```
    insert into persons(first_name, second_name) values('sveta', 'svetova');
    ```
20) Во второй сессии выполняю:
    ```
    select * from persons;
    ```
21) Запись не видна.
22) Завершаю первую транзакцию
    ```
    commit;
    ```
23) Во второй сессии запись не видна.
    select * from persons;
24) Завершаю вторую транзакцию
    ```
    commit;
    ```
25) Записи видны.
    ![image](https://user-images.githubusercontent.com/97864676/199062005-70135097-36b3-4689-9708-aec60a25148c.png)

26) Вывод:
    На уровне Repeatable Read не видны изменения, произведенные другими транзакциями до фиксации текущей транзакции. 
