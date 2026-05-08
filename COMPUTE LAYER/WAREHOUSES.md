[[COMPUTE LAYER]]

A virtual warehouse provides CPU, memory, and temporary storage resources to process queries and run data load jobs. Each node in a virtual warehouse computes cluster has its own memory, computing resources, and local cache stored on a solid-state disk.

Domyślnie uprawnienia do warehousow CREATE WAREHOUSE przyznany jest tylko rolom o najwyższych uprawnieniach:

- SYSADMIN
- ACCOUNTADMIN

Ale można je przyznac tez innym rolom. Inne systemowe role jak SECURITYADMIN czy USERADMIN domyślnie nie mają tego przywileju – służą głównie do zarządzania dostępem, rolami i użytkownikami, a nie zasobami compute.

aby inna (niestandardowa) rola mogła tworzyć warehouses, musisz jej nadać przywilej:

GRANT CREATE WAREHOUSE ON ACCOUNT TO ROLE nazwa_roli;

CREATE WAREHOUSE ETL_VWH WITH

WAREHOUSE_SIZE = 'SMALL'

WAREHOUSE_TYPE = 'STANDARD'

AUTO_SUSPEND = 300

AUTO_RESUME = TRUE;

A virtual warehouse in Snowflake is typically a multi node compute cluster. A virtual warehouse provides resources such as CPU, RAM memory and temporary storage, which is used to process queries and data load jobs. Each node in a compute cluster is generally a low cost virtual machine instance. Multiple virtual warehouses can be created for a given snowflake account, but it is worth noting that each of them accessed the same shared data, hence the term multi clustered shared data.


Można suspendowac warehousy - suspended warehouse nie generuje zadnych kosztow i nie pobiera credits. Defaultowo  Jeśli klikniemy żeby suspendowac warehouse, a on obecnie przetwarza jakies query, najpierw je skonczy, a dopiero potem się zawiesi, nie przerwie przetwarzania. Żeby zawiesic go od razu, bez czekania na zakonczenie, można zrobic forced suspend z poziomu resource monitors. Auto suspend i auto resume sa defaultowo wlaczone.

Można ustawic auto resume, co spowoduje ze warehouse sam się podniesie jak jakies query będzie executed.

Scaling: Warehousy mogą być resizowane (scale up, scale down). Jak robimy scale up, kolejne compute nodes sa dodawane do warehousu. Query które już były executed zanim zwiekszymy rozmiar warehousu nie będą uzywaly dodatkowych nodow. Tylko nowe query uruchomione po powiekszeniu.

![[Zrzut ekranu 2026-05-4 o 21.35.29.png]]
![[Zrzut ekranu 2026-05-4 o 21.38.00.png]]

Each virtual warehouse has its own cache and can cache query results in local memory and ssd disks. Virtual warehouse cache is dropped when the warehouse is suspended.  Suppose a future query executing on the same virtual warehouse requires the same data. In that case, the virtual warehouse can fetch the results from the cache instead of hitting the physical storage layer. Therefore, improving the query performance. 

 The size of the cache is determined by the compute resources in the warehouse (that is, the larger the warehouse and, therefore, more compute resources in the warehouse, the larger the cache). cache is dropped when the warehouse is suspended.

Decreasing the size of a running warehouse removes compute resources from the warehouse. When the computer resources are removed, the cache associated with those resources is dropped, which can impact performance in the same way that suspending the warehouse can impact performance after it is resumed.

**WIECEJ O local disk CACHE**

[[Local Disk Cache]] (warehouse-level) - to jest ten cache o którym jest napisane wyzej w paragrafie

🔹 Co to jest? = Cache danych na lokalnym dysku konkretnego warehouse.

	🔹 Kluczowe cechy (EGZAMIN 🔥)
	
	⚠️ ZALEŻNY od warehouse
	
	⚠️ PRZEPADA przy SUSPEND warehouse
	
	⚠️ Tylko dla tego konkretnego warehouse
	
	🚀 Przyspiesza skany danych (nie wyniki zapytań)

AUTO_SUSPEND = mniejszy koszt, ale gorszy local cache


Snowflake bardzo lubi pytania typu:

„dlaczego pierwsze zapytanie po przerwie jest wolniejsze?”

👉 Bo local disk cache był pusty.



Warehouses are required for queries, as well as all DML operations, including loading data into tables. In addition to being defined by its type as either Standard (default, optimized for SQL, BI, ETL, DML) or Snowpark-optimized (optimized for memory-hravy jobs, obliczen, ma inne proporcje CPU do RAMu (wiecej pamieci), dobry do ML i obrobki danych, nie wymagany ale zalecany do snowpark), a warehouse is defined by its size.

A default warehouse can be specified when creating or modifying the user

When a user connects to Snowflake and start a session, Snowflake determines the default warehouse for the session in the following order: Default warehouse for the user, overridden by Default warehouse in the configuration file for the client utility (SnowSQL, JDBC driver, etc.) overridden by » Default warehouse specified on the client command line or through the driver/connector parameters. default warehouse for a session can be changed at any time by executing the [USE WAREHOUSE](https://docs.snowflake.com/en/sql-reference/sql/use-warehouse) command.

✅ Wymaga warehouse

- DML (INSERT, UPDATE, DELETE, MERGE)
- CREATE TABLE AS SELECT (CTAS)
- INSERT … SELECT
- UPDATE … FROM
- Materialized Views (refresh)
- Streams + Tasks (task używa warehouse!)
- Cloning with data transformation
- COPY INTO

Wyjątek (ważny detal): Samo UPLOAD pliku do stage (PUT) nie używa warehouse

Ale: COPY INTO → tak

❌ NIE wymaga warehouse

- CREATE DATABASE
- CREATE SCHEMA
- CREATE TABLE (bez AS SELECT)
- DROP, ALTER (same metadane)
- SHOW, DESCRIBE
- Prosty COUNT(*) z metadata (jak omawiałyśmy)

Snowflake dzieli świat na:

- metadata-only operations → Cloud Services
- data-processing operations → Virtual Warehouse

👉 Jeśli dotykasz realnych danych → warehouse

👉 Jeśli manipulujesz opisem danych → nie uzywa się warehouse

## CO MUSISZ WIEDZIEĆ O WAREHOUSES (SnowPro Core)

1️⃣ Kto może tworzyć warehouse

- Wymagany privilege: CREATE WAREHOUSE na ACCOUNT
- Domyślnie mają:
- ✅ SYSADMIN
- ✅ ACCOUNTADMIN
- ❌ SECURITYADMIN, USERADMIN – nie
- Egzamin lubi pytania: „która rola?”

👉 Złota zasada egzaminowa

Obiekty (DB, schema, warehouse) = SYSADMIN

2️⃣ Warehouse ≠ storage
- Warehouse = compute
- Storage jest:
	- Wspólny (shared pomiedzy warehousami)
	- niezależny od warehouse
	- płatny osobno  

📌 Snowflake oddziela compute od storage

💡 Częste pytanie:

Czy wyłączenie warehouse powoduje utratę danych? → ❌ NIE

4️⃣ AUTO-SUSPEND i AUTO-RESUME (mega ważne)
-AUTO_SUSPEND = <sekundy>
- AUTO_RESUME = TRUE/FALSE

Egzamin:

- ✅ Najlepsza praktyka → krótki auto-suspend
- ❌ Brak auto-suspend = marnowanie credits

📌 Częste pytanie:

Co się stanie gdy query przyjdzie na suspended warehouse?
→ Auto-resume (jeśli włączone)

8️⃣ Uprawnienia do warehouse

USAGE - korzystać
OPERATE - suspend/resume
MODIFY - zmieniać parametry
OWNERSHIP - wszystko

📌 Częsty haczyk:
USAGE ≠ możliwość start/stop warehouse

9️⃣ Billing (często podchwytliwe)

- Liczone per sekundę
- Minimum 60 sekund
- Liczy się:
	- czas działania
	- rozmiar
	- liczba klastrów

❌ Nie płacisz za:
- warehouse w stanie SUSPENDED

3️⃣ Rozmiary warehouse

Znaj na pamięć:
- Każdy rozmiar ≈ 2× więcej mocy niż poprzedni
- Billing rośnie liniowo z rozmiarem
- Nie pytają o liczbę vCPU — tylko relacje
**Zasada podwojenia:** Każdy kolejny rozmiar (np. z S do M) **podwaja** zasoby obliczeniowe (CPU/RAM) i **podwaja** koszt kredytowy na godzinę.

Rozmiar warehouse ≠ liczba klastrów. To są dwa różne wymiary skalowania.



Warehouse = dwa niezależne „pokrętła”

🔼 POKRĘTŁO 1: ROZMIAR (SIZE)

- X-SMALL, SMALL, MEDIUM, …
- wpływa na:

- moc CPU
- ilość pamięci

- dotyczy pojedynczego klastra

👉 rozmiar = jak silny jest JEDEN klaster

↔️ POKRĘTŁO 2: LICZBA KLASTRÓW (MULTI-CLUSTER)

- MIN_CLUSTER_COUNT
- MAX_CLUSTER_COUNT
- wpływa na:

- ile zapytań równolegle

- dotyczy liczby kopii tego samego warehouse

👉 liczba klastrów = ile równoległych „silników” masz



TIPs z dokumentacji o warehousach:

Unless you are bulk loading a large number of files concurrently (i.e. hundreds or thousands of files), a smaller warehouse (Small, Medium, Large) is generally sufficient. Using a larger warehouse (X-Large, 2X-Large, etc.) will consume more credits and may not result in any performance increase.

The size of a warehouse can impact the amount of time required to execute queries. BUT Larger is not necessarily faster for small, basic queries.




## PYTANIA O WAREHOUSE Z EGZAMINU

❓ JAKIE PYTANIA MOGĄ PAŚĆ (przykłady)

🧪 Typ 1: Rola

Która rola może tworzyć warehouse?

- A) USERADMIN
- B) SECURITYADMIN
- C) SYSADMIN ✅
- D) PUBLIC

🧪 Typ 2: Concurrency

Użytkownicy skarżą się na kolejki zapytań. Co zrobić?

- A) Zwiększyć rozmiar warehouse
- B) Włączyć multi-cluster ✅
- C) Replikować dane
- D) Zwiększyć storage

🧪 Typ 3: Koszty

Jak zminimalizować koszty warehouse?

- A) Wyłączyć auto-resume
- B) Długi auto-suspend
- C) Krótki auto-suspend ✅
- D) Większy rozmiar

🧪 Typ 4: Multi-cluster

Co poprawia multi-cluster warehouse?

- A) Szybkość pojedynczego query
- B) Storage
- C) Concurrency ✅
- D) Cache




