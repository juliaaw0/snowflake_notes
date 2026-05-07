[[CONTINUOUS DATA PROTECTION]]


Dawniej w Snowflake replikowało się tylko bazy danych. Teraz standardem jest **Account Replication**.

- **Database Replication:** Kopiuje tylko dane i strukturę (tabele, widoki).
    
- **Account Replication:** Kopiuje dane ORAZ metadane konta (użytkowników, role, magazyny, integracje bezpieczeństwa). Dzięki temu po awarii całego regionu AWS możesz zalogować się na Azure i wszystko (teoretycznie) działa tak samo.


### Replication & Failover groups

Zamiast wysyłać każdą bazę danych, każdego użytkownika i każdą rolę z osobna na drugie konto (co byłoby logistycznym koszmarem), tworzysz **Grupę**.

- **Grupa to obiekt w Snowflake**, który mówi: „Wszystko, co włożysz do środka, będzie podróżować razem, w tym samym czasie i według tego samego harmonogramu”. Zarówno **Replication Group**, jak i **Failover Group** to po prostu **"paczki" (kontenery)**. Zamiast wysyłać pojedynczo bazę danych, potem użytkowników, potem role, wrzucasz wszystko do jednej paczki. Snowflake dba o to, żeby zawartość tej paczki na Koncie B była identyczna z tą na Koncie A.
    

#### Replication Group (Kopia zapasowa / Lustro)

To jest Twoja "kopia bezpieczeństwa" lub "lustro" do podglądu. Regularnie przesyła zmiany z Konta A na Konto B. Wszystko jest tam **tylko do odczytu (Read-Only)**. Możesz tam wejść i sprawdzić dane, ale nie możesz nic dopisać ani zmienić. **W razie awarii:** Jeśli Konto A wybuchnie, masz dane na Koncie B, ale **nie możesz ich tam edytować**. Musiałbyś ręcznie kopiować dane do nowej bazy, co zajmuje mnóstwo czasu.
    
- **Główny cel:** Raportowanie w innym regionie (żeby analitycy z USA nie "mulili" konta w Europie) lub prosty backup.
* Pamiętaj, że replikacja zużywa kredyty na obu kontach (Primary do obliczenia zmian, Secondary do ich zapisu).

### Failover Group (Zapasowy silnik / Koło ratunkowe)

To jest wersja "Premium" z jednym, magicznym przyciskiem: **PRZEŁĄCZ**. Wymaga edycji **Business Critical** (lub wyższej). "Database failover and failback between Snowflake accounts are provided first in the Business Critical edition and are also available in the virtual private Snowflake (VPS) edition."

- **Co robi:** Robi dokładnie to samo co Replication Group (kopiuje paczkę), ale ma dodatkową funkcję logiki biznesowej.
    
- **Status na Koncie B:** Zazwyczaj też jest Read-Only... **DOPÓKI** nie wydasz komendy Failover.
    
- **W razie awarii:** Jeśli Konto A padnie, wchodzisz na Konto B i mówisz: _"Teraz ty jesteś szefem"_. **Gdzie wpisujesz komendę?** To jest najczęstsze pytanie na egzaminie! Komendę failover wpisujesz na **Koncie Zapasowym (Secondary)**.
    
- **Magia:** W ułamku sekundy paczka na Koncie B staje się **Read-Write (do zapisu)**. Twoja firma może pracować dalej na koncie zapasowym, dopóki główne nie zostanie naprawione.
    
- **Główny cel:** Przetrwanie katastrofy (Disaster Recovery). To Twoja polisa ubezpieczeniowa na wypadek, gdyby cały region chmury (np. AWS Frankfurt) przestał działać.

Jeśli zobaczysz pytanie o **"Business Continuity"** lub **"Disaster Recovery"** – Twoją odpowiedzią jest zawsze **Failover Group**. Tylko ten obiekt pozwala na zmianę roli konta z "zapasowego" na "główne" (Primary).

CLIENT REDIRECT Z FAILOVER GROUP:

To jest "magiczny składnik", o którym warto wiedzieć: Jeśli masz skonfigurowany **Client Redirect**, Twoi użytkownicy (analitycy, narzędzia BI) używają jednego, specjalnego adresu URL. Gdy robisz Failover, Snowflake automatycznie przekierowuje ten adres na nowe konto. Użytkownik nawet nie musi zmieniać adresu w swojej przeglądarce czy lookerze.




Failback - przywrócenie statusu Primary na koncie, ktore umarło i musielismy zrobic failover

