# Домашнее задание к уроку 12. Настройка PostgreSQL #   
1. Создала виртуальную машину в Yandex-Cloud.   
![Шаг4](/12_1_create_vm.jpg)  
Установила PostgreSQL 15.   
![Шаг4](/12_2_inst_postgre.jpg)  
![Шаг4](/12_3_postgre_ok.jpg)  
1. Подготовила утилиту pgbench к запуску.      
![Шаг4](/12_4_pgbench_bef.jpg)  
1. Провела тестирование работы Postgres на базовой конфигурации.  
![Шаг4](/12_5_pgbench_first.jpg)  
1. Сохранила исходный файл postgresql.conf и изменила настройки для повышения производительности без учета надежности работы системы.   
![Шаг4](/12_6_postgresql_conf1.jpg)  
![Шаг4](/12_7_postgresql_conf2.jpg)  
В файле postgresql.conf сделала следующие настройки:      
   maintenance_work_mem            1GB   
   shared_buffers                  1GB   
   work_mem                        16MB   
   checkpoint_completion_target    0.9   
   checkpoint_timeout              30min   
   min_wal_size                    500MB   
   max_wal_size                    5GB   
   bgwriter_lru_maxpages           1000   
   bgwriter_lru_multiplier         10.0   
   effective_cache_size            12GB   
   random_page_cost                1.0   
   wal_compression                 on   
   fsync                           off   
   full_page_writes                off   
   synchronous_commit              off   
1. Перезагрузила кластер.   
![Шаг4](/12_8_restart.jpg)  
1. Провела тестирование работы Postgres в новой конфигурации.   
![Шаг4](/12_9_pgbench_second.jpg)   
Итог: tps со значения 574 вырос до 2370, т.е. более чем в 4 раза.   
Для повешения кол-ва tps я постаралась ограничить нагрузку на диск, улучшить работу с паматью, не обращая внимание на надежность системы.   
# shared_buffers # - используется для кеширования данных, чем больше данных в памяти, тем выше скорость.
# shared_buffers # - используется для кеширования данных, чем больше данных в памяти, тем выше скорость.
Задачу под звездочкой (аналогично протестировать через утилиту sysbench-tpcc) постараюсь сделать позже.
