[[STORAGE LAYER]]

All data in Snowflake is maintained in databases. Each database consists of one or more schemas, which are logical groupings of database objects, such as tables and views. Snowflake does not place any hard limits on the number of databases, schemas (within a database), or objects (within a schema) you can create. Two of the principal concepts utilized in Snowflake physical table structures are: micro-partitions and data clustering.

Po co powstalo: 
-bo pliki w chmurze jak już sa tam wrzucone to nie mogą być modyfikowane (cloud storage jest immutable). Stanowi podstawe dla time-travel, secure data sharing i cloning.
-Bo w snowflake nie ma tradycyjnych indeksow jak w db, tylko sprawdzane sa metadane micropartitions (w [[Metadata Cache]] w cloud services layer) i jest robiony pruning.

Compared to traditional static partitioning, micro partitions and snowflake are managed automatically and don't require intervention by the user. As the name suggests, micro partitions are relatively small and each micro partition will generally contain 50 MP to 500 MP of uncompressed data. However, do note that the actual stored data is smaller as data in snowflake is always stored with compression.

Micro partitions are added to a table in the order of how the data arrived in the table.

So if additional data is added to a table, another micro partition or possibly multiple micro partitions, depending on the size of the data, will be created to accommodate this data. Micro partitions are immutable, which means they cannot be changed once created. Any update to existing data or loading of new data into a table will result in new micro partitions being created. Therefore, it is not necessary that similar partition values will always be in the same physical partition. Snowflake must keep track of what range of data is in which partitions so that it can use that information for efficient query processing. Snowflake maintains several different kinds of metadata (clustering metadata) for a given table for this purpose. It stores the range of column values in its metadata.That is the maximum and minimum value for each column in each micro partition. With this clustering information (czyli tymi metadanymi), snowflake can intelligently decide which partitions to read when processing a query. Now another important aspect is that within each micro partition, the data is stored in a columnar format, so each column is stored, compressed, and snowflake automatically determines the most appropriate and best compression algorithm. Storing data in columnar format enables Snowflake to optimize queries even further when a subset of columns are accessed.


![[Pasted image 20260225003251.png]]

Benefits of Micro-partitioning:

- derived automatically; they don’t need to be explicitly defined up-front or maintained by users.

- Small ->  efficient DML and fine-grained pruning for faster queries. Pruning to pomijanie partycji które na pewno nie pasuja do zapytania dzieki patrzeniu na metadane

1. First, prune micro-partitions that are not needed for the query.
2. Then, prune by column within the remaining micro-partitions.

- Micro-partitions can overlap in their range of values, which, combined with their uniformly small size, helps prevent skew. Tu (w skew) chodzi o to, ze jak w tradycyjnej bazie dane sa popartycjonowane np. po kraju, i 1 kraj stanowi 90% danych to jedna partycja bd wielka i będzie pracowala caly czas, a inne sa male i pracuja rzadko. Zamiast tego w snowflake w mikropartycjach wartosci mogą się powtarzac bo dane sa pakowane do małych, równych micro-partitions które mogą być skanowane rownolegle

- Columns are stored independently within micro-partitions, often referred to as columnar storage. This enables efficient scanning of individual columns; only the columns referenced by a query are scanned.
- Columns are also compressed individually within micro-partitions. Snowflake automatically determines the most efficient compression algorithm for the columns in each micro-partition.

Mikropartycje przyspieszaja operacje DML. Jednoczesnie np. gdy dropujemy kolumne z tabeli, to mikropartycje które zawieraja dane tej kolumny pozostaja takie same - dane tej kolumny dalej pozostaja w storage.

Clustering Information Maintained for Micro-partitions

Snowflake maintains clustering metadata for the micro-partitions in a table, including:

- The total number of micro-partitions that comprise the table.

- The number of micro-partitions containing values that overlap with each other (in a specified subset of table columns).

- The depth of the overlapping micro-partitions - czyli clustering depth - o co w tym chodzi: Depth to miara tego, ile micro-partitions potencjalnie zawiera tę samą wartość (lub zakres wartości) danej kolumny. Depth = ile micro-partitions ma w swoim [MIN, MAX] tę daną. Jak żadne partycje na siebie nie nachodzą to depth = 1, a jak zaczynaja na siebie nachodzic to depth rosnie. Clustering depth can be used for a variety of purposes, including:

- Monitoring the clustering “health” of a large table, particularly over time as DML is performed on the table. (DML powoduje pogorszenie jakosci clusteringu tabeli)
- Determining whether a large table would benefit from explicitly defining a [clustering key](https://docs.snowflake.com/en/user-guide/tables-clustering-keys).

The clustering depth for a table is not an absolute or precise measure of whether the table is well-clustered. Ultimately, query performance is the best indicator of how well-clustered a table is.