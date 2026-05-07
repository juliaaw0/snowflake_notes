[[STORAGE LAYER]]

Możesz o Search Optimization Service (SOS) myśleć jak o „inteligentnym indeksie” przeznaczonym dla standardowych tabel (nie aplikuje się do hybrydowych bo one maja swój wlasny koncept indeksow). SOS to usługa działająca w tle (background service), która tworzy specjalną strukturę danych (rodzaj indeksu - nazywa się to search access path) dla tabeli. Ta struktura search access path to jest persistent structure. The search access path keeps track of which values of the table’s columns might be found in each of its [[Micropartitions]], allowing some micro-partitions to be skipped when scanning the table. Nie tworzy się tych "indeksow" recznie tylko są same tworzone przez snowflake po wlaczeniu SOS.

Kiedy użyć SOS? (Pewniak egzaminacyjny)

- Tabela ma setki gigabajtów lub terabajty danych.
- Zapytanie nie używa klucza klastrowania (Clustering Key).
- Zapytanie zwraca bardzo małą liczbę rekordów.
- Standardowy Pruning (odrzucanie mikro-partycji) nie działa wystarczająco dobrze.

na jakich query NAJLEPIEJ zadziala SOS:
- The query involves a column or columns other than the primary cluster key.
- The query typically runs for a few seconds or longer
- At least one of the columns accessed by the query filter operation has 100,000 distinct values
- kolumny uzyte w query nie moga byc type FLOAT

na jakich tabelach zadziala SOS:
- Standard Snowflake tables
* Interactive tables
* Iceberg tables
* Dynamic tables
* Transient tables

na jakich NIE:
* External tables
* Hybrid tables
* Temporary tables
* materialized views

Serverless: SOS to usługa bezserwerowa. Snowflake pobiera kredyty za zarządzanie i aktualizowanie usługi wyszukiwania. Nie wymaga wlaczania warehousu. 

Storage: Indeks stworzony przez SOS zajmuje dodatkowe miejsce, więc wzrosną koszty Storage.

DML: Każdy INSERT, UPDATE lub DELETE na tabeli z włączonym SOS powoduje, że usługa musi się zaktualizować, co kosztuje dodatkowe kredyty.

To add, configure, or remove search optimization for a table, you must:
- Have OWNERSHIP privilege on the table.
- Have ADD SEARCH OPTIMIZATION privilege on the schema that contains the table.

mozna sprawdzic w query profile czy query uzylo SOS
![[Zrzut ekranu 2026-05-4 o 22.33.29.png]]
