[[TABLES]]

Dynamiczna tabela - "Chcę, żeby ta tabela wyglądała TAK (podajesz SELECT), i ma być aktualna co 5 minut". Jest to tabela która się sama aktualizuje na podstawie kilku innych tabel. 

![[Zrzut ekranu 2026-03-12 o 21.38.27.png]]
![[Zrzut ekranu 2026-03-12 o 21.43.48.png]]


**target lag:** Jest to docelowy czas, o jaki dane w Dynamic Table mogą być "do tyłu" względem zmian w tabelach źródłowych. np: jeśli odpytasz tę tabelę o 12:10, zobaczysz w niej dane, które trafiły do źródła co najmniej do godziny 12:05. target lag nie oznacza że tabela odswieza sie rowno co 5 min. Jeśli w tabeli bazowej nic się nie zmienia przez 3 godziny, Snowflake **nie będzie** odświeżał Dynamic Table. 

są 2 sposoby ustawiania tego target lag:
- **Measure of freshness**: - Tabela "patrzy" na swoje źródło i mówi: "Muszę się odświeżyć tak, żeby nie być do tyłu o więcej niż 5 minut". Kto decyduje o starcie: Sam Snowflake. On wylicza, kiedy uruchomić Warehouse, żeby zdążyć przeliczyć dane przed upływem tych 5 minut.
- **DOWNSTREAM** - To jest ustawienie reaktywne (pasywne). To najważniejsza koncepcja pod egzamin, bo pozwala oszczędzać pieniądze. Tabela z ustawieniem `DOWNSTREAM` nie ma własnego "zegara". Ona mówi: "Mnie jest wszystko jedno, jak bardzo jestem spóźniona. Odświeżę się tylko wtedy, gdy ktoś 'wyżej' (tabela, która ze mnie korzysta) będzie mnie potrzebował". **Kto decyduje o starcie:** Tabela, która znajduje się dalej w łańcuchu i ma ustawiony konkretny czas (np. 10 min).

Czyli zwykle to dziala tak ze "srodkowe" tabele w jakims pipelinie maja typ downstream, a koncowa - podany na sztywno czas typu 10 min. 

Odświeżanie wykonuje **Virtual Warehouse**, który przypisałeś podczas tworzenia Dynamic Table. Warehouse musi być dostępny dla roli, która jest właścicielem Dynamic Table, i musi być w stanie "Started"

Typy odświeżania dynamic tables:
- incremental - Snowflake analizuje zmiany w tabelach źródłowych i przetwarza **tylko nowe lub zmienione dane**.
- full - czyta całą tabelę źródłową od zera i przelicza wszystko. Używany gdy zapytanie SQL jest zbyt skomplikowane dla trybu przyrostowego (np. niektóre funkcje niedeterministyczne) lub gdy struktura tabeli źródłowej drastycznie się zmieniła.


**Ograniczenia dynamic tables:**
- nie ma DML, zmiany mozna zrobic tylko odswiezaniem
- constraints (te z definicji - np unique key, primary key, foreign key) - nie są enforcowane w dynamic tables - mozna je ustawic ale snowflake ich nie sprawdza jak sie tworzy tabela. Sprawdzane jest tylko NOT NULL constraint. Constrainty są generalnie we wszystkich typach tabel używane przez optymalizator do przyspieszania zapytań
- Dynamic Tables **nie dziedziczą** automatycznie constraintow z tabel źródłowych.

**Immutability:** Immutability constraints let you mark portions of a dynamic table as static. When you define an immutability constraint, Snowflake skips those rows during refresh, which improves performance, especially for tables that contain large amounts of historical data. Powoduje to drastycznie niższe koszty Compute (kredytów) i szybsze odświeżanie.

Backfill: To jest "skrót klawiszowy" przy tworzeniu nowej Dynamic Table.
- **Problem:** Chcesz stworzyć nową DT na podstawie tabeli, która ma 5 lat historii. Zwykle Snowflake musiałby uruchomić ogromny Warehouse, żeby wszystko przeliczyć od zera (to jest ta "expensive initialization").
- **Rozwiązanie:** `BACKFILL FROM`. Używasz technologii **Zero-copy** (tej samej co przy Clone), żeby błyskawicznie "przepiąć" gotowe dane z innej tabeli do nowej DT.
- **Efekt pod egzamin:** Dane są kopiowane natychmiast bez użycia mocy obliczeniowej (Compute) do przeliczeń.

---

### 3. Jak to działa razem? (Scenariusz)

Kiedy tworzysz DT z oboma tymi parametrami, Snowflake dzieli Twoją tabelę na dwa regiony:

1. **Immutable Region (Region Niezmienny):** To dane "wsteczne" (Backfilled). Snowflake kopiuje je zero-copy i obiecuje, że **nigdy** nie będzie ich dotykał ani odświeżał. One po prostu tam są.
    
2. **Mutable Region (Region Zmienny):** To dane bieżące. Tylko te wiersze podlegają regularnemu odświeżaniu (Incremental Refresh).

