Dotyczy [[hybrid tables]]!!!
W standardowych tabelach (zapis kolumnowy) Snowflake używa Micro-partitions i Clusteringu. Snowflake ma "notatki" o tym, jakie wartości (min i max) są w każdym bloku danych. Dzięki temu wie, które bloki może pominąć (Pruning). To działa świetnie przy przeszukiwaniu milionów wierszy, ale jest zbyt wolne, by w ułamku sekundy znaleźć jeden konkretny wiersz. Ponieważ Hybrid Tables służą do obsługi transakcji (np. szybkich zamówień w sklepie internetowym), muszą działać jak tradycyjne bazy (Row Store). Tutaj nie ma czasu na sprawdzanie metadanych partycji – potrzebny jest konkretny "adres" wiersza.

W Hybrid Tables mamy dwa rodzaje indeksów:

A. Indeksy tworzone przez użytkownika (Secondary Indexes)

To Ty decydujesz, po czym chcesz szybko wyszukiwać.

Przykład: Masz tabelę klientów. Kluczem głównym (Primary Key) jest ID_KLIENTA. Ale często szukasz ich po NUMER_TELEFONU.

Tworzysz indeks na kolumnie TELEFON. Dzięki temu zapytanie SELECT * FROM klienci WHERE telefon = '123456' zadziała natychmiastowo.

B. Indeksy automatyczne (Built-in)

Snowflake automatycznie tworzy i zarządza indeksami dla:

Primary Keys (Klucze główne): Muszą być unikalne i Snowflake musi błyskawicznie sprawdzić, czy dany rekord już istnieje.

Unique Keys (Klucze unikalne): Podobnie jak wyżej.

Foreign Keys (Klucze obce): Aby szybko powiązać np. Zamówienie z Klientem

Uwaga: minusy indeksow:

- Dodatkowy storage: Indeks to fizyczna, dodatkowa struktura danych. Snowflake musi skopiować dane z kolumn, które indeksujesz, i ułożyć je w specjalny sposób (tzw. drzewo B-Tree).
- Dodatkowe obciazenie przy operacjach DML: Kiedy dodajesz nowy wiersz do hybrid table z indeksem, Snowflake musi zapisać dane w głównej tabeli i natychmiast (synchronicznie) zatrzymać się i zaktualizować każdy indeks, który stworzyłeś, żeby "mapa" pasowała do rzeczywistości. Efekt: Jeśli masz 10 indeksów na jednej tabeli, każde dopisanie nowego wiersza trwa dłużej, bo baza musi wykonać 11 operacji zapisu zamiast jednej

Zasada kciuka na egzamin:  Im więcej masz indeksów, tym szybciej czytasz dane (SELECT), ale tym wolniej je zapisujesz (INSERT/UPDATE).

Include columns: Zrób mi indeks po kolumnie Email, ale dołącz (INCLUDE) do niego kolumnę Nazwisko" - Bez INCLUDE: Snowflake znajduje email w indeksie, bierze "adres" wiersza, idzie do głównej tabeli (Row Store), wyciąga nazwisko i Ci je pokazuje. Z INCLUDE: Snowflake znajduje email w indeksie i od razu obok widzi nazwisko. Wyświetla je natychmiast. Nie musi dotykać głównej tabeli. To się nazywa Index-Only Access.

Indeksow robionych recznie (secondary indexes) nie można modyfikowac, można je tylko dropnac i zrobic nowy