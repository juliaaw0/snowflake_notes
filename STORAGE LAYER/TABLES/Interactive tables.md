[[TABLES]]

INTERACTIVE TABLES (tylko istnieja jako preview)

Bonus: co to sa interactive warehouses i interactive tables:

Tak naprawdę Interactive Tables to po prostu Hybrid Tables, które są zoptymalizowane pod jeszcze niższą latencję dzięki połączeniu ich z Interactive Warehouses.

To specjalny typ wirtualnego magazynu (warehouse) w Snowflake zoptymalizowany pod niską latencję zapytań i wysoką współbieżność.

Jest projektowany tak, aby jak najszybciej odpowiadać na krótkie, „interaktywne” zapytania (np. dashboardy, API, aplikacje działające w czasie rzeczywistym).

Kluczowe cechy:

- Tunelowane optymalizacje silnika Snowflake pod małe, szybkie zapytania.
- Zawsze „ON” — nie wstrzymuje się automatycznie przy bezczynności (co odróżnia je od standardowych warehouse).
- Domyślny timeout SELECT = 5s, więc dłuższe zapytania zostaną przerwane jeśli trwają zbyt długo.
- Może obsługiwać wysoką liczbę równoległych zapytań, ale nie auto-skalowanie w multi-cluster (min=max).
- Może zapytywać jedynie Interactive Tables

Interactive tables wymagaja cluster by przy tworzeniu
