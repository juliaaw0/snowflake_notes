[[CHANGE DATA CAPTURE (CDC)]]
[[CLOUD SERVICES LAYER]]

Task to obiekt, który wykonuje pojedynczą instrukcję SQL lub store procedure wg harmonogramu.

Możesz łączyć zadania w logiczne ciągi.

- **Root Task (Zadanie główne):** Jedyne zadanie w grafie, które ma zdefiniowany harmonogram (`SCHEDULE`). To ono uruchamia całą lawinę.
    
- **Child Tasks (Zadania potomne):** Uruchamiają się po zakończeniu zadania nadrzędnego. Definiujesz je za pomocą słowa kluczowego `AFTER`.
    
    - _Przykład:_ `CREATE TASK task_2 AFTER task_1 ...`
        
- **Limit:** Graf może mieć do **1000 zadań**.
    
- **Zasada działania:** Zadanie potomne uruchomi się tylko wtedy, gdy zadanie nadrzędne (Parent) zakończy się **sukcesem**.



**Dwa modele obliczeniowe (Compute Models):**

1. **User-managed Warehouse:** Task korzysta z Twojego wirtualnego magazynu. Musisz go podać przy tworzeniu (`WAREHOUSE = my_wh`). Płacisz standardowo za czas pracy Warehouse.
    
2. **Serverless Tasks:** Snowflake sam przydziela zasoby. Nie musisz definiować Warehouse. Płacisz za faktyczny czas procesora (compute time). To świetne dla krótkich zadań.

WAŻNE:
**Stan początkowy:** Nowo stworzony Task jest zawsze w stanie **SUSPENDED**. Aby zaczął działać, musisz go „obudzić”: `ALTER TASK my_task RESUME;`. (To najczęstsze pytanie typu „dlaczego mój task nie działa?”).



### "Triggered" Tasks – Inteligentne sprawdzanie

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

To jest brakujące ogniwo Twojej wiedzy o Strumieniach. Używając parametru `WHEN`, możesz sprawić, by Task był ekstremalnie oszczędny:

- **Składnia:** `WHEN SYSTEM$STREAM_HAS_DATA('my_stream')`
    
- **Jak to działa?** Task budzi się zgodnie z harmonogramem (np. co minutę), ale zanim odpali Warehouse, sprawdza tę funkcję. Jeśli w Strumieniu nie ma żadnych nowych zmian – **Task zasypia natychmiast**, a Ty nie płacisz za pracę Warehouse (płacisz grosze za sprawdzenie metadanych).


### Monitorowanie i błędy

**Ważność na egzaminie:** ⭐⭐⭐⭐

- **Historia:** Widok `TASK_HISTORY` (w Information Schema oraz Account Usage).
    
    - `Information Schema`: Historia z ostatnich **7 dni**.
        
    - `Account Usage`: Historia z ostatnich **365 dni**.
        
- **Statusy:** `SUCCEEDED`, `FAILED`, `SKIPPED` (gdy `WHEN` zwróciło false), `CANCELLED`.
    
- **Overrun (Nakładanie się):** Domyślnie, jeśli Task trwa dłużej niż wynosi jego interwał (np. task trwa 2 minuty, a ma się odpalać co minutę), Snowflake **pominie** kolejne uruchomienie, dopóki bieżące się nie skończy.
    
- **Timeout:** Możesz ustawić `USER_TASK_TIMEOUT_MS`, aby przerwać zadanie, które „utknęło” i zżera kredyty.




Przykladowe pytanie: _Czy zadanie potomne (Child Task) uruchomi się, jeśli zadanie główne (Root Task) zakończy się błędem?_ **Odpowiedź:** Nie. Domyślnie zadania potomne czekają na status `SUCCEEDED` swojego poprzednika.


