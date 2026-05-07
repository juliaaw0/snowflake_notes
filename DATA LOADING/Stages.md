[[DATA LOADING]]
### External stages
A named external stage is a database object created in a schema. This object stores the URL to files in cloud storage, the settings used to access the cloud storage account, and convenience settings such as the options that describe the format of staged files. Create stages using the [CREATE STAGE](https://docs.snowflake.com/en/sql-reference/sql/create-stage) command.

external stage nie przechowuje niczego fizycznie tylko pointuje do lokalizacji w cloudzie

![[Zrzut ekranu 2026-04-12 o 16.12.09.png]]
### Internal stages
one przechowują fizycznie pliki. To co w nich sie znajduje liczy sie do ogolnego storage'u w snowflake za ktory sie placi. Dlatego zacheca sie zeby po zaladowaniu plikow ze stage do tabeli wyczyscic stage. 

Do internal stage'a wklada sie pliki komendą PUT
z internal stage'a mozna potem przeniesc dane do tabeli za pomoca COPY INTO

DANE SA ENCRYPTED PRZED PRZYJSCIEM NA INTERNAL STAGE I NA NIM!
Data is encrypted automatically at the client machine before being transmitted to the Snowflake internal stage. Once the data is in an internal stage, it is stored encrypted. This is part of the end-to-end encryption managed by Snowflake.

### Komenda PUT – Klucz do pytań pułapek

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Bardzo wysoka)

To najczęstszy punkt zapalny na egzaminie. Musisz zapamiętać te zasady:
- **Gdzie działa?** Polecenie `PUT` **NIE MOŻE** być wykonane w interfejsie webowym (Snowsight). Musi być wykonane przez **SnowSQL** lub sterowniki (Python, JDBC, .NET).
    
- **Automatyczna kompresja:** `PUT` domyślnie kompresuje pliki algorytmem **GZIP** przed ich wysłaniem (chyba że ustawisz `AUTO_COMPRESS = FALSE`).
    
- **Struktura:** `PUT file:///sciezka/do/pliku.csv @nazwa_stage;`
    
- **Parametr OVERWRITE:** Jeśli plik o tej samej nazwie już istnieje w stage, `PUT` go nie nadpisze, chyba że dodasz `OVERWRITE = TRUE`.

* Put nie uzywa warehousu snowflakowego tylko zasobow komputera do załadowania plikow do stage


### Kopiowanie danych (COPY INTO )

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ - koniecznie zapamietac

Gdy pliki są już w stage (po komendzie `PUT`), używasz `COPY INTO`. Oto co musisz wiedzieć na egzamin:

- **Wymaga Warehouse:** W przeciwieństwie do `PUT` (który używa zasobów Twojego komputera do wysyłki), `COPY INTO` wymaga uruchomionego Virtual Warehouse.
    
- **Transformacja w locie:** Podczas kopiowania możesz zrobić SELECT i transformowac kolumny ktore ladujesz:
    
    - Zmienić kolejność kolumn (np. `$2, $1`).
        
    - Pominąć kolumny.
        
    - Użyć funkcji `CAST` (zmiana typu danych).

	* **Proste funkcje:** `SUBSTR`, `CONCAT`, `CASE`.
	
    - **Uwaga:** W `COPY INTO` nie można używać funkcji agregujących (np. `SUM`, `COUNT`) ani `JOIN`, ani window functions
        
- **Selekcja plików:** Możesz ładować:
    
    - Wszystko: `FROM @my_stage`
        
    - Konkretne pliki: `FILES = ('file1.csv', 'file2.csv')`
        
    - Według wzorca: `PATTERN = '.*sales.*\.csv'`
    - 
There is no requirement for your data files to have the same number and ordering of columns as your target table.
 Snowflake can transform data after a partner software has loaded it.



Przykładowe podchwytliwe pytanie z tym:
_"Użytkownik chce załadować plik 1GB przez Snowsight (Web UI) używając komendy PUT. Dlaczego to się nie uda?"_, prawidłowe odpowiedzi są dwie: 1. `PUT` nie działa w UI, 2. Przez UI ładujemy tylko do 50MB bez komendy PUT (używając kreatora).

### TYPY INTERNAL STAGE

Snowflake maintains the following internal stage types in your account:

**User**
A user stage is allocated to each user for storing files. This stage type is designed to store files that are staged and managed by a single user but can be loaded into multiple tables. User stages cannot be altered or dropped.
Nie mozna podac formatu ladowanego pliku - musi to byc podane w COPY INTO
odwolujemy sie do nich przez @~

**Table**
A table stage is available for each table created in Snowflake. This stage type is designed to store files that are staged and managed by one or more users but only loaded into a single table. Table stages cannot be altered or dropped.
Note that a table stage is not a separate database object; rather, it is an implicit stage tied to the table itself. A table stage has no grantable privileges of its own. To stage files to a table stage, list the files, query them on the stage, or drop them, y**ou must be the table owner (have the role with the OWNERSHIP privilege on the table**).
Nie mozna w table stage podac od razu formatu ladowanego pliku - musi to byc podane w COPY INTO.
Odwolujemy sie do nich za pomoca ich nazwy: @%tablename

**Named (internal)**

A named internal stage is a database object created in a schema. This stage type can store files that are staged and managed by one or more users and loaded into one or more tables. Because named stages are database objects, the ability to create, modify, use, or drop them can be controlled using security access control privileges. Create stages using the [CREATE STAGE](https://docs.snowflake.com/en/sql-reference/sql/create-stage) command.
Upload files to any of the internal stage types from your local file system using the [PUT](https://docs.snowflake.com/en/sql-reference/sql/put) command. GET to pobieranie plikow internal stage'a na nasz komputer

### uwaga!

The table stages do not allow basic transformations during the COPY process; thus, basic transformations may only be performed while loading data from external stages, named internal stages or user stages.




**Kluczowe parametry przy tworzeniu internal stage'a:**

- `ENCRYPTION`: Wszystkie dane w Internal Stages są **automatycznie szyfrowane** przez Snowflake. Możesz zarządzać kluczami (Master Key), ale nie możesz wyłączyć szyfrowania.
    
- `FILE_FORMAT`: Możesz przypisać domyślny format (np. CSV z konkretnym separatorem) bezpośrednio do stage'a, żeby nie musieć go definiować za każdym razem w `COPY INTO`.



## QUERYING DATA IN STAGES

Snowflake pozwala na użycie SQL-a bezpośrednio na plikach leżących w Stage (Internal lub External) bez ich ładowania (`COPY INTO`).

- **Notacja dolarowa (`$`):** Zamiast nazw kolumn używasz `$1, $2, $3...` odnosząc się do pozycji kolumny w pliku (np. CSV).
    
- **Przykład:** `SELECT $1, $2 FROM @my_stage/data.csv;`
    
- **Zastosowanie:** * Szybki podgląd danych przed załadowaniem. Sprawdzenie, czy format pliku (`FILE FORMAT`) jest poprawnie skonfigurowany.
        
- **Wymagania:** Do wykonania zapytania potrzebny jest **Virtual Warehouse** (to nie jest operacja darmowa/metadata-only).

QUEROWANIE METADANYCH:

Podczas odpytywania stage'a lub ładowania danych, Snowflake udostępnia dwie wirtualne kolumny metadanych, które są **niezwykle pomocne przy audycie**:

1. **`METADATA$FILENAME`**: Nazwa pliku, z którego pochodzi dany rekord (wraz ze ścieżką w stage).
    
2. **`METADATA$FILE_ROW_NUMBER`**: Numer wiersza wewnątrz tego konkretnego pliku.

> **Tip pod egzamin:** Często pojawia się pytanie, jak sprawdzić, z którego pliku pochodzi błędny rekord w tabeli. Odpowiedź to zapisanie `METADATA$FILENAME` do dodatkowej kolumny w tabeli podczas wykonywania `COPY INTO`.


SCHEMA EVOLUTION:
To funkcja, która pozwala tabelom "rosnąć" automatycznie, gdy w plikach źródłowych pojawią się nowe kolumny. Jeśli w pliku Parquet pojawi się nowa kolumna, której nie ma w tabeli, Snowflake automatycznie zrobi `ALTER TABLE ... ADD COLUMN`.

- **Wymóg:** Tabela musi mieć włączony parametr: `ENABLE_SCHEMA_EVOLUTION = TRUE`.
    
- **Wspierane formaty:** Działa głównie dla plików półstrukturalnych, które niosą ze sobą schemat: **Parquet, Avro, ORC**. (Dla CSV/JSON jest to trudniejsze i wymaga dodatkowej konfiguracji).
    
- **Uprawnienia:** Rola wykonująca `COPY INTO` musi mieć uprawnienie `EVOLVE SCHEMA` na tabeli.




Przykladowe pytanie na egz:
_Czy możesz użyć funkcji `SUM($3)` podczas ładowania danych poleceniem `COPY INTO`?_ **Odpowiedź:** Nie. Transformacje w `COPY INTO` nie wspierają funkcji agregujących. Musisz najpierw załadować dane, a potem użyć `INSERT INTO ... SELECT SUM(...)`.

