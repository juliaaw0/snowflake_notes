
[[COSTS, BILLING]]
### Trzy filary kosztów Snowflake

Na całkowity koszt Snowflake składają się trzy główne komponenty (plus czwarty, rzadszy):

1. **Compute (Obliczenia):** Największa część kosztów. Zużycie kredytów przez Virtual Warehouses i usługi Serverless.
    
2. **Storage (Przechowywanie):** Miesięczny koszt za przetrzymywanie danych w chmurze.
    
3. **Cloud Services (Usługi chmurowe):** "Mózg" Snowflake (metadane, autoryzacja). Często darmowy, ale ma swoje limity.
    
4. **Data Transfer (Transfer danych):** Koszt przesyłania danych między regionami lub chmurami.




### Kontrola dostępu do danych o kosztach (Access Control)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Domyślnie dane o kosztach są bardzo restrykcyjnie chronione. Nie każdy użytkownik powinien wiedzieć, ile firma wydaje na zapytania.

- **ACCOUNTADMIN:** Ma pełny wgląd we wszystkie koszty konta (Compute, Storage, Transfer).
    
- **ORGADMIN:** Widzi koszty skonsolidowane dla wszystkich kont w całej organizacji.

* ***`MONITOR USAGE`:** Globalne uprawnienie pozwalające roli na podgląd kosztów.
    
- **Custom Roles (Role własne):** Aby zwykły użytkownik mógł widzieć koszty, musi otrzymać specjalne uprawnienia:
    
    - Uprawnienie **`MONITOR`** na poziomie konkretnego Virtual Warehouse.
        
    - Dostęp do bazy danych **`SNOWFLAKE`** (szczególnie schematów `ACCOUNT_USAGE` i `ORGANIZATION_USAGE`), co wymaga nadania uprawnienia `IMPORTED PRIVILEGES`.




### Compute Costs (Koszty obliczeniowe)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Tutaj płacisz za **Kredyty Snowflake**.

1. **Virtual Warehouse Usage:** Kredyty spalane przez Twoje "silniki".Płacisz tylko wtedy, gdy magazyn jest uruchomiony. Rozmiary (XS, S, M, L...) podwajają liczbę kredytów na godzinę (1, 2, 4, 8...).Naliczanie jest sekundowe (minimalnie 60 sekund przy każdym uruchomieniu).
        
- **Serverless Features:** Koszty usług zarządzanych przez Snowflake - Niektóre funkcje nie potrzebują Twojego Warehouse'u, bo działają na zasobach Snowflake (np. **Snowpipe, Automatic Clustering, Materialized Views, Search Optimization**). One również zużywają kredyty, które widzisz na osobnym zestawieniu - nie widzisz ich jako "Warehouse" na liście – mają swoje własne kategorie
    
**Kluczowy widok SQL:** `WAREHOUSE_METERING_HISTORY`.




### Storage Costs (Koszty przechowywania)

**Ważność na egzaminie:** ⭐⭐⭐⭐

Snowflake pobiera opłatę za **średnią dzienną ilość danych** (po kompresji).

- **Co wchodzi w skład?** Dane aktywne + Time Travel + Fail-safe.
    
- **Cena:** Zależy od Twojej umowy (On-demand vs. Pre-purchased/Capacity). Cena jest zbliżona do natywnych kosztów S3/Azure Blob, ale Snowflake dolicza niewielką marżę za zarządzanie tymi danymi.
    
- **Rodzaj konta:** Konta typu _Reader Accounts_ (dla klientów) też generują koszty storage'u na koncie dostawcy.

Koszty storage'u nie zmieniają się tak gwałtownie jak compute, ale Snowflake pozwala je rozbić na:

- **Database Storage:** Dane w tabelach.
    
- **Stage Storage:** Pliki czekające na załadowanie (wewnętrzne stage).
    
- **Fail-safe Storage:** Dane chronione przez 7 dni po upływie Time Travel.
    

**Kluczowy widok SQL:** `STORAGE_USAGE`.






### Cloud Services – Zasada 10%

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (To jest ulubione pytanie o koszty!)

Cloud Services to warstwa, która działa zawsze (metadane, optymalizator).

- **Zasada:** Snowflake nie pobiera opłat za Cloud Services, dopóki ich zużycie nie przekroczy **10% dziennego zużycia Compute**.
    
- **Przykład:** Jeśli Twoje Warehouse'y zużyły dziś 100 kredytów, a Cloud Services zużyły 12 kredytów, to płacisz za 100 (Compute) + 2 (nadwyżka Cloud Services). Razem 102 kredyty. Jeśli zużyłyby 5, płacisz tylko 100.

Jak już wiesz, Cloud Services są darmowe do 10% dziennego zużycia Compute. Jeśli jednak Twoja firma płaci za tę warstwę, dokumentacja sugeruje optymalizację:

- **Batching DML:** Zamiast robić 1000 małych `INSERT`, zrób jeden duży. Każde zapytanie (nawet małe) wymaga pracy Cloud Services (kompilacja, autoryzacja).
    
- **Unikanie zbędnych `SHOW`:** Komendy typu `SHOW TABLES` lub `SHOW SCHEMAS` obciążają tylko metadane (Cloud Services). Jeśli używasz ich w pętli w skryptach, generujesz koszty.
    
- **Tuning zapytań:** Im lepszy Pruning (przycinanie partycji), tym mniej metadanych musi przetworzyć optymalizator.



### Data Transfer (Transfer danych)

**Ważność na egzaminie:** ⭐⭐⭐

- Wrzucanie danych do Snowflake (**Ingress**) jest **zawsze darmowe**.
    
- Wypychanie danych na zewnątrz lub przesyłanie ich między regionami (np. Replikacja) jest **płatne** (według stawek dostawcy chmury).



| **Co chcesz sprawdzić?** | **Gdzie szukać (SQL)?**  | **Ważna uwaga**                         |
| ------------------------ | ------------------------ | --------------------------------------- |
| **Ogólne kredyty**       | `METERING_DAILY_HISTORY` | Zagregowane dane dzień po dniu.         |
| **Koszty zapytań**       | `QUERY_HISTORY`          | Pozwala znaleźć "najdroższe" zapytania. |
| **Koszty Organizacji**   | `ORGANIZATION_USAGE`     | Dane mogą mieć do 24h opóźnienia.       |
| **Koszty Konta**         | `ACCOUNT_USAGE`          | Opóźnienie od 45 min do 3 godzin.       |


### Attributing Costs (Kto za to płaci?)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

W dużych firmach faktura jest jedna, ale działy (Marketing, Sprzedaż, IT) muszą się rozliczyć wewnętrznie. Snowflake daje Ci do tego trzy główne narzędzia:

#### A. Osobne Virtual Warehouses (Najlepsza praktyka)

Najprostszy sposób. Tworzysz `WH_MARKETING` i `WH_FINANCE`. Ponieważ każdy Warehouse ma własne statystyki zużycia, na koniec miesiąca dokładnie widzisz, ile kredytów spalił dany dział. Mozna tez tagować warehousy tagami

#### B. Query Tags (Tagowanie zapytań)

Parametr sesyjny `QUERY_TAG`.

- Możesz ustawić go tak: `ALTER SESSION SET QUERY_TAG = 'Raport_Roczny_2024';`.
    
- Wszystkie zapytania wysłane w tej sesji będą miały ten „stempel”. Potem w widoku `QUERY_HISTORY` możesz łatwo pogrupować koszty po tych tagach.




### Resource Monitors (Hamulec bezpieczeństwa)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Resource Monitor (RM) to narzędzie służące do monitorowania zużycia kredytów i **automatycznego reagowania**, gdy limity zostaną przekroczone.

- **Co monitorują?** Kredyty zużywane przez **Virtual Warehouses** oraz (pośrednio) Cloud Services.
    
- **Czego NIE monitorują?** Funkcji typu Serverless (np. Snowpipe, Automatic Clustering) – do nich używamy wspomnianych wcześniej **Budgets**.
    
- **Poziomy (Hierarchy):**
    
    - **Account Level:** Jeden monitor pilnuje wszystkich magazynów na koncie.
        
    - **Warehouse Level:** Monitor pilnuje konkretnego magazynu lub grupy magazynów.

#### Akcje Resource Monitora (To musisz znać!)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Gdy zużycie osiągnie określony procent (np. 80%, 100%), RM może wykonać akcje:

|Akcja|Co się dzieje?|
|---|---|
|**Notify**|Wysyła e-mail do administratorów. Warehouse działa dalej.|
|**Suspend**|Czeka, aż trwające zapytania się skończą, a potem zawiesza magazyn. Nowe zapytania nie wystartują.|
|**Suspend Immediate**|**Zabija wszystkie trwające zapytania** i natychmiast zawiesza magazyn.|

Export to Sheets

> **Tip egzaminacyjny:** Pamiętaj, że jeśli RM jest ustawiony na poziomie Konta, akcja "Suspend" zawiesi **wszystkie** magazyny na tym koncie.


RM może działać w różnych cyklach:

- **Monthly (Domyślny):** Licznik zeruje się pierwszego dnia każdego miesiąca.
    
- **Daily, Weekly, Yearly:** Inne interwały czasowe.
    
- **Never-ending:** Licznik nigdy się nie zeruje (kumuluje zużycie).


Uprawnienie do tworzenia RM ma domyślnie tylko `ACCOUNTADMIN`.
![[Zrzut ekranu 2026-05-5 o 22.01.08.png]]

From a privilege perspective, only Account Administrators (users with ACCOUNTADMIN role) can create new resource monitors. However, account administrators can grant privileges to an existing resource monitor to allow other users to view and modify the resource monitor configuration. The MONITOR and MODIFY privileges on a resource monitor allow other users to view and modify a specific resource monitor.




### Czym jest Snowflake Budget?

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Budget to obiekt typu **Serverless**, który monitoruje zużycie kredytów w ujęciu miesięcznym.

- **Kluczowa różnica:** W przeciwieństwie do Resource Monitors, Budgets śledzą **wszystko** – nie tylko Virtual Warehouses, ale także usługi **Serverless** (Snowpipe, Automatic Clustering, Materialized Views itp.).
    
- **Działanie:** Budgets co wyznaczony czas (zazwyczaj raz dziennie) sprawdzają Twoje wydatki i na podstawie trendu przewidują, czy do końca miesiąca zmieścisz się w limicie.

| **Cecha**             | **Resource Monitor**                                                                                | **Budget**                                      |
| --------------------- | --------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| **Co monitoruje?**    | ![[Zrzut ekranu 2026-05-5 o 22.00.55.png]]Tylko Virtual Warehouses. Cloud services i serverless nie | **Wszystko** (Warehouses + Serverless).         |
| **Akcja "Hard Kill"** | **TAK.** Może natychmiast uśpić Warehouse.                                                          | NIE (domyślnie tylko powiadamia).               |
| **Analiza trendów**   | Brak.                                                                                               | **TAK.** Przewiduje zużycie na koniec miesiąca. |
| **Poziom**            | Konto (account level) lub (warehouse level) - dowolna grupa zasobow, np wiele lub 1 warehouse       | Konto lub dowolna grupa zasobów.                |





### SCHEMY W BAZIE SNOWFLAKE DO MONITOROWANIA
![[Zrzut ekranu 2026-05-5 o 22.08.25.png]]
Głównie informacje historyczne


![[Zrzut ekranu 2026-05-5 o 22.11.26.png]]
This schema serves as a database dictionary. It contains metadata about the objects created in that database, as well as views containing information on account level items such as roles, databases and warehouses. Some of the data provided by the information schema is also provided by the account usage schema.
Typowy czas przechowywania danych do 14 dni (ale range jest od 7 dni do 6 msc)

Większość baz danych, gdy zadasz im ciężkie zapytanie, po prostu "mieli" je długo, aż w końcu wypluje wynik. Snowflake w przypadku `INFORMATION_SCHEMA` mówi: **"Nie, stop! To zapytanie jest zbyt szerokie, nie będę go nawet próbował dokończyć"**.

Dzieje się tak, ponieważ `INFORMATION_SCHEMA` to widoki **generowane "na żywo"** bezpośrednio z metadanych w warstwie **Cloud Services**.

- Cloud Services to "mózg" Snowflake'a, który obsługuje wszystkich użytkowników naraz.
    
- Jeśli dopuściłbyś do sytuacji, w której ktoś robi gigantyczny skan wszystkich kolumn we wszystkich tabelach w ogromnej bazie, mógłbyś "zapchać" ten mózg.
    
- Dlatego Snowflake narzuca **sztywne limity na objętość danych**, jakie te widoki mogą wygenerować w jednym zapytaniu.

![[Zrzut ekranu 2026-05-5 o 23.45.19.png]]



ACCOUNT_USAGE ma widoki m.in. login history - na biezaco logowanie wszystkich uzytkownikow
Login usage by user - logowanie ojedynczych uzytkownikow

![[Zrzut ekranu 2026-05-5 o 22.14.27.png]]

Once a month, Snowflake also releases a behavior change release. These changes to existing behaviors may affect customers who already use the service. Over two months, the new behavior is adopted by everyone. The behavior change is not enabled during the first month unless the customer opts in. The behavior modification is enabled automatically in the second month, but a customer can opt out.

Snowflake does not instantly deploy a new version to all Snowflake accounts; rather, customer accounts are moved into the new release over time in a phased manner. Day 1 (early access): Deployed for Enterprise edition (or higher) accounts that have elected for early access. You can enroll an Enterprise edition (or higher) account for early access by contacting Snowflake support. Day 1 or 2 (regular access): Deployment of all Snowflake accounts on the Standard edition. Day 2 (last): All remaining Enterprise edition (or higher) accounts are deployed. Between an early access deployment and a final deployment, a minimum of 24 hours must pass. This staged release strategy enables Snowflake to identify and address any software issues uncovered during early access. https://docs.snowflake.com/en/user-guide/intro-releases