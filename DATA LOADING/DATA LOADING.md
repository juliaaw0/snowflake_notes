![[Zrzut ekranu 2026-04-12 o 16.57.52.png]]![[Zrzut ekranu 2026-04-12 o 17.01.51.png]]![[Zrzut ekranu 2026-04-12 o 17.02.05.png]] You can use a SELECT query to unload data from a table

The GET command is used to download data from an internal stage to an on-premises system. The PUT command uploads data from an on-premises system to an internal stage. To download or upload data to an external stage, cloud provider utilities or other tools are used to interact with data in the cloud storage pointed to by the external stage.
 zapamietaj 3 litery: PUT/GET to ladowanie/wyladowywanie internal stage'a




### 1. Preparing Your Data Files

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Kluczowe)

- **Rozmiar plików:**
    
    - **Zalecenie:** Snowflake zaleca pliki o rozmiarze **100–250 MB (po skompresowaniu)**.
    - **Dlaczego?** Pozwala to na optymalne wykorzystanie równoległości (parallelism) serwerów wewnątrz Warehouse. Zbyt małe pliki generują narzut (overhead), zbyt duże (np. >100 GB) uniemożliwiają efektywne rozłożenie pracy.

	- Przez Web UI (Snowsight) możesz załadować pliki o rozmiarze maksymalnie **50 MB**. (To bardzo popularne pytanie egzaminacyjne!). Jeśli plik jest większy, musisz użyć `PUT` i `COPY INTO` przez SnowSQL lub inne narzędzie.
	    
- **Automatyzacja:** UI pod spodem tworzy tymczasowy internal stage, ładuje tam plik i uruchamia `COPY INTO`. Po zakończeniu plik jest usuwany.
    
- **Kiedy używać?** Tylko do małych plików i doraźnych zadań (ad-hoc). Nie nadaje się do produkcji ani dużych wolumenów danych.
        
- **Kompresja:**
    
    - Snowflake **automatycznie wykrywa** większość typów kompresji (GZIP, BZIP2, LZO, ZSTD).
        
    - Zaleca się kompresowanie plików przed ich przesłaniem (oszczędność pasma i kosztów storage).
        
- **Kodowanie (Encoding):**
    
    - Domyślnie Snowflake oczekuje **UTF-8**. Inne kodowania muszą być jawnie określone w `FILE FORMAT`.
        
- **Dane Półstrukturalne (JSON, Parquet, Avro):**
    
    - Zaleca się usuwanie zewnętrznych nawiasów tablic (`STRIP_OUTER_ARRAY = TRUE`), aby ładować rekordy jako osobne wiersze, a nie jeden wielki dokument.
        
    - Snowflake najlepiej radzi sobie z plikami o spójnej strukturze (np. te same klucze w każdym obiekcie JSON).
        

---

### 2. Planning a Data Load

**Ważność na egzaminie:** ⭐⭐⭐⭐

- **Bulk Loading vs Snowpipe:**
    
    - **Bulk Loading:** Używa polecenia `COPY INTO`. Wymaga **Virtual Warehouse** (Ty płacisz za czas jego pracy). Przetwarza duże partie danych naraz.
        
    - **Snowpipe:** Rozwiązanie serverless (nie potrzebujesz własnego Warehouse). Ładuje dane w sposób ciągły w małych paczkach (micro-batches), gdy tylko pojawią się w stage'u.
        
- **Zasada Idempotency (Load History):**
    
    - Snowflake zapamiętuje, które pliki zostały już załadowane (przez 64 dni dla `COPY INTO` i 14 dni dla Snowpipe).
        
    - Dzięki temu, jeśli uruchomisz to samo polecenie dwa razy, te same pliki **nie zostaną** załadowane ponownie (chyba że użyjesz flagi `FORCE = TRUE`).
        
- **Warehouse Sizing:**
    
    - Dla małych i średnich ilości plików zwykle wystarczy **X-Small**. Zwiększenie rozmiaru Warehouse ma sens tylko wtedy, gdy ładujesz **tysiące plików naraz** (wtedy więcej serwerów pracuje równolegle).
        
### 4. Loading Data

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

- **Polecenie COPY INTO:**
    
    - Podstawowe polecenie do ładowania danych.
    - Snowflake pozwala na użycie zapytania `SELECT` wewnątrz komendy `COPY INTO <table>`, co jest określane mianem **transformacji podczas ładowania** (data transformation during load). Jest to niezwykle potężne narzędzie, ponieważ pozwala na modyfikację danych „w locie”, zanim fizycznie wylądują w Twojej tabeli (np. zmiana kolejności kolumn, rzutowanie typów, proste funkcje, truncate).
			    - `COPY INTO my_table
			FROM (
			  SELECT 
			    s.$1,                           -- Pierwsza kolumna z pliku
			    s.$3,                    -- Trzecia kolumna z pliku (pomijamy drugą)
			    UPPER(s.$4),            -- Zmiana liter na wielkie
			    TO_DATE(s.$5, 'YYYY-MM-DD') -- Konwersja tekstu na datę
			  FROM @my_stage s
			)
			FILE_FORMAT = (TYPE = CSV);`

    * You can load files using COPY INTO by providing exact file names, load all files from a specific path, or load files that match a pattern.
    
- **Obsługa błędów (`ON_ERROR`):**
    
    - `ABORT_STATEMENT` (domyślne): Jeśli jeden błąd w jednym pliku – cały proces staje, nic nie jest załadowane.
    - `CONTINUE`: Ładuje wszystko co poprawne, pomija błędne wiersze.
    - `SKIP_FILE`: Pomija cały plik, w którym wystąpił błąd.
        
- **Walidacja:**
    
    - Parametr `VALIDATION_MODE = RETURN_ERRORS` pozwala sprawdzić błędy w plikach **bez ich faktycznego ładowania** do tabeli (oszczędność czasu i credits).
        
- **Usuwanie plików po ładowaniu:**
    
    - Parametr `PURGE = TRUE` automatycznie usuwa pliki ze stage'u po udanym załadowaniu. Zalecane, aby nie płacić za storage w stage'u.
        
- Aby załadować dane, rola musi posiadać:
    - `USAGE` na Database i Schema.
    - `INSERT` na docelowej tabeli.
    - `USAGE` na Stage (jeśli używamy nazwanego stage'a).
        
- **Wspierane formaty:** Snowflake natywnie wspiera: **CSV, JSON, XML, Avro, Parquet, ORC**. Zapamiętaj tę listę – często pojawiają się pytania typu "którego formatu Snowflake NIE wspiera bezpośrednio".
    
- **Struktura tabeli:** Dane w pliku muszą odpowiadać typom danych w kolumnach tabeli. Jeśli nie odpowiadają, musisz użyć transformacji wewnątrz polecenia `COPY`.
---

**Wskazówka pod egzamin:** Zapamiętaj, że `COPY INTO` do działania wymaga aktywnego Virtual Warehouse (czyli statusu 'Started'), podczas gdy Snowpipe zarządza zasobami automatycznie w tle.


### 5. Monitoring loads
Snowflake oferuje trzy główne sposoby sprawdzania, co się stało z Twoim ładowaniem:

1. **Metadane polecenia COPY INTO:**
    
    - Po wykonaniu `COPY INTO`, Snowflake zwraca tabelę z podsumowaniem (ile wierszy załadowano, ile było błędów).
        
2. **Information Schema (Widok LOAD_HISTORY):**
    
    - Przechowuje historię ładowania danych przez polecenie `COPY INTO`.
    - **Retencja:** Dane są dostępne przez **14 dni**.
    - Zasięg: Tylko dla tabel w konkretnej bazie danych.
        
3. **Account Usage (Widok COPY_HISTORY):**
    
    - Widok w bazie danych `SNOWFLAKE`.
    - **Retencja:** Dane dostępne przez **365 dni (1 rok)**.
    - **Opóźnienie (Latency):** Dane w tym widoku mogą pojawić się z opóźnieniem (nawet do 2 godzin), podczas gdy Information Schema jest niemal natychmiastowa. (Ważna różnica na egzamin!).

DO ZAPAMIETANIA DO EGZAMINU: 
LOAD HISTORY -> W INFORMATION SCHEMA, DOSTEPNE OD RAZU, NA 14 DNI
COPY HISTOR -> W BAZIE SNOWFLAKE, DOSTEPNE Z OPOZNIENIEM, NA ROK
        
4. **Funkcja VALIDATE:**
    
    - Możesz użyć `VALIDATE(<table_name>, job_id => '<query_id>')`, aby zobaczyć szczegóły błędów z poprzedniego ładowania, które nie zostało ukończone z powodu błędów.

Monitoring Snowpipe odbywa się inaczej (przez funkcję `PIPE_STATUS` lub widok `PIPE_USAGE_HISTORY`)


### MIKROPARTYCJE
### Micropartitions/metadata
When new data is loaded into a table, new micro partitions are written to the storage, and every time a new micro partition for a table is written, the table's metadata has changed, because this metadata contains information on the micro partitions. 
Informacje o tym jakie pliki zostały załadowane do tabeli jest przechowywana w load metadata (zeby nie zaladowac drugi raz tych samych plikow niechcacy. To load metadata jest przechowywane przez 64 dni)



### Ingestion Metadata (Metadane ładowania)

[[CHANGE DATA CAPTURE (CDC)]]

**Ważność na egzaminie:** ⭐⭐⭐

Podczas ładowania danych (`COPY INTO` lub Snowpipe), Snowflake pozwala na automatyczne przechwycenie informacji o samym procesie ładowania. Najważniejszą nowością w tej sekcji, poza tymi, które już znamy (`FILENAME`), jest czas skanowania.

- **`METADATA$START_SCAN_TIME`**:
    
    - To znacznik czasu (**Timestamp**), który mówi dokładnie, kiedy Snowflake zaczął odczytywać dany plik podczas operacji ładowania. Jest to wyrażone w strefie czasowej **UTC**.
        
- **Zastosowanie:** * Pozwala na śledzenie opóźnień (Latency) – porównujesz czas zdarzenia w danych z czasem `METADATA$START_SCAN_TIME`.  Pomaga w rozwiązywaniu problemów z kolejnością danych w systemach typu CDC.