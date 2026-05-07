[[UPRAWNIENIA]]

SHARES
**Kto tworzy Share?** Domyślnie tylko **`ACCOUNTADMIN`**. Można jednak nadać uprawnienie `CREATE SHARE` innej roli (np. `SYSADMIN`), aby mogła zarządzać udziałami.
    
**Własność (Ownership):** Aby dodać tabelę do Share, rola musi posiadać uprawnienie `OWNERSHIP` do tej tabeli (lub posiadać uprawnienie `SELECT` i działać w imieniu roli, która ma prawo do dzielenia się danymi).


READER ACCOUNTS
Tylko **`ACCOUNTADMIN`** (lub rola z uprawnieniem `CREATE ACCOUNT`) może stworzyć Reader Account.