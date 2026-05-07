
[[CONTINUOUS DATA PROTECTION]]
### Czym jest Fail-safe? (Ostatnia deska ratunku)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐

To nieusuwalny, 7-dniowy okres ochrony danych, który następuje automatycznie **zaraz po** tym, jak dane wypadną z okna Time Travel. Nie da sie zmienic tych 7 dni na inna ilosc albo wylaczyc failsafe dla tabel permanentnych. 

- **Kluczowa różnica:** Ty (jako użytkownik czy admin) **nie masz dostępu** do tych danych. Nie możesz ich odpytać przez `SELECT`, ani przywrócić przez `UNDROP`.
    
- **Kto może pomóc?** Jedynie **Snowflake Support**. Jeśli skasowałaś dane i minął już okres Time Travel, musisz pisać ticket do wsparcia technicznego, żeby spróbowali je odzyskać z Fail-safe.


|**Typ tabeli**|**Time Travel**|**Fail-safe**|**Możliwość odzyskania przez Support?**|
|---|---|---|---|
|**Permanent**|0–90 dni|**7 dni**|✅ TAK|
|**Transient**|0–1 dzień|**Brak (0 dni)**|❌ NIE|
|**Temporary**|0–1 dzień|**Brak (0 dni)**|❌ NIE|

### Koszty (Storage)

**Ważność na egzaminie:** ⭐⭐⭐⭐

Dane w Fail-safe zajmują miejsce na dysku (w chmurowym storage'u), więc Snowflake za nie nalicza opłaty.

- Płacisz za nie taką samą stawkę jak za aktywne dane.
    
- **Problem "churnu":** Jeśli masz tabelę, w której codziennie podmieniasz (Delete/Insert) miliony rekordów, a jest to tabela **Permanent**, to przez 7 dni w Fail-safe będą zalegać "duchy" tych wszystkich usuniętych danych. To może drastycznie podnieść rachunek za Storage!
    
- **Rozwiązanie:** Dla dużych, często zmienianych tabel, które nie są krytyczne, używa się tabel **Transient**, aby uniknąć kosztów Fail-safe.