[[UPRAWNIENIA]]

Domyślnie uprawnienia do warehousow CREATE WAREHOUSE przyznany jest tylko rolom o najwyższych uprawnieniach:

- SYSADMIN
- ACCOUNTADMIN

Ale można je przyznac tez innym rolom. Inne systemowe role jak SECURITYADMIN czy USERADMIN domyślnie nie mają tego przywileju – służą głównie do zarządzania dostępem, rolami i użytkownikami, a nie zasobami compute.

aby inna (niestandardowa) rola mogła tworzyć warehouses, musisz jej nadać przywilej:

GRANT CREATE WAREHOUSE ON ACCOUNT TO ROLE nazwa_roli;

CREATE WAREHOUSE ETL_VWH WITH

WAREHOUSE_SIZE = 'SMALL'

WAREHOUSE_TYPE = 'STANDARD'

AUTO_SUSPEND = 300

AUTO_RESUME = TRUE;