[[CACHES]]

Metadata cache żyje w Cloud Services Layer

i trzyma metadane, np.:

- liczbę wierszy w micro-partitions
- min/max wartości kolumn
- informacje o partycjach
- statystyki potrzebne do optymalizacji

To nie są dane, tylko opis danych.

Dlaczego COUNT(*) jest specjalny - Snowflake zna liczbę wierszy na poziomie metadanych (micro-partitions). Dlatego dla najprostszego przypadku: SELECT COUNT(*) FROM table; Snowflake: nie skanuje danych, nie uruchamia warehouse, odpowiada z metadata cache i koszt compute = 0

To NIE znaczy, że każdy count zawsze jest „za darmo”.

✅ KIEDY COUNT NIE UŻYWA WAREHOUSE

✔ COUNT(*)

✔ Bez WHERE

✔ Bez JOIN

✔ Bez GROUP BY

✔ Bez DISTINCT

✔ Na fizycznej tabeli (nie view)

Ważne zdanie egzaminowe (zapamiętaj to dokładnie)

Snowflake can answer some COUNT queries using metadata, but not all COUNT queries.

Albo jeszcze lepsze:

Only simple COUNT(*) queries without filters can be satisfied from metadata.

Podobnie jest z MIN i MAX (ale tylko na liczbach, na literach nie)

