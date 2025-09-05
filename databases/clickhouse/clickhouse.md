
## Диагностика медленной работы узла

возможные причины медленной работы:
- долгие запросы, можно посмотреть запросом, который вернет самые тяжелые запросы по памяти со всех узлов кластера:
	- логи узла `tail -f /var/log/clickhouse-server/clickhouse-server.log`
	- получение запросов за сегодня, отсортированных по cpu [stackoverflow](https://stackoverflow.com/questions/71178274/is-there-a-way-to-see-cpu-usage-for-a-specific-clickhouse-query)
		```SQL
			select any(query), sum(`ProfileEvents.Values`[indexOf(`ProfileEvents.Names`, 'UserTimeMicroseconds')]) userCPU from system.query_log 
			where type = 2 and event_date >= today()
			group by normalizedQueryHash(query)
			order by userCPU desc
			limit 10
		```
	- [docs](https://clickhouse.com/docs/knowledgebase/find-expensive-queries) , [docs для отдельных полей запросов](https://clickhouse.com/docs/operations/system-tables/query_log)
	- получение данных долгих запросов
	 ```SQL
		SELECT  
			type,  
			event_time,  
			initial_query_id,  
			query_id,  
			formatReadableSize(memory_usage) AS memory,  
			ProfileEvents.Values[indexOf(ProfileEvents.Names, 'UserTimeMicroseconds')] AS userCPU,  
			ProfileEvents.Values[indexOf(ProfileEvents.Names, 'SystemTimeMicroseconds')] AS systemCPU,  
			normalizedQueryHash(query) AS normalized_query_hash  
			FROM clusterAllReplicas(default, system.query_log)  
			ORDER BY memory_usage DESC  
			LIMIT 10;
	    ```
    ```SQL
	    # данные контретного запроса по его initial_query_id
	    SELECT  
			hostname(),  
			type,  
			event_time,  
			initial_query_id,  
			query_id,  
			formatReadableSize(memory_usage) AS memory,  
			ProfileEvents.Values[indexOf(ProfileEvents.Names, 'UserTimeMicroseconds')] AS userCPU,  
			ProfileEvents.Values[indexOf(ProfileEvents.Names, 'SystemTimeMicroseconds')] AS systemCPU,  
			normalizedQueryHash(query) AS normalized_query_hash  
			FROM clusterAllReplicas(default, system.query_log)  
			WHERE initial_query_id = 'a7262fa2-bd8b-4b51-a359-621ccf282417'  
			ORDER BY memory_usage DESC  
			LIMIT 10 FORMAT Pretty;
    ```
	
- из-за mutations [github issue](https://github.com/ClickHouse/ClickHouse/issues/39403) - можно увидеть в grafana по резкому повышению нагрузки без ее спада после записи в таблицу
	```SQL
		SELECT *
		FROM system.mutations
		ORDER BY latest_fail_time DESC
	```
	- для удаления мутаций, которые падают с ошибками из-за того, что напрмер не могут преобразовать некорректный тип:
	```
		kill mutation where mutation_id =  `mutation_97807.txt`;
	    alter table logging.`shop-fast-query` modify MODIFY COLUMN `cost` String;
		alter table update cost = 0 where cost = '' settings mutations_sync=2;
		alter table logging.`shop-fast-query` modify MODIFY COLUMN `cost` Int64;
	```
