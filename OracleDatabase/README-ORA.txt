 Инструкция по сборке контейнера Oracle Database 23.5.0 Free
 

 1. Подготовка рабочего окружения

Перед тем как начать сборку контейнера, нужно подготовить рабочее окружение и скачать необходимые файлы 
1 Установить Docker. У пользователя должен быть доступ на его запуск без sudo 
2 Установить Git. Вероятно при попытке загрузить некоторые репозитории Oracle потребуется дополнительная авторизация

 1.1. Создание каталогов для хранения репозиториев и файлов

 Переходим в домашнюю директорию cd
Создаем папку для репозиториев куда скачаем репозиторий гита и переходим в папку dbreps
mkdir  git
mkdir -p  git/dbreps  
cd git/dbreps  

 1.2. Клонирование репозитория Docker Images от Oracle

 Клонируем репозиторий Docker-образов Oracle
git clone https://github.com/oracle/docker-images.git

- Репозиторий содержит Dockerfile и сценарии для создания контейнеров с Oracle Database.

 1.3. Создание папки для скачивания RPM-файла Oracle Database

 Возвращаемся в домашнюю директорию cd
Создаем папку для хранения RPM-файла и переходим в созданную папку
mkdir orafree        
cd orafree           

 1.4. Скачивание RPM-файла Oracle Database 23.5.0 Free

 Скачиваем файл с официального сайта Oracle(может запросить логин/пасс)
wget https://download.oracle.com/otn-pub/otn_software/db-free/oracle-database-free-23ai-1.0-1.el8.x86_64.rpm

 2. Перемещение RPM-файла в нужный каталог
После скачивания, переместите RPM-файл в нужный каталог в репозитории Docker-образов Oracle.

 Перемещаем RPM-файл в каталог Dockerfiles
mv oracle-database-free-23ai-1.0-1.el8.x86_64.rpm /home/oracle/git/dbreps/docker-images/OracleDatabase/SingleInstance/dockerfiles/23.5.0/

- Это позволяет Docker-образу использовать этот файл для установки базы данных.

 3. Сборка контейнера

Теперь можно приступить к сборке контейнера Oracle Database с помощью скрипта buildContainerImage.sh.
Переходим
cd /home/oracle/git/dbreps/docker-images/OracleDatabase/SingleInstance/dockerfiles/

 3.1. Запуск сборки

 Выполняем сборку контейнера с тегом oracle/database:23.5.0-free
./buildContainerImage.sh -f -v 23.5.0 -t oracle/database:23.5.0-free

- Флаг `-f` — для сборки с использованием Dockerfile для версии 23.5.0.
- Флаг `-v 23.5.0` — указывает версию.
- Флаг `-t oracle/database:23.5.0-free` — присваивает тег образу.

 3.2. Процесс сборки

- Скрипт загрузит Oracle Linux 8, установит необходимые зависимости и файлы, а затем настроит Oracle Database для запуска в контейнере.
- В процессе сборки вы увидите различные этапы, такие как добавление файлов, установка пакетов, настройка среды и т.д.

Сборка может занять некоторое время (примерно 10-15 минут), в зависимости от мощности вашей машины и скорости интернета. 
В конце при успешной сборке покажет:

  Oracle Database container image for 'free' version 23.5.0 is ready to be extended:

    --> oracle/database:23.5.0-free

  Build completed in 607 seconds.

 4. Проверка собранного Docker-образа

После завершения сборки, убедитесь, что образ был успешно создан.

 Проверяем список доступных образов Docker
docker images
В выводе команды должен быть виден ваш новый образ с тегом `oracle/database:23.5.0-free`:

REPOSITORY           TAG           IMAGE ID       CREATED         SIZE
oracle/database      23.5.0-free   87db81b8bda7   2 minutes ago   4.92GB

 5. Запуск контейнера Oracle Database

После того как образ был собран, можно запустить контейнер Oracle Database:

 Запуск контейнера Oracle Database
docker run -d --name oracle-db -p 1521:1521 -p 5500:5500 -p 2484:2484 \
  --ulimit nofile=1024:65536 --ulimit nproc=2047:16384 --ulimit stack=10485760:33554432 --ulimit memlock=3221225472 \
  -e ORACLE_SID=FREE -e ORACLE_PDB=FREEPDB -e ORACLE_PWD=Oracle2024 \
  -e INIT_SGA_SIZE=1024 -e INIT_PGA_SIZE=1024 -e INIT_CPU_COUNT=2 -e INIT_PROCESSES=300 \
  -e ORACLE_EDITION=FREE -e ORACLE_CHARACTERSET=AL32UTF8 \
  -v /opt/oracle/oradata \
  oracle/database:23.5.0-free

Параметры:
- `-d` — запуск контейнера в фоновом режиме.
- `-p` — проброс портов для подключения (1521 для SQLNet, 5500 для Oracle Enterprise Manager, 2484 для TCPS).
- Переменные среды (например, `ORACLE_SID`, `ORACLE_PDB`, `ORACLE_PWD`) позволяют настроить параметры базы данных.
- Флаг `-v` — монтирование каталога данных на хосте.
-Так как данная версия бесплатная, у нее недоступно переименование БД. Даже если поставить  -e ORACLE_SID=ORCL -e ORACLE_PDB=ORCL, контейнер или не соберется или переименует бд по умолчанию
[oracle@dis-work-srv ~]$ docker run -d --name oracle-db -p 1521:1521 -p 5500:5500 -p 2484:2484   --ulimit nofile=1024:65536 --ulimit nproc=2047:16384 --ulimit stack=10485760:33554432 --ulimit memlock=3221225472   -e ORACLE_SID=FREE -e ORACLE_PDB=ORCL -e ORACLE_PWD=Oracle2024   -e INIT_SGA_SIZE=1024 -e INIT_PGA_SIZE=1024 -e INIT_CPU_COUNT=2 -e INIT_PROCESSES=300   -e ORACLE_EDITION=FREE -e ORACLE_CHARACTERSET=AL32UTF8   -v /opt/oracle/oradata   oracle/database:23.5.0-free
a751ae656249c4c912b1b2498f7ad46b39290a3f31a99477e94d19ced361e59a
[oracle@dis-work-srv ~]$ docker ps
CONTAINER ID   IMAGE                         COMMAND                  CREATED         STATUS                            PORTS                                                                                                                             NAMES
a751ae656249   oracle/database:23.5.0-free   "/bin/bash -c $ORACL…"   7 seconds ago   Up 6 seconds (health: starting)   0.0.0.0:1521->1521/tcp, :::1521->1521/tcp, 0.0.0.0:2484->2484/tcp, :::2484->2484/tcp, 0.0.0.0:5500->5500/tcp, :::5500->5500/tcp   oracle-db  
Если хотим следить за процессом:
[oracle@dis-work-srv ~]$ docker logs -f  a751ae656249
.......
Configuring Oracle Listener.
Prepare for db operation
7% complete
...........
     Pluggable database: a751ae656249/FREEPDB1
     Multitenant container database: a751ae656249

 6. Проверка работы контейнера
После запуска контейнера можно проверить его статус:

 Проверяем запущенные контейнеры
docker ps
Покажет 
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS                             PORTS                                                                                                                             NAMES
a751ae656249   oracle/database:23.5.0-free   "/bin/bash -c $ORACL…"   31 seconds ago   Up 31 seconds (health: starting)   0.0.0.0:1521->1521/tcp, :::1521->1521/tcp, 0.0.0.0:2484->2484/tcp, :::2484->2484/tcp, 0.0.0.0:5500->5500/tcp, :::5500->5500/tcp   oracle-db
Это отобразит информацию о работающих контейнерах. Вы должны увидеть контейнер с именем `oracle-db`.
Скопируем CONTAINER ID  и нажимаем 
docker logs a751ae656249 
При правильном запуске лог будет выглядеть примерно так, бд создается 10-15 минут
Specify a password to be used for database accounts. Oracle recommends that the password entered should be at least 8 characters in length, 
contain at least 1 uppercase character, 1 lower case character and 1 digit [0-9]. Note that the same password will be used for SYS, SYSTEM and PDBADMIN accounts:
Confirm the password:
Configuring Oracle Listener.
Listener configuration succeeded.
Configuring Oracle Database FREE.
Enter SYS user password:
*************
Enter SYSTEM user password:
************
Enter PDBADMIN User Password:
*********
Prepare for db operation
7% complete
Copying database files
29% complete
Creating and starting Oracle instance
30% complete

и в самом конце будет

#########################
DATABASE IS READY TO USE!
#########################

Если в логах ошибок нет то идем дальше, если есть то по коду ошибки ORA-123 в процессе придется исправлять или донастраивать БД

 7. Подключение к базе данных
 Подключение к базе данных с использованием SQLPlus
C самого сервера не получится подключиться к бд так как ora_home находится в контейнере
Работать с БД надо утилитой докера
docker exec -ti a751ae656249 sqlplus / as sysdba
Если sqlplus необходим то нужно скачать и установить Oracle Instant Client на сайте https://www.oracle.com/database/technologies/instant-client/downloads.html
На линукс ставим Oracle Instant Client Basic Package и SQL*Plus Package
Если пароль для sys был задан при создании контейнера то заходит так
sqlplus sys/Oracle2024@localhost:1521/FREE as sysdba
Если нет то нужно будет настроить пароль  
Заходим:
docker exec -ti a751ae656249 sqlplus / as sysdba
SQL> ALTER USER sys IDENTIFIED BY Oracle2024;

User altered.
После этого можем заходить 

Контейнер содержит 2 БД  FREE и FREEPDB1
По умолчанию стоит FREE которая является контейнерной базой данных (CDB) все пользователи на уровне CDB должны иметь имя, 
начинающееся с C##, чтобы отличать их от пользователей на уровне pluggable database (PDB).
Перед тем как создавать объекты надо определиться в какой работать 
Чтобы исключить ошибку создания пользователя ORA-65096 переключаемся на FREEPDB1: 
ALTER SESSION SET CONTAINER=FREEPDB1; 
 Или можно просто создавать пользователей с префиксом, например
 SQL>  CREATE USER C##user IDENTIFIED BY Oracle2024;

 8. Доступ к Oracle Enterprise Manager Express

Для доступа к Oracle Enterprise Manager Express (OEM Express) используйте следующий URL в браузере:

https://localhost:5500/em/

Введите учетные данные для входа и управляйте базой данных через веб-интерфейс.

9 Подключение с ПК

9.1 Если на пк установлен Oracle  то переходим в раздел 9.2

Скачиваем  Oracle Instant Client Downloads for Microsoft Windows Basic Package и SQL*Plus Package	
 https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html#ic_winx64_inst 
 Скачаный архив надо распаковать в C:\oracle\instantclient\
 Создать C:\oracle\instantclient_23_6\network\admin  файлы  listener.ora  и  tnsnames.ora в которые надо скопировать данные из пункта 9.2
Так же при необходимости нужно установить Microsoft Visual C++ https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170

9.2 Заходим в контейнер
docker exec -ti a751ae656249  bash
Смотрим статус БД
 lsnrctl status
Он покажет 
LSNRCTL for Linux: Version 23.0.0.0.0 - for Oracle Cloud and Engineered Systems on 27-NOV-2024 07:31:17

Copyright (c) 1991, 2024, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=TCP)(HOST=a751ae656249)(PORT=1521)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for Linux: Version 23.0.0.0.0 - for Oracle Cloud and Engineered Systems
Start Date                27-NOV-2024 05:54:37
Uptime                    0 days 1 hr. 36 min. 40 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Default Service           FREE
Listener Parameter File   /opt/oracle/product/23ai/dbhomeFree/network/admin/listener.ora
Listener Log File         /opt/oracle/diag/tnslsnr/a751ae656249/listener/alert/log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=a751ae656249)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(KEY=EXTPROC1521)))
Services Summary...
Service "27df968603950f81e063040011accf3d" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
Service "FREE" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...

Находим путь до Listener Parameter File /opt/oracle/product/23ai/dbhomeFree/network/admin/
Смотрим содержание 
bash-4.4$ ls /opt/oracle/product/23ai/dbhomeFree/network/admin/
listener.ora  samples  shrept.lst  sqlnet.ora  tnsnames.ora
Смотрим  listener.ora  и  tnsnames.ora

bash-4.4$ cat /opt/oracle/product/23ai/dbhomeFree/network/admin/tnsnames.ora
# tnsnames.ora Network Configuration File: /opt/oracle/product/23ai/dbhomeFree/network/admin/tnsnames.ora
# Generated by Oracle configuration tools.

FREE =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = a751ae656249)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = FREE)
    )
  )

LISTENER_FREE =
  (ADDRESS = (PROTOCOL = TCP)(HOST = a751ae656249)(PORT = 1521))

FREEPDB1 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = FREEPDB1)
    )
  )


bash-4.4$ cat /opt/oracle/product/23ai/dbhomeFree/network/admin/listener.ora
# listener.ora Network Configuration File: /opt/oracle/product/23ai/dbhomeFree/network/admin/listener.ora
# Generated by Oracle configuration tools.

DEFAULT_SERVICE_LISTENER = FREE

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = a751ae656249)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
  
Эти данные нужно отобразить на локальной машине.  
Если обратить внимание то HOST = a751ae656249 это id контейнера. При копировании данных на свой tnsnames.ora нужно изменить на ip-адрес хоста на котором запущен контейнер бд.
Данные надо скопировать в локальные файлы тнс и листенер

Можно подключиться с помощью DBeaver  и сделать простой запрос 
SELECT * FROM all_tables where rownum < 10;
SYS	SYS_IOT_OVER_9644	SYSAUX		RULE_SET_PR$	VALID	10		1	255								YES	N	0	0	0	0	0	0	0	0	         1	         0	    N	ENABLED	0	2024-10-18 01:55:09.000	NO	IOT_OVERFLOW	N	N	NO	DEFAULT	DEFAULT	DEFAULT	DISABLED	YES	NO		DISABLED	YES		DISABLED	DISABLED		NO	NO	NO	DEFAULT	NO			NO	NO	DISABLED					USING_NLS_COMP	N	N	N	N	N	NO	NO		NO	NO	NO	NO			NO	DISABLED	DISABLED	NO	NO	NO	ENABLED	NO	NO	NO
SYS	SYS_IOT_OVER_9657	SYSAUX		RULE_SET_PTPDTY$	VALID	10		1	255								YES	N	0	0	0	0	0	0	0	0	         1	         0	    N	ENABLED	0	2024-10-18 01:55:09.000	NO	IOT_OVERFLOW	N	N	NO	DEFAULT	DEFAULT	DEFAULT	DISABLED	YES	NO		DISABLED	YES		DISABLED	DISABLED		NO	NO	NO	DEFAULT	NO			NO	NO	DISABLED					USING_NLS_COMP	N	N	N	N	N	NO	NO		NO	NO	NO	NO			NO	DISABLED	DISABLED	NO	NO	NO	ENABLED	NO	NO	NO
SYS	SYS_IOT_OVER_9867	SYSAUX		CHNF$_CLAUSES	VALID	10		1	255								YES	N	0	0	0	0	0	0	0	0	         1	         0	    N	ENABLED	0	2024-10-18 01:54:47.000	NO	IOT_OVERFLOW	N	N	NO	DEFAULT	DEFAULT	DEFAULT	DISABLED	YES	NO		DISABLED	YES		DISABLED	DISABLED		NO	NO	NO	DEFAULT	NO			NO	NO	DISABLED					USING_NLS_COMP	N	N	N	N	N	NO	NO		NO	NO	NO	NO			NO	DISABLED	DISABLED	NO	NO	NO	ENABLED	NO	NO	NO
SYS	SYS_IOT_OVER_9907	SYSAUX		CHNF$_GROUP_FILTER_IOT	VALID	10		1	255								YES	N	0	0	0	0	0	0	0	0	         1	         0	    N	ENABLED	0	2024-10-18 01:54:48.000	NO	IOT_OVERFLOW	N	N	NO	DEFAULT	DEFAULT	DEFAULT	DISABLED	YES	NO		DISABLED	YES		DISABLED	DISABLED		NO	NO	NO	DEFAULT	NO			NO	NO	DISABLED					USING_NLS_COMP	N	N	N	N	N	NO	NO		NO	NO	NO	NO			NO	DISABLED	DISABLED	NO	NO	NO	ENABLED	NO	NO	NO

Если все ОК то это все
Для повторного запуска контейнера в случае сбоев и пр. в нашем случае необходимо запустить контейнер, 
а не образ, потому что контейнер уже был создан и содержал все данные и настройки базы данных.
Находим оснтановленный контейнер
[oracle@dis-work-srv ~]$ docker ps -a
CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS                     PORTS                                      NAMES                                                                                                                                                                                                                                                                           great_bardeen
a751ae656249   oracle/database:23.5.0-free   "/bin/bash -c $ORACL…"   23 hours ago      Stopped     0.0.0.0:1521->1521/tcp, :::1521->1521/tcp  oracle-db

Берем CONTAINER ID и запускаем его. 
[oracle@dis-work-srv ~]$ docker start a751ae656249 
БД поднимается пару минут. Можно повторно посмотреть статус и если статус будет  Up 4 minutes (healthy) то все ОК.
Можем доступность проверить через DBeaver или командой docker exec -ti a751ae656249 sqlplus / as sysdba
Можно так же посмотреть внутри контейнера 
Заходим в контейнер
[oracle@dis-work-srv ~]$ docker exec -it  a751ae656249  bash
Смотрим статус(надо обратить внимание на то что мы внутри контейнера) и проверяем статус листенера
bash-4.4 lsnrctl status/start/stop

в конце строки должно быть  строки о готовности бд
Service "freepdb1" has 1 instance(s).
  Instance "FREE", status READY, has 1 handler(s) for this service...
The command completed successfully

Если будут какие то ошибки и бд не поднимется то смотрим логи контейнера и исследуем ошибки
 docker logs   a751ae656249  
