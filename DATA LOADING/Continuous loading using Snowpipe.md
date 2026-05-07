[[DATA LOADING]]

This option is designed to load small volumes of data (i.e. micro-batches) and incrementally make them available for analysis. Snowpipe loads data within minutes after files are added to a stage and submitted for ingestion. This ensures users have the latest results, as soon as the raw data is available.

Snowpipe może ładować dane z external (z uzyciem notifications z tego clouda gdzie jest external stage ze przyszly nowe dane) i internal stage (z uzyciem REST API Snowpipe'a)

#### Compute resources

Snowpipe uses compute resources provided by Snowflake (i.e. a serverless compute model). These Snowflake-provided resources are automatically resized and scaled up or down as required, and are charged and itemized using per-second billing. Data ingestion is charged based upon the actual workloads.

**Składowe kosztu:**
1. **Compute:** Faktyczny czas procesora zużyty na ładowanie.
2. **Overhead (Narzut):** Koszt zarządzania plikami i kolejkami.

**Złota zasada optymalizacji:** **Lepiej ładować mniej dużych plików niż mnóstwo małych.**
- Snowflake dolicza mały narzut za każdy plik. Jeśli ładujesz pliki KB-towe, koszt overheadu może przewyższyć koszt compute.
- **Zalecenie pod egzamin:** Agreguj pliki tak, aby miały **100-250 MB** (po skompresowaniu).

Liczbe zuzytych kredytow przez Snowpipe mozna sprawdzic w widoku `PIPE_USAGE_HISTORY` w schemacie `ACCOUNT_USAGE`.

#### Simple transformations during a load

The COPY statement in a pipe definition supports the same COPY transformation options as when bulk loading data.
* Column reordering
- Column omission
- Casts
- Truncating text strings that exceed the target column length

In addition, data pipelines can leverage Snowpipe to continuously load micro-batches of data into staging tables for transformation and optimization using automated tasks and the change data capture (CDC) information in streams.

### Auto-ingest

- Plik trafia do S3 → S3 wysyła sygnał do Snowpipe (Wykorzystuje powiadomienia z chmury (Cloud Messaging)) → Snowpipe ustawia plik w kolejce → Dane trafiają do tabeli.
- **Obiekt PIPE:** Tworzysz go komendą `CREATE PIPE`. Wewnątrz definicji pipe'a znajduje się polecenie `COPY INTO` (bez parametrów takich jak Warehouse, bo to serverless).

Jeśli pipe jest wstrzymany (`PAUSED = TRUE`), powiadomienia o nowych plikach (z S3/Azure/GCS) "odkładają się" w kolejce (event queue). Snowflake zachowuje je przez **ograniczony czas (zazwyczaj 14 dni)**. Po wznowieniu (`RESUME`), pipe spróbuje przetworzyć te zaległe eventy.

**Recreate (OR REPLACE):** Jeśli użyjesz `CREATE OR REPLACE PIPE`, **historia ładowania tego pipe'a zostaje wyczyszczona**. Oznacza to, że jeśli w stage'u są pliki, które już raz zostały załadowane, zostaną załadowane **ponownie**(duplikaty!).

Na egzaminie może paść pytanie o to, jak wymusić załadowanie plików, które Snowflake uważa za już załadowane
- Dla `COPY INTO`: Używamy flagi `FORCE = TRUE`.
- Dla obu metod: Zmiana nazwy pliku na dysku/stage'u (Snowflake identyfikuje pliki po nazwie i sumie kontrolnej).

### Snowpipe REST API
- Używamy REST API, gdy nie możemy skorzystać z auto-ingestu (np. gdy system źródłowy nie wysyła powiadomień do chmury, ale może wykonać zapytanie HTTP).
    
- **Uwierzytelnianie:** Snowpipe REST API wymaga **Key-pair authentication** (Public/Private Key). To częste pytanie – nie używa się tu zwykłego hasła użytkownika.
    
- **Metoda `insertFiles`:** To główny punkt końcowy (endpoint), który informuje Snowpipe o nowych plikach do załadowania.

### Monitoring 
- **Pipe Status:** Używasz funkcji `SYSTEM$PIPE_STATUS('nazwa_pipe')`, aby sprawdzić, czy pipe działa, czy ma błędy i ile plików czeka w kolejce.
    
- **Load History:** Do sprawdzania historii ładowania przez Snowpipe służy funkcja tabelaryczna `VALIDATE_PIPE_LOAD` lub widok `SNOWFLAKE.ACCOUNT_USAGE.PIPE_USAGE_HISTORY`.

 ![[Zrzut ekranu 2026-04-25 o 14.35.30.png]]

### RETENCJA Meta-danych (Idempotentność):

SNOWPIPE: 14 dni  - Jeśli ten sam plik zostanie wysłany ponownie po 15 dniach, Snowpipe uzna go za nowy i **załaduje duplikat**.

COPY INTO (bulk): 64 dni - Jeśli plik o tej samej nazwie zostanie w stage'u po 65 dniach i uruchomisz `COPY INTO`, Snowflake **załaduje go ponownie**.

### Retencja Historii (Widoki i Raportowanie)

- Bardzo szybki dostęp (real-time) - LOAD HISTORY / PIPE_USAGE_HISTORY
14 dni
Ograniczony do bieżącej bazy danych.


* Dostep audytowy - `COPY_HISTORY` lub `PIPE_USAGE_HISTORY`
1 rok
dane pojawiaja sie z opoznieniem kilku h
Całe konto (widzisz ładowania we wszystkich bazach).




# Snowpipe Streaming

### Snowpipe Streaming: Czym różni się od "zwykłego" Snowpipe?

- **Brak plików w Stage (No Files):** To najważniejsza różnica. Klasyczny Snowpipe ładuje dane z plików (CSV, JSON itp.) umieszczonych w stage'u. Snowpipe Streaming przesyła **wiersze danych bezpośrednio** do Snowflake bez tworzenia plików pośrednich. 
    
- **Model Ingestu:** Jest to proces "row-based" (wierszowy), a nie "file-based" (plikowy).
    
- **Opóźnienie (Latency):** Snowpipe Streaming oferuje znacznie niższe opóźnienia (sekundy) w porównaniu do klasycznego Snowpipe (minuty), ponieważ eliminuje etap zapisu pliku na S3/Azure/GCS.
    
- **Narzędzia:** Wykorzystuje **Snowflake Ingest SDK** (głównie Java) lub **Snowflake Connector for Kafka**


### Mechanizm "Channels" (Kanały)

- **Definicja:** Kanał to logiczne połączenie między klientem (Twoją aplikacją lub Kafką) a konkretną tabelą w Snowflake.
    
- **Zastosowanie:** Służy do przesyłania danych do tabeli. Jedna tabela może mieć wiele kanałów otwartych jednocześnie (np. z różnych instancji Twojej aplikacji). - Otwieranie i zamykanie kanału (`Channel`) jest kosztowne. Najlepiej trzymać kanał otwarty tak długo, jak długo aplikacja przesyła dane. Snowflake gwarantuje kolejność wierszy **w obrębie jednego kanału**. Jeśli wysyłasz dane przez wiele kanałów do tej samej tabeli, kolejność między kanałami nie jest gwarantowana. Jeśli zmienisz strukturę tabeli (np. dodasz kolumnę), kanał musi zostać odświeżony/ponownie otwarty, aby "zauważyć" zmianę.
    
- **Offset (Przesunięcie):** Kanał śledzi postęp ładowania. Jeśli połączenie zostanie zerwane, SDK może zapytać Snowflake o ostatni pomyślnie zapisany "offset", aby uniknąć duplikatów. Snowpipe Streaming nie usuwa duplikatów automatycznie. Musisz zarządzać offsetami (przesunięciami) po stronie klienta (Twojej aplikacji), aby w razie awarii wiedzieć, od którego momentu wznowić przesyłanie.
    

### "Exactly-once" Delivery (Gwarancja dostarczenia)

Snowflake gwarantuje, że dane wysłane przez Snowpipe Streaming trafią do tabeli dokładnie raz, pod warunkiem poprawnej implementacji w SDK:

- Wykorzystywany jest mechanizm **Client-side sequencing** i **Offset tracking**.
    
- Dzięki temu, nawet jeśli sieć zawiedzie, system wie, od którego momentu wznowić przesyłanie, aby nie stworzyć duplikatów ani nie zgubić danych.

![[Zrzut ekranu 2026-04-25 o 15.28.59.png]]
![[Zrzut ekranu 2026-04-25 o 16.36.37.png]]

![[Zrzut ekranu 2026-04-25 o 16.13.15.png]]


### Ograniczenia Snowpipe Streaming

- Wspiera tylko **Standardowe Tabele** (Permanent) i **Transient Tables** i Dynamic tables
- **NIE wspiera:** Temporary Tables (tabel tymczasowych) oraz External Tables, Materialized Views