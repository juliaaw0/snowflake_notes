[[ORGANIZATIONS AND ACCOUNTS]]

Organizacja to najwyższy poziom w hierarchii Snowflake. To „parasol”, pod którym znajdują się wszystkie Twoje konta.

- **Multicloud & Multi-region:** Jedna Organizacja może zarządzać kontami na AWS, Azure i GCP jednocześnie, w różnych częściach świata. Aby móc kopiować bazy danych lub dane o użytkownikach między kontem na AWS (USA) a kontem na Azure (Europa), musisz mieć włączoną Organizację.
    
- **Centralizacja:** Pozwala na wspólny billing, ujednolicone nazewnictwo i łatwą replikację danych między kontami.


### Rola ORGADMIN
 To rola systemowa, która zarządza całą Organizacją. Musisz znać różnicę między nią a `ACCOUNTADMIN`.
 
![[Zrzut ekranu 2026-05-3 o 00.58.54.png]]



`ORGADMIN` nie jest automatycznie przypisany do żadnego użytkownika przy zakładaniu pierwszego konta (często trzeba go jawnie włączyć). Dobrą praktyką jest posiadanie co najmniej dwóch użytkowników z tą rolą.

Snowflake udostępnia specjalną bazę danych o nazwie `SNOWFLAKE`, a w niej schemat **`ORGANIZATION_USAGE`**.

- Znajdziesz tam widoki takie jak `ACCOUNTS` (lista kont), `RATE_SHEETS` (ceny usług) i `USAGE_IN_CURRENCY_DAILY`.
    
- **Opóźnienie:** Dane w `ORGANIZATION_USAGE` mogą mieć do **24h opóźnienia** (to częsty szczegół na egzaminie).



### Tworzenie kont jako organizacja

To jedna z najważniejszych operacji. Kiedy `ORGADMIN` tworzy nowe konto, musi określić 4 krytyczne parametry:

1. **Account Name:** Unikalna nazwa wewnątrz organizacji.
    
2. **Edition:** Standard, Enterprise lub Business Critical (wybór edycji determinuje dostępne funkcje i cenę).
    
3. **Region:** Region dostawcy chmury (np. `aws_us_east_1`).
    
4. **Admin Name/Password:** Dane pierwszego użytkownika, który otrzyma rolę `ACCOUNTADMIN` na nowym koncie.
    

> **Ważne:** Nie można zmienić **Regionu** ani **Dostawcy Chmury** po utworzeniu konta. Jeśli się pomylisz, musisz usunąć konto i stworzyć nowe.


Przy tworzeniu konta wybierasz edycję. Musisz znać różnice pod kątem egzaminu: rozszerz wiedze o tym

- **Standard:** Podstawowa. Brak Time Travel powyżej 1 dnia, brak Dynamic Data Masking.
    
- **Enterprise:** Większość firm wybiera tę edycję. Posiada **Multi-cluster Warehouse**, **Time Travel do 90 dni**, Materialized Views.
    
- **Business Critical:** Dla danych wrażliwych. Posiada **Tri-Secret Secure**, Private Link, zwiększone bezpieczeństwo (HIPAA/PCI).
    
- **VPS (Virtual Private Snowflake):** Najwyższa, odizolowana instalacja dla gigantycznych instytucji (np. banków). Ma np wlasny metadata store. 


Pytanie testowe:
_Czy ACCOUNTADMIN może zmienić nazwę swojego konta?_ **Odpowiedź:** Nie. Może to zrobić tylko **ORGADMIN**, ponieważ zmiana nazwy konta wpływa na całą strukturę organizacji i adresy URL.



