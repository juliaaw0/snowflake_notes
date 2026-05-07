
[[DATA GOVERNANCE]]
### Dynamic Data Masking (DDM)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

To funkcja poziomu kolumny (**Column-level Security**). Najważniejsza rzecz do zapamiętania: **Dane na dysku pozostają niezaszyfrowane (w formie surowej), a maskowanie dzieje się "w locie" (at query time).**

- Tworzysz politykę (`CREATE MASKING POLICY`), która zawiera logikę typu: „Jeśli użytkownik ma rolę `ANALYST`, pokaż mu tylko ostatnie 4 cyfry PESEL-u, a jeśli ma rolę `HR_ADMIN`, pokaż całość”.
    
- **Kluczowe funkcje SQL:** W politykach maskowania prawie zawsze używa się funkcji:
    
    - `CURRENT_ROLE()`: Sprawdza, jaką rolę ma obecnie użytkownik.
        
    - `INVOKER_ROLE()`: Sprawdza, jaka rola uruchomiła zapytanie (ważne przy widokach).


* Tag based masking - Przypisujesz politykę maskowania do **Tagu** (np. tag `Email`). Od tego momentu każda kolumna, którą oznaczysz tym tagiem, jest automatycznie chroniona tą polityką.

* Aby stworzyć politykę maskowania, rola musi mieć uprawnienie `CREATE MASKING POLICY` w schemacie.





### Tokenizacja - alternatywa dla maskingu

- W Tokenizacji dane wpadają do Snowflake'a już jako "tokeny" (bełkot). Snowflake wysyła zapytanie do zewnętrznego systemu, aby "odtokenizować" dane dla uprawnionego użytkownika. Na egzamin wystarczy wiedzieć, że to rozwiązanie dla firm, które nie chcą trzymać nawet surowych danych wrażliwych w chmurze.


### Aggregation policy

- Zapobiega tzw. "atackom różnicowym". Jeśli w tabeli jest tylko jeden pracownik z Wrocławia zarabiający 50 000 zł, to zapytanie o `AVG(Salary) WHERE City = 'Wrocław'` zdradzi jego zarobki. Jeśli przypiszesz tę politykę do tabeli, Snowflake **zablokuje zapytania**, które zwracają wyniki dla zbyt małych grup (np. jeśli grupa ma mniej niż 5 osób).


### "Pewniaki" i pułapki:

1. **DML a Maskowanie:** Czy jeśli mam prawo `UPDATE` na tabeli z zamaskowaną kolumną, to mogę ją zepsuć? Snowflake ma zabezpieczenia, które uniemożliwiają przypadkowe nadpisanie surowych danych ich zamaskowaną wersją.
    
2. **Kolejność:** Co jest sprawdzane pierwsze? Zazwyczaj **Row Access Policy** (filtrowanie wierszy), a dopiero potem **Masking Policy** (ukrywanie wartości w tych wierszach).


