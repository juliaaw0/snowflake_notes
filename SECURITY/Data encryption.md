
[[SECURITY]]

Snowflake zapewnia szyfrowanie danych na każdym etapie ich drogi.

- **W locie (In-flight):** Snowflake encrypts all data in transit using Transport Layer Security (TLS) 1.2. This applies to all Snowflake connections, including those made through the Snowflake Web interface, JDBC, ODBC, and the Python connector.

- **W spoczynku (At-rest):** Dane zapisane w micro-partitions na cloud storage (S3, Azure Blob, GCS) są zawsze zaszyfrowane za pomocą AES-256. **Użytkownik nie może wyłączyć szyfrowania w spoczynku.** 

![[Zrzut ekranu 2026-05-1 o 11.36.52.png]]


By default, all client data in snowflake is encrypted with AES 256 bit encryption. Snowflake manages the encryption keys by default with no customer intervention required.

every 30 days rotation - The snowflake managed keys are rotated by snowflake when they reach the age of 30 days as part of the rotation process, new keys are created and made active and previously active keys are retired. Retired keys are then used for decryption only during the data access. Active keys are used to both encrypt and decrypt.

Yearly rekeying - (ta opcje trzeba wlaczyc najpierw) If a retired encryption key is more than a year old, Snowflake generates a new encryption key and reencrypts any data previously protected by the retired encryption key. Rekeying requires a minimum of enterprise edition and must be manually enabled by the account administrator.

Tri secret secure - To funkcja typu **BYOK (Bring Your Own Key)**. Pozwala na połączenie klucza Snowflake z kluczem klienta (np. z AWS KMS lub Azure Key Vault). Dane można odczytać tylko wtedy, gdy oba klucze są dostępne. Jeśli klient wyłączy swój klucz, Snowflake nie może odczytać danych (tzw. "cyfrowy wyłącznik bezpieczeństwa"). Funkcja dostępna tylko w edycji **Business Critical** lub wyższej.

![[Zrzut ekranu 2026-05-1 o 11.38.41.png]]


### KEY HIERARCHY:

Snowflake używa hierarchicznej struktury kluczy, przypominającej piramidę. Każdy poziom jest szyfrowany przez poziom wyższy:

1. **Root Key:** Przechowywany w sprzętowym module bezpieczeństwa (HSM - Hardware Security Module) dostawcy chmury.
    
2. **Account Key:** Unikalny dla Twojego konta.
    
3. **Table Key:** Unikalny dla każdej tabeli.
    
4. **File Key:** Szyfruje pojedyncze pliki (micro-partitions).
    

**Dlaczego to ważne?** Taka struktura ogranicza "scope" (zakres) potencjalnego wycieku. Jeśli ktoś złamałby klucz tabeli, nie uzyska dostępu do reszty konta.





_Pytanie egzaminowe:_ Która edycja Snowflake jest wymagana, aby klient mógł zarządzać własnymi kluczami szyfrującymi w celu zwiększenia kontroli nad danymi? _Odpowiedź:_ Business Critical (lub VPS).