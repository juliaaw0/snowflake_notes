[[TABLES]]

An external table is a Snowflake feature that you can use to query data stored in an [external stage](https://docs.snowflake.com/en/user-guide/data-load-overview.html#label-data-load-overview-external-stages) as if the data were inside a table in Snowflake

Wyjasnienie po ludzku czym jest ta external table i "pliki":

Tabela to cały folder (Stage), reprezentuje cały zbiór plików znajdujących się w danej lokalizacji. Plik to "paczka" wierszy: Każdy plik (np. .csv, .json, .parquet) znajdujący się w tym folderze jest traktowany przez Snowflake jako źródło rekordów. 

Organizations with established data lakes and significant amounts of data in cloud object storage will find external tables helpful. When data is accessed infrequently, or only a portion of the data has to be queried, external tables can expose data from data lakes in a cost-effective manner. However, storing the data in a typical Snowflake table may be more economical if all of the data is viewed or if access is made often.

An external table is a metadata definition; that is, you register the definition of an external table, but the external table itself doesn't contain any data. Instead, the table metadata contains column definition, the name of the external stage from where the data for the external table is, and the file format which should be used to read that data. The external stage, in turn, points to object storage on the cloud, for example, an AWS bucket or Azure Blob storage, which contains the data for the external table. Note that an external table can only point to an external stage. An internal stage cannot be used to create an external table.

If Snowflake encounters an error while scanning a file in cloud storage during a query operation, the file is skipped and scanning continues on the next file. A query can partially scan a file and return the rows scanned before the error was encountered. To różni tabele zewnętrzne od COPY INTO. W tabelach zewnętrznych Snowflake priorytetyzuje dostępność danych nad ich idealną spójnością, bo nie ma pełnej kontroli nad tym, co ktoś "dorzucił" do folderu na S3. For optimal performance when querying large data files, create and query [materialized views over external tables](https://docs.snowflake.com/en/user-guide/tables-external-intro#label-materialized-views-over-external-tables).

External tables let you store (within Snowflake) certain file-level metadata, including filenames, version identifiers, and related properties. External tables can access data stored in any format that the COPY INTO command supports, except XML.

External tables are read-only. **You can’t perform data manipulation language (DML) operations on external tables. However, you can use external tables for query and join operations. You can also create views against external tables.**

Żeby stworzyc external tabele nie trzeba znac wewnetrznej struktury plikow w chmurze. Caly 1 wiersz z pliku może isc do pojedynczej kolumny o nazwie value. Oto podstawowa wersja external table



PARTYCJONOWANIE EXTERNAL TABLES:

W tradycyjnej tabeli Snowflake sam dzieli dane na mikro-partycje. W tabeli zewnętrznej Snowflake nie ma takiej władzy – dane leżą na S3/Azure, więc musisz mu "podpowiedzieć", jak ma filtrować pliki. Wyobraź sobie, że masz na S3 tysiące plików z logami, ułożone w folderach:s3://my-bucket/logs/2024/02/10/s3://my-bucket/logs/2024/02/11/s3://my-bucket/logs/2024/02/12/. Jeśli zapytasz o dane z 12 lutego, a nie masz partycjonowania, Snowflake musi otworzyć i przeczytać każdy plik z każdego folderu (latami wstecz!), żeby sprawdzić, co jest w środku. To się nazywa "Full Table Scan" i jest bardzo drogie. Partycjonowanie polega na tym, że uczysz Snowflake'a rozumieć strukturę Twoich folderów. Mówisz mu: "Pierwszy podfolder to ROK, drugi to MIESIĄC, trzeci to DZIEŃ" (to jest przykład automatycznego partycjonowania)

Są 2 sposoby partycjonowania external tables:

1. Automatyczne - podajemy pattern jak wygląda ścieżka do plikow, np. sales/2024/03/data.csv - podajemy ze to jest partycjonowaie z wzorcem dzial/rok/miesiac/. 

	Ale: W tym typie partycjonowania Snowflake wysyła prośbę do AWS (przez API): "Hej, wypisz mi wszystkie ścieżki plików w folderze /logs/". Mielenie nazw: AWS zwraca listę 100 000 ścieżek. Snowflake musi każdą z nich "przemielić" przez Twój kod (np. split_part), żeby wyliczyć, do której partycji dany plik należy. Zasoby: To skanowanie listy plików zajmuje czas (latency) i zużywa zasoby Twojego Warehouse'u (kredyty). Jeśli masz miliony plików, samo "sporządzenie listy" może trwać dłużej niż samo czytanie danych!

2. Mechaniczne/ręczne/manualne - musimy sami podac kazda partycje (kazda jej wersje, np. podac rok = 2023, rok = 2024, rok = 2025 itp.)

W Manual Partitioning Snowflake w ogóle nie pyta AWS-a o listę plików w całym bucketcie. On zagląda do swojej własnej, wewnętrznej bazy metadanych (która jest super szybka).

- Zaufanie: Snowflake mówi: "Użytkownik powiedział mi kiedyś komendą ADD PARTITION, że folder /2024/02/12/istnieje. Wierzę mu na słowo".
- Precyzyjne uderzenie: Gdy robisz SELECT ... WHERE p_date = '2024-02-12', Snowflake idzie bezpośrednio do tego jednego folderu. Nie prosi o listę wszystkiego, co jest w /logs/. Przeskakuje etap "śledztwa" i od razu przechodzi do czytania.

RÓŻNICE PARTYCJONOWANIA (AUTO VS MANU) NA SnowPro Core:

- Automatic: Wygodny, bo Snowflake sam "odkrywa" nowe pliki (jeśli masz włączony REFRESH lub AUTO_REFRESH), ale przy ogromnej skali (miliony plików/folderów) samo "odkrywanie" staje się wąskim gardłem.
- Manual: Usuwa problem "skanowania listy plików" (API overhead). Jest to rozwiązanie zalecane dla systemów typu Data Lake, gdzie danych jest tak dużo, że automatyczne odświeżanie metadanych trwałoby wieczność.

Ważny detal: Przy AUTO_REFRESH = TRUE w trybie automatycznym, Snowflake korzysta z komunikatów (SNS/SQS). To trochę ratuje sytuację, bo chmura sama "mówi" Snowflake'owi o nowym pliku, więc on nie musi co chwila skanować całego folderu. Ale wciąż musi wyliczyć partycję na podstawie nazwy tego konkretnego pliku.

Do zapamietania na egzamin:

1. Storage Integration: To obiekt (Object), który pozwala uniknąć wpisywania kluczy (Access Keys) w kodzie SQL. To jest hit egzaminacyjny. **Storage Integration:** To obiekt w Snowflake, który przechowuje "bezpieczne połączenie" z chmurą. Pozwala uniknąć podawania kluczy (Access Keys) w kodzie SQL. 

2. Formaty plików: Wiedzieć, że External Tables najlepiej działają z formatami kolumnowymi jak Parquet, ORC czy właśnie Delta Lake.
3. Billing: Pamiętaj, że za dane w External Table płacisz dostawcy chmury (np. Microsoftowi), a Snowflake'owi płacisz tylko za Compute (Warehouse), gdy odpytujesz te dane + minimalną opłatę za zarządzanie metadanymi - przy automatycznym odswiezaniu metadanych z external table z uzyciem event notification service placi się troche wiecej, przy manualnym odswiezaniu - mniej.
4. W przypadku Delta Lake partycjonowanie jest specyficzne i o to mogą zapytaćAutomatyczne wykrywanie:.  Snowflake potrafi automatycznie wykryć partycje z logów Delta Lake. Nie musisz ich ręcznie definiować za pomocą split_part, jeśli używasz TABLE_FORMAT = DELTA.

Creating and managing external tables requires a role with a minimum of the following role permissions:

|   |   |
|---|---|
|Object|Privilege|
|Database|USAGE|
|Schema|USAGE, CREATE STAGE (if creating a new stage), CREATE EXTERNAL TABLE|
|Stage (if using an existing stage)|USAGE|

If ownership of an external table (that is, the OWNERSHIP privilege on the external table) is transferred to a different role, the AUTO_REFRESH parameter for the external table is set to FALSE by default.