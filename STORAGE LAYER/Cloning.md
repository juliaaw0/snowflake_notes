[[STORAGE LAYER]]

Snowflake nazywa tę funkcję **Zero-Copy Cloning**, co jest kluczem do zrozumienia, jak to działa i dlaczego jest tak genialne.

W tradycyjnych bazach danych, aby skopiować tabelę o rozmiarze 1 TB, musisz fizycznie skopiować dane (co trwa długo i kosztuje drugie tyle za przechowywanie).
W Snowflake, polecenie `CLONE`:
- **Nie kopiuje danych fizycznie.** Tworzy jedynie nową nazwę obiektu (metadane), która "wskazuje" na te same mikropartycje (pliki z danymi), co oryginał. Klonowanie to operacja tylko na metadanych.
- **Jest natychmiastowe.** Niezależnie od tego, czy tabela ma 10 wierszy czy 10 petabajtów, klonowanie trwa sekundy.
- **Jest darmowe (na początku).** Ponieważ nie powstają nowe kopie plików, nie płacisz za dodatkowy _Storage_.
- jest operacją na metadanych (konkretnie na mikropartycjach)
- jest wykonywane przez Cloud Services Layer

### Co się dzieje przy zmianie danych? (Copy-on-Write)
To ulubiony temat pytań na egzaminie: **"Skoro oba obiekty patrzą na te same dane, to czy zmiana w klonie zepsuje oryginał?"**
- **Odpowiedź brzmi: NIE.** Obiekty są od siebie niezależne.
- Jeśli w klonie zmienisz wiersz, Snowflake utworzy **nową mikropartycję** tylko dla tego zmienionego fragmentu danych.
- Oryginalna tabela nadal patrzy na stare partycje, a klon patrzy na mieszankę: stare partycje (niezmienione) + nowe partycje (twoje zmiany).
- Dopiero od tego momentu zaczynasz płacić za dodatkowy Storage (tylko za te nowe, zmienione partycje).

### Co da i nie da sie klonować
Na egzaminie mogą Cię zapytać, co da się sklonować.
- **Databases** (Bazy danych)
- **Schemas** (Schematy)
- **Tables** (Tabele)
 * file format, sequences, tasks
 * external stages
 * pipes do internal objects
**Ważne:** Kiedy klonujesz bazę danych, Snowflake automatycznie klonuje wszystkie schematy i tabele wewnątrz niej.

To jest klasyczny "haczyk". Niektóre obiekty **nie przechodzą** do klona:
- **Internal Stages** -- **Internal Stages** - Generalnie zasada jest że "nie mogą być klonowane'. Wynika to z faktu, że przechowują one fizyczne pliki na wewnętrznym systemie plików Snowflake'a, a mechanizm _Zero-copy cloning_ jest zaprojektowany głównie dla metadanych tabel (mikro-partycji), a nie dla surowych plików wewnątrz stage'a.  W praktycw: defaultowo komendą CLONE internal stage nie są kopiowane. Takim jakby wyjątkiem jest table stage, bo nowa sklonowana tabela dostanie table stage - ale jest on zawsze pusty. Żeby sklonować PUSTE internal stage, musisz jawnie dopisać frazę: `CREATE DATABASE klon CLONE oryginał INCLUDE INTERNAL STAGES. Ale i tak nigdy nie zostanie skopiowana ich zawartość bo sa to pliki a nie metadane. 
- **Temporary Tables** (Tabele tymczasowe) – nie mogą być klonowane do stałych tabel.
- **Shares** (Udostępnienia danych).
- external tables 
- pipes do external objects
- **Storage Integrations** (integracje z magazynem chmurowym).
- Cloning does not copy the load metadata; therefore, any files previously loaded in the source table can be reloaded into the cloned table without any issues.

### Uprawnienia (Privileges)

Kolejny pewniak egzaminacyjny:
- **Uprawnienia do klonowania:** Aby sklonować obiekt, musisz mieć uprawnienie SELECT do obiektu źródłowego (i bazy/schematu) oraz uprawnienie `CREATE [OBJECT]` w miejscu docelowym.
- **Uprawnienia w klonie:** 

	Zasada ogólna: Klon jest "czysty": Jeśli wykonasz `CREATE TABLE tabela_B CLONE tabela_A`, to domyślnie **żadne uprawnienia nie przechodzą**. Jeśli rola `ANALYST` miała dostęp do `tabela_A`, to **nie będzie** miała dostępu do `tabela_B`, dopóki jej go nie nadasz.

	Parametr COPY GRANTS: Dla konkretnych obiektów (takich jak **Tables** czy **Views**) Snowflake pozwala dopisać magiczną frazę: CREATE TABLE tabela_B CLONE tabela_A COPY GRANTS. Wtedy wszystkie uprawnienia (poza `OWNERSHIP`) zostaną skopiowane. **Na egzaminie:** Pamiętaj, że `OWNERSHIP` (własność) zawsze przypada roli, która wykonała klonowanie, nawet jeśli użyjesz `COPY GRANTS`.

	Klonowanie schemy/bazy (wyzszy lvl niz tabela): sama baza-klon czy schema-klon nie dostaje tych samych grantów co baza oryginal. Jest "czysta". Ale wszystkie nizsze obiekty np tabele i widoki w niej, ZACHOWUJA granty. **Dlaczego tak jest?** Snowflake zakłada, że skoro klonujesz cały schemat lub bazę, to chcesz zachować strukturę bezpieczeństwa **wewnątrz** tego środowiska, ale chcesz kontrolować, kto w ogóle ma do tego środowiska (kontenera) wejście.

	**Pipes:** Rola, która klonuje, zawsze staje się właścicielem (`OWNERSHIP`)

### Cloning i Time Travel

Możesz sklonować tabelę tak, jak wyglądała np. 2 dni temu, łącząc polecenie `CLONE` z `AT` lub `BEFORE`: `CREATE TABLE clone_table CLONE original_table AT(TIMESTAMP => ...);` To potężne narzędzie do odzyskiwania danych po błędach.

### Cloning i rozne cechy obiektow

CLUSTERING KEY
When a table with a clustering key is cloned, the new table is created with a clustering key. By default, [Automatic Clustering](https://docs.snowflake.com/en/user-guide/tables-auto-reclustering) is suspended for the new table.

FOREIGN KEY
Tabela z foreign key jest klonowana:
 - jesli clone jest na calej bazie/schemie i tabela do ktorej pointuje foreign key jest w tej bazie/schemie, to foreign key pointuje tez do klona
 - jesli tabela do ktorej pointuje foreign key nie jest w klonowanej bazie/schemie (albo clone jest po prostu na tabeli) to foreign ket pointuje do oryginalu

STAGES
**External:**
When cloning a database or schema: external named stages that were present in the source when the cloning operation started are cloned. 
You can also clone external named stages individually. An external stage references a bucket or container in external cloud storage; cloning an external stage has no impact on the referenced cloud storage.

**Table stages:**
Każda tabela w Snowflake ma swój własny „table stage” (do którego możesz wgrywać pliki przez `PUT`). **Co się dzieje przy klonie:** Tabela w nowym miejscu (klon) dostaje swój własny, nowy table stage. **Haczyk egzaminacyjny:** Same **pliki**, które leżały w stage'u oryginalnej tabeli, **NIE są kopiowane**. **Wynik:** Masz sklonowaną tabelę z danymi, ale jej stage jest całkowicie pusty.

**Internal stages: Tylko na życzenie**
**Domyślnie:** Nie są klonowane.  **Jak je sklonować:** Musisz jawnie dopisać frazę: `CREATE DATABASE klon CLONE oryginał INCLUDE INTERNAL STAGES;`.

PIPES
When a database or schema is cloned, any pipes in the source container that reference an internal (that is, Snowflake) stage are not cloned. However, any pipes that reference an external stage are cloned. When you clone a database or schema that contains any pipes through a CREATE .. CLONE command, the role that creates the clone takes ownership of the cloned pipe.

Po stworzeniu bazy/schemy, jesli w niej znajdowal sie pipe. Problem polega na tym, że Pipe to w zasadzie "instrukcja", która mówi: _"Zabierz dane ze Stage X i wrzuć je do Tabeli Y"_. Pytanie brzmi: do której tabeli Pipe ma wrzucać dane **po tym, jak zostanie sklonowany**? Do starej (oryginalnej), czy do nowej (sklonowanej)? Snowflake rozstrzyga to na podstawie tego, jak zapisałaś nazwę tabeli w definicji Pipe'a (`COPY INTO ...`).
- Nazwa w pełni kwalifikowana (Fully Qualified) - COPY INTO BAZA.SCHEMAT.TABELA podane w kodzie pipe'a na sztywno. Wtedy Sklonowany Pipe **nadal wskazuje na oryginalną tabelę** w oryginalnej bazie/schemacie.
- Nazwa niepełna (Not Fully Qualified) - COPY INTO TABELA. Wtedy snowflake odniesie sie do nowej tabeli w nowym schemacie/bazie. 

SEARCH OPTIMIZATION SERVICE
klonowanie "kopiuje" nie tylko tabelę, ale i jej strukturę pomocniczą (tzw. _search access path_)
Kiedy klonujesz tabelę, która ma włączone SOS: Sama "ścieżka dostępu" (access path), która odpowiada za przyspieszenie wyszukiwania, również jest klonowana metodą **Zero-Copy**. Czyli początkowo nie płacisz za dodatkowe miejsce (storage) na tę strukturę w klonie.

**Problem "nieaktualnego klona" (Haczyk kosztowy!)**:
możesz zapłacić za utrzymanie SOS w klonie, **nawet jeśli nic w tej sklonowanej tabeli nie zmieniłaś**. **Jak to możliwe?** SOS to usługa działająca w tle, która stale "indeksuje" tabelę bazową. Jeśli zrobisz klon w momencie, gdy SOS w oryginale nie skończył jeszcze przetwarzać wszystkich danych (indeks nie był w 100% gotowy), to sklonowana tabela "dziedziczy" ten nieukończony stan. Snowflake automatycznie uruchomi serwis utrzymania (Maintenance Service) dla Twojego klona, aby "dociągnąć" indeks do końca, co generuje**ukryte koszty Compute**

STREAMS
**Stream to "licznik zmian" (Change Tracking)** nałożony na tabelę.
Wyobraź sobie, że masz tabelę `KLIENCI`. Stream nie jest kopią tej tabeli, ale "zakładką", która obserwuje tę tabelę i zapisuje:
- Które wiersze zostały dodane (**INSERT**).
- Które wiersze zostały usunięte (**DELETE**).
- Jak wiersze wyglądały przed i po zmianie (**UPDATE**).
**Kluczowe cechy pod egzamin:** **To nie są dane:** Stream przechowuje tylko metadane (informacje o tym, co się zmieniło).

Currently, when a database or schema that contains source tables and streams is cloned, any unconsumed records in the streams (in the clone) are inaccessible. This behavior is consistent with [Time Travel](https://docs.snowflake.com/en/user-guide/data-time-travel) for tables. If a table is cloned, historical data for the table clone begins at the time/point when the clone was created.

Jeśli masz Stream w oryginalnej bazie, który "widzi" 10 nowych wierszy, i zrobisz klon tej bazy, to sklonowany Stream w nowym miejscu będzie **pusty**.


TASKS
When a database or schema that contains tasks is cloned, the tasks in the clone are suspended by default.

ALERTS
When a database or schema that contains alerts is cloned, the alerts in the clone are [suspended](https://docs.snowflake.com/en/user-guide/alerts.html#label-alerts-suspend-resume) by default.

MASKING & ROW POLICY
Wyobraź sobie, że masz tabelę chronioną polityką maskowania (np. ukrywanie numerów PESEL).
- Cloning an individual policy object is not supported: **Polityki (Masking/Row Access) nie mogą być klonowane same w sobie** (nie zrobisz `CREATE POLICY ... CLONE`). Klonują się tylko "przy okazji" klonowania schematu lub bazy.
- **Klonowanie samej tabeli:** Klon "dziedziczy" przypisanie do tej samej polityki, co oryginał. Dane w klonie będą tak samo zamaskowane.
- **Klonowanie całego schematu (najważniejszy punkt):**
    - Jeśli tabela i polityka są w tym samym schemacie `PROD`, to po sklonowaniu do `DEV`, tabela w `DEV` będzie używać **sklonowanej polityki z `DEV`**.        
- **Referencje zewnętrzne (Foreign references):** Jeśli Twoja tabela w `PROD` używa polityki, która siedzi w dedykowanym schemacie `SECURITY`, to klon tabeli nadal będzie "dzwonił" do tego samego schematu `SECURITY`. Link się nie zmienia.

TAGS
Tagi służą do klasyfikacji danych (np. tag `Sensitive`).
Tagi są utrzymywane w klonach. Logika działa identycznie jak w politykach. Jeśli klonujesz bazę/schemat, a tag był w środku, klon obiektu będzie używał klona tagu. Jeśli tag był poza bazą (np. w centralnym schemacie GOVERNANCE), klon obiektu nadal wskazuje na ten centralny tag.

TAG BASED MASKING POLICIES
To sytuacja, gdzie polityka maskowania nie jest przypisana bezpośrednio do kolumny, ale do **tagu**, który jest na tej kolumnie. 

Jeśli wszystko (tabela, tag i masking policy) jest w jednym schemacie, klonowanie schematu "przepina" wszystko na nową wersję w nowym schemacie.

Jeśli tag jest w innym schemacie niż polityka, po sklonowaniu tabela może nadal podlegać polityce z oryginału (źródła).

Jeśli tabela była chroniona tylko dlatego, że "dziedziczyła" tag z całego schematu (czyli tag nalezy do schematu/bazy, a nie samej tabeli - tabela miala ochrone tylko z powodu bycia w danej schemie/bazie), to po przeniesieniu/sklonowaniu jej do innego schematu **traci tę ochronę**, chyba że nowy schemat też ma taką politykę.

DIFFERENCIAL PRIVACY
Differential Privacy to metoda dodawania precyzyjnie wyliczonego **szumu matematycznego** do wyników zapytań. Dzięki temu statystyki ogólne (np. „ile średnio zarabiają pracownicy”) pozostają **prawdziwe**, a dane pojedynczych osób stają się **niemożliwe do wyodrębnienia**.

Polityka prywatności (Differential Privacy) zawsze chroni sklonowany obiekt, ale pytanie brzmi, czy ten klon używa „starej” polityki, czy „nowej” (sklonowanej). - czyli nie ma sytuacji że po sklonowaniu obiekt nie ma żadnej differencial policy. 
- jak kopiujemy samą tabele (nie cala scheme itp) - to differencial policy nie jest klonowane, nowa tabela pointuje caly czas do starej polityki
- jak kopiujemy całą scheme/baze (czyli i tabele i polityke ktora jest w schemie) to nowa tabela bedzie pointować do nowej polityki
* jak tabela i polityka znajdują się w różnych miejscach, to klonowana jest tylko tabela i nowa tabela pointuje do starej polityki

DATABASE ROLES
You can clone a database role using the CREATE DATABASE ROLE … CLONE command if the database role doesn’t already exist in the target database.

### Cloning i DDL/DML

DDL
zrobienie operacji DDL na obiekcie bazowym (tabeli,schemie itp) podczas gdy jest klonowana moze spowodowac bledy i wywalic error. Snowflake nie ma jakichs specjalnych zabezpieczen typu zapamietywanie nazw tabeli przed i po sklonowaniu. Zaleca sie nie robic DDL jak cos jest klonowane. 

DML:
Chociaż klonowanie jest „natychmiastowe” z punktu widzenia metadanych, to przy ogromnych tabelach (PB danych) system musi „podpiąć” pod klon tysiące mikropartycji. Trwa to zazwyczaj sekundy, ale przy gigantycznych obiektach może zająć nieco więcej czasu.

**Co się dzieje w tym czasie?**

1. Rozpoczynasz klonowanie tabeli.
2. W tym samym momencie inny proces wykonuje `UPDATE` lub `DELETE` na tabeli źródłowej (DML).
3. Snowflake chce, aby klon był **spójny** – musi on zawierać dane dokładnie z sekundy, w której kliknęłaś „Clone”. Normalnie w tej sytuacji clone skorzysta z danych zapisanych w historii time travel zeby miec spojne dane z jednego punktu w czasie. ALE:

	Jeśli ustawisz retencję danych na **0 dni**, wyłączasz Time Travel.
- W momencie, gdy DML (np. `DELETE`) zmienia dane w tabeli źródłowej, stara wersja tych danych jest **natychmiast usuwana** (purged), bo system nie ma nakazu jej trzymania.
- Jeśli proces klonowania akurat w tej milisekundzie potrzebował tej „starej” wersji, aby dokończyć tworzenie spójnego klona – **nie znajdzie jej**.
- **Wynik:** Błąd `Data is not available`.

	Jak zaradzić: albo powstrzymac sie przed operacjami DML w momencie klonowania, albo przed sklonowaniem ustawic time travel = 1


### Cloning i error tables

When a base table with an associated error table is cloned, the behavior is as follows:
    - The base table’s schema and content are cloned.
    - The error table’s content isn’t cloned
    - The cloned base table has the ERROR_LOGGING property turned on, which implicitly creates an empty error table for it.
