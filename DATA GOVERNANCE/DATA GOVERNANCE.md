tu wrzucam inne tematy dotyczace data governance, poza tymi glownymi

### Data Quality & Monitoring (DMFs) (malo wazne)

Snowflake wprowadził **Data Metric Functions (DMFs)**, aby zautomatyzować sprawdzanie jakości danych. Zamiast pisać własne skrypty, używasz gotowych funkcji.

- **System DMFs:** Snowflake daje Ci gotowce, takie jak:
    
    - `NULL_COUNT`: Ile jest nulii w kolumnie.
        
    - `DUPLICATE_COUNT`: Szukanie duplikatów.
        
    - `FRESHNESS`: Kiedy ostatnio dane zostały zaktualizowane.
        
- **User-Defined DMFs:** Możesz napisać własną funkcję sprawdzającą specyficzną logikę biznesową.
    
- **Schedule (Harmonogram):** DMF-y nie działają "ciągle". Ustawiasz harmonogram (np. raz na godzinę/dzień), a Snowflake sam uruchamia testy i zapisuje wyniki.



### Access History

- **Co to jest?** Widok w `ACCOUNT_USAGE`, który rejestruje każde zapytanie odczytujące lub zapisujące dane.
    
- **Dlaczego jest wyjątkowy?** Śledzi **bezpośredni** i **pośredni** dostęp do danych.
    
    - _Przykład:_ Jeśli zrobisz `SELECT` na widoku, który pobiera dane z tabeli, `ACCESS_HISTORY` pokaże oba te obiekty. Dzięki temu wiesz, że użytkownik "dotknął" danych źródłowych, nawet jeśli zrobił to przez widok.
        
- **Zastosowanie:** Wykrywanie nieużywanych tabel (nikt ich nie czytał od 90 dni) oraz śledzenie wycieków danych.


### Differential Privacy
- Snowflake dodaje niewielką ilość "szumu" (losowych danych) do wyników zapytań agregujących (np. `SUM`, `AVG`). Wynik jest statystycznie poprawny dla dużej grupy, ale uniemożliwia sprawdzenie, czy konkretna osoba (np. Jan Kowalski) znajduje się w zbiorze danych.


