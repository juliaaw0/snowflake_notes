[[UPRAWNIENIA]]

COPY INTO (BULK):

- Aby załadować dane, rola musi posiadać:
    - `USAGE` na Database i Schema.
    - `INSERT` na docelowej tabeli.
    - `USAGE` na Stage (jeśli używamy nazwanego stage'a).


SNOWPIPE:
![[Zrzut ekranu 2026-04-25 o 16.15.42.png]]


SNOWPIPE STREAMING: 
![[Zrzut ekranu 2026-04-25 o 16.14.55.png]]



DO UNLOAD:
**Uprawnienia:** Aby wyeksportować dane, Twoja rola musi mieć:

- `SELECT` na tabeli źródłowej.
    
- `WRITE` na stage'u (jeśli jest internal).
    
- Uprawnienie do _Storage Integration_ (jeśli eksportujesz bezpośrednio do S3/Azure/GCS).