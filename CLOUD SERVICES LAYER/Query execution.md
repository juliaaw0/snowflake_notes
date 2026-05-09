
[[CLOUD SERVICES LAYER]]
# QUERY EXECUTION AND CACHING


All client tools connecting to Snowflake, whether the web UI is SnowSQL or any other snowflake drivers, use the REST interface to connect to Snowflake. All queries and connections are to the cloud services layer which authenticates and authorizes a query. The query is compiled and optimised by the cloud services layer, after which its execution starts.

New queries (never executed before): The first step towards query execution starts in the cloud services layer where the query compilation occurs. The SQL is converted into a query plan consisting of several steps. The query optimizer in the Cloud services layer also participates in optimizing the query to ensure the most efficient execution. After the query plan is generated, the cloud services layer typically passes the query to a virtual warehouse. The virtual warehouse reads data from the required tables in the storage layer and performs the necessary  processing as dictated by the query.  This is where micro partition pruning may occur. That is, that snowflake can decide to read only micro partitions that are relevant to the query. As the micro partitions are scanned, the data is brought to the virtual warehouse ram where processing occurs on it. If the data does not fit in the RAM, then it may be spilled to the disk. Note that as a virtual warehouse processes a query, it creates a [[local disk cache]] called Warehouse Cache. A virtual warehouse may re-use the warehouse cache for query processing. This cache is local to the virtual warehouse executing the query, and any other virtual warehouses cannot see it. If the virtual warehouse is suspended or deleted, this cache will also be removed. 

In the next step of query processing, the results of the processed query are sent back to the cloud services layer.

The Cloud Services Layer stores the results in [[query result cache]].  The purpose of query result cache is to improve the performance of queries that have already been processed. 

suppose a query that is identical to a previously executed query is sent. In that case, the query result cache is used by Snowflake to return results directly back to the user using the cache. It does not require invoking virtual warehouse processing and reduces processing costs.

Another important question mechanism is the [[metadata cache]], which maintains specific statistics such as the count of rows at the metadata level. Therefore, a count query will never invoke a virtual warehouse, but rather always return results from the metadata cache.

czyli query korzystajace z query result cache i metadata cache nie uzywaja warehouse. Query result cache i metadata cache są w cloud services layer, a local disk cache (ta warehousowa) jest częścią compute layer bo siedzi w środku konkretnego warehousu. 




### Diagnostyka: Query Profile & Query History

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Narzędzia administratora)

Musisz wiedzieć, gdzie szukać problemów, gdy zapytanie "muli":

- **Query History:** Widok wszystkich zapytań z ostatnich 14 dni (w UI) lub 365 dni (w Account Usage).
    
- **Query Profile:** Graficzne narzędzie pokazujące, co "zjadło" czas w zapytaniu. Szukaj tych pojęć:
    
    - **Exploding Joins:** Gdy Join produkuje miliony wierszy (źle dobrane klucze).
        
    - **Spilling to Disk:** Gdy Warehouse ma za mało RAM i musi zapisywać dane tymczasowe na dysku lokalnym (Local Disk) lub zdalnym (Remote Disk – najwolniej!). To sygnał, że potrzebujesz większego Warehouse (**Scale Up**).
        
    - **Data Pruning:** Ile mikropartycji zostało przeskanowanych, a ile pominiętych.
