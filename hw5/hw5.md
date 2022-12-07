## **Домашнее задание №5:**
**Настройка autovacuum с учетом оптимальной производительности**

1.  Создаем кластер pg-instance-hw5, устанавливаем на него  PostgreSQL 14 с дефолтными настройками:
    
    ![1](https://user-images.githubusercontent.com/97864676/206087913-0ab7b0c6-58e7-497a-bbfb-b73d1077ec77.png)
2.  Посмотрим на наши дефолтные настройки:

    ![2](https://user-images.githubusercontent.com/97864676/206087928-9fad11fd-faa6-4cf5-adf0-da563a21b367.png)
3.  Применим параметры настройки PostgreSQL из прикрепленного к материалам занятия файла
4.  Здесь два пути было, или например через команду psql(добавятся параметры в postgresql.auto.conf):
    ```
    alter system set max_connections = '40';
    ```
    Я же поступил проще, добавил в конец postgresql.conf ну и обязательно перезагрузим 
    ```
    sudo pg_ctlcluster 14 main restart
    ```

5.  Посмотрим на наши недефолтные настройки:
    
    ![3](https://user-images.githubusercontent.com/97864676/206087938-a13b07d4-753c-46c2-b951-f9f87ac9d346.png)
6.  Выполняем pgbench -i postgres и запускаем pgbench -c8 -P 60 -T 600 -U       postgres postgres
    ```
    ubuntu@pg-instance-hw5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
    pgbench (14.6 (Ubuntu 14.6-1.pgdg22.04+1))
    starting vacuum...end.
    progress: 60.0 s, 449.0 tps, lat 17.801 ms stddev 20.600
    progress: 120.0 s, 461.2 tps, lat 17.347 ms stddev 11.856
    progress: 180.0 s, 401.4 tps, lat 19.813 ms stddev 29.223
    progress: 240.0 s, 434.2 tps, lat 18.540 ms stddev 31.296
    progress: 300.0 s, 418.2 tps, lat 19.119 ms stddev 23.061
    progress: 360.0 s, 467.4 tps, lat 17.111 ms stddev 11.350
    progress: 420.0 s, 448.3 tps, lat 17.852 ms stddev 21.393
    progress: 480.0 s, 411.0 tps, lat 19.461 ms stddev 13.774
    progress: 540.0 s, 404.7 tps, lat 19.770 ms stddev 14.142
    progress: 600.0 s, 398.6 tps, lat 20.068 ms stddev 14.971
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 257647
    latency average = 18.630 ms
    latency stddev = 20.270 ms
    initial connection time = 20.431 ms
    tps = 429.383899 (without initial connection time)
    ```
7.  Посмотрим настройки автовакуума
    
    ![5](https://user-images.githubusercontent.com/97864676/206087990-34933313-688f-412e-81c1-4b68f28a1ffd.png)
8.  Поменяем настройки автовакуума, в postgresql.conf
    
    ![6](https://user-images.githubusercontent.com/97864676/206088019-0e75c05c-1ff1-455c-9fd9-187a6769c481.png)
9.  Стало хуже в некоторых точках....
    ```
    ubuntu@pg-instance-hw5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
    pgbench (14.6 (Ubuntu 14.6-1.pgdg22.04+1))
    starting vacuum...end.
    progress: 60.0 s, 428.8 tps, lat 18.645 ms stddev 13.730
    progress: 120.0 s, 428.7 tps, lat 18.662 ms stddev 13.907
    progress: 180.0 s, 414.7 tps, lat 19.286 ms stddev 14.859
    progress: 240.0 s, 429.3 tps, lat 18.636 ms stddev 12.872
    progress: 300.0 s, 450.3 tps, lat 17.763 ms stddev 12.854
    progress: 360.0 s, 413.3 tps, lat 19.356 ms stddev 13.413
    progress: 420.0 s, 450.0 tps, lat 17.777 ms stddev 11.781
    progress: 480.0 s, 425.8 tps, lat 18.784 ms stddev 15.647
    progress: 540.0 s, 449.6 tps, lat 17.797 ms stddev 11.595
    progress: 600.0 s, 408.2 tps, lat 19.596 ms stddev 22.386
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 257936
    latency average = 18.608 ms
    latency stddev = 14.539 ms
    initial connection time = 23.108 ms
    tps = 429.891806 (without initial connection time)
    ```
10. Поменял   autovacuum_max_workers на 1
    ```
    ubuntu@pg-instance-hw5:~$ sudo -u postgres pgbench -c8 -P 60 -T 600 -U postgres postgres
    pgbench (14.6 (Ubuntu 14.6-1.pgdg22.04+1))
    starting vacuum...end.
    progress: 60.0 s, 458.5 tps, lat 17.434 ms stddev 18.603
    progress: 120.0 s, 442.7 tps, lat 18.071 ms stddev 14.242
    progress: 180.0 s, 499.4 tps, lat 16.017 ms stddev 10.358
    progress: 240.0 s, 468.3 tps, lat 17.085 ms stddev 11.947
    progress: 300.0 s, 484.4 tps, lat 16.515 ms stddev 11.884
    progress: 360.0 s, 457.1 tps, lat 17.499 ms stddev 13.222
    progress: 420.0 s, 427.6 tps, lat 18.709 ms stddev 14.208
    progress: 480.0 s, 476.1 tps, lat 16.805 ms stddev 10.996
    progress: 540.0 s, 458.0 tps, lat 17.450 ms stddev 21.732
    progress: 600.0 s, 434.5 tps, lat 18.430 ms stddev 14.279
    transaction type: <builtin: TPC-B (sort of)>
    scaling factor: 1
    query mode: simple
    number of clients: 8
    number of threads: 1
    duration: 600 s
    number of transactions actually processed: 276408
    latency average = 17.365 ms
    latency stddev = 14.503 ms
    initial connection time = 20.780 ms
    tps = 460.673732 (without initial connection time)
    ```
Результаты замеров на графике. Третий замер самый лучший.

    ![7](https://user-images.githubusercontent.com/97864676/206088287-3ecfc2d2-798a-4673-81bc-a41abda99d14.png)

  
