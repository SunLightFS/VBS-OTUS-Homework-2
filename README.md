# Занятие №3: Физический уровень PostgreSQL

## Основное задание
-    создайте виртуальную машину c Ubuntu 20.04 LTS (bionic) в GCE типа e2-medium в default VPC в любом регионе и зоне, например us-central1-a
-    поставьте на нее PostgreSQL через sudo apt
-    проверьте что кластер запущен через sudo -u postgres pg_lsclusters
-    зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым  
postgres=# create table test(c1 text);  
postgres=# insert into test values('1'); \q
-    остановите postgres например через sudo -u postgres pg_ctlcluster 13 main stop
-    создайте новый standard persistent диск GKE через Compute Engine -> Disks в том же регионе и зоне что GCE инстанс размером например 10GB
-    добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
-    проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
-    сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
-    перенесите содержимое /var/lib/postgres/13 в /mnt/data - mv /var/lib/postgresql/13 /mnt/data
-    попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 13 main start
-    напишите получилось или нет и почему  
**Ответ: нет**, т.к. для запуска postgres'у нужна БД, которую по умолчанию он ищет в /var/lib/pgsql/13/data. Т.к. содержимое /var/lib/postgres/13 мы перенесли, postgres не может найти базу в обычном месте и завершает свою работу.
-    задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/10/main который надо поменять и поменяйте его
-    напишите что и почему поменяли  
**Ответ: поменял значение параметра PGDATA в файле севиса /usr/lib/systemd/system/postgresql-13.service.** Т.к. я создал виртуальную машину с ОС CentOS 8, мне для корректного запуска СУБД нужно было поменять значение переменной среды PGDATA, используемое сервисом postgresql-13.
-    попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 13 main start
-    напишите получилось или нет и почему  
**Ответ: получилось**, после выполнения предыдущего пункта и запуска команды systemctl deamon-reload для перечитывания конфигурации сервиса, postgresql удалось успешно запустить.
-    зайдите через через psql и проверьте содержимое ранее созданной таблицы   
**Результат: таблица на месте и содержит те данные, что вставляли ранее.**
## Задание со звездочкой *
Не удаляя существующий GCE инстанс сделайте новый, поставьте на его PostgreSQL, удалите файлы с данными из /var/lib/postgres, перемонтируйте внешний диск который сделали ранее от первой виртуальной машины ко второй и запустите PostgreSQL на второй машине так чтобы он работал с данными на внешнем диске, расскажите как вы это сделали и что в итоге получилось.
-   Остановил postgres на instance-1
-   Отмонтировал диск на instance-1 через umount
-   Убрал запись с этим диском из fstab на instance-1
-   Отмонтировал этот диск от виртуальной машины через её редактирование в GCE
-   Создал вторую виртуальную машину (instance-2)
-   Установил на неё postgresql
-   Удалил содержимое /var/lib/pgsql (на CentOS БД изначально не инициализируется, но я во время установки по привычке это сделал, поэтому все же содержимое каталога почистил)
-   Добавил на неё внешний диск через интерфейс GCE
-   Добавил диск в fstab и выполнил команду mount -a, чтобы проверить конфигурацию fstab и подмонтировать диск в /var/lib/pgsql
-   Запустил postgres
-   Выполнил запрос 'select * from test;', чтобы убедиться что данные перенеслись.



