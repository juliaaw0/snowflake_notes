[[Clustering Keys]]
[[Search Optimization Service (SOS)]]
[[Query Acceleration Service (QAS)]]
[[Materialized views]]


---

### 1. Trzy Poziomy Caching (Absolutny "must-know")

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Zanim zaczniesz zwiększać rozmiar Warehouse, musisz wiedzieć, skąd Snowflake bierze dane. Istnieją trzy "warstwy" pamięci:

1. [[Metadata Cache]] (Cloud Services Layer): Snowflake przechowuje statystyki o mikro-partycjach (np. min/max wartości, liczba rekordów). Zapytania typu `SELECT COUNT(*) FROM table` lub `SELECT MIN(col)` nie potrzebują nawet włączonego Warehouse! Są darmowe i błyskawiczne.
    
2. [[Query Result Cache]] (24h):** Jeśli zadasz identyczne zapytanie dwa razy w ciągu 24h i dane się nie zmieniły, Snowflake zwróci wynik z Cloud Services. **Warehouse nie zostanie uruchomiony.**
    
3. **Warehouse Cache (Local SSD):** Gdy Warehouse pracuje, pobiera dane z Cloud Storage i zapisuje je na swoich dyskach SSD. Kolejne zapytania do tych samych danych będą znacznie szybsze, bo nie muszą "lecieć" z dalekiego S3/Azure Blob.


### Query Profile (Twój stetoskop)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Jeśli zapytanie działa wolno, `Query Profile` w Snowsight powie Ci dlaczego. Musisz znać te pojęcia:

- **Pruning (Przycinanie):** To najważniejszy wskaźnik. Jeśli widzisz "Partitions scanned: 1000" a "Partitions total: 1,000,000", to znaczy, że Snowflake świetnie odrzucił niepotrzebne dane. Jeśli te liczby są bliskie siebie – masz problem (tzw. Full Table Scan).
    
- **Spilling to Disk:** Jeśli Warehouse ma za mało pamięci RAM na Twoje zapytanie, zaczyna zrzucać dane na dysk lokalny (Local Disk Spilling), a potem na zdalny (Remote Disk Spilling). To zabójca wydajności. **Rozwiązanie: Większy Warehouse.**
    
- **Exploding Joins:** Gdy zapytanie zwraca znacznie więcej wierszy niż ma tabela źródłowa (np. przez błąd w `JOIN`).

* query profile jest dostepne dla query w kazdym statusie: running, failed, success



### Opcje Wydajności (Query Options)

**Ważność na egzaminie:** ⭐⭐⭐

Możesz sterować zachowaniem Snowflake'a za pomocą parametrów:

- **`STATEMENT_TIMEOUT_IN_SECONDS`:** Domyślnie 2 dni (172,800 sek.). Warto to skrócić, aby "wiszące" zapytania nie zżerały kredytów przez weekend.
    
- **`USE_CACHED_RESULT`:** Możesz wyłączyć Result Cache (często robimy to podczas testów wydajności, żeby sprawdzić realną moc Warehouse).
    


### Queuing (Kolejkowanie) vs. Spilling (Wyciek na dysk)

To dwa najczęstsze problemy wydajnościowe. Musisz wiedzieć, jak je odróżnić w `Query Profile`.

- **Queuing (Kolejkowanie):** Zapytanie czeka na start, bo Warehouse jest zajęty innymi zadaniami.
    
    - **Rozwiązanie:** **Multi-cluster Warehouse (Scale Out)**. Dodanie klastrów pozwoli obsłużyć więcej zapytań naraz.
        
- **Spilling (Wyciek do pamięci masowej):** Zapytanie nie mieści się w pamięci RAM i musi zapisywać dane tymczasowe na dysku.
    
    - **Local Disk Spilling:** Dane trafiają na lokalny SSD Warehouse'u. Jest wolniej, ale znośnie.
        
    - **Remote Disk Spilling:** Dane trafiają do S3/Azure Blob. Jest **bardzo wolno**.
        
    - **Rozwiązanie:** **Scale Up** (większy rozmiar Warehouse, np. z Small na Large).




### Warehouse Cache (Local SSD)

**Ważność na egzaminie:** ⭐⭐⭐⭐

To "ciepłe" dane. Kiedy Warehouse pobiera dane z Cloud Storage (S3), zapisuje je na swoich lokalnych dyskach SSD.

- Jeśli zadasz inne zapytanie, które potrzebuje tych samych danych, Warehouse weźmie je z SSD (błyskawicznie), a nie z S3 (wolniej).
    
- **Ważne:** Jeśli Warehouse zostanie uśpiony (Suspend), jego **Lokalny Cache zostaje wyczyszczony!** Po ponownym uruchomieniu Warehouse jest "zimny" i musi znów pobierać dane z chmury.


### Max Concurrency Level

**Ważność na egzaminie:** ⭐⭐⭐

Parametr `MAX_CONCURRENCY_LEVEL` (domyślnie 8) mówi Snowflake'owi, ile zapytań może próbować uruchomić jednocześnie na jednym klastrze.

- Snowflake sam zarządza zasobami. Jeśli widzi, że 2 zapytania są gigantyczne i zjadły cały RAM, to 3. zapytanie trafi do kolejki (Queuing), nawet jeśli limit 8 nie został osiągnięty.


# STORAGE OPTIMIZATION


### Mikro-partycje i Pruning (Przycinanie)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Absolutny fundament)

Snowflake nie używa tradycyjnych indeksów. Zamiast tego dzieli każdą tabelę na **Micro-partitions**.

- **Charakterystyka:** Są niezmienne (immutable), mają rozmiar od 50 do 500 MB nieskompresowanych danych.
    
- **Metadata:** Dla każdej mikro-partycji Snowflake przechowuje w warstwie Cloud Services statystyki: **min/max wartość** każdej kolumny.
    
- **Pruning:** Kiedy zadajesz pytanie `WHERE Date = '2024-01-01'`, Snowflake patrzy na metadane i odrzuca (pruning) wszystkie partycje, których zakres min/max nie obejmuje tej daty.
    
- **Wynik:** Warehouse skanuje tylko ułamek danych, co drastycznie przyspiesza zapytanie.

It is possible to accomplish better partition pruning by redistributing data in micro partitions using [[clustering keys]].


### Search Optimization Service (SOS)

**Ważność na egzaminie:** ⭐⭐⭐⭐

To funkcja dla specyficznych przypadków: "szukania igły w stogu siana".

- **Kiedy użyć?** Gdy masz ogromną tabelę (terabajty) i często zadajesz zapytania o pojedyncze rekordy (np. `WHERE Customer_ID = 12345`), a kolumna ta nie jest kluczem klastrowania.
    
- **Jak działa?** To jak "super-indeks" działający w tle, który przyspiesza punktowe wyszukiwanie.
    
- **Koszt:** SOS zużywa kredyty na utrzymanie struktury wyszukiwania (Serverless).



## podsumowanie

![[Zrzut ekranu 2026-05-4 o 23.01.58.png]]
![[Zrzut ekranu 2026-05-4 o 23.05.03.png]]
![[Zrzut ekranu 2026-05-4 o 23.06.26.png]]
![[Zrzut ekranu 2026-05-4 o 23.07.39.png]]

