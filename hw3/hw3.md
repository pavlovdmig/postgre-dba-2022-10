## **Домашнее задание №3:**
**Установка и настройка PostgreSQL**

1.   Создаем и подключаемся к нашей VM в ЯО.
    ![1](https://user-images.githubusercontent.com/97864676/202900533-b50d9df0-6759-48e5-b87d-e3fce17a4f6f.png)

2.  Устанавливем PostgreSQL 14. 
3.  Проверяем, что кластер запущен.
    ![2](https://user-images.githubusercontent.com/97864676/202900546-f63c5ef6-fa19-496f-8f5d-3f0d853d22d5.png)
4.  Заходим из под пользователя postgres в psql и делаем произвольную таблицу с произвольным содержимым
    ![3](https://user-images.githubusercontent.com/97864676/202900554-f29b11de-7ffd-40c4-a823-a89d4590e8d7.png)
5.  Останавливаем postgres через sudo -u postgres pg_ctlcluster 14 main stop
    ![4](https://user-images.githubusercontent.com/97864676/202900559-ca48bc39-2a00-4bb2-ab32-0ca73e3af122.png)
6.  Создаем диск и добавляем его к нашей VM
    ![5](https://user-images.githubusercontent.com/97864676/202900573-4e6474f9-17b2-4027-812b-50a9ef0beab5.png)

7.  Проверяем наш диск
    ![6](https://user-images.githubusercontent.com/97864676/202900578-49532bc0-c3a7-492a-b2fc-e81feec2725a.png)
8.  Создаем раздел: 
    ```
    sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
    ```
    ![7](https://user-images.githubusercontent.com/97864676/202900590-7fd6a733-7d60-44ab-bcbf-420dedfa708f.png)
9.  Меняем метку раздела и монтируем новый диск.
    ```
    sudo e2label /dev/vdb1 postgre
    sudo mkdir -p /mnt/data
    sudo mkfs.ext4 -L /data /dev/vdb1
    ```
    ![9](https://user-images.githubusercontent.com/97864676/202900611-68cab5e8-5320-40d9-84fa-72b90ec510ba.png) 
10.  Делаем изменения в /etc/fstab
11. ПРоверяем и тестируем что получилось
    ![10](https://user-images.githubusercontent.com/97864676/202900630-42dd45dc-d6e7-416e-b133-a44157ca7d11.png)  
    ![11](https://user-images.githubusercontent.com/97864676/202900637-3b3cf4c3-4ac7-4171-9500-a53020ea403e.png)

12. Делаем пользователя postgres владельцем /mnt/data
    ```
    sudo chown -R postgres:postgres /mnt/data/
    ```
13. Переносим содержимое /var/lib/postgres/14 в /mnt/data
    ```
    sudo mv /var/lib/postgresql/14 /mnt/data
    ```
14. При старте, кластер не запускается,так как данные перенесены, чтобы заработало, ножно поменять конфигурационный файл.
    ![12](https://user-images.githubusercontent.com/97864676/202900645-0f4f2579-4a8f-451c-9247-f98a0709cdf8.png)
15. Проверяем, что все работает.
    ![13](https://user-images.githubusercontent.com/97864676/202900707-a1088def-e9a7-4b7f-ab63-2d23a367f8a8.png)
