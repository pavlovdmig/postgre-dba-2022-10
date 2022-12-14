## **Домашнее задание №7:**
**Механизм блокировок**
1.   Создад кластер, установил на него  PostgreSQL 15 с дефолтными настройками:

![1](https://user-images.githubusercontent.com/97864676/207563894-dd114ae2-f22d-4bea-bd3f-5b113f0d2420.png)


2.   Для того, чтобы в журнал сообщений сбрасывалась информация о блокировках, удерживаемых более 200 миллисекунд, нужна включить настройку log_lock_waits и у настройки deadlock_timeout выставить значение 200.
Выставление настроек делал через alter system:

![2](https://user-images.githubusercontent.com/97864676/207563948-f2258040-d972-4530-9a11-359504345f00.png)

3.  Создал базу данных locks, создал таблицу account и заполнил ее данными:
    ```
    postgres=# \c locks
    SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off)
    You are now connected to database "locks" as user "postgres".
    locks=# \dt
            List of relations
    Schema |   Name   | Type  |  Owner
    --------+----------+-------+----------
    public | accounts | table | postgres
    (1 row)

    locks=# select * from accounts;
    acc_no | amount
    --------+---------
        2 | 2000.00
        3 | 3000.00
        1 | 1400.00
    (3 rows)

    locks=#
    ```
4.  Смоделируем ситуацию обновления одной и той же строки тремя командами UPDATE в разных сеансах.
5.  Первый сеанс, открываем транзыкцию и выполняем update, видим:   
    
    ![3](https://user-images.githubusercontent.com/97864676/207564078-64f53ccb-ab5e-411e-b7d1-ba9f4fecb2d2.png)

6.  Второй сеанс, выполняем тоже update
    
    ![4im](https://user-images.githubusercontent.com/97864676/207564141-941feff0-40fe-4fc1-9516-878681fc56e4.png)

7. Третий сеаенс и видим, что в втором и в третьем update не выполняется:

    ![5im](https://user-images.githubusercontent.com/97864676/207564211-f5bbcd3d-ea1e-4f5b-9539-433040ae2b18.png)

8.  Посмотрим лог, наблюдаем блокировки во второй и в третьих сессиях:

    ![6](https://user-images.githubusercontent.com/97864676/207564248-1d51761a-aaab-46f6-a560-a1b6090c7ea6.png)


9.  В первой сессии мы можем наблюдать, что RowExclusiveLock установлен на отношение accounts
    
    ![7](https://user-images.githubusercontent.com/97864676/207564301-20f49fb6-56fe-453d-a959-6029cdae9b83.png)
    
10. Во второй сессии мы наблюдаем, что у нас появилась блокировка transactionid в режиме ShareLock и tuple для выполняемого обновления.

    ![8](https://user-images.githubusercontent.com/97864676/207564346-340723f4-ab9c-4240-b2d7-c9cd509602fb.png)

11. В третьей же мы видим блокировку tuple в режиме ExclusiveLock

    ![9](https://user-images.githubusercontent.com/97864676/207564392-66f3016a-5c91-4856-bed3-98baa6422a5c.png)

12. Завершим первую сессию. Все блокировки ушли!

    ![10](https://user-images.githubusercontent.com/97864676/207564432-c112fa4c-13df-40d9-ae25-cdc866a17414.png)

13. Воспроизведите взаимоблокировку трех транзакций.
    Воспроизвести удалось:
    
    ![11](https://user-images.githubusercontent.com/97864676/207564485-850bbb34-8a52-4e8e-8484-b1fd1bcc66bc.png)
    
14. Могут ли две транзакции, выполняющие единственную команду UPDATE одной и той же таблицы (без where), заблокировать друг друга?
 Могут! Если обновление будет происходить в обратных последовательностях

    

