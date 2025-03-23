# Лабораторная работа №5: Запуск сайта в контейнере

## Цель работы:

Выполнив данную работу студент сможет подготовить образ контейнера для запуска веб-сайта на базе Apache HTTP Server + PHP (mod_php) + MariaDB.

## Задание:
Создать Dockerfile для сборки образа контейнера, который будет содержать веб-сайт на базе Apache HTTP Server + PHP (mod_php) + MariaDB. База данных MariaDB должна храниться в монтируемом томе. Сервер должен быть доступен по порту 8000.

Установить сайт WordPress. Проверить работоспособность сайта.

## Подготовка:
Для выполнения данной работы необходимо иметь установленный на компьютере Docker.

Для выполнения работы необходимо иметь опыт выполнения лабораторной работы №3.

## Выполнение работы:
### 1. Создал репозиторий containers 05.

- Ссылка на репозиторий:
[Repo](https://github.com/alexyakimenko/containers05/blob/main/files/wp-config.php)

### 2. Создание Dockerfile
В файле Dockerfile добавил следующий код:
```
# create from debian image
FROM debian:latest

# install apache2, php, mod_php for apache2, php-mysql and mariadb
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php php-mysql mariadb-server && \
    apt-get clean
```
---
### Создаю образ из докер файла
![build-apache2-php-mariadb-image](images/1.%20build-apache2-php-mariadb-image.png)

Смотрим итоговый список образов:

![](images/2.%20build-apache2-php-mariadb-image-result.png)
---
### 3. Извлечение конфигурационных файлов из контейнера
---
1. Для начала запускаем контейнер

![](images/3.%20create-container-apache2-php-mariadb.png)
---
2. Далее копируем необходимые конфигурационные файлы для apache2,php,mariadb в папку **files/**:

![](images/4.%20copy-the-apache2-php-mariadb-configs.png)
---
3. Проверяю наличие файлов после копирования и далее останавливаю и удаляю контейнер:

![](images/5.%20stop-remove-apache-php-mariadb-container.png)
---
### 4. Настройка конфигурационных файлов
---
### 4.1 Конфигурационные файлы apache2
---

1. Меняю в **apache2\000-default.conf**,  **ServerName** на **localhost**:

![](images/6.%20apache-server-name-change.png)
---
2. Меняю **ServerAdmin** на свою почту:

![](images/7.%20change-server-admin.png)
---
3. Устанавливаю **DirectoryIndex** в *index.php* *index.html*:

![](images/8.%20add-directory-index.png)
---
4. В конце файла **files/apache2/apache2.conf** добавляю следующую строку:

![](images/9.%20add-server-name-to-the-end-apache-conf.png)
---
### 4.2 Конфигурационный файл php
---
1. В php.ini заменяю **;error_log = php_errors.log** на **error_log = /var/log/php_errors.log**:

![](images/10.%20add-error-log-in-php-ini.png)
---
2. Настраиваю параметры memory_limit, upload_max_filesize, post_max_size и max_execution_time следующим образом:

```
memory_limit = 128M
upload_max_filesize = 128M
post_max_size = 128M
max_execution_time = 120
```
![](images/11.%20change-upload-max-filesize-to-128m.png)
![](images/12.%20set-post_max_size-to-128m.png)
![](images/13.%20set-max_execution_time-to-120.png)

---
### 4.3 Конфигурационный файл mariadb
---

1. Раскоментирую параметр **log_error** в **50-server.cnf** :

![](images/14.%20uncomment-log_error-in-50-server-cnf.png)

---

### 5. Создание скрипта запуска

---

1. Добавляю **supervisor.cnf** в новую папку **supervisor** в **files/**:

![](images/15.%20add-supervisor-cnf-to-supervisor-folder.png)
---


### 6. 

1. Добавляю монтирование волюмов для **mysql** и **логов** в **Dockerfile**:

![](images/16.%20add-volume-mount-after-from.png)

---

2. Добавляю установку supervisor вместе установкой wordpress и его распоковкой:

![](images/17.%20add-supervisor-installation-and-wordpress-installation.png)

**Примечание:**

в скрине забыто добавление команды по распаковке архива:
```
RUN tar xf /var/www/html/latest.tar.gz -C /var/www/html/
```
![](images/25.%20fix-image.png)

---

3. Добавляю копирование конфигурационных файлов **apache2, php, mariadb,** а также скрипта запуска:
![](images/18.%20add-copy-for-apache-php-mariadb-conf-files-in-dockerfile.png)

---

4. Добавляю для для функционирования **mariadb** создайте папку **/var/run/mysqld** и установите права на неё:

![](images/19.%20create-mysql-socket-directory.png)

---

5. Открываю порт **80** и добавляю команду запуска **supervisord**:

![](images/20.%20expose-80-port-and-supervisor-start-cmd.png)

---

6. Пересобираю образ из измененного **Dockerfile** и создаю новый контейнер:

![](images/21.%20rebuild-image.png)

---

![](images/22.%20create%20new%20container.png)

---

### 7. Создание базы данных и пользователя

---

1. Создаю базу данных **wordpress** и пользователя **wordpress** с паролем **wordpress** в контейнере **apache2-php-mariadb**:
![](images/23.%20mariadb%20setup.png)


### 8. Создание файла конфигурации WordPress

1. По ходу выполнения была ошибка при запуске контейнера из за неверного копирования конфига supervisor, исправление:

![](images/24.%20fix-image.png)

---

2. Открываю wordpress и ввожу данные для фомирования ```wp-config.php```

![](images/26.%20open-wp-enter-data.png)

---

3. Создаю в **files/** ```wp-config.php``` и копирую его внутрь контейнера при сборке образа:

![](images/27.%20copy-wp-config-php.png)

---

4. Далее продолжаю установку ввожу необходимую информацию и в конечном итоге попадаю на страницу с wordpress:

![](images/28.%20add-wordpress-info.png)

---

![](images/29.%20finish-installation.png)

# Ответы на контрольные вопросы:
- Какие файлы конфигурации были изменены?

> 000-default.conf (Apache)
>
>apache2.conf (Apache)
>
>php.ini (PHP)
>
>50-server.cnf (MariaDB)
>
>wp-config.php (WordPress)

- За что отвечает инструкция DirectoryIndex в файле конфигурации Apache?

> Определяет файлы, которые сервер будет загружать в первую очередь при обращении к директории (например, index.php, index.html).

- Зачем нужен файл wp-config.php?

> Содержит настройки WordPress, включая параметры подключения к базе данных.

- За что отвечает параметр post_max_size в файле конфигурации PHP?

> Определяет максимальный размер данных, которые можно отправить в POST-запросе.

- Какие недостатки есть в созданном образе контейнера?

> Нет разделения сервисов (Apache, PHP, MariaDB) на отдельные контейнеры.
>
> Используется latest-версия Debian, что может привести к непредвиденным проблемам.
>
> WordPress загружается внутри контейнера, что усложняет обновления.
# Выводы
В ходе выполнения лабораторной работы №5 был успешно создан и настроен контейнер для веб-сайта на базе Apache HTTP Server + PHP (mod_php) + MariaDB. 
1. Подготовлен Dockerfile, содержащий установку необходимых пакетов и настройку окружения.
2. Настроены ключевые параметры, включая ServerName, логирование ошибок, размер загружаемых файлов и таймаут выполнения скриптов.
3. Добавлен и настроен Supervisor для управления процессами в контейнере.
4. Установлен и настроен WordPress, включая создание базы данных и конфигурационного файла wp-config.php.

В результате лабораторной работы был получен полностью функционирующий контейнеризированный веб-сайт, доступный по порту 80, с правильно настроенными сервисами и возможностью развертывания в любом окружении с Docker.