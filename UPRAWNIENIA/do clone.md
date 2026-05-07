[[UPRAWNIENIA]]

- **Uprawnienia do klonowania:** Aby sklonować obiekt, musisz mieć uprawnienie SELECT do obiektu źródłowego (i bazy/schematu) oraz uprawnienie `CREATE [OBJECT]` w miejscu docelowym.
- **Uprawnienia w klonie:** Sklonowany obiekt **nie dziedziczy** uprawnień (Grants) z oryginału. Jest "czysty". Wyjątek: jeśli klonujesz całą bazę lub schemat, możesz użyć opcji `COPY GRANTS` (ale domyślnie ich nie ma).


