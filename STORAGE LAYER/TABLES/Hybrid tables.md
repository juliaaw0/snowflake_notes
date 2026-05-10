[[TABLES]]

Klasyczne tabele w Snowflake są column-based (kolumnowe) i zoptymalizowane pod analytics (OLAP). Dobre do skanowania milionów wierszy. Ale słabe do: częstych pojedynczych UPDATE i losowych odczytów jednego wiersza

Hybrid table to tabela zoptymalizowana pod workload transakcyjny (OLTP). Czyli szybkie pojedyncze INSERT/UPDATE, szybki dostęp do konkretnego wiersza. dużo małych operacji. HIGH CONCURRENCY, LOW LATENCY.


Row locking = blokowanie pojedynczego wiersza podczas jego modyfikacji, żeby inni nie mogli go jednocześnie zmieniać.

[[Index-based access]]
W standardowych tabelach (zapis kolumnowy) Snowflake używa Micro-partitions i Clusteringu. Snowflake ma "notatki" o tym, jakie wartości (min i max) są w każdym bloku danych. Dzięki temu wie, które bloki może pominąć (Pruning). To działa świetnie przy przeszukiwaniu milionów wierszy, ale jest zbyt wolne, by w ułamku sekundy znaleźć jeden konkretny wiersz. Ponieważ Hybrid Tables służą do obsługi transakcji (np. szybkich zamówień w sklepie internetowym), muszą działać jak tradycyjne bazy (Row Store). Tutaj nie ma czasu na sprawdzanie metadanych partycji – potrzebny jest konkretny "adres" wiersza.

W Hybrid Tables mamy dwa rodzaje indeksów:

A. Indeksy tworzone przez użytkownika (Secondary Indexes)
To Ty decydujesz, po czym chcesz szybko wyszukiwać.
Przykład: Masz tabelę klientów. Kluczem głównym (Primary Key) jest ID_KLIENTA. Ale często szukasz ich po NUMER_TELEFONU.
Tworzysz indeks na kolumnie TELEFON. Dzięki temu zapytanie SELECT * FROM klienci WHERE telefon = '123456' zadziała natychmiastowo.

B. Indeksy automatyczne (Built-in indexes)
Snowflake automatycznie tworzy i zarządza indeksami dla:
- Primary Keys (Klucze główne): Muszą być unikalne i Snowflake musi błyskawicznie sprawdzić, czy dany rekord już istnieje.
- Unique Keys (Klucze unikalne): Podobnie jak wyżej.
- Foreign Keys (Klucze obce): Aby szybko powiązać np. Zamówienie z Klientem

Uwaga: minusy indeksow:

- Dodatkowy storage: Indeks to fizyczna, dodatkowa struktura danych. Snowflake musi skopiować dane z kolumn, które indeksujesz, i ułożyć je w specjalny sposób (tzw. drzewo B-Tree).
- Dodatkowe obciazenie przy operacjach DML: Kiedy dodajesz nowy wiersz do hybrid table z indeksem, Snowflake musi zapisać dane w głównej tabeli i natychmiast (synchronicznie) zatrzymać się i zaktualizować każdy indeks, który stworzyłeś, żeby "mapa" pasowała do rzeczywistości. Efekt: Jeśli masz 10 indeksów na jednej tabeli, każde dopisanie nowego wiersza trwa dłużej, bo baza musi wykonać 11 operacji zapisu zamiast jednej

**Zasada kciuka na egzamin:  Im więcej masz indeksów, tym szybciej czytasz dane (SELECT), ale tym wolniej je zapisujesz (INSERT/UPDATE).**

(funkcja Include columns: Zrób mi indeks po kolumnie Email, ale dołącz (INCLUDE) do niego kolumnę Nazwisko" - Bez INCLUDE: Snowflake znajduje email w indeksie, bierze "adres" wiersza, idzie do głównej tabeli (Row Store), wyciąga nazwisko i Ci je pokazuje. Z INCLUDE: Snowflake znajduje email w indeksie i od razu obok widzi nazwisko. Wyświetla je natychmiast. Nie musi dotykać głównej tabeli. To się nazywa Index-Only Access.)

Indeksow robionych recznie (secondary indexes) nie można modyfikowac, można je tylko dropnac i zrobic nowy



You can join hybrid tables with other Snowflake tables.

Hybrid table ma dwa „sposoby przechowywania danych” naraz (I Snowflake sam decyduje, którego użyć).:

- szybki row store do transakcji (czyli dane zapisywane sa wierszami a nie kolumnami. Na dysku "obok siebie" w jednym miejscu leżą nie np. same imiona, w innym same nazwiska, a jeszcze gdzies indziej same maile, tylko w row store razem jest 1 wiersz: konkretne imie, nazwisko, mail itp..
- kopię zoptymalizowaną pod analitykę - do tej kopii (object storage) dane sa kopiowane asynchronicznie po tym jak pierwotnie trafią do row store.
- Dodatkowo dla jeszcze szybszych zapytan OPAL może istniec cache w warehousie, w formacie kolumnowym

To dlatego nazywa się to Unistore (OLTP + OLAP razem)

Required vs enforced:

ENFORCED = baza aktywnie sprawdza i blokuje operacje, które łamią constraint. Nie można np. wstawic dwoch wierszy o tym samym ID jeśli mamy enforced primary key. Dostaniemy blad.  ENFORCED --> przy insert/update

REQUIRED = musisz zdefiniować ten constraint, żeby w ogóle utworzyć tabelę. Jeśli zrobimy komende create table bez tego constrainta dostaniemy blad. REQUIRED dotyczy samego tworzenia tabeli a nie operacji na niej.

Jak to się stosuje w hybrid tables:

PK w tabeli hybrydowej: REQUIRED, ENFORCED

Foreign key (A FOREIGN KEY constraint informs the query optimizer that a particular record in a child table points to exactly one record in a parent table), UNIQUE i NOT NULL: nie required (optional), ENFORCED - nie trzeba go definiowac ale jak już to zrobimy, to jest pilnowane ze musi być

Dla porownania w standard table: PK, FK i UNIQUE nie jest ani required ani enforced (nawet jak dodamy taki constraint do create table, potem w insertach itp. Można np. dodac 2x ten sam PK i nie dostaniemy bledu)

NOT NULL jest not required (optional), enforced.

TWORZENIE HYBRID TABLE:

To create a hybrid table, you must have a running warehouse that is specified as the current warehouse for your session. Hybrid table można zrobic normalnie (pusta) albo CTAS (create hybrid table as select - ale trzeba pamietac ze CTAS nie obsluguje FOREIGN KEY constraint). Nie można zaladowac hybrid table np. za pomoca Snowpipe. Dla hybrid table najlepszy jest bulk load (przez COPY lub INSERT INTO SELECT) , ale można go zrobic tylko na pustej tabeli!

Snowsight vs. Drivery: Wydajność mierzona w interfejsie Snowsight nie odzwierciedla rzeczywistej wydajności dla aplikacji. Snowsight dodaje narzut (overhead). Dla Hybrid Tables kluczowe jest używanie programistycznych driverów (JDBC, Python itp.)

Zaleca się oddzielenie magazynów dla zapytań operacyjnych (Hybrid Tables) od analitycznych, aby uniknąć rywalizacji o pamięć cache. Hybrid Tables mają swój własny mechanizm cache'owania (plan cache, data cache,metadata cache). Nie korzystają z Result Cache. Oto wyjasnienie rodzajow cache typowych dla hybrid tables:

- [[Plan cache]] - Plan Cache przechowuje "gotowy przepis" na wykonanie konkretnego zapytania. Jeśli Twoja aplikacja wysyła tysiące razy to samo zapytanie (np. SELECT * FROM orders WHERE id = ?), Snowflake pomija etap myślenia ("przetłumaczenia" na instrukcje zrozumiałe dla procesora (kompilacja, optymalizacja, wybór indeksów)) i od razu przechodzi do działania. Cel: zmniejszenie narzutu kompilacji (compilation overhead)
- [[Column store data cache]] -  Hybrid Tables, mimo że służą do OLTP, pod spodem utrzymują też format kolumnowy, aby zapytania typu "zsumuj sprzedaż z dzisiaj" również były szybkie. Dane te lądują w cache'u, aby kolejne zapytania nie musiały sięgać do wolniejszej pamięci dyskowej.
- [[Metadata cache]] - przechowuje strukturę tabeli i — co najważniejsze — informacje o indeksach. Dzięki temu silnik bazy danych błyskawicznie wie, do którego konkretnego wiersza się "wkłuć", zamiast przeszukiwać całą tabelę.

Dlaczego "Hybrid tables do not use a result cache"?

To najważniejszy punkt pod certyfikat i do zrozumienia Hybrid Tables.

Standardowy Result Cache w Snowflake działa tak: jeśli nikt nie zmienił danych w tabeli i zadasz to samo pytanie, Snowflake zwróci wynik z globalnej pamięci (Service Layer) bez uruchamiania Warehouse.

W Hybrid Tables to nie ma sensu, ponieważ:

1. Dane zmieniają się non-stop: W systemach transakcyjnych (np. sklep internetowy) stany magazynowe czy zamówienia zmieniają się co milisekundę. Result Cache byłby unieważniany (invalidated) niemal natychmiast po utworzeniu.
2. Unikalność zapytań: W OLTP każde zapytanie zazwyczaj dotyczy innego ID (inny klient, inny koszyk). Prawdopodobieństwo, że dwóch użytkowników zapyta o dokładnie ten sam rekord w tym samym ułamku sekundy, jest małe.

Throttling i quotas w hybrid tables:

W standardowym Snowflake'u jedynym ograniczeniem jest moc Twojego Warehouse'a (jeśli jest za mały, zapytania po prostu trwają dłużej). W Hybrid Tables (Unistore) dochodzą dodatkowe "bezpieczniki" – Quotas (limity ) - ok 8000 zaoytan na sekunde. Aby jeden użytkownik nie "zapchał" całej infrastruktury Snowflake'a swoimi zapytaniami OLTP, Snowflake nakłada limity przepustowości na poziomie bazy danych. Throttling to zjawisko, w którym Snowflake celowo spowalnia Twoje zapytania lub zwraca błąd, ponieważ przekroczyłaś przyznany Ci limit operacji na sekundę (throughput). Jeśli otworzysz Query Profile dla zapytania do Hybrid Table, możesz zobaczyć statystykę "Percentage of time throttled" - wysoka wartosc = przekroczony limit przepustowosci. Jak sobie z tym poradzic: zoptymalizowac query i upewnic się ze korzysta z indeksow, albo wlaczyc platny per hour (nawet jak nie ma zapytan) Dedicated Storage Mode (który powoduje ze hybrid table nie wspoldzieli zasobow z innymi tabelami). Standardowo (jak nie wlaczymy tego dediacted) to mamy Shared Storage Model który wlasnie może prowadzic do throttlingu.

KOSZTY HYBRID TABLES:

1. Trzy filary kosztów (Triple Billing)

W przypadku standardowych tabel płacisz głównie za Compute (Warehouse) i Storage. Przy Hybrid Tables dochodzi trzeci element:

1. Compute (Virtual Warehouse): Płacisz za kredyty, gdy Warehouse pracuje. To działa tak samo jak zawsze.
2. Storage (Baza danych): Płacisz za GB danych. Uwaga: dane w Hybrid Tables są droższe niż w standardowych tabelach (Standard Storage), ponieważ są przechowywane w sposób umożliwiający szybki dostęp transakcyjny.
3. Hybrid Storage Charge: To dodatkowa opłata za utrzymanie danych w formacie zoptymalizowanym pod OLTP.

4. Brak Result Cache = Większe zużycie kredytów

Jak już ustaliliśmy wcześniej, Hybrid Tables nie używają Result Cache.

- Egzaminacyjny haczyk: Jeśli to samo zapytanie zostanie wykonane dwa razy pod rząd, za każdym razem Snowflake użyje Warehouse'a i pobierze za to kredyty. W przypadku standardowych tabel drugie zapytanie byłoby "darmowe" (pobrane z Result Cache).

3. Koszty operacji zapisu (Write overhead)

Każdy indeks, który dodasz do Hybrid Table, zwiększa koszt:

- Storage: Indeksy zajmują miejsce.
- Compute: Każdy INSERT czy UPDATE musi zaktualizować wszystkie indeksy, co sprawia, że operacja trwa odrobinę dłużej i zużywa więcej "mocy" Warehouse'a.

Oto lista "zakazów" i ograniczeń, które musisz znać na pamięć:

1. Ograniczenia funkcjonalne (Co "nie działa")

- Brak Result Cache: To już wiesz, ale powtarzam, bo to absolutny pewniak egzaminacyjny.
- Brak wsparcia dla Snowpipe: Nie możesz używać Snowpipe do ciągłego ładowania danych bezpośrednio do Hybrid Tables.
- Brak Streams & Tasks: Obecnie Hybrid Tables nie wspierają bezpośrednio strumieni danych (Streams).
- Brak Search Optimization Service: Ta funkcja przyspieszania wyszukiwania w Snowflake nie działa z tabelami hybrydowymi (bo one mają własne indeksy).
- Hybrid tables nie ma na Google clour platform

2. Ograniczenia Typów Danych i Rozmiaru

- Rozmiar wiersza: Cały pojedynczy wiersz w Hybrid Table nie może przekroczyć 16 MB. A cala zawartosc danych w hybrid tables w 1 bazie nie może być wieksza niż 2 tb.
- Typy danych: Niektóre bardzo specyficzne typy danych lub funkcje geoprzestrzenne mogą mieć ograniczenia w porównaniu do standardowych tabel.

3. Ograniczenia Operacyjne

- Brak Time Travel dla klonowania: Nie możesz sklonować tabeli hybrydowej "z przeszłości". Możesz użyć Time Travel tylko do zapytania SELECT, ale nie do operacji CLONE.
- Brak Undrop: Standardowy UNDROP może mieć ograniczenia w zależności od tego, jak powiązane są klucze obce (Foreign Keys) z innymi tabelami.
- Zmiana typu tabeli: NIE MOŻNA zmienić istniejącej tabeli standardowej na hybrydową (i odwrotnie) za pomocą polecenia ALTER. Musisz stworzyć nową tabelę i przenieść dane (np. przez INSERT INTO ... SELECT). Nie możesz zamienić Hybrid Table w tabelę Iceberg (zewnętrzną).

Unsupported features, At this time, hybrid tables do not support:

- [Clustering ke](https://docs.snowflake.com/en/user-guide/tables-clustering-keys)ys - Data in hybrid tables is ordered by the primary key.
- [Data sharing](https://docs.snowflake.com/en/guides-overview-sharing)
- [Dynamic tables](https://docs.snowflake.com/en/user-guide/dynamic-tables-about)
- [Fail-safe](https://docs.snowflake.com/en/user-guide/data-failsafe)
- [Materialized views](https://docs.snowflake.com/en/user-guide/views-materialized)
- [Query Acceleration Service](https://docs.snowflake.com/en/user-guide/query-acceleration-service)
- [Replication](https://docs.snowflake.com/en/user-guide/account-replication-intro)
- [Search Optimization Service](https://docs.snowflake.com/en/user-guide/search-optimization-service)
- [Snowpipe](https://docs.snowflake.com/en/user-guide/data-load-snowpipe-intro)
- [Snowpipe Streaming API](https://docs.snowflake.com/en/user-guide/snowpipe-streaming/data-load-snowpipe-streaming-overview)
- [Streams](https://docs.snowflake.com/en/user-guide/streams-intro)
- [UNDROP](https://docs.snowflake.com/en/sql-reference/sql/undrop-table)

