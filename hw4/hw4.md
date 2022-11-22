## **Домашнее задание №4:**
**Работа с базами данных, пользователями и правами**

1.  Создаем новый кластер postgreSQL 14 и заходим в него под пользователем postgres.
    ![1](https://user-images.githubusercontent.com/97864676/203247069-17925d4d-0fa3-48b2-81d9-ef83668022ce.png)
2.  Создаем новую базу данных testdb и заходим в созданную базу данных под пользователем postgres.
    ![2](https://user-images.githubusercontent.com/97864676/203247086-05f59b10-da04-49d8-b245-0d3efb5d8d76.png)
3.  Создаем новую схему testnm, далее создадим новую таблицу t1 с одной колонкой c1 типа integer и вставим строку со значением
     c1=1. 
    
    ![3](https://user-images.githubusercontent.com/97864676/203247100-f0816497-ccc4-4b72-bfae-ad2629f29445.png)
4.  Cоздадим новую роль readonly,дадим новой роли право на подключение к базе данных testdb, предоставим новой роли право на 
    использование схемы testnm, дадим новой роли право на select для всех таблиц схемы testnm.
    
    ![4](https://user-images.githubusercontent.com/97864676/203247105-906448d6-9a7c-4fe0-b22a-45ceea01db73.png)
5.  Создадим пользователя testread с паролем test123 и дадим роль readonly пользователю testread.
    ![5](https://user-images.githubusercontent.com/97864676/203247117-2659554c-5413-4e3e-b737-027ab1422ee9.png)
6.  Зайдем под пользователем testread в базу данных testdb и выполним команду: select * from t1;
    ![6](https://user-images.githubusercontent.com/97864676/203247128-ab418fc9-164f-4014-b45a-429e105f475d.png)
7. Не получилось, запрос выдал ошибку, так как нашу таблицу мы создали в схеме public.
    ![7](https://user-images.githubusercontent.com/97864676/203247153-d20e3de2-aad9-4ec8-b414-39c7261ab92c.png)
8. Вернемся в базу данных testdb под пользователем postgres, удалите таблицу t1, создадим ее заново но уже с 
    явным указанием имени схемы testnm и вставим строку со значением c1=1.
    ![8](https://user-images.githubusercontent.com/97864676/203247168-cf2cb6df-4117-40db-8ef2-73f5e9b3d6a3.png)
9.  Зайдем под пользователем testread в базу данных testdb, сделаем select * from testnm.t1. Как видим, запрос выдал ошибку. 
    Так как давали права до того, как создали t1 в нужной нам схеме.
    ![9](https://user-images.githubusercontent.com/97864676/203247189-38f3c89e-ab40-4bf7-84f6-839043ad366c.png)
10. Раздаем grant по новой и выполняем select * from testnm.t1;. Запрос без ошибок.
    ![10](https://user-images.githubusercontent.com/97864676/203247203-4912927f-7b51-4ed3-bc61-547c78a7df7d.png)
11.  Теперь попробуйте выполнить команды
    create table t2(c1 integer); insert into t2 values (2);
    Отработало без ошибок, так как у нас в searth_path указана схема public, а создавать в ней данные можно под любым пользовавтелем.
    
    ![11](https://user-images.githubusercontent.com/97864676/203248012-fcc70cab-b495-42e4-a4f6-7e6f86e2b437.png)
    
12. Чтобы такое избежать, уберем у роли public права.
    
    ![12](https://user-images.githubusercontent.com/97864676/203247252-266a7482-584b-4cc6-8963-e3da262bc5ef.png)
13. Порядок, пользователь теперь не имеет прав на создание таблиц.
    
    ![13](https://user-images.githubusercontent.com/97864676/203247260-981633de-89e8-42c3-918b-51f374430a4a.png)

