### Query Acceleration Service (QAS)

**Ważność na egzaminie:** ⭐⭐⭐⭐

To stosunkowo nowa funkcja, którą Snowflake nazywa "dodatkowym zastrzykiem mocy".

- **Jak to działa?** Snowflake wykrywa zapytanie, które skanuje gigantyczne ilości danych (np. potężny skan tabeli bez filtrów). Zamiast zwiększać cały Warehouse, system "wypożycza" dodatkowe zasoby z puli Snowflake'a tylko dla tego jednego zadania.
    
- **Zaleta:** Nie musisz płacić za większy Warehouse przez cały czas, a tylko za te sekundy, gdy QAS pomaga Twojemu zapytaniu.
    
- **Gdzie to widać?** W Query Profile zobaczysz statystykę: "Query acceleration upper bound".




The query acceleration service (QAS) can accelerate parts of the query workload in a [[WAREHOUSES]]. When it is enabled for a warehouse, it can improve overall warehouse performance by reducing the impact of outlier queries, which are queries that use more resources than the typical query. The query acceleration service does this by offloading portions of the query processing work to shared compute resources that are provided by the service.

Examples of the types of workloads that might benefit from the query acceleration service include:

- Ad hoc analytics.
- Workloads with unpredictable data volume per query.
- Queries with large scans and selective filters.

The query acceleration service supports the following SQL commands:

- SELECT
- INSERT
- CREATE TABLE AS SELECT (CTAS)
- COPY INTO table

Within a supported SQL command, QAS might accelerate an entire query, or a subquery or clause within the query.

The query acceleration service might increase the credit consumption rate of a warehouse. The maximum scale factor can help limit the consumption rate.

For example, suppose that you set the scale factor to 5 for a medium warehouse. This means that the warehouse can lease additional compute resources up to 5 times the size of a medium warehouse.  If you use QAS for a multi-cluster warehouse, consider increasing the scale factor. That way, all the warehouse clusters can take advantage of the QAS optimizations.Not all queries require the full set of resources that are made available by the scale factor. The query acceleration service only uses as many resources as it needs and that are available at the time the query is executed.

In general, queries are eligible because they have a portion of the query plan that can be run in parallel using QAS compute resources. These queries fall into one of two patterns:

- Large scans with an aggregation or selective filter.
- Large scans that insert or copy many new rows (for example, INSERT and COPY commands).

Żeby wyswietluc widok QUERY_ACCELERATION_ELIGIBLE view trzeba mieć role ACCOUNTADMIN

The cost is the same no matter how many queries are using the query acceleration service at the same time. The query acceleration service is billed by the second, only when the service is in use. These credits are billed separately from warehouse usage.