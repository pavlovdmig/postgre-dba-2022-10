## **Домашнее задание №3:**
**Установка и настройка PostgreSQL**

1.   Создаем и подключаемся к нашей VM в ЯО.
    im.1
2.  Устанавливем PostgreSQL 14. 
3.  Проверяем, что кластер запущен.
im2
4.  Заходим из под пользователя postgres в psql и делаем произвольную таблицу с произвольным содержимым
im3
5.  Останавливаем postgres через sudo -u postgres pg_ctlcluster 14 main stop
4
6.  Создаем диск и добавляем его к нашей VM
5
7.  Проверяем наш диск
6
8.  Создаем раздел: 
    ```
    sudo parted -a opt /dev/sdb mkpart primary ext4 0% 100%
    ```
    7
9.  Меняем метку раздела и монтируем новый диск.
    ```
    sudo e2label /dev/vdb1 postgre
    sudo mkdir -p /mnt/data
    sudo mkfs.ext4 -L /data /dev/vdb1
    ```

    9

10.  Делаем изменения в /etc/fstab

11. ПРоверяем и тестируем что получилось
10,11
12. Делаем пользователя postgres владельцем /mnt/data
    ```
    sudo chown -R postgres:postgres /mnt/data/
    ```
13. Переносим содержимое /var/lib/postgres/14 в /mnt/data
    ```
    sudo mv /var/lib/postgresql/14 /mnt/data
    ```
14. При старте, кластер не запускается,так как данные перенесены, чтобы заработало, ножно поменять конфигурационный файл.
12
15. Проверяем, что все работает.
13