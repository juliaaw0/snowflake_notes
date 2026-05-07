[[UPRAWNIENIA]]

 Aby użytkownik mógł odpytać widok, musi mieć uprawnienie `SELECT` do widoku oraz `USAGE` do bazy i schematu. **Ciekawostka:** Jeśli właściciel widoku ma uprawnienia do tabel źródłowych, to użytkownik końcowy odpytujący widok **nie potrzebuje** bezpośrednich uprawnień do tych tabel

uprawnienia w secure views:
W Secure View definicja widoku w SQL jest widoczna **tylko dla właściciela** (roli z uprawnieniem OWNERSHIP) lub ról nadrzędnych.
