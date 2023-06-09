# Домашнее задание к уроку 10. #   
1. Проверила состояние параметра интервала контрольной точки checkpoint_timeout.  
![Шаг4](/10_1_chp_to_before.jpg)  
Выполнила настройку контрольной точки раз в 30 секунд. Поменяла checkpoint_timeout через alter system и перезагрузила конфигурацию.   
![Шаг4](/10_2_chp_to_after.jpg)  
1. Подготовила утилиту pgbench к запуску.   
![Шаг4](/10_3_pgbench_start.jpg)  
Cохранила данные по количеству контрольных точек и текущему LSN.  
![Шаг4](/10_4_pgbench_before.jpg)   
1. Подала нагрузку на 10 минут c помощью утилиты pgbench.   
![Шаг4](/10_5_pgbench.jpg)  
Cохранила данные по количеству контрольных точек и текущему LSN.  
![Шаг4](/10_6_pgbench_after.jpg)  
**Видим, что количество выполненных контрольных точек 51-15=36.**    
1. Определим число байт в журнале предзаписи (WAL) до и после выполнения утилиты pgbench (нужно было наооборот вычитать из конца начало, тогда не было бы минуса):   
![Шаг4](/10_7_pgbench_size.jpg)   
**Получим, что на одну контрольную точку приходится 466246296/36 = 12 951 286, т.е. примерно 12Мб.**   
1. Посмотрела в журнале все ли контрольные точки выполнялись точно по расписанию. Из лога видно, что, например, контрольная точка стартует в 20:18:35 и завершается в 20:19:02.   
![Шаг4](/10_8_pgbench_logX.jpg)  
![Шаг4](/10_10_pgbench_log3.jpg)  
**За это отвечает параметр checkpoint_completion_target. 0,9*30сек=27сек**   
1. Данные tps в режиме синхронной фиксации транзакций   
![Шаг4](/10_5_pgbench.jpg)  
1. Для замера tps в режиме асинхронной фиксации транзакций изменила параметр synchronous_commit   
![Шаг4](/10_11_sync1.jpg)  
![Шаг4](/10_12_sync2.jpg)  
![Шаг4](/10_13_sync3.jpg)  
1. Запустила проверку в режиме асинхронной фиксации транзакций   
![Шаг4](/10_14_sync4.jpg)  
**Видем прирост tps в 6 раз (3461/551) по сравнению с синхронным режимом фиксации транзакций. В асинхронном режиме не нужно ждать, пока записи из WAL сохранятся на диске, прежде чем сообщить об успешном завершении операции. Поэтому кол-во обработанных транзакций становится выше.**   
1. Создала новый кластер с включенной контрольной суммой страниц.   
![Шаг4](/10_15_cluster1.jpg)  
![Шаг4](/10_16_cluster2.jpg)  
1. Создала таблицу с несколькими значениями.   
![Шаг4](/10_17_cluster3.jpg)  
Уточнила, где ее рыть на диске.   
![Шаг4](/10_18_cluster4.jpg)  
1. Выключила кластер и изменила пару байт в таблице.   
![Шаг4](/10_19_cluster5.jpg)  
1. Включила кластер и сделала выборку из таблицы. Вышла ошибка проверки суммы контрольных страниц.   
![Шаг4](/10_20_cluster6.jpg)  
Игнорировать ошибку можно через параметр ignore_checksum_failure=true
![Шаг4](/10_21_cluster7.jpg)  
**Данные прочитаны успешно, значит заголовок блока с данными уцелел. При повреждении заголовка, была бы выдана ошибка вне зависимости от параметра ignore_checksum_failure**
