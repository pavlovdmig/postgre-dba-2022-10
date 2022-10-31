## **Домашнее задание №1:**
**Работа с уровнями изоляции транзакции в PostgreSQL**

**Цель**:
* научиться работать с Google Cloud Platform на уровне Google Compute Engine (IaaS)ЯндексОБлаке на уровне Compute Cloud
* научиться управлять уровнем изоляции транзакций в PostgreSQL и понимать особенность работы уровней read commited и repeatable read

**Установка Yandex Cloud и подключение по ssh**
1) Для выполнения дз выбрал YC и создал в нем новый проект:
1.png
2) Установи Yandex Cloud (CLI) для работы с командной строкой.
3) Проверим настройки профиля командой:
    
    ```
    yc config list
    ```
    получаем следующее:
    2.png

4) Создаем SSh ключ командой

    ``` 
    ssh-keygen
    ```
    получаем следующее:
    3.png

5) Для успешного подключения к VM, необходимо создать текстовый файл и поместить в него публичный ssh ключ, следуюшим образом <Пользователь>:<Ключ>
6) Создаем нашу VM

    ```
    yc compute instance create --name pg-instance --hostname pg-instance --create-boot-disk size=15G,type=network-ssd,image-folder-id=standard-images,image-family=ubuntu-2204-lts --network-interface subnet-name=default-ru-central1-a,nat-ip-version=ipv4 --zone ru-central1-a --metadata-from-file ssh-keys=C:\Users\dipavlov\.ssh\id_rsa.txt
    ```
    после успешного создания получаем следующее:
    4.png

7) Для того, чтобы успешно подключиться к нашей VM используем следующую команду, указав имя пользователя ubuntu и публичный ip адрес VM
 ```
    ssh ubuntu@178.154.255.24
 ```
    Мы успешно подключились:
    5.png

**Установка PostgreSQL**

1) Установим PostgreSQL 15 следующей командой
    ```
    sudo apt update && sudo apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt -y install postgresql-15
    ```
    после успешной установки получаем следующее:
    6.png
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
    7.png

4) Зайдем вторым SSH
7.png
5) Выключаем автокомит
    ```
    \set AUTOCOMMIT OFF
    ```
6) Создаем таблицу и вставляем данные
    ```
    create table persons(id serial, first_name text, second_name text); insert into persons(first_name, second_name) values('ivan', 'ivanov'); insert into persons(first_name, second_name) values('petr', 'petrov'); commit;
    ```
7) Проверяем, что получилось
    ```
    select * from persons
    ```

8) Смотрим на текущий уровень изоляции:
    ```
    show transaction isolation level
    ```
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
    13
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
14
26) Вывод:
    На уровне Repeatable Read не видны изменения, произведенные другими транзакциями до фиксации текущей транзакции. 