[[CONTINUOUS DATA PROTECTION]]

### Co to jest Time Travel?

To funkcja **Continuous Data Protection (CDP)**, która umożliwia dostęp do danych (tabel, schematów, baz), które zostały zmodyfikowane lub usunięte, w dowolnym punkcie w przeszłości w ramach określonego okna czasowego.

**Co możesz zrobić dzięki Time Travel?**

- **Odpytać (Query):** Zobaczyć, jak wyglądała tabela wczoraj o 14:00.
    
- **Odzyskać (Restore):** Przywrócić przypadkowo usuniętą tabelę komendą `UNDROP`.
    
- **Sklonować (Clone):** Stworzyć nową tabelę na podstawie stanu starej tabeli sprzed 3 dni.


### Okres retencji (Retention Period)

To czas, przez który Snowflake przechowuje "stare" wersje danych. Można go ustawić na poziomie konta, bazy, schematu lub konkretnej tabeli. TIme travel jest wspierane przez wszystkie edycje snowflake, ale na egzaminie musisz znać różnice między edycjami. 

| **Edycja Snowflake**      | **Domyślny okres** | **Maksymalny okres**                                       |
| ------------------------- | ------------------ | ---------------------------------------------------------- |
| **Standard**              | 1 dzień            | **1 dzień** (nie da się zwiększyć)                         |
| **Enterprise (i wyższe)** | 1 dzień            | **90 dni** (konfigurowalne ale nie moze byc wiecej niz 90) |


### Składnia SQL: Jak podróżować w czasie?

Aby "cofnąć się", używamy klauzuli `AT` (w tym konkretnym momencie) lub `BEFORE` (tuż przed tym momentem). Mamy trzy sposoby nawigacji:

1. **Timestamp:** Dokładna data i godzina.
    
    - `SELECT * FROM my_table AT(TIMESTAMP => '2024-05-01 10:00:00'::timestamp_tz);`
        
2. **Offset:** Czas w sekundach wstecz od teraz.
    
    - `SELECT * FROM my_table AT(OFFSET => -3600);` (stan sprzed godziny)
        
3. **Statement ID:** Stan danych po wykonaniu konkretnego zapytania (Query ID).
    
    - `SELECT * FROM my_table BEFORE(STATEMENT => 'query_id_tutaj');`



### Funkcja UNDROP

Jeśli przypadkowo usuniesz obiekt (`DROP TABLE`, `DROP SCHEMA`, `DROP DATABASE`), możesz go natychmiast przywrócić, dopóki jest w oknie Time Travel. Dziala dla tabel, schem i baz. Nie dziala dla np userow i ról. 

- **Przykład:** `UNDROP TABLE my_deleted_table;`
    
- **Ważne:** Jeśli stworzyłeś nową tabelę o tej samej nazwie, musisz najpierw zmienić nazwę tej nowej, aby móc "od-dropować" starą.



### Time Travel dla różnych typów tabel

To jest najczęstszy "haczyk" egzaminacyjny!

- **Permanent Tables:** 0-90 dni (wymaga Enterprise dla >1 dnia). Posiadają Fail-safe.
    
- **Transient Tables:** 0-1 dzień. **Nie posiadają Fail-safe.**
    
- **Temporary Tables:** 0-1 dzień. **Nie posiadają Fail-safe.** Znikają po zamknięciu sesji.



### Koszty (Storage Costs)

Time Travel nie jest darmowy.

- Snowflake przechowuje "delty" (różnice) w micro-partitions. Jeśli zmienisz 1 GB danych w tabeli, Snowflake musi zachować stare wersje tych mikro-partycji. "when users request to read data from a table using time travel extension and they want to read data as it existed at a specific point, snowflake reads data from deleted historical partitions to fulfill those time travel queries."
    
- Płacisz za **Storage** tych starych danych tak samo, jak za aktywne dane.
    
- Dobrą praktyką jest ustawianie krótkiego Time Travel (np. 0-1 dzień) dla tabel, które bardzo często się zmieniają (np. tabele stagingowe), aby uniknąć gigantycznych kosztów.

"Płacisz za **każdą wersję mikro-partycji**, która jeszcze nie "wygasła" z Time Travel lub Fail-safe, dlatego dla ogromnych, często zmienianych tabel o niskim znaczeniu **zawsze wybieraj typ Transient**, aby uniknąć 7 dni płatnego Fail-safe."




