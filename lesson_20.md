# Домашнее задание к уроку 20. Секционирование. Секционировать большую таблицу из демо базы flights #   
1. Создала виртуальную машину в Yandex-Cloud. (IP адрес 51.250.25.251)   
![Шаг4](/20_01_vm.jpg)  
1. Установила PostgreSQL 15.   
![Шаг4](/20_02_postgres.jpg)  
![Шаг4](/20_03_cluster.jpg)  
1. Скачала архив с демо-базой. Скачала архиватор unzip и распаковала демо базу.  
![Шаг4](/20_04_get_arh.jpg)  
![Шаг4](/20_05_unzip.jpg).  
![Шаг4](/20_06_unpack.jpg)  
1. Командой \i <имя файла> запустила импорт демо базы в свою версию Postgres.   
![Шаг4](/20_07_insert_bd.jpg)  
1. Создала секционированную таблицу p_flights по диапазону поля scheduled_departure со структурой, аналогичной flights но без констрейнтов. Затем создала на ней все констрейнты от оригинальной таблицы, кроме уникальных (иначе не получится секционировать, уникальные ограничения должны быть в ключе секционирования).    
![Шаг4](/20_08_create_tbl.jpg)  
1. Подключила таблицу flights как дефолтную секцию к новой таблице p_flights. Переименовала таблицы. Теперь получаем секционированную таблицу flights с одной секцией.   
![Шаг4](/20_09_def_renames.jpg)  
1. Сделала анализ данных поля scheduled_departure. Решила создать 2 секции для данных на 2017год с разбивкой по полугодиям. Создала эти таблицы как копии дефолтной секции flights_def. Затем скопировала данные 1 полугодия в таблицу flights_17_01_06, а данные 2 полугодия в таблицу flights_17_07_12. Попробовала удалить эти данные из дефолтной секции. Не прокатило, т.к. есть таблица, которые ссылается по ключу на дефолтную секцию.   
![Шаг4](/20_10_get_data.jpg)  
1. Удалила ограничение с таблицы ticket_flights. Очистила данные в дефолтной секции.    
![Шаг4](/20_11_truncate.jpg)  
1. Выполнила attach partition поочередно для двух созданных секций добавив их к таблице flights. Посмотрела структуру  таблицы таблицу flights: таблица секционирована, согласно заданию, есть дефолтная секция и 2 секции по диапазону дат, сохранены ограничения, кроме уникальных.   
![Шаг4](/20_12_add_part.jpg)
Цель ДЗ достигнута, секционирование выполнено. Из потерь первичный ключ flights_pkey, уникальное ограничение flights_flight_no_scheduled_departure_key, которые нельзя создать на базовой таблице, т.к. они не в ключе секционирования. Также на таблице ticket_flights удалено ограничение ticket_flights_flight_id_fkey, которое не может больше поддерживаться из-за отсутствия первичного ключа на flights.
