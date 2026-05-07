
[[STORAGE LAYER]]
designating one or more table columns/expressions as a clustering key for the table (recommended maximum of 3 or 4 columns per key). A table with a clustering key defined is considered to be clustered. You can cluster [materialized views](https://docs.snowflake.com/en/user-guide/views-materialized), as well as tables. Cant cluster hybrid tables. after a clustered table is defined, reclustering does not necessarily start immediately. Snowflake only reclusters a clustered table if it will benefit from the operation.
Clustering keys are not intended for all tables due to the costs of initially clustering the data and maintaining the clustering. Clustering is optimal when either:

- You require the fastest possible response times, regardless of cost.
- Your improved query performance offsets the credits required to cluster and maintain the table. The more frequently a table is queried, the more benefit clustering provides. However, the more frequently a table changes, the more expensive it will be to keep it clustered. Therefore, clustering is generally most cost-effective for tables that are queried frequently and do not change frequently.

Typically, queries benefit from clustering when the queries filter or sort on the clustering key for the table. Defining a clustering key will generally benefit queries that require filtering or sorting on the clustering keys during the query execution. ORDER BY, GROUP BY & certain joins require sorting during query execution. Queries that use the WHERE clause on the clustering keys will also benefit from an adequately defined clustering key.

Wskazniki ze clustering key może pomoc:

- Queries on the table are running slower than expected or have noticeably degraded over time.
- The [clustering depth](https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions.html#label-clustering-depth) for the table is large. Clustering Depth to liczba, która reprezentuje średnią liczbę mikro-partycji, do których Snowflake musi zajrzeć, aby znaleźć konkretną wartość. Im wieksza tym gorzej i trzeba clustering/reclustering robić. Minimalna wartosc depth to 1. 
![[Zrzut ekranu 2026-05-4 o 21.31.31.png]]
![[Zrzut ekranu 2026-05-4 o 21.33.39.png]]


Na jakich tabelach clustering się sprawdzi:

- Duzo mikropartycji (terabajty danych w tabeli)
- Queries czytaja tylko maly % micropartitions tabeli I/LUB dane sa sortowane (order by albo inner join)
- Wiekszosc queries na danej tabeli robi selecta i sort na tych samych kolumnach (wiec duzo queries skorzysta z clustering keys). Dobra praktyka jest robienie cluster keys na kolumnach które najwiecej sa uzywane w filtrowaniu  (jeśli często filtrujemy po wartosciach z 2 tych samych kolumn to najlepiej robic cluster key na obu), albo na kolumnach uzywanych do joinow.
- Clustering key dobrze robic na kolumnach które maja nie za duzo (np. nanosecond timestamp - żeby się dalo sensownie pogrupowac dane w mikropartycje) i nie za malo (np. boolean - żeby dalo się robic pruning i odrzucac jakies sensowne czesci danych) distinct wartosci. Im wiecej distinct wartosci (wyzsze cardinality) to utrzymywanie clustering key jest drozsze. Na kolumnach z bardzo duza cardinality polecane jest utworzenie expression (żeby ogranicze liczbe distinct values) i dopiero z tego zrobienie clustering key. Expression to np. zamiana timestampa na date, dokladnej liczby na mniej dokladna itp.
- The order in which the data is loaded doesn’t match the dimension by which it is most commonly queried (for example, the data is loaded by date, but reports filter the data by ID). If your existing scripts or reports query the data by both date and ID (and potentially a third or fourth column), you might see some performance improvement by creating a multi-column clustering key.
- [Query Profile](https://docs.snowflake.com/en/user-guide/ui-snowsight-activity) indicates that a significant percentage of the total duration time for typical queries against the table is spent scanning. This applies to queries that filter on one or more specific columns.

Jak robimy multi-column clustering key, to kolejnosc kolumn podanych w CLUSTER BY ma znaczenie -

Snowflake recommends ordering the columns from lowest cardinality to highest cardinality

When clustering on a text field, the cluster key metadata tracks only the first several bytes (typically 5 or 6 bytes). 

**RECLUSTERING**
Reclustering - As DML operations (INSERT, UPDATE, DELETE, MERGE, COPY) are performed on a clustered table, the data in the table might become less clustered. Periodic/regular reclustering of the table is required to maintain optimal clustering.  During reclustering, Snowflake uses the clustering key for a clustered table to reorganize the column data, so that related records are relocated to the same micro-partition.  Reclustering in Snowflake is automatic; no maintenance is needed.

Reclustering kosztuje credits i powoduje zajecie storage (i koszty jego). Dlaczego - bo to operacja DML, wiec kosztuje kredyty. A storage zuzywa bo podczas przegrupowywania danych tworzone sa nowe mikropartycje (bo one sa immutable). Note that reclustering rewrites existing data with a different order. 

 Original micro-partitions (te sprzed reclusteringu) are marked as deleted, but retained for 7 days in the system to enable Time Travel and Fail-safe. The original micro-partitions are purged only after both the Time Travel retention period and the subsequent Fail-safe period have passed (min. 8 days and up to 97 days for extended Time Travl).

constant state micropartition oznacza ze nie może być już poprawione przez clustering.  In a well-clustered large table, most micro-partitions will fall into this category.

An existing clustering key is copied when a table is created using CREATE TABLE … CLONE. However, Automatic Clustering is [suspended for the cloned table](https://docs.snowflake.com/en/user-guide/object-clone.html#label-cloning-and-clustering-keys) and must be resumed.It is not copied when table is created using CREATE TABLE … AS SELECT; cluster key może być dropniety za pomoca komendy

AUTOMATIC CLUSTERING

Snowflake monitors and evaluates the tables to determine whether they would benefit from reclustering, and automatically reclusters them, as needed. You can suspend and resume Automatic Clustering for a clustered table at any time using ALTER TABLE … SUSPEND / RESUME RECLUSTER.![[Zrzut ekranu 2026-05-4 o 21.10.35.png]]



## NIE POMYL CLUSTERINGU TABELI Z CLUSTERING KEYS!
To rozróżnienie jest jednym z najbardziej podchwytliwych elementów w całym Snowflake. Twoje zmieszanie wynika z tego, że w Snowflake istnieją dwa "poziomy" klastrowania, a ich nazwy są do siebie bardzo podobne.

Oto proste wyjaśnienie, które pozwoli Ci to poukładać:

### 1. Natural Clustering (Klastrowanie Naturalne) – Automat "domyślny"

Masz rację, że zazwyczaj nie musisz nic włączać. Snowflake **zawsze** klastruje dane podczas ich ładowania. Dzieje się to automatycznie, ponieważ dane lądują w mikro-partycjach w takiej kolejności, w jakiej zostają wstawione do tabeli.

- **Przykład:** Jeśli codziennie ładujesz logi z danego dnia, to mikro-partycje będą "naturalnie" pogrupowane według daty. To jest to **automatyczne klastrowanie**, o którym mówiła poprawna odpowiedź. Snowflake po prostu nie wrzuca danych do jednego worka, tylko układa je w mikro-partycje na bieżąco.
    

---

### 2. Clustering Keys (Klucze Klastrowania) – Automat "na żądanie"

To jest to, o czym myślałaś – specjalna funkcja dla bardzo dużych tabel (zazwyczaj powyżej kilku terabajtów), gdzie "naturalny" porządek nie wystarcza.

- Kiedy definiujesz **Clustering Key**, mówisz Snowflake'owi: "Ten domyślny porządek ładowania mi nie pasuje, chcę, żebyś fizycznie poukładał dane według kolumny `CITY`, a nie według czasu ładowania".
    
- Po zdefiniowaniu klucza, Snowflake uruchamia usługę **Automatic Reclustering**. To jest proces w tle (Serverless), który pilnuje, aby dane były zawsze dobrze "poukładały" według Twojego klucza, nawet gdy dopisujesz nowe rekordy.