[[STORAGE LAYER]]

Storage is calculated and charged for data regardless of whether it is in the Active, Time Travel, or Fail-safe state (updated/deleted data protected by CDP will continue to incur storage costs until the data leaves the Fail-safe state).

If you have been assigned the **ACCOUNTADMIN** role (i.e. you serve as the top-level administrator for your Snowflake account), you can use [Snowsight](https://docs.snowflake.com/en/user-guide/ui-snowsight-gs.html#label-snowsight-getting-started-sign-in) to view data storage across your **entire account**
Singular table - Any user with the appropriate privileges can view data storage for individual tables.

STORAGE AND STAGES
Data files staged in Snowflake internal stages are not subject to the additional costs associated with Time Travel and Fail-safe, but they do incur standard data storage costs. to help manage your storage costs, Snowflake recommends that you monitor these files and remove them from the stages once the data has been loaded and the files are no longer needed. Periodic purging of staged files can have other benefits, such as improved data loading performance.

STORAGE AND CLONING
When a clone is created of a table, the clone utilizes no data storage because it shares all the existing micro-partitions of the original table at the time it was cloned.  The storage associated with these micro-partitions is owned by the oldest table in the clone group and the clone references these micro-partitions. This storage behavior applies to standard tables but not hybrid tables. When you clone a database that contains hybrid tables, additional storage costs are incurred.
however, in standard tables rows can then be added, deleted, or updated in the clone independently from the original table. Each change to the clone results in new micro-partitions that are owned exclusively by the clone and are protected through CDP - that incurs costs. 

