[[DATA LOADING]]

Data Unloading to proces odwrotny do ładowania – eksportujemy dane z tabel Snowflake do plików w chmurze (S3, Azure, GCS) lub do wewnętrznego stage'a.

Eksport danych odbywa się w dwóch krokach (analogicznie do ładowania, tylko w drugą stronę):
1. **Etap 1:** Używasz polecenia `COPY INTO <lokalizacja>` (lokalizacja to stage).
2. **Etap 2:** Pobierasz pliki ze stage'a na swój dysk lokalny (za pomocą komendy `GET` w SnowSQL) lub zostawiasz je w chmurze dla innych systemów.


### Polecenie COPY INTO 

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

To jest najważniejsza komenda. Musisz wiedzieć, że:

- **Źródło:** Możesz eksportować całą tabelę albo **wynik zapytania SELECT**.
    
    - _Przykład:_ `COPY INTO @my_stage/exports/ FROM (SELECT name, email FROM users WHERE active = true);`
        
- **Formaty plików:** Snowflake potrafi zapisać dane jako: **CSV, JSON, Parquet**. (Ważne: Nie wspiera eksportu do XML na ten moment).
    
- **Kompresja:** Domyślnie Snowflake kompresuje eksportowane pliki algorytmem **GZIP**. Możesz to zmienić w `FILE_FORMAT` lub wylaczyc kompresje



### Kluczowe parametry eksportu (Częste pytania!)

Snowflake uwielbia pytać o to, jak kontrolować pliki wynikowe:

- **`SINGLE = TRUE | FALSE`**:
    
    - `FALSE` (domyślne): Snowflake wyeksportuje dane do **wielu plików** równolegle (parallelism). Każdy serwer w Warehouse zapisuje swoją część danych. To jest najszybsza metoda.
        
    - `TRUE`: Wszystkie dane trafią do **jednego pliku**. Uwaga: To jest wolniejsze i ma limit rozmiaru (zależy od chmury).
        
- **`MAX_FILE_SIZE`**: Pozwala określić maksymalny rozmiar pojedynczego pliku. Gdy Snowflake go przekroczy, zacznie pisać do nowego pliku. defaultowo = 16 mb. the maximum allowed size per file is 5GB if you export data to cloud storage
    
- **`OVERWRITE = TRUE | FALSE`**: Czy Snowflake ma nadpisać pliki w stage'u, jeśli już tam są? (Domyślnie `FALSE`).
    
- **`HEADER = TRUE | FALSE`**: (Tylko dla CSV) Czy dodać pierwszy wiersz z nazwami kolumn?
    
- **`INCLUDE_QUERY_ID = TRUE | FALSE`**: Dodaje unikalny identyfikator zapytania do nazwy pliku, co zapobiega konfliktom nazw przy częstym eksporcie.



**Uprawnienia:** Aby wyeksportować dane, Twoja rola musi mieć:

- `SELECT` na tabeli źródłowej.
    
- `WRITE` na stage'u (jeśli jest internal).
    
- Uprawnienie do _Storage Integration_ (jeśli eksportujesz bezpośrednio do S3/Azure/GCS).



![[Zrzut ekranu 2026-04-27 o 22.37.25.png]]

_Pytanie:_ Chcesz wyeksportować dane z tabeli do 10 równych plików, aby przyspieszyć proces. Który parametr za to odpowiada? _Odpowiedź:_ Snowflake robi to automatycznie dzięki równoległości (parallelism), o ile parametr `SINGLE` jest ustawiony na `FALSE`.
