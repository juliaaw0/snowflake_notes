[[CHANGE DATA CAPTURE (CDC)]]

Stream to **obiekt bazy danych**, który **śledzi zmiany (DML)** dokonane na tabeli źródłowej (lub widoku/tabeli katalogowej). Stream **nie przechowuje danych** tabeli. Przechowuje jedynie minimalne metadane o zmianach i sumy kontrolne wierszy, aby móc wyciągnąć różnice (delta).
Np:

INSERT INTO PRODUKTY VALUES (1, 'Chleb', 5);

| **ID** | **NAZWA** | **CENA** | **METADATA$ACTION** | **METADATA$ISUPDATE** | **METADATA$ROW_ID** |
| ------ | --------- | -------- | ------------------- | --------------------- | ------------------- |
| 1      | Chleb     | 5        | **INSERT**          | **FALSE**             | _jakis_kod_123_     |
UPDATE PRODUKTY SET cena = 7 WHERE id = 1;

|**ID**|**NAZWA**|**CENA**|**METADATA$ACTION**|**METADATA$ISUPDATE**|**METADATA$ROW_ID**|
|---|---|---|---|---|---|
|1|Chleb|5|**DELETE**|**TRUE**|_jakis_kod_123_|
|1|Chleb|7|**INSERT**|**TRUE**|_jakis_kod_123_|

### NA PAMIEC:
Każdy Stream dodaje do Twoich danych trzy wirtualne kolumny, które pozwalają zidentyfikować rodzaj zmiany:

1. **`METADATA$ACTION`**: Wskazuje na akcję: `INSERT` lub `DELETE`.
    
2. **`METADATA$ISUPDATE`**: Typu Boolean (`TRUE`/`FALSE`). Mówi, czy wiersz jest częścią operacji `UPDATE`.

3. **`METADATA$ROW_ID`**: Unikalny identyfikator wiersza (służy do śledzenia zmian w czasie).
    
**Jak interpretować UPDATE?** W SQL `UPDATE` to jedna operacja, ale w Streamie są to **dwa wiersze**:

- Wiersz z akcją `DELETE` i `METADATA$ISUPDATE = TRUE`.
- Wiersz z akcją `INSERT` i `METADATA$ISUPDATE = TRUE`.



### Typy Strumieni (Stream Types)

**Ważność na egzaminie:** ⭐⭐⭐⭐

1. **Standard (Default):** Śledzi wszystkie zmiany DML (`INSERT`, `UPDATE`, `DELETE`).
    
2. **Append-only:** Śledzi **tylko INSERTy**. Jest bardziej wydajny dla tabel, do których tylko dopisujemy dane (np. logi), bo ignoruje operacje kasowania i aktualizacji.
    
3. **Insert-only:** Specjalny typ używany wyłącznie dla **External Tables**. Śledzi tylko nowe pliki pojawiające się w zewnętrznym stage'u.


### !!!!!!!!!!!
### Kluczowy moment: Konsumpcja (To jest najważniejsze na egzamin!)

Pytasz: "Jak strumień wygląda po otwarciu?". Odpowiedź brzmi: tak długo, jak tylko robisz `SELECT` * from strumień, on będzie pokazywał te zmiany (chleb za 5 i chleb za 7).

Ale jeśli użyjesz tych danych, żeby coś zrobić, np.:

```
INSERT INTO TABELA_RAPORTOWA SELECT * FROM produkt_stream;
```

W momencie, gdy ta operacja się uda (**COMMIT**), Stream robi się **pusty**. Snowflake uznaje: "Okej, użytkownik już przetworzył te zmiany, przesuwam moją zakładkę na koniec". Jeśli teraz dodasz 'Masło' do tabeli `PRODUKTY`, Stream pokaże tylko 'Masło'.



### Retencja i "Staleness" (Przedawnienie)

**Ważność na egzaminie:** ⭐⭐⭐⭐

Streams polegają na mechanizmie **Time Travel** tabeli źródłowej.

- **Zależność:** Jeśli okres retencji tabeli (Time Travel) wynosi 1 dzień, to teoretycznie masz 1 dzień na skonsumowanie strumienia.
    
- **Extended Stream Retention:** Snowflake stara się chronić strumień przed przedawnieniem. Rozszerza okres retencji tabeli źródłowej (maksymalnie do 14 dni lub do wartości `MAX_DATA_EXTENSION_TIME_IN_DAYS`), aby zapobiec sytuacji, w której strumień staje się "stale" (nieaktualny).
    
- **Stale Stream:** Jeśli nie skonsumujesz strumienia w wyznaczonym oknie, staje się on **Stale** (nieużyteczny). Musisz go wtedy usunąć i stworzyć na nowo, ale **stracisz historię zmian**, których nie zdążyłeś pobrać.



_Pytanie:_ Użytkownik wykonał `SELECT * FROM my_stream` trzy razy pod rząd. Co zobaczył? _Odpowiedź:_ Za każdym razem to samo, ponieważ samo zapytanie `SELECT` nie przesuwa offsetu strumienia. Strumień zostanie "wyczyszczony" dopiero po użyciu go np. w komendzie `INSERT INTO ... SELECT * FROM my_stream`.