
## Najważniejsza zasada (często pytanie na egzaminie)

👉 **Snowflake NIE wymaga tradycyjnych backupów**

Dlaczego?

- architektura oparta o **storage + metadata**
- dane są:
    - automatycznie replikowane
    - przechowywane w durable storage (np. S3, Azure Blob)

📌 **Wniosek egzaminacyjny:**

> Backupy są „wbudowane”, nie robisz ich ręcznie jak w klasycznych bazach.

---

## 🔁 2. Co zastępuje backupy w Snowflake?

### ✅ a) **Time Travel**

- pozwala cofnąć dane do przeszłości
- działa na:
    - tabelach
    - schematach
    - bazach

📌 zakres:

- Standard: **1 dzień**
- Enterprise: **do 90 dni**

📌 użycia:

- odzyskanie usuniętych danych
- undo operacji (DELETE, UPDATE)

---

### 🧊 b) **Fail-safe**

- automatyczne po Time Travel
- trwa **7 dni**
- tylko dla Snowflake (support)

📌 ważne:

- ❌ NIE masz dostępu samodzielnie
- ❌ NIE do codziennego recovery

📌 pytanie egzaminacyjne:

> Fail-safe ≠ backup dla użytkownika

---

### 🌍 c) **Replication**

- kopia danych do innego regionu / konta
- ochrona przed:
    - awarią regionu
    - disaster recovery

📌 typy:

- cross-region
- cross-cloud

---

### 🔄 d) **Failover**

- przełączenie na replikę
- używane razem z replication