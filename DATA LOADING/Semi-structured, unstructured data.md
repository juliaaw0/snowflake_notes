[[DATA LOADING]]

# SEMI-STRUCTURED

**Najważniejszy typ: `VARIANT`**

- To uniwersalny typ danych, który może przechowywać te formaty danych polstrukturalnych - JSON Avro ORC Parquet XML.
    
- **Limit rozmiaru:** Pojedynczy rekord w kolumnie `VARIANT` może mieć maksymalnie **16 MB** (po skompresowaniu). To bardzo częste pytanie!
    
- Snowflake automatycznie optymalizuje kolumny `VARIANT`, przechowując je w formacie binarnym i wyciągając pod-kolumny do osobnych mikro-partycji, co przyspiesza zapytania.

**Obsługiwane formaty:** JSON, Avro, ORC, Parquet, XML. (Pamiętaj o XML – Snowflake obsługuje go, ale traktuje nieco inaczej, często wymagając rzutowania na `VARIANT`).

**Schemaless:** Nie musisz definiować schematu (kolumn) przed załadowaniem danych JSON do kolumny `VARIANT`. To podejście nazywa się **Schema-on-read**.
    
**Wydajność:** Dzięki "columnarization" (kolumnowaniu) danych wewnątrz `VARIANT`, zapytania o konkretny klucz (np. `data:user:id` - to sie nazywa dot notation, ta ścieżka po jsonie) są niemal tak szybkie jak zapytania o zwykłe kolumny relacyjne.
    
**NULLe w JSON:** Snowflake odróżnia SQL-owy `NULL` od JSON-owego `null`. Przy ładowaniu warto znać parametr `STRIP_NULL_VALUES = TRUE`, który pozwala usunąć puste pola, oszczędzając miejsce.

**Querowanie typu VARIANT:**

* ***Dot Notation (Kropka):** `src:customer:id` – proste i najczęstsze.
    
- **Bracket Notation (Nawiasy):** `src['customer']['id']` – przydatne, gdy nazwa klucza zawiera spacje lub znaki specjalne.
    
- **Rzutowanie (Casting):** Dane wyciągnięte z `VARIANT` są domyślnie traktowane jako `VARIANT`. Aby ich używać w obliczeniach, musisz je rzutować: `src:age::int` lub `src:total_price::float`.
    
- **Flattening:** Funkcja `FLATTEN()` służy do zamiany tablic (`ARRAY`) na osobne wiersze. Często używana z `LATERAL JOIN`.

### Pułapki Egzaminacyjne (Uważaj na to!):

1. **Pytanie:** Jaki jest limit rozmiaru kolumny `VARIANT`?
    
    - **Odpowiedź:** 16 MB skompresowanych danych.
        
2. **Pytanie:** Czy Snowflake obsługuje XML?
    
    - **Odpowiedź:** Tak, ale XML jest konwertowany na strukturę obiektową wewnątrz `VARIANT`.
        
3. **Pytanie:** Co jest szybsze: zapytanie o kolumnę relacyjną (NUMBER) czy o klucz wewnątrz `VARIANT`?
    
    - **Odpowiedź:** Różnica jest minimalna dzięki optymalizacji Snowflake, ale kolumna relacyjna jest technicznie zawsze odrobinę szybsza. Jednak pod kątem egzaminu: Snowflake optymalizuje `VARIANT` do postaci kolumnowej.
        
4. **Pytanie:** Jakiej funkcji użyjesz, żeby "rozbić" tablicę JSON na wiele wierszy?
    
    - **Odpowiedź:** `FLATTEN`.



# UNSTRUCTURED
Dane nieustrukturyzowane (**Unstructured Data**) to pliki, których Snowflake nie potrafi "przeczytać" jako tabeli (np. obrazy, PDF-y). W przypadku tych danych Snowflake nie przechowuje ich treści w kolumnach, ale zarządza ich **metadanymi** i **dostępem**

**Directory tables:**

To nie jest osobny obiekt (jak zwykła tabela), ale **warstwa metadanych** nałożona na Stage.
Podczas tworzenia stage'a dodajesz parametr `DIRECTORY = (ENABLE = TRUE).
- **Co to daje?** Pozwala na przeszukiwanie plików w stage'u za pomocą SQL, tak jakby były tabelą.
- **Metadane:** Directory Table automatycznie wyciąga informacje o plikach:
    - `RELATIVE_PATH`: Ścieżka do pliku względem stage'a.
    - `SIZE`: Rozmiar pliku.
    - `LAST_MODIFIED`: Data ostatniej zmiany.
    - `FILE_URL`: Stały adres URL do pliku.
- **Koszt:** Snowflake pobiera niewielką opłatę za przechowywanie tych metadanych (storage) oraz za ewentualny proces odświeżania (compute).

Wiecej o tym URL:
Snowflake oferuje trzy rodzaje adresów URL do plików:
1. **Scoped URL:** Najbardziej bezpieczne. Generowane przez funkcję `BUILD_SCOPED_FILE_URL.` Działa tylko w ramach bieżącej sesji użytkownika. Jeśli wyślesz ten link komuś innemu, nie zadziała. Idealne do aplikacji webowych zintegrowanych ze Snowflake.
        
2. **File URL:** Wymaga zalogowania do Snowflake. Stały link dostępny przez UI lub API. Aby go otworzyć, musisz mieć uprawnienia `USAGE` do stage'a.
        
3. **Pre-signed URL:**  Udostępnianie plików osobom/systemom **bez konta w Snowflake**. Generowane przez funkcję `GET_PRESIGNED_URL.` Ma określony czas wygaśnięcia (ustawiany przez użytkownika). Każdy, kto ma ten link, może pobrać plik.


