[[CACHES]]

Column store data cache-  Hybrid Tables, mimo że służą do OLTP, pod spodem utrzymują też format kolumnowy, aby zapytania typu "zsumuj sprzedaż z dzisiaj" również były szybkie. Dane te lądują w cache'u, aby kolejne zapytania nie musiały sięgać do wolniejszej pamięci dyskowej.
