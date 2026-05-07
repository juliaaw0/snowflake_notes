[[CACHES]]

### Query Results Caching (Wyniki w pamięci podręcznej)

**Ważność na egzaminie:** ⭐⭐⭐⭐⭐ (Kluczowe dla wydajności i kosztów)

Snowflake uwielbia pytać o **Result Cache**. To mechanizm, który pozwala na natychmiastowe pobranie wyniku zapytania bez użycia Warehouse.

- **Czas trwania:** Wyniki są trzymane przez **24 godziny**.
    
- **Odświeżanie:** Licznik 24h resetuje się za każdym razem, gdy ktoś wywoła to samo zapytanie (maksymalnie do 31 dni).

* Wyniki są dostępne dla **całego konta** (jeśli inny użytkownik z tą samą rolą zada to samo pytanie, dostanie wynik z cache). Wazna jest ta sama rola! Tylko wtedy query result cache zostanie uzyte jesli pierwotne query ktore wygenerowalo cache i kolejne query są zrobione przez uzytkownikow o tych samych rolach
    
- **Warunki Cache Hit (Kiedy to zadziała?):**
    
    Snowflake uses the query result cache if the following conditions are met. 
    - A new query matches an old query,
	* underlying data contributing to the query results remains unchanged. 
	* The table micro-partitions have not changed as a result of clustering or consolidation. * The query makes no use of user-defined and external, or runtime functions. Note that queries that use the CURRENT DATE function are eligible for query result caching.
	
- **Koszt:** Korzystanie z Result Cache jest **darmowe** (nie zużywa kredytów Warehouse).


Query Result Cache jest integralną częścią warstwy **Cloud Services**. To właśnie ta „lokalizacja” nadaje mu jego unikalne supermoce, które odróżniają go od pozostałych rodzajów cache’u w Snowflake.

Oto dlaczego to powiązanie jest tak istotne dla Ciebie (i dla Twojego egzaminu):

Ponieważ Result Cache znajduje się w „mózgu” Snowflake (Cloud Services), a nie w „mięśniach” (Virtual Warehouse), ma on kilka kluczowych cech:

- **Niezależność od Warehouse'u:** Jeśli wynik zapytania jest w cache, Snowflake może go zwrócić, nawet jeśli Twój Virtual Warehouse jest uśpiony (**Suspended**). Nie musisz płacić za kredyty compute, żeby dostać ten wynik!
    
- **Dostępność Globalna:** Wynik jest dostępny dla każdego użytkownika na Twoim koncie (o ile ma te same uprawnienia/rolę i zapytanie jest identyczne), niezależnie od tego, którego Warehouse'u używa.
    
- **Szybkość:** Omijamy cały proces planowania zapytania i uruchamiania maszyn. Cloud Services po prostu sprawdza „czy już to robiliśmy?” i wysyła gotową paczkę danych.

Za włączanie i wyłączanie Query Result Cache odpowiada parametr o nazwie `USE_CACHED_RESULT`. W architekturze Snowflake parametry mają określoną hierarchię i poziomy, na których można je ustawiać.

Ten konkretny parametr steruje **zachowaniem optymalizatora zapytań**, a nie sposobem przechowywania danych.

### 2. Dlaczego te poziomy mają sens?

- **Account (Konto):** Administrator może zdecydować, że ze względów bezpieczeństwa lub specyficznych testów wydajnościowych nikt na całym koncie nie powinien korzystać z cache'u.
    
- **User (Użytkownik):** Możesz chcieć, aby konkretny użytkownik (np. bot testujący wydajność) zawsze wymuszał świeże zapytania, podczas gdy inni pracownicy korzystają z cache'u.
    
- **Session (Sesja):** To najczęstszy przypadek. Jako analityk piszesz: `ALTER SESSION SET USE_CACHED_RESULT = FALSE;`, bo chcesz sprawdzić, jak szybko Twoje zapytanie zadziała "naprawdę", bez wspomagania się gotowym wynikiem.
    

---

### 3. Dlaczego nie poziom Tabeli?

To jest klucz do zrozumienia architektury:

1. **Cache jest "ponad" tabelami:** Result Cache nie wie, że "należy" do tabeli A czy B. On przechowuje wynik konkretnego **tekstu zapytania SQL**. Jedno zapytanie może łączyć 10 tabel. Na którym poziomie miałabyś wtedy wyłączyć cache? Na wszystkich dziesięciu? To byłoby nieefektywne.
    
2. **Inwalidacja vs Wyłączenie:** Tabela ma wpływ na cache tylko w jeden sposób: jeśli dane w niej się zmienią, cache staje się nieważny (**invalidated**). Ale nie możesz zakazać Snowflake'owi "pamiętania" wyników dla konkretnej tabeli, bo system cache'owania działa globalnie w warstwie Cloud Services.