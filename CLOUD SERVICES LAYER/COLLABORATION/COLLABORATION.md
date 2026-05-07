
![[Zrzut ekranu 2026-04-29 o 23.40.10.png]]




### Czym jest Data Clean Room (DCR)?

**Ważność na egzaminie:** ⭐⭐⭐ (Koncepcja)

Wyobraź sobie DCR jako **"bezpieczny, wirtualny pokój spotkań"** dla danych.

- **Problem:** Dwie firmy (np. Bank i Sklep) chcą sprawdzić, ilu mają wspólnych klientów, ale prawo (RODO/GDPR) zabrania im pokazywania sobie nawzajem list nazwisk czy maili.
    
- **Rozwiązanie (DCR):** Obie strony "wkładają" swoje dane do Clean Roomu. Żadna ze stron nie może podejrzeć surowych danych drugiej strony. Mogą jedynie uruchomić **zatwierdzone niezmienialne zapytania** (templates), które zwrócą wynik zagregowany (np. "Macie 5000 wspólnych klientów"). Konsument może tylko uzupełnić parametry (np. wybrać konkretne miasto), ale nie może zmienić struktury zapytania.

* **Data Offerings:** Konkretne zbiory danych, które Provider udostępnia wewnątrz Clean Roomu.*

* Aktywacja: - Clean Room znajduje 1000 wspólnych klientów banku i linii lotniczych. **Aktywacja** to przesłanie listy tych (zakodowanych/zaszyfrowanych) ID do platformy marketingowej (np. Facebook Ads czy Google Ads), aby wyświetlić im reklamy. Surowe dane nadal nie są widoczne dla żadnej ze stron, ale wynik analizy staje się "używalny".