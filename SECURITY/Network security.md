[[SECURITY]]
![[Zrzut ekranu 2026-05-2 o 00.55.14.png]]
At the network level, Snowflake encrypts all communication by default using TLS 1.2. The security is further enhanced through network policies through which specific IP addresses may be allowed to connect and others may be blocked. Additionally, Snowflake supports private connectivity, which means your connection to Snowflake can be via a private link to the cloud.

### Network policies
Żeby stworzyc network policy musimy miec role SECURITY ADMIN albo wyzszą, albo role posiadającą privilege CREATE NETWORK POLICY

To podstawowe narzędzie do filtrowania ruchu przychodzącego (**Ingress**). Działają one na podstawie adresów IP lub identyfikatorów Private Link. Mozna podawac konkretne IP albo IP range (zakres)

- **Allowed List:** Lista adresów IP, które mają wstęp.
- **Blocked List:** Lista adresów IP, które mają zakaz wstępu (mają pierwszeństwo przed Allowed List).

Hierarchia (Bardzo ważne!):
1. **Account Level:** Obowiązuje wszystkich użytkowników na koncie.
2. **User Level:** Przypisana do konkretnego użytkownika (np. administratora). **Polityka na poziomie użytkownika nadpisuje (overrides) politykę na poziomie konta.**


Ingress vs. Egress (Ruch przychodzący i wychodzący)
Musisz rozróżniać te dwa kierunki:

- **Ingress (Przychodzący):** Ktoś chce się zalogować do Snowflake. Kontrolujemy to przez **Network Policies**.
    
- **Egress (Wychodzący):** Snowflake chce się połączyć z czymś na zewnątrz (np. z Twoim bucketem S3, zewnętrznym API lub bazą danych przez External Network Access).
    - **Egress IPs:** To adresy IP, których Snowflake używa, gdy "wychodzi" na zewnątrz. Musisz je znać, aby dopisać je do białej listy (whitelist) na swoim firewallu (np. w AWS S3). Znajdziesz je za pomocą funkcji `SYSTEM$ALLOWLIST()`.


**Pytanie "pewniak" na egzamin:** _Pytanie:_ Do konta przypisana jest polityka sieciowa A (poziom konta), a do użytkownika polityka B (poziom użytkownika). Która z nich decyduje o dostępie tego użytkownika? _Odpowiedź:_Polityka B, ponieważ polityki na poziomie użytkownika mają pierwszeństwo.




### Private connectivity
So by default, your snowflake instance is available over the public internet with access protected by different security measures such as MFA, HTTPS and network rules. If your organization demands that your snowflake instance not be available through the Internet, Snowflake supports private connectivity through which you can ensure that access to your snowflake instances via a private connection.
**And then you can optionally block all Internet access**.

It's good to note that private connectivity to Snowflake requires at least the business critical edition.

Depending on your cloud provider, Snowflake provides private connectivity methods specific to that cloud.


### Encryption in transit (TLS 1.2)
Snowflake encrypts all communication end to end. TLS 1.2 is used to encrypt data while it is in transit. 
Everything in snowflake is connected over https, including connectivity to the Snowflake web UI, JDBC, Python Connector and other connection mechanisms. And then all access to Snowflake's services is accomplished through rest APIs, which are also invoked over the HTTPS protocol.