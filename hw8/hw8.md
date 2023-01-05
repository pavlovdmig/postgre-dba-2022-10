## **Домашнее задание №8:**
**Нагрузочное тестирование и тюнинг PostgreSQL**
1.  Развернул VM в YC со следующими характеристиками
    ![1](https://user-images.githubusercontent.com/97864676/210778538-d57b12d5-a4a0-42c3-b0ba-40dced32df7f.png)

2.  Поставил на неё PostgreSQL 14.
    
    ![2](https://user-images.githubusercontent.com/97864676/210778553-95d76dbc-c585-4b6d-8fda-4ae25ba28f38.png)

3.  Запустим pgbench с дефолтными настройками:
    ```
    sudo -u postgres pgbench -c 50 -j 2 -P 10 -T 60 postgres
    ```
4. Смотрим на результаты:

    ![3](https://user-images.githubusercontent.com/97864676/210778578-4f2d8715-f379-44ca-8c12-5542dd7e7f54.png)
   
5. Пробуем подкрутить следующие настройки(так как у нас есть настройка с контекстом postmaster нам необходимо рестартовать наш сервер)
    -   shared_buffers TO '512MB' так как памяти у нас 2GB, а в рекомендациях устанавливаь в 25% от общей оперативной памяти на сервере;
    -   work_mem увеличил в два раза;
    -   effective_cache_size установил в 75% от общей оперативной памяти на сервере;
    
    ![4](https://user-images.githubusercontent.com/97864676/210778691-29cdbe2f-7bc4-4b72-a096-1f06ffdb7c07.png)

6. Получаем следующие результаты:
    
    ![5](https://user-images.githubusercontent.com/97864676/210778709-493c5b80-fbf5-4809-8629-4f24f1f3f642.png)

7. Прирост получился не плохой, в районе 45 tps
8. Воспользуемся pgtune :
    
    ![6](https://user-images.githubusercontent.com/97864676/210778782-dcebe7e3-2871-4ab9-98b4-069f11646d34.png)

9. Применим настройки, рестартанем постгрес, и запустим наш pgbench(применял настройки с генереннымм скриптом через alter system)
    
    ![7](https://user-images.githubusercontent.com/97864676/210778806-a9371964-696d-4927-bb43-aa8bb4c83068.png)

10. Ну и чудеса, результат хуже чем с дефолтом и с тюнингом, 299 tps...
11. Попробуем перезапустить инстанс в клауде.
12. Получаем результат:

    ![8](https://user-images.githubusercontent.com/97864676/210778840-6abb895d-38eb-457d-9510-98ddc0139b28.png)

13. Стало лучше.  373 tps
