[[COLLABORATION]]



![[Zrzut ekranu 2026-04-29 o 23.51.56.png|416]]


![[Zrzut ekranu 2026-04-29 o 23.28.12.png]]

**Share** object to nazwany obiekt Snowflake, który działa jak "pojemnik" na uprawnienia. Zamiast dawać komuś dostęp do swojego konta, tworzysz Share, "wrzucasz" do niego tabele i mówisz: "Konto XYZ może to widzieć". Dane nie są kopiowane, tylko tabele dodane do share odnoszą się do [[Micropartitions]] oryginalnej tabeli u providera. Zatem stworzenie share nie pozera dodatkowego storage (niz to co juz zajmuje tabela) i consumer widzi zmiany na zywo bo to po prostu odwolanie do oryginalnej tabeli. 


**Co zawiera Share?**
    
    - Uprawnienia do bazy danych i schematu (`USAGE`).  Normalnie uzytkownikom korzystajacym z share mozna dodac role based access (RBAC) tak jak zwyklym kontom snowflake. 
        
    - Uprawnienia do konkretnych obiektów (`SELECT` na tabelach/widokach).
        
    - Listę kont konsumentów, którzy mogą to podpiąć.
        
- **Najważniejsza zasada:** Dane w Share są **tylko do odczytu (Read-Only)** dla konsumenta.

- **Provider (Dostawca):** Konto, które udostępnia dane.
    
- **Consumer (Konsument):** Konto, które widzi udostępnione dane (w trybie **read-only**).
    
- **Brak ruchu danych:** Dane nie są kopiowane. Konsument widzi dane "na żywo" (live metadata sharing).
    
- **Koszty:**
    
    - **Provider** płaci za **Storage** (bo dane fizycznie leżą na jego koncie).
        
    - **Consumer** płaci za **Compute** (za swój Warehouse, którego używa do odpytania danych).
        
- **Co można udostępnić?** Tabele, Secure Views, Secure UDFs (Funkcje bezpieczne).

- **No Data Movement:** Dane nigdy nie opuszczają konta dostawcy. Nie ma kopiowania, przesyłania plików ani FTP.
    
- **Live Data:** Konsument widzi dane natychmiast po ich zaktualizowaniu przez dostawcę.

* Jakie typy kont snowflake mogą uzywac Share - wszystkie oprocz VPS (virtual private snowflake). czyli shares moga uzywac: Standard, Enterprise, Business edition. 


JAK ZROBIC SHARE:
Aby udostępnić dane, musisz wykonać te kroki w SQL (warto znać kolejność):
1. **`CREATE SHARE`**: Tworzysz pusty pojemnik.
2. **`GRANT USAGE ON DATABASE ... TO SHARE`**: Dajesz dostęp do "drzwi" bazy.
3. **`GRANT USAGE ON SCHEMA ... TO SHARE`**: Dajesz dostęp do schematu.
4. **`GRANT SELECT ON TABLE ... TO SHARE`**: Dajesz dostęp do konkretnych danych.
5. **`ALTER SHARE ... ADD ACCOUNTS`**: Wskazujesz, kto może to widzieć.


- **Kto tworzy Share?** Domyślnie tylko **`ACCOUNTADMIN`**. Można jednak nadać uprawnienie `CREATE SHARE` innej roli (np. `SYSADMIN`), aby mogła zarządzać udziałami.
    
- **Własność (Ownership):** Aby dodać tabelę do Share, rola musi posiadać uprawnienie `OWNERSHIP` do tej tabeli (lub posiadać uprawnienie `SELECT` i działać w imieniu roli, która ma prawo do dzielenia się danymi).



### Co robi konsument jak pojawi sie dla niego share (Consumer Workflow)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Absolutny "must-know")

To proces, o który Snowflake pyta bardzo często. Co musi zrobić Konsument, gdy Provider doda jego konto do Share?

1. **Widoczność:** Konsument widzi przychodzący Share (może to sprawdzić komendą `SHOW SHARES` lub w UI).
    
2. **Montowanie (Mounting):** Share to tylko "link". Konsument musi stworzyć z niego lokalną bazę danych.
    
    - **Komenda:** `CREATE DATABASE <nazwa_bazy> FROM SHARE <nazwa_konta_providera>.<nazwa_share>;`
        
3. **Uprawnienia wewnętrzne:** Po stworzeniu bazy, `ACCOUNTADMIN` konsumenta musi nadać uprawnienia `USAGE` do tej nowej bazy innym rolom wewnątrz swojego konta (np. `ANALYST`), aby mogli z niej korzystać.

![[Zrzut ekranu 2026-04-30 o 00.07.43.png|697]]

Warto zapamiętać te techniczne limity:

- **Time Travel:** Konsument **nie może** korzystać z Time Travel na udostępnionych danych (nie może sprawdzić, jak tabela wyglądała 2 dni temu, chyba że Provider udostępnił mu konkretny widok historyczny).
    
- **Account Usage:** Dane o zużyciu (billing) dotyczące zapytań Konsumenta na udostępnionych danych są widoczne na koncie Konsumenta (bo to on płaci za Warehouse).


### Reader Accounts 

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Bardzo częste pytania!)

To funkcja dla sytuacji, gdy chcesz udostępnić dane komuś, **kto nie ma konta w Snowflake**.

- **Kto tworzy?** Dostawca (Provider).
    
- **Kto płaci?** **Dostawca** płaci za wszystko (Storage + Compute używane przez Reader Account). Zużycie kredytów przez Reader Account pojawia się na rachunku Providera jako osobna pozycja. Jako Provider masz pełny wgląd w to, co robią użytkownicy na Reader Account (przez widok `READER_ACCOUNT_USAGE`).

- **Relacja:** Jeden Provider (Dostawca) może stworzyć wiele Reader Accounts.
    
- **Ograniczenia:** Reader Account może tylko odczytywać dane udostępnione przez swojego "rodzica" (Providera). Nie może ładować własnych danych. reader accounts Mogą tworzyć własne magazyny (Virtual Warehouses), ale ich koszt zawsze obciąża Providera.
    
- **Zastosowanie:** Udostępnianie raportów klientom zewnętrznym (B2B). Użytkownicy Reader Account logują się do Snowflake przez dedykowany URL

* Tylko **`ACCOUNTADMIN`** (lub rola z uprawnieniem `CREATE ACCOUNT`) może stworzyć Reader Account. x


Jeśli padnie pytanie: _"Czy Konsument może zaktualizować (UPDATE) dane udostępnione przez Dostawcę?"_ **Odpowiedź zawsze brzmi: NIE.** Współdzielenie danych w Snowflake jest zawsze **Read-Only**.



### Udostępnianie z wielu baz danych

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Bardzo częsta "pułapka")

To jest kluczowy detal techniczny: **Jeden Share może zawierać obiekty tylko z JEDNEJ bazy danych.**

- **Problem:** Co jeśli chcesz udostępnić Tabelę A z Bazy 1 i Tabelę B z Bazy 2 w jednym udziale?
    
- **Rozwiązanie:** Musisz stworzyć **Secure View** w jednej z tych baz (lub w trzeciej, dedykowanej bazie do sharingu), który łączy te dane (`JOIN`), i to ten widok dodać do Share. W Shares ZAWSZE uzywamy secure views 


### Regiony - co zrobic jak sa rozne regiony providera i consumera

Standardowy "Direct Share" działa tylko wtedy, gdy Provider i Consumer są w tym samym **regionie** i u tego samego **dostawcy chmury** (np. obaj na AWS Frankfurt).

Jeśli chcesz udostępnić dane do innego regionu albo innej chmury (np. z AWS Frankfurt do AWS New York) lub innej chmury (z AWS do Azure):

1. Musisz użyć **Database Replication**.
2. Najpierw replikujesz bazę danych do regionu konsumenta.
3. Dopiero tam tworzysz Share z tej zreplikowanej bazy. (Wyjątkiem są Listings z funkcją Auto-fulfillment– one robią to automatycznie "pod maską").


### Re-sharing i "Override Restrictions"

**Ważność na egzaminie:** ⭐⭐⭐⭐ (Pułapka!)

To jeden z ulubionych tematów egzaminatorów: **Czy można podać dalej dane, które ktoś nam udostępnił?**

- **Zasada ogólna:** **NIE.** Nie możesz dodać do swojego Share'a tabeli, która pochodzi z bazy danych udostępnionej Ci przez kogoś innego.
    
- **Dlaczego?** Snowflake chroni łańcuch własności danych.
    
- **Override (Wyjątek):** Możesz stworzyć **Secure View** na bazie udostępnionych danych i spróbować go udostępnić dalej, ale wymaga to specyficznych ustawień i jest zazwyczaj ograniczone do udostępniania wewnątrz tej samej organizacji (np. między kontami tej samej firmy).

![[Zrzut ekranu 2026-04-29 o 00.50.19.png]]