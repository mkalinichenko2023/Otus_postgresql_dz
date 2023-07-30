# Домашнее задание к уроку 18. Работа с индексами, join'ами, статистикой #   
## Вариант 1. Создать индексы на БД, которые ускорят доступ к данным. ##   
1. Создала виртуальную машину в Yandex-Cloud. (IP адрес 158.160.24.50)   
![Шаг4](/18_01_vm.jpg)  
1. Установила PostgreSQL 15.   
![Шаг4](/18_02_postgres.jpg)  
![Шаг4](/18_03_cluster.jpg)  
1. Выполнять задания решила на готовой демо базе (как в дз к 20-занятию). Скачала базу, распакова и загрузила в установленный Postgres.  
![Шаг4](/18_04_get_arh.jpg)
![Шаг4](/18_05_insert_bd.jpg)
*Ниже схема БД*
![Шаг4](/18_00_db.jpg)
1. Посмотрела, какие индексы есть на таблице flights.  
![Шаг4](/18_06_view_ind.jpg)  
1. Решила создать индексы по полям departure_airport (аэропорт вылета) и arrival_airport (аэропорт прибытия).  
![Шаг4](/18_07_create_ind.jpg)  
Выполнила поиск кол-ва рейсов, отправляющихся из Внуково. Проверила план, индекс flights_dep_airport используется.  
![Шаг4](/18_08_use_ind1.jpg)  
Выполнила поиск кол-ва рейсов, прибывающих в Пулково. Проверила план, индекс flights_arr_airport используется.  
![Шаг4](/18_09_use_ind2.jpg)  
1. С целью показать реализацию индекса для полнотекстового поиска создала новое поле в таблице tickets и заполнила его контактной информацией пассажира через вызов to_tsvector.  
*alter table bookings.tickets add column tsv_contact_data tsvector;*  
*update bookings.tickets set tsv_contact_data = to_tsvector('english',contact_data);*  
Создала индекс для полнотекстового поиска по полю tsv_contact_data, попробовала найти пассажира по номеру телефона и проверила командой explain, что индекс задействован.  
![Шаг4](/18_10_txt_ind.jpg)  
1. Пример реализации индекса на часть таблицы сделала по таблице flights. В таблице есть поле status. Посмотрела, какие статусы есть в таблице и решила выделить в отдельный поиск рейсы, которые еще не выполнены (статус Scheduled). Проверила, что индекс используется.  
*create index flights_status on bookings.flights(status) where status='Scheduled';*   
![Шаг4](/18_11_part_ind1.jpg)  
1. Индекс на несколько полей таже создала по таблице flights. Выбрала поля дата прибытия и номер рейса. Проверила, что индекс используется.   
![Шаг4](/18_12_2field_ind.jpg)  
## Вариант 2. Запросы с различными типами соединений. ##  
1. Прямое соединение двух или более таблиц выполнила для запроса поиска номеров мест бизнес класса для опеределенной модели самолета.  
*select * from bookings.aircrafts_data a  
              inner join bookings.seats s on a.aircraft_code=s.aircraft_code  
where a.model['en']='"Boeing 737-300"' and s.fare_conditions='Business';*   
![Шаг4](/18_21_inner.jpg)  
Выбираются строки, которые одновременно есть в первой и во второй таблице.  
1. Для иллюстрации левостороннего соединения двух таблиц сделала выборку из таблиц аэропортов и рейсов. Выбрала аэропорты, в которые не прибывает ни один рейс.  
*select a.airport_code,a.airport_name['en'],a.city['en'],a.timezone,f.flight_no,f.scheduled_departure,f.scheduled_arrival  
from bookings.airports_data a  
     left join bookings.flights f on a.airport_code=f.arrival_airport where f.arrival_airport is null;*  
![Шаг4](/18_22_left.jpg)  
1. В качестве примера полного соединения двух таблиц сделала выборку аэропортов, которых нет в таблице рейсов как по отправлению, так и по прибытию.  
*select p1.airport_code,p1.airport_name['en'],p2.airport_code,p2.airport_name['en']  
from (select a.* from bookings.airports_data a left join bookings.flights f on a.airport_code=f.departure_airport where f.departure_airport is null) p1  
     inner join  
     (select a.* from bookings.airports_data a left join bookings.flights f on a.airport_code=f.arrival_airport where f.arrival_airport is null) p2  
     on p1.airport_code=p2.airport_code;*  
![Шаг4](/18_23_full.jpg)  
1. Запрос с разными типами соединений выбирает модели самолетов, которые не использованы в рейсах, и считает кол-во мест в них.   
*select a.model,count(case when s.fare_conditions='Business' then 1 else null end) B_cnt,count(case when s.fare_conditions='Business' then null else 1 end) E_cnt  
from bookings.aircrafts_data a  
              inner join bookings.seats s on a.aircraft_code=s.aircraft_code  
              left join bookings.flights f on a.aircraft_code=f.aircraft_code  
where f.aircraft_code is null  
group by a.model;*  
![Шаг4](/18_24_diff.jpg)  
1. Реализовать кросс соединение двух или более таблиц. Ничего особо разумного по данной демо базе в голову не пришло. Сделала кросс соединение моделей самолетов со сгенерированной таблицей дат, может пригодится для отчетности.  
*select * from (  
select a.model['en'],grid_date  
from bookings.aircrafts_data a  
     cross join generate_series(date'2017-07-01',date'2017-07-05','1 day') grid_date  
order by 1,2) as q limit 15;*  
![Шаг4](/18_23_full.jpg)  


