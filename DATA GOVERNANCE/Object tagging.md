[[DATA GOVERNANCE]]

**Object Tagging** to fundament nowoczesnego zarządzania danymi (Data Governance) w Snowflake. To narzędzie pozwala Ci tagowac obiekty, aby łatwiej było nimi zarządzać, śledzić koszty czy chronić prywatność. Tagi łączą się z maskowaniem danych.

Tag to obiekt pierwszego poziomu (first-class object), który składa się z **klucza** i **wartości** (key-value pair).

- **Przykład:** Klucz: `Sensitivity`, Wartość: `Public`, `Internal`, `Secret`.
    
- **Gdzie można przypiąć tag?** Do niemal wszystkiego: konta, bazy danych, schematu, tabeli, a nawet pojedynczej kolumny.
    
- **Limit:** Do jednego obiektu można przypisać maksymalnie **50 unikalnych tagów**.

Thumb rule: Tagi służą do **klasyfikacji** danych. Jeśli widzisz pytanie o "klasyfikację wrażliwych danych" (sensitive data classification) – **Object Tagging** to właściwy kierunek.



DZIEDZICZENIE:
Tagi działają kaskadowo. Jeśli przypiszesz tag na wyższym poziomie, obiekty "dzieci" automatycznie go dziedziczą.

- **Account → Database → Schema → Table → Column.**
    
- **Przykład:** Jeśli oznaczysz całą Bazę Danych jako `CostCenter = 'Sales'`, to każda tabela w tej bazie będzie miała ten tag, chyba że...
    
- **Nadpisywanie:** Możesz nadpisać odziedziczony tag na niższym poziomie (np. konkretna tabela wewnątrz bazy może mieć `CostCenter = 'Marketing'`).

* **Cloning (Klonowanie):** Kiedy klonujesz bazę, schemat lub tabelę, **tagi są kopiowane** do klona. **Views (Widoki):** Jeśli kolumna w tabeli źródłowej ma tag, widok stworzony na tej tabeli może go odziedziczyć (tzw. column-level tag propagation).*


### Tag-based Masking (To musisz znać!)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Ulubione pytanie egzaminacyjne)

To jest "killer feature" tagowania. Zamiast ręcznie przypisywać politykę maskowania (np. ukrywanie maili) do 100 różnych kolumn, robisz tak:

1. Tworzysz tag (np. `PII_Type = 'Email'`).
    
2. Przypisujesz politykę maskowania do **tagu**, a nie do kolumny.
    
3. Każda kolumna, którą oznaczysz tym tagiem, zostanie **automatycznie zamaskowana**.



PYTANIE EGZ:
_Czy usunięcie tagu (DROP TAG) usuwa go z obiektów?_ **Odpowiedź:** Tak, jeśli usuniesz definicję tagu, znika on ze wszystkich przypisanych obiektów. Jeśli jednak tylko "odepniesz" (unset) tag od tabeli, sama definicja tagu nadal istnieje w schemacie.