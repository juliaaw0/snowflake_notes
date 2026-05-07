[[COSTS, BILLING]]

### Modele płatności (Billing Models)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

Na egzaminie musisz znać różnicę między tymi dwoma podejściami, bo wpływają one na cenę kredytu i storage’u:

- **On-Demand (Na żądanie):** Płacisz „z dołu” za to, co zużyłeś w danym miesiącu. Stawki są zazwyczaj wyższe, ale nie masz żadnych zobowiązań.
    
- **Capacity (Pojemność):** Kupujesz określoną pulę kredytów „z góry” (pre-paid). Dzięki temu otrzymujesz niższe ceny za kredyt i często niższe stawki za przechowywanie danych (Storage). Jak kredyty w capacity sie skoncza - Konto płynnie przechodzi w tryb **On-Demand**.

Gdzie szukać papierów? Tylko `ACCOUNTADMIN` (dla jednego konta) lub `ORGADMIN` (dla całej organizacji) mają dostęp do zakładki Billing.



### Uzgadnianie kosztów (Reconciling)

**Ważność na egzaminie:** ⭐⭐⭐

Zadanie dla administratora: „Sprawdź, czy faktura zgadza się z rzeczywistością”. Snowflake sugeruje użycie widoków SQL, aby porównać fakturę z logami:

- **`METERING_HISTORY`:** Pokazuje zużycie kredytów przez magazyny.
    
- **`DATABASE_STORAGE_USAGE_HISTORY`:** Pokazuje zużycie storage’u.
    
- **`DATA_TRANSFER_HISTORY`:** Pokazuje koszty przesyłu danych.


### Waluta: Kredyt vs. Pieniądz

Pamiętaj o złotej zasadzie:

- **Kredyty Snowflake** to uniwersalna miara zużycia Compute.
    
- Cena jednego kredytu zależy od **Edycji Snowflake** (Standard, Enterprise, Business Critical) oraz **Regionu/Chmury** (AWS, Azure, GCP).
    
- Na fakturze zużyte kredyty są przeliczane na walutę zgodnie z Twoją umową.




pewniak egzaminacyjny:
> **Pytanie:** W którym miejscu w interfejsie Snowsight administrator może sprawdzić aktualne zużycie kredytów oraz pobrać miesięczne faktury?
> 
> **Odpowiedź:** W zakładce Admin, podsekcja Billing.