[[STORAGE LAYER]]
A view allows the result of a query to be accessed as if it were a table.

Snowflake supports two types of views:
- Non-materialized views (usually simply referred to as “views”)
- Materialized views

**NON-MATERIALIZED VIEWS:**
A non-materialized view is basically a named definition of a query. A non-materialized view’s results are created by executing the query at the time that the view is referenced in a query. The results are not stored for future use. Performance is slower than with materialized views.

[[Materialized views]]
in many ways it behaves more like a table. A materialized view’s results are stored, almost as though the results were a table. This allows faster access, but requires storage space and active maintenance, both of which incur additional costs. Materialized views are designed to improve query performance for workloads composed of common, repeated query patterns. Materialized views support clustering, and you can create multiple materialized views on the same data, with each materialized view being clustered on a different column, so that different queries can each run on the view with the best clustering for that query. 

**KIEDY NAJLEPIEJ UZYC MATERIALIZED VIEWS**
- Query results contain a small number of rows and/or columns relative to the base table (jak tabela jest wielka)- jesli nie zrobimy materialized view to za kazdym odpaleniem widoku musialaby byc skanowana ogromna tabela i spalac kredyty

- The view’s base table does not change frequently - To jest najważniejszy punkt pod kątem kosztów. Za każdym razem, gdy w tabeli bazowej zmieni się choć jeden wiersz, Snowflake musi "zaktualizować" widok zmaterializowany w tle. Zajmuje się tym proces _Background Maintenance Service_. Za to automatyczne odswiezanie placi sie w jednostkach compute. 

- Analiza danych półstrukturalnych (JSON/Parquet): Funkcje takie jak `FLATTEN` czy wyciąganie wartości z kolumn typu `VARIANT` są procesorożerne. MV pozwala "wyciągnąć" te dane do płaskiej tabeli raz, a potem czytać je błyskawicznie.
    
- Agregaty (SUM, AVG): Jeśli obliczenie średniej z lat zajmuje 10 minut, MV policzy to raz i poda gotowy wynik w milisekundach.

- External Tables (Tabele zewnętrzne): Dane leżące na S3 lub Azure Blob są zawsze wolniejsze, bo Snowflake musi je pobrać przez sieć. MV na takiej tabeli tworzy "lokalną kopię" wyników wewnątrz Snowflake, co drastycznie przyspiesza pracę.

ogólna zasada kiedy tworzyć materialized views - wtedy gdy jakikolwiek z tych warunkow jest spelniony: **Dane są rzadko zmieniane + zapytania są bardzo częste + obliczenia są bardzo ciężkie.**

**POROWNANIE MATERIALIZED VIEWS I INNYCH FORM  PRZECHOWYWANIA WYNIKOW QUERY:**

![[Zrzut ekranu 2026-03-7 o 08.45.44.png]]


Dlaczego materialized views są szbsze niz zwykle tabele: kiedy odpytujesz zwykłą tabelę, Snowflake musi wykonać całą "brudną robotę": przefiltrować wiersze, połączyć tabele (JOIN) i wykonać obliczenia (np. `SUM`, `AVG`), a Materialized View to w praktyce **wstępnie obliczony wynik**. Gdy tabela bazowa się zmienia, a widok zmaterializowany nie zdążył się jeszcze w pełni odświeżyć w tle, Snowflake bierze gotowe wyniki z pamięci cache dla 99% danych, które się nie zmieniły i są już zapisane w MV i doczytuje "w locie" tylko te nowe 1% danych (zmiany/deltę), które pojawiły się w tabeli bazowej od ostatniego odświeżenia widoku. Pamiętaj, że ten "cache" (zapisany wynik) kosztuje Cię dodatkowe pieniądze za **Storage** (bo to fizyczna kopia danych) - czyli materialized views kosztują i za storage i za compute jak sie odswiezaja. Odswiezanie (background maintenance) mozna zawiesic (suspend)

How the Query Optimizer Uses Materialized Views:  You don’t need to specify a materialized view in a SQL statement in order for the view to be used. The query optimizer can automatically rewrite queries against the base table or regular views to use the materialized view instead.

**There are three types of privileges that are related to materialized views:**
- **Privileges on the schema that contains the materialized view** - To create a materialized view, you need the CREATE MATERIALIZED VIEW privilege on the schema that will contain the materialized view. 
- **Privileges directly on the materialized view itself** - You can grant SELECT privilege on a materialized view. A materialized view (normalny view tez) does not automatically inherit the privileges of its base table. You should explicitly grant privileges on the materialized view to the roles that should use that view. - wyjatkiem jest sytuacja gdy to query optimizer uzywa MV zamiast zwyklej tabeli - wtedy nie trzeba uprawnien
- Privileges on the database objects (e.g. tables) that the materialized view accesses - nie trzeba mieć uprawnien do obiektu z ktorego powstaje view zeby moc zobaczyc sam view

**Materialized views limitations:**
- brak operacji DML na nich (nie mozna np wstawic wiersza prosto do MV ani robic zadnych COPY, DELETE, INSERT, MERGE, UPDATE)
- user nie moze zrobic truncate
- no using the Time Travel feature to query materialized views at a point in the past. (However, you can use Time Travel to clone a database or schema containing a materialized view at a point in the past. )
- A materialized view can query only a single table.
- Joins, including self-joins, are not supported. - nie mozna zrobiv materialized view na joinie 2 tabel
- A materialized view cannot query:
    - A materialized view.
    - A non-materialized view.
    - A hybrid table.
    - A dynamic table.
    - A UDTF (user-defined table function).

A materialized view cannot include:
- UDFs     
- Window functions.
- HAVING clauses.
- ORDER BY clause.
- LIMIT clause.
- subqueries

modyfikacje tabeli bazowej a MV:
*  jak do tabeli bazowej MV dodamy nowa kolumne - Nowa kolumna nie pojawi się automatycznie w Materialized View. Dlaczego? Bo Widok zmaterializowany (MV) jest tworzony na podstawie konkretnego zapytania `SELECT`. Jeśli w definicji MV miałaś np. `SELECT col1, col2 FROM table`, to dodanie `col3` do tabeli bazowej w żaden sposób nie zmienia definicji widoku. MV nadal będzie przechowywał i wyświetlał tylko `col1` i `col2`. Nawet jeśli MV zostanie stworzony jako `SELECT *` to nowa kolumna i tak sie NIE POJAWI, bo `SELECT *` jest "rozwijany" do listy kolumn w momencie tworzenia widoku.

* Changing or Dropping Columns in the Base Table - If a base table is altered so that existing columns are changed or dropped, then all materialized views on that base table are suspended; the materialized views cannot be used or maintained. (This is true even if the modified or dropped column was not part of the materialized view.) You cannot RESUME that materialized view. If you want to use it again, you must recreate it.

* Renaming or Swapping the Base Table - materialized view is suspended

* Dropping the Base Table - the materialized view is suspended (but not automatically dropped). In most cases, the materialized view must be dropped.


MATERIALIZED VIEWS VS DYNAMIC TABLES
![[Zrzut ekranu 2026-03-12 o 21.33.06.png]]


[[Secure views]]
Both non-materialized and materialized views can be defined as _secure_. Secure views have advantages over standard views, including improved data privacy and data sharing;

"Po co używać Secure Views?". Odpowiedź ma dwa filary:

- **Ukrycie definicji (kodu SQL):** W standardowym widoku każdy, kto ma rolę z dostępem, może zobaczyć kod źródłowy (przez `GET_DDL` lub `SHOW VIEWS` lub INFORMATION_SCHEMA.VIEWS). W Secure View definicja w GET_DDL() jest widoczna **tylko dla właściciela** (roli z uprawnieniem OWNERSHIP) lub ról nadrzędnych. **Query Profile:** W interfejsie Snowflake (Snowsight/Classic UI), detale wykonania Secure View (np. konkretne operacje na tabelach pod spodem) są **ukryte**, nawet dla właściciela widoku.
    
- **Ochrona przed wyciekiem danych (Side-channel attacks):** Snowflake optymalizuje zapytania, co czasem może ujawnić dane, których użytkownik nie powinien widzieć (np. poprzez błędy typu „division by zero” w wierszach, które teoretycznie powinny być odfiltrowane). Secure Views wymuszają inną kolejność optymalizacji, by do tego nie dopuścić.

**W takim razie skoro nie każdy może zobaczyć DDL secure widoku, to kto moze?** 
 GET_DDL() jest widoczna **tylko dla właściciela** (roli z uprawnieniem OWNERSHIP) lub ról nadrzędnych
Użytkownicy z rolą `ACCOUNTADMIN` lub uprawnieniem `IMPORTED PRIVILEGES` na bazie danych `SNOWFLAKE` lub uzytkownicy z rola SNOWFLAKE.OBJECT_VIEWER mogą zobaczyć definicję w widokach `ACCOUNT_USAGE`.

**Kiedy uzywac secure widokow:** 
- gdy celem powstania widoku jest ograniczenie uzytkownikom widocznosci jakichs danych z tabeli zrodlowej (jak nie bd secure to i tak moga sobie zobaczyc w ddl)

Secure widoki są wolniejsze niz zwykle widoki - tradeoff szybkosc vs bezpieczenstwo - Ponieważ Snowflake celowo rezygnuje z niektórych optymalizacji

Mozna przeksztalcic istniejacy widok w secure komenda ALTER VIEW mój_widok SET SECURE i w druga strone - UNSET

Jeśli udostępniasz dane (Data Sharing) innym kontom, **zawsze** powinieneś używać Secure Views, aby odbiorca nie widział struktury Twoich baz danych ani kodu SQL. **Funkcje kontekstowe:** W Secure Views często używa się `CURRENT_ROLE()` lub `CURRENT_USER()`, aby filtrować wiersze dynamicznie (np. użytkownik widzi tylko wiersze ze swojego regionu).


**Zalety widokow (po co one są):**
- Views help you to write clearer, more modular SQL code.
- Pozwalają ograniczyć dostęp do konkretnych kolumn lub wierszy bez dawania uprawnień do całej tabeli. Views allow this because you can grant privileges on a particular view to a particular role, without the grantee role having privileges on the table(s) underlying the view.
			Note
			Zwróć uwagę na uprawnienia. Aby użytkownik mógł odpytać widok, musi mieć uprawnienie `SELECT` do widoku oraz `USAGE` do bazy i schematu. **Ciekawostka:** Jeśli właściciel widoku ma uprawnienia do tabel źródłowych, to użytkownik końcowy odpytujący widok **nie potrzebuje** bezpośrednich uprawnień do tych tabel


**DDL NA WIDOKACH:** To częsta pułapka na egzaminie. **Nie można** zmienić samej definicji (kodu SQL) widoku za pomocą `ALTER VIEW`. Aby zmienić logikę widoku, musisz go stworzyć na nowo, używając: `CREATE OR REPLACE VIEW. Polecenie ALTER VIEW. służy głównie do zmiany metadanych (np. zmiany nazwy widoku, dodania komentarza lub ustawienia go jako "Secure").

