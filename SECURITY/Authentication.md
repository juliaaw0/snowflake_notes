[[SECURITY]]

haslo/MFA (Duo)- uzytkownik ludzki
SSO (federated) - uzytkownik ludzki (przez zewnetrzne apki jak Okta)
key pair auth - skrypty
OAuth - programy jak looker/power bi



Musisz znać te 5 głównych sposobów logowania do Snowflake:

1. **Username / Password:** Standardowe logowanie ( Snowflake wspiera polityki haseł, np. długość, znaki specjalne). Haslo musi miec 8 znakow, 1 duzy, 1 maly, 1 cyfra
    
2. **MFA (Multi-Factor Authentication):** Zasilane przez **Duo Security**.
    
    - **Kluczowy fakt:** MFA jest dostępne w **każdej edycji** Snowflake i jest darmowe.
        
3. **Federated Authentication (SSO):** Oparte na standardzie **SAML 2.0**. Pozwala na logowanie przez zewnętrznych dostawców (IdP) jak Okta, Azure AD czy Google.
    
4. **Key Pair Authentication:** Używane głównie do połączeń **nie-ludzkich** (skrypty, narzędzia ETL, SnowSQL, Python Connector). Generujesz parę kluczy (publiczny/prywatny) zamiast używać hasła.
    
5. **OAuth:** Pozwala zewnętrznym aplikacjom (np. Tableau, Looker) na dostęp do Snowflake bez przesyłania hasła użytkownika. Snowflake wspiera zarówno własny (Snowflake OAuth), jak i zewnętrzny OAuth.



### Authentication Policies

Polityki uwierzytelniania pozwalają administratorom (zazwyczaj rola `SECURITYADMIN` lub `ACCOUNTADMIN`) na ścisłą kontrolę nad tym, jak użytkownicy wchodzą do systemu.

- **Co można kontrolować?**
    
    - **Allowed Authentication Methods:** Możesz wymusić, aby np. rola `ACCOUNTADMIN` mogła logować się **tylko przez SSO**, a narzędzie ETL **tylko przez Key Pair**.
        
    - **Allowed Client Types:** Możesz ograniczyć dostęp np. tylko do Snowsight (web UI) lub tylko do SnowSQL.
        
- **Hierarchia przypisania:**
    
    1. Poziom **Account** (domyślna dla wszystkich).
        
    2. Poziom **User** (nadpisuje politykę konta dla konkretnego użytkownika).
        

> **Ważne na egzamin:** Nie pomyl **Authentication Policies** z **Network Policies**.
> 
> - Authentication Policy = _Jak_ się logujesz (hasło, SSO, klucz).
>     
> - Network Policy = _Skąd_ się logujesz (lista adresów IP).




## MULTI FACTOR AUTHENTICATION
To najważniejszy punkt tej sekcji. Zapamiętaj te fakty:

- **Partner technologiczny:** Snowflake używa **Duo Security** jako dostawcy usługi MFA.
    
- **Dostępność:** Funkcja jest dostępna dla **wszystkich edycji** Snowflake (od Standard wzwyż) i nie wiąże się z dodatkowymi kosztami. All Snowflake client tools, including the web interface, SnowSQL, and the various connectors and drivers, support MFA.
    
- **Samodzielna rejestracja (Self-enrollment):** Użytkownicy sami rejestrują się w MFA przez interfejs Snowflake (Snowsight lub Classic Console). Nie musi tego robić administrator dla każdego z osobna. Admin moze komus wylaczyc MFA. 
    
- **Co jest potrzebne?** Zazwyczaj aplikacja Duo Mobile na smartfonie, ale wspierane są też klucze bezpieczeństwa, SMS-y i połączenia głosowe.
    

Egzaminatorzy lubią pytać o detale dotyczące wygody użytkownika:

- Przy logowaniu z MFA użytkownik może zaznaczyć opcję, która pozwoli mu nie wpisywać kodu przez pewien czas. Standardowo jest to **4 godziny** (można to zmienić w parametrach sesji). To ustawienie jest powiązane z konkretną przeglądarką/urządzeniem.




### FEDERATED AUTHENTICATION (SSO) – Detale

**Ważność na egzaminie:** ⭐⭐⭐⭐

Egzaminatorzy lubią pytać o techniczne podstawy SSO:

- Standard: **SAML 2.0**. Jeśli na egzaminie widzisz pytanie o **Federated Authentication** lub **SSO**, w odpowiedziach szukaj skrótu **SAML 2.0**(Security Assertion Markup Language). To jedyny standard wspierany przez Snowflake dla federacji tożsamości. "Snowflake supports federated authentication, allowing for single sign-on (SSO). Users authenticate using a SAML 2.0-compliant external identity provider (IdP). After IdP authentication, users can access Snowflake without logging in. Snowflake natively supports the majority of SAML 2.0 compliant identity providers, including Okta, ADFS, OneLogin, and Ping Identity PingOne."
    
- Jeśli SSO jest włączone, użytkownik może być automatycznie przekierowany do swojego panelu logowania Identy Providera (IdP) (np. Okta). Natomiast Snowflake pelni role Service provider - czyli odbiera info od identity providera. 

* Security Integration - To jest najważniejszy obiekt do zapamiętania. W Snowflake nie konfigurujesz SSO "w ustawieniach profilu". Tworzysz do tego specjalny obiekt: **Security Integration**. Służy on jako "most" między Snowflake a Twoim IdP. Możesz mieć **wiele security integrations (np. jeśli firma łączy się z dwoma różnymi systemami logowania).*


- **External Browser Auth:** Jeśli używasz SnowSQL (CLI), Snowflake może otworzyć przeglądarkę, abyś zalogował się przez SSO firmy – to też jest część Federated Auth.

* SSO jest dostepne dla wszystkich poziomow kont snowflake, nawet Standard.

* SCIM - Dzięki SCIM (System for Cross-domain Identity Management), gdy dodasz kogoś do grupy w Okta, ta informacja "leci" do Snowflake i konto tworzy się samo. Gdy pracownik odchodzi – konto w Snowflake zostaje automatycznie zablokowane.


## KEY PAIR AUTHENTICATION

To temat, który na egzaminie pojawia się niemal wyłącznie w jednym, konkretnym kontekście: **automatyzacji**. Jeśli widzisz pytanie o bezpieczne łączenie się skryptów, narzędzi ETL (np. Informatica, Matillion) lub konektorów (Python, Node.js) bez wpisywania hasła "na sztywno" – odpowiedzią jest **Key-pair authentication**.

To metoda uwierzytelniania przeznaczona dla **kont technicznych (service accounts)** i procesów automatycznych.

- **Zaleta:** Nie musisz przechowywać hasła w kodzie aplikacji czy pliku konfiguracyjnym.
    
- **Mechanizm:** Wykorzystuje parę kluczy kryptograficznych: **Klucz Prywatny** (zostaje u Ciebie, np. na serwerze ETL) i **Klucz Publiczny** (wędruje do Snowflake).

Snowflake pozwala na przypisanie **dwóch kluczy publicznych** do jednego użytkownika jednocześnie:

- `RSA_PUBLIC_KEY`
- `RSA_PUBLIC_KEY_2`
    
**Dlaczego to ważne?** Pozwala to na rotację kluczy **bez przerywania pracy skryptów**.

1. Dodajesz nowy klucz jako "dwójkę".
2. Aktualizujesz skrypt, by używał nowego klucza prywatnego.
3. Gdy wszystko działa, usuwasz stary klucz z pozycji "jeden".

Key pair authentication jest dostepne dla wszystkich poziomow kont snowflake, nawet Standard.




## AUTHORIZATION (OAuth i inne)
![[Zrzut ekranu 2026-05-1 o 20.00.50.png]]

### ROLES (Access Control) - BARDZO BARDZO WAZNE NA EGZ

Aby zrozumieć ten system, musisz widzieć relację między trzema elementami:

1. **Securable Object:** "Rzecz", którą chcemy chronić (tabela, widok, warehouse, baza danych).
    
2. **Privilege:** "Czynność", którą można wykonać na obiekcie (np. `SELECT` na tabeli, `USAGE` na warehouse).
    
3. **Role:** "Kontener" (nasz mundur), który zbiera uprawnienia i jest przypisywany do użytkownika.
    

**złota zasada:** W Snowflake nigdy nie nadajemy uprawnień bezpośrednio użytkownikowi. Zawsze: **Uprawnienie →Rola → Użytkownik.**


RBAC - role based access control - Nie dajemy privileges userom, tylko rolom. Dopiero potem role mozna dac userowi
Hierarchie ról buduje sie przez przypisanie jednej roli do innej
Defaultowe role dostepne w snowflake (nie mozna ich dropnąć):

PUBLIC - podstawowa rola, automatycznie dawana wszystkim

USERADMIN -  Może zarządzać tylko tymi użytkownikami i rolami, które **sam stworzył**. Nie może "wtrącać się" w uprawnienia do tabel stworzonych przez SysAdmina (bo nie ma przywileju MANAGE GRANTS)

SECURITYADMIN - Szeryf. To rola "polityczna". On nie buduje baz danych – on decyduje, kto ma do nich klucze. 

- **Co MOŻE robić:**
    
    - Zarządzać wszystkimi uprawnieniami na koncie (`MANAGE GRANTS`). Dzięki `MANAGE GRANTS` może modyfikować **każde uprawnienie na całym koncie**. Może podejść do dowolnego obiektu (np. tabeli finansowej stworzonej przez SysAdmina) i nadać do niej dostęp dowolnej roli, nawet jeśli nie jest właścicielem tej tabeli.
        
    - Dziedziczy uprawnienia roli `USERADMIN` – czyli tworzy i usuwa użytkowników oraz role.
        
    - Modyfikować dowolną rolę na koncie.
        
    - Tworzyć polityki bezpieczeństwa (Network Policies, Authentication Policies).
        
- **Czego NIE MOŻE (domyślnie):**
    
    - Stworzyć bazy danych (on tylko daje komuś prawo, by to zrobił).
        
    - Stworzyć Virtual Warehouse.
        
    - Oglądać danych w Twoich tabelach (chyba że sam sobie nada do nich uprawnienie, co zostanie odnotowane w logach audytowych).


SYSADMIN - inżynier. To rola, której używasz do "pracy z danymi". Jeśli coś zajmuje miejsce (storage) albo zużywa moc obliczeniową (compute), zajmuje się tym `SYSADMIN`.

- **Co MOŻE robić:**
    
    - Tworzyć bazy danych, schematy i tabele.
        
    - Tworzyć i modyfikować Virtual Warehouses.
        
    - Wykonywać zapytania na danych (jeśli ma do nich dostęp).
        
    - Zarządzać wszystkimi obiektami, które stworzył on sam lub role podległe mu w hierarchii.
        
- **Czego NIE MOŻE (domyślnie):**
    
    - Tworzyć nowych użytkowników.
        
    - Tworzyć nowych ról.
        
    - Resetować haseł kolegom.
        
    - Zarządzać politykami sieciowymi (Network Policies).

ACCOUNTADMIN - It is the account administrator role with full access rights as the most powerful role in the organisation. Access to this role should be rigorously managed (MFA musi byc). This role has all the privileges of sysadmin and securityadmin. 

Istnieje kilka "czerwonych przycisków", do których dostęp ma wyłącznie ta rola (lub role, którym on jawnie nada te prawa):

- **Zarządzanie billingiem:** Tylko on widzi szczegóły faktur, zużycie kredytów w skali całego konta i może podpinać karty płatnicze/umowy.
    
- **Zmiana parametrów konta:** Może zmieniać globalne ustawienia (np. `STATEMENT_TIMEOUT_IN_SECONDS` dla całego konta).
    
- **Zarządzanie kontami czytelnika (Reader Accounts):** Tylko on może je tworzyć i usuwać.
    
- **Data Sharing i Marketplace:** Może tworzyć udziały (Shares) i publikować dane w Snowflake Marketplace.
    
- **Replikacja i Failover:** Zarządzanie replikacją baz danych między regionami/chmurami.

* CZEGO NIE MOZE ACCOUNTADMIN - widziec wynikow query innych uzytkownikow for privacy reasons


Snowflake kładzie ogromny nacisk na to, jak korzystasz z tej roli:

1. **Zasada minimum:** Na koncie powinno być tylko **2-3 użytkowników** z rolą `ACCOUNTADMIN`. Bo jak jedyny accountadmin zgubi telefon z MFA, to nie ma kto mu cofnac tego MFA.
    
2. **Obowiązkowe MFA:** Każdy użytkownik z tą rolą **musi** mieć włączone uwierzytelnianie wieloskładnikowe.
    
3. **Nie używaj jej do pracy:** `ACCOUNTADMIN` nie służy do tworzenia tabel, ładowania danych czy pisania codziennych zapytań. Do tego używamy `SYSADMIN` lub ról funkcyjnych.



### Przykład "egzaminacyjny":

Wyobraź sobie, że rola `SYSADMIN` stworzyła bazę danych `HR_DB`.

1. **UserAdmin** widzi tę bazę, ale **nie może** nadać do niej uprawnień roli `ANALYST`, bo nie jest właścicielem tej bazy.
    
2. **SecurityAdmin** może wejść i powiedzieć: _"Nie obchodzi mnie, że SysAdmin to stworzył – jako Szeryf nadaję dostęp roli ANALYST"_. I to zadziała.
    

**Zapamiętaj pod egzamin:** Jeśli pytanie dotyczy **tylko** tworzenia użytkowników i ról – wystarczy `USERADMIN`. Jeśli w pytaniu pojawia się **zarządzanie uprawnieniami w skali całego konta** (global grant management) – poprawną odpowiedzią jest `SECURITYADMIN`.

![[Zrzut ekranu 2026-05-2 o 00.50.28.png]]

	
Mozna tez tworzyc custom role. All custom roles should be granted to the sysadmin role either directly or indirectly, so that the system administrator can control all objects in the snowflake system, regardless of which role owns those items.

Przywileje daje sie roli slowem GRANT a zabiera REVOKE

Normalnie w sesji używasz jednej roli (`PRIMARY ROLE`). Snowflake pozwala jednak włączyć **Secondary Roles**. Jeśli je włączysz (`USE SECONDARY ROLES ALL`), Twoja sesja sumuje uprawnienia **wszystkich ról**, które zostały Ci przypisane. Dzięki temu możesz robić `SELECT` na tabeli z Roli A i `JOIN` z tabelą z Roli B bez przełączania się między nimi.



### DAC 
DAC - discretionary access control -  „Kto zbudował, ten rządzi i decyduje”. Każdy obiekt w Snowflake musi mieć właściciela. **Właściciel** to rola, która stworzyła obiekt (ma uprawnienie `OWNERSHIP`). Tylko właściciel (lub rola wyżej w hierarchii) może nadawać uprawnienia do tego obiektu innym rolom. `OWNERSHIP` można przekazać innej roli za pomocą `GRANT OWNERSHIP`.


### OAuth
Key-Pair był dla skryptów, to **OAuth jest dla aplikacji**. **OAuth pozwala Ci powiedzieć: „Snowflake, daj tej aplikacji dostęp do moich danych w moim imieniu, ale bez pokazywania jej mojego hasła”.**

To jest najważniejszy podział. Snowflake oferuje dwa sposoby obsługi OAuth:

- **Snowflake OAuth (Native):** Snowflake sam występuje jako „serwer autoryzacji”. Sam generuje tokeny. Gdy chcesz szybko połączyć partnerów (Tableau, Looker) lub własną aplikację.
        
- **External OAuth:** Autoryzacją zarządza zewnętrzny system (np. Okta, Azure AD, PingFederate). Gdy Twoja firma ma rygorystyczne zasady i chce, aby to centralny system decydował, która aplikacja ma do czego dostęp.

Tak jak przy SSO (SAML), w OAuth również wszystko kręci się wokół obiektu **Security Integration**.

Musisz znać różnicę między tymi dwoma "przepustkami":

- **Access Token:** Krótkotrwały (np. 10 minut). Aplikacja wysyła go z każdym zapytaniem do Snowflake.
    
- **Refresh Token:** Długotrwały (np. 90 dni). Gdy Access Token wygaśnie, aplikacja używa Refresh Tokena, aby po cichu (bez pytania Cię o hasło) dostać nowy Access Token.
    
- **Single-use Refresh Tokens:** Snowflake wspiera funkcję, gdzie każde użycie Refresh Tokena unieważnia stary i generuje nowy. To potężne zabezpieczenie przed kradzieżą tokenów.


### Inne metody:
WIF - Workload Identity Federation - metoda uwierzytelniania **bez haseł i kluczy** dla usług chmurowych (Cloud Workloads).