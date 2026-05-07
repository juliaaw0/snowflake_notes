[[COLLABORATION]]

![[Zrzut ekranu 2026-04-29 o 23.48.44.png|292]]
## Czym są Listings? (Koncepcja)

![[Zrzut ekranu 2026-04-30 o 00.11.27.png|439]]

**Listing** to pełna "strona produktu" w sklepie (Amazonie danych). 

Listing pozwala dostawcy (Provider) nie tylko udostępnić dane, ale też dołączyć do nich:

- **Tytuł i opis** (co to za dane?).
    
- **Przykładowe zapytania SQL** (jak z tego korzystać?).
    
- **Metadane** (częstotliwość odświeżania, regiony).
    
- **Warunki biznesowe/licencyjne** (czy to jest darmowe, czy płatne?). Listing moze tez byc gotowy od reki czyli generic, albo personalizowany dla klienta. 


**Snowflake Marketplace (Public):** Dostępny dla wszystkich klientów Snowflake na całym świecie. Możesz tam znaleźć dane pogodowe, finansowe czy demograficzne od firm zewnętrznych. Każdy poziom konta Snowflake (nawet standard) z wyjątkiem VPS moze z tego korzystac. Nie jest to funkcja enterprise. Pod spodem Marketplace i tak korzysta z direct data sharing jako mechanizmu.  Snowflake Marketplace can be browsed by non-Snowflake users as well, however they need to sign up to a Snowflake edition in order to consume data from the marketplace. Data Marketplace is supported in all Snowflake editions;

czyli w pytaniu - jaka jest najnizsza rola zeby BROWSE marketplace - zadna, nie trzeba miec edycji snowfloake
w pytaniu - jaka jest najnizsza rola zeby SKORZYSTAC z zasobow marketplace - STANDARD

## Direct Sharing vs. Listings (Klucz do pytań!)

To jest najczęstszy temat pytań porównawczych na egzaminie.

![[Zrzut ekranu 2026-04-28 o 22.09.30.png]]


## Role i Uprawnienia

To jest techniczny aspekt, o który Snowflake lubi pytać:

- **ACCOUNTADMIN:** Domyślnie tylko ta rola może przeglądać i "kupować" (montować) dane z Marketplace. Inne role mogą to robić tylko, jeśli dostaną specjalne uprawnienie `CREATE DATA EXCHANGE LISTING` lub `IMPORT SHARE`.
    
- **Data Consumer:** Konto, które przegląda Marketplace i dodaje dane do siebie.
    
- **Data Provider:** Konto, które publikuje listing.


## "Auto-fulfillment" (Kluczowy termin!)

Jeśli na egzaminie padnie pytanie: _"Jak najłatwiej udostępnić te same dane klientom w różnych regionach chmurowych (np. AWS Frankfurt i Azure Singapore)?"_ – poprawną odpowiedzią często jest **Listing z włączonym Auto-fulfillmentem**. Snowflake w tle zajmuje się replikacją danych, a Ty nie musisz ręcznie konfigurować kont w każdym regionie.