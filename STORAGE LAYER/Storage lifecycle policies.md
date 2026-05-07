[[STORAGE LAYER]]
**Storage Lifecycle Policies** służą do oszczędzania pieniędzy na danych, których już prawie nie używasz, ale musisz je trzymać (np. ze względów prawnych).

Snowflake wprowadził pojęcie warstw, aby obniżyć koszty przechowywania:

- **Standard Storage (Active):** Dane, które odpytujesz na co dzień. Płacisz standardową stawkę za TB/miesiąc.
    
- **Archive Storage (Cold):** Dane archiwalne. Koszt przechowywania jest znacznie niższy, ale ich odczyt jest wolniejszy i dodatkowo płatny.
    

**Storage Lifecycle Policy** to zestaw reguł, które automatycznie przenoszą dane z warstwy Standard do Archive po określonym czasie bezczynności.


**Ważność na egzaminie:** ⭐⭐⭐

- **Obiekt:** Jest to obiekt na poziomie konta (`Account-level object`).
    
- **Reguły:** Polityka opiera się na parametrze `MIN_DAYS_SINCE_LAST_ACCESSED`. Jeśli nikt nie dotykał tabeli np. przez 90 dni, Snowflake przenosi jej dane do archiwum.
    
- **Przypisanie:** Politykę przypisuje się do konkretnych tabel.


To jest punkt, w którym egzaminatorzy mogą Cię "złapać". Musisz znać te zasady:

- **Niższy koszt Storage:** Przechowywanie danych w Archive jest tańsze (często porównywalne z cenami S3 Glacier czy Azure Archive Storage).
    
- **Koszt Odczytu (Retrieval Cost):** Wyciągnięcie danych z archiwum z powrotem do warstwy Standard kosztuje kredyty.
    
- **Opóźnienie (Latency):** Dane w archiwum **nie są dostępne natychmiast**. Proces "przywracania" (restoration) może potrwać od kilku do kilkunastu godzin.
- 

Snowflake udostępnia widoki w `ACCOUNT_USAGE`, abyś mógł śledzić:

- Które tabele są zarchiwizowane.
    
- Ile zaoszczędziłeś dzięki politykom.
    
- Status procesów przywracania danych.