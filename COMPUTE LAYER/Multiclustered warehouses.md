Po co są multi-cluster [[WAREHOUSES]]:

If queries are queuing more than desired, another warehouse can be created and queries can be manually redirected to the new warehouse. In addition, resizing a warehouse can enable limited scaling for query concurrency and queuing; however, warehouse resizing is primarily intended for improving query performance.To enable fully automated scaling for concurrency, Snowflake recommends [multi-cluster warehouses](https://docs.snowflake.com/en/user-guide/warehouses-multicluster), which provide essentially the same benefits as creating additional warehouses and redirecting queries, but without requiring manual intervention

Defaultowo jest tworzony 1-klastrowy warehouse.

Multi-cluster warehouses are an [Enterprise Edition](https://docs.snowflake.com/en/user-guide/intro-editions) feature.

Multi-cluster ustawia się DWIEMA dodatkowymi opcjami:
MIN_CLUSTER_COUNT = <liczba>  
MAX_CLUSTER_COUNT = <liczba>

Np.:
CREATE WAREHOUSE ETL_VWH WITH
  WAREHOUSE_SIZE = 'SMALL'
  WAREHOUSE_TYPE = 'STANDARD'
  MIN_CLUSTER_COUNT = 1
  MAX_CLUSTER_COUNT = 4
  AUTO_SUSPEND = 300
  AUTO_RESUME = TRUE

5️⃣ [[Scaling: SCALE UP vs SCALE OUT]]

**Scale UP**
- Zwiększasz rozmiar warehouse
- Lepsze dla:
	- pojedynczych ciężkich zapytań
	- ETL

**↔ Scale OUT** - polega na dodawaniu kolejnego klastra do multi cluster warehouse
- MIN_CLUSTER_COUNT
- MAX_CLUSTER_COUNT
- Kilka klastrów równolegle

Najważniejsze:

- Multi-cluster = concurrency, nie szybsze pojedyncze query

💡 Egzamin kocha to zdanie:

Multi-cluster warehouses improve concurrency, not query performance

Można te min i max ustawic na 2 sposoby i to wplywa na tryb w którym uzywamy multicluster warehousu:

MIN_CLUSTER=MAX_CLUSTER_COUNT --> maximized mode:  when the warehouse is started, Snowflake starts all the clusters so that maximum resources are available while the warehouse is running. This mode is effective for statically controlling the available compute resources, particularly if you have large numbers of concurrent user sessions and/or queries and the numbers do not fluctuate significantly.

MIN_CLUSTER != MAX_CLUSTER_COUNT --> autoscale mode:  different values for maximum and minimum number of clusters. In this mode, Snowflake starts and stops clusters as needed to dynamically manage the load on the warehouse.  To help control the usage of credits in Auto-scale mode, Snowflake provides a property, SCALING_POLICY, that determines the scaling policy to use when automatically starting or shutting down additional clusters.

[[SCALING POLICY: standard vs economy]]

- STANDARD - Prevents/minimizes queuing by favoring starting additional clusters over conserving credits.
- ECONOMY - Conserves credits by favoring keeping running clusters fully-loaded rather than starting additional clusters, which may result in queries being queued and taking longer to complete. Wlacza nowy warehouse dopiero gdy przewiduje że kolejka bedzie trwac ponad 6 minut. 

Zapamiętaj:

- STANDARD → performance > cost
- ECONOMY → cost > performance


![[Zrzut ekranu 2026-05-4 o 22.01.31.png]]


📌 Typowe pytanie:

Który tryb minimalizuje koszty przy burstach? → ECONOMY

Currently, Snowsight supports updating MAX_CLUSTER_COUNT to a maximum of 10 clusters. To increase MAX_CLUSTER_COUNT beyond 10, use the ALTER WAREHOUSE command in SQL. You can increase or decrease the maximum and minimum number of clusters for a warehouse at any time, even while it is running and executing statements. 

7️⃣ Warehouse a concurrency

- Jeden warehouse:

- obsługuje wiele zapytań
- przy dużej liczbie → kolejka

- Rozwiązania:

- multi-cluster
- więcej warehouse’ów
- query acceleration (tylko świadomość istnienia)

Monitorowanie wykorzystania warehousu:

Warehouse query load measures the average number of queries that were running or queued within a specific interval.

Query load is calculated by dividing the execution time (in seconds) of all queries in an interval by the total time (in seconds) for the interval.

If the chart shows recurring time periods when the warehouse was running and consuming credits, but the total query load was less than 1 for substantial periods of time, the warehouse use is inefficient. You might consider any of the following actions:

- Decrease the warehouse size. Note that decreasing the warehouse size generally increases the query execution time.
- For a multi-cluster warehouse, decrease the MIN_CLUSTER_COUNT parameter value.