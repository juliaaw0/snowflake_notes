![[Zrzut ekranu 2026-05-4 o 00.07.35.png]]


TEMPORARY [[TABLES]]

In addition to permanent tables, which is the default table type when creating tables, Snowflake supports defining tables as either temporary or transient. These types of tables are especially useful for storing data that does not need to be maintained for extended periods of time (i.e. transitory data).

 Temporary tables only exist within the session in which they were created and persist only for the remainder of the session. As such, they are not visible to other users or sessions. Once the session ends, data stored in the table is purged completely from the system and, therefore, is not recoverable, either by the user who created the table or Snowflake. For the duration of the existence of a temporary table, the data stored in the table contributes to the overall storage charges that Snowflake bills your account. Żeby nie generowac duzych kosztow temporary tables w sesji poleca się dropnac je recznie po skonczeniu uzywania albo wylogowac z sesji

temporary tables belong to a specified database and schema; however, because they are session-based, they aren’t bound by the same uniqueness requirements. This means you can create temporary and non-temporary tables with the same name within the same schema. the temporary table takes precedence in the session over any other table with the same name in the same schema. This can lead to potential conflicts - as the normal table is hidden by the temporary table - all queries and other operations performed in the session on the table affect only the temporary table.

Temporary tables can have a Time Travel retention period of 1 day; however, a temporary table is purged once the session (in which the table was created) ends so the actual retention period is for 24 hours or the remainder of the session, whichever is shorter.

**A connection and a session are different concepts within Snowflake. When logged into Snowflake, one or more sessions may be created. A Snowflake session is only terminated if the user explicitly terminates the session or the session times out due to inactivity after 4 hours. Disconnecting from Snowflake does not terminate the active sessions. Thus, a Snowflake session may be very long-lived and any temporary tables created within that session will continue to exist until they are dropped or the session is terminated.**

To avoid unexpected storage costs for temporary tables, Snowflake recommends creating them as needed within a session and dropping them when they are no longer required.

Uwaga:  You cannot create [hybrid tables](https://docs.snowflake.com/en/user-guide/tables-hybrid) that are temporary or transient. you cannot create hybrid tables within transient schemas or databases

In addition to tables, Snowflake supports creating certain other database objects as temporary (e.g. stages). These objects follow the same semantics (i.e. they are session-based, persisting only for the remainder of the session).




TRANSIENT TABLES

tables that persist until explicitly dropped and are available to all users with the appropriate privileges. Transient tables are similar to permanent tables with the key difference that they do not have a Fail-safe period (and costs). As a result, transient tables are specifically designed for transitory data that needs to be maintained beyond each session (in contrast to temporary tables), but does not need the same level of data protection and recovery provided by permanent tables.

Snowflake also supports creating transient databases and schemas. All tables created in a transient schema, as well as all schemas created in a transient database, are transient by definition.

Transient tables can have a Time Travel retention period of 1 day;

Transient and temporary tables have no Fail-safe period. Because transient tables do not have a Fail-safe period, they provide a good option for managing the cost of very large tables used to store transitory data; however, the data in these tables cannot be recovered after the Time Travel retention period passes.

PYTANIE EGZ:
As part of a data processing pipeline, you are required to store data in an interim table. The subsequent processes then use the table in the pipeline. The data is deleted and reloaded every time the pipeline is executed. You are required to minimize data storage costs. Which type of table will you create?
Based on the requirement, a Transient table is a good option. Transient tables don't have fail-safe storage and have only up to 1 day of Time Travel. Because the data in this table is deleted and reloaded daily, a transient table provides the best solution. Transient tables are available across sessions, so different processes and sessions can access them.