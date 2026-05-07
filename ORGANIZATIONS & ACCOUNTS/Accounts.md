
[[ORGANIZATIONS AND ACCOUNTS]]

### Account Identifiers (To jest "pewniak" na test!)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Istnieją dwa główne sposoby identyfikacji konta. Musisz wiedzieć, kiedy użyć którego:

- **Account Name (wewnątrz Organizacji):**
    
    - Format: `ORNAME.ACCOUNTNAME` (np. `MYCORP.PROD_DATA`).
        
    - Zastosowanie: Logowanie przez URL, replikacja, udostępnianie danych (Data Sharing).
        
    - **Zaleta:** Jest przenośny. Jeśli zmigrujesz konto z AWS do Azure przez replikację, nazwa może zostać ta sama.
        
- **Account Locator (Legacy ID):**
    
    - Format: Krótki ciąg znaków (np. `XY12345`).
        
    - Zastosowanie: Niektóre starsze sterowniki (JDBC/ODBC), funkcje systemowe, wsparcie techniczne Snowflake.
        
    - **Wada:** Jest przypisany do regionu i chmury. Jeśli zmienisz region, locator się zmieni.



### Trial Accounts (Konta próbne)

**Ważność na egzaminie:** ⭐⭐⭐

Mimo że to podstawowy temat, egzamin pyta o limity kont testowych:

- **Kredyty:** Zazwyczaj dostajesz **400$** wirtualnych kredytów na start.
    
- **Wybór edycji:** Zakładając triala, możesz wybrać **dowolną edycję** (Standard, Enterprise, Business Critical), aby przetestować wszystkie funkcje.
    
- **Przekształcenie:** Po 30 dniach lub zużyciu kredytów konto przechodzi w stan "suspended", chyba że podepniesz kartę płatniczą (staje się kontem typu _On-Demand_).