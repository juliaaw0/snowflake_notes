[[DATA GOVERNANCE]]

Klasyfikacja to proces, w którym Snowflake analizuje zawartość Twoich tabel (używając algorytmów uczenia maszynowego), aby wykryć dane **PII** (Personally Identifiable Information), czyli dane osobowe (np. nazwiska, numery kont, maile).

- **Cel:** Automatyczne wykrycie wrażliwych danych, aby móc je potem zamaskować. - nowflake skanuje próbkę danych w kolumnach. System sugeruje kategorie dla każdej kolumny. Jeśli zaakceptujesz sugestie, Snowflake przypisuje systemowe tagi (`SNOWFLAKE.CORE.SEMANTIC_CATEGORY` oraz `PRIVACY_CATEGORY`) do tych kolumn.
    
- **Obiekty:** Działa na tabelach i widokach.
    
- **Koszt:** Proces klasyfikacji zużywa kredyty (Compute) jako operacja typu **Serverless**.


