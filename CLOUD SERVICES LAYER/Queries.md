[[CLOUD SERVICES LAYER]]

Automatyczny timeout query (jak tego nie zmienimy) to 48h!!! 172 800 sekund


### Snowflake-Specific Syntax (Time Travel Queries)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (To jest pewniak)

Nie pytają o kod jako taki, ale o to, **jakiej klauzuli użyć**, aby "cofnąć się w czasie". Musisz znać te trzy sposoby użycia `AT` lub `BEFORE`:

- **TIMESTAMP:** `SELECT * FROM table AT(TIMESTAMP => '2023-10-24 12:00:00'::timestamp);`
    
- **OFFSET (sekundy wstecz):** `SELECT * FROM table AT(OFFSET => -60*5);` (5 minut temu).
    
- **QUERY_ID:** `SELECT * FROM table BEFORE(STATEMENT => 'query_id_tutaj');` (stan tabeli tuż przed konkretnym zapytaniem).




- **SAMPLE:** Jak pobrać losowe 10% tabeli? `SELECT * FROM table SAMPLE (10);`. Bardzo przydatne przy dużych tabelach.

- **FLATTEN:** Funkcja do "rozbijania" danych półstrukturalnych (JSON) na wiersze. Zawsze występuje z **LATERAL JOIN**. To jest jedyny "haczyk" w temacie Joinów, który na 100% pojawi się na egzaminie w kontekście danych półstrukturalnych (JSON).

	- **Koncepcja:** Lateral Join pozwala prawej stronie joina odwołać się do kolumn z lewej strony.
	    
	- **Zastosowanie:** Używamy go prawie zawsze z funkcją `FLATTEN`. Jeśli masz kolumnę `VARIANT` z tablicą w środku, `FLATTEN` rozbija ją na wiersze, a `LATERAL JOIN` łączy te wiersze z powrotem z oryginalnym rekordem.
	    
	- **Składnia do zapamiętania:** `SELECT ... FROM table, LATERAL FLATTEN(input => col_name)`. (Przecinek w Snowflake często zastępuje słowo kluczowe `CROSS JOIN LATERAL`).

![[Zrzut ekranu 2026-04-28 o 21.36.11.png]]

![[Zrzut ekranu 2026-04-28 o 21.36.38.png]]


