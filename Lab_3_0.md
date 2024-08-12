# Lab_3_0: Advanced Storage Management in Oracle Databases

## Objectives:
- To understand the creation and usage of private temporary tables.
- To implement data compression to save space in Oracle databases.
- To reclaim wasted space from tables and indexes using the segment shrink functionality.
- To manage resumable space allocation in Oracle databases.

## Use Case:
As your Oracle Database grows, managing storage effectively becomes increasingly important. This lab will guide you through advanced storage management techniques that help in optimizing space usage. You'll learn how to create private temporary tables for session-specific operations, use compression to reduce storage space, reclaim unused space in tables and indexes, and manage resumable space allocation to handle large transactions efficiently.

## Prerequisites:
- Completion of the previous lab (Lab 2) where you learned about creating and managing tablespaces.
- Access to the Oracle Database system as the `oracle` user.
- SQL*Plus or SQL Developer for executing SQL commands.

## Optional Steps:

### 1. **Identify and Stop All Running Databases and Listeners**
   (Refer to Lab 1 if needed)

### 2. **Start the CDB and PDBs**
   (Use the same startup commands as Lab 1 to ensure that CDBLAB and the PDBs are running.)

## Main Steps:

### 3. **Creating Private Temporary Tables**

   **In the CDB or PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Open **SQL Developer**.
   2. In the **Connections** pane, right-click on `CDBLAB` or `PDBLAB1` and select **Connect**.
   3. Once connected, click on the **SQL Worksheet** icon to open a new worksheet.
   4. In the worksheet, execute the following SQL command to create a private temporary table:
   ```sql
   CREATE PRIVATE TEMPORARY TABLE ora$ptt_temp_table
   ON COMMIT PRESERVE DEFINITION AS
   SELECT * FROM employees;
   ```
   5. Click the **Run Script** button (or press F5) to execute the command.

   **Using SQL*Plus:**
   1. Open a terminal and log in as the `oracle` user:
      ```bash
      su - oracle
      ```
   2. Connect to SQL*Plus:
      ```bash
      sqlplus / as sysdba
      ```
   3. Connect to the desired CDB or PDB:
      ```sql
      CONNECT system@PDBLAB1
      ```
   4. Execute the following command to create the private temporary table:
      ```sql
      CREATE PRIVATE TEMPORARY TABLE ora$ptt_temp_table
      ON COMMIT PRESERVE DEFINITION AS
      SELECT * FROM employees;
      ```

   **Confirmation:**
   1. In SQL Developer, execute the following command to confirm the creation:
   ```sql
   SELECT table_name, temporary FROM user_tables WHERE table_name = 'ORA$PTT_TEMP_TABLE';
   ```
   2. In SQL*Plus, use the same SQL command:
   ```sql
   SELECT table_name, temporary FROM user_tables WHERE table_name = 'ORA$PTT_TEMP_TABLE';
   ```

### 4. **Saving Space by Using Compression**

   **In the CDB or PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Connect to `PDBLAB1` in SQL Developer.
   2. Open a **SQL Worksheet** and execute the following SQL command to create a table with compression enabled:
   ```sql
   CREATE TABLE employees_compressed
   COMPRESS FOR OLTP
   AS SELECT * FROM employees;
   ```
   3. Click the **Run Script** button to execute.

   **Using SQL*Plus:**
   1. Connect to `PDBLAB1` in SQL*Plus:
      ```sql
      CONNECT system@PDBLAB1
      ```
   2. Execute the following command to create the compressed table:
      ```sql
      CREATE TABLE employees_compressed
      COMPRESS FOR OLTP
      AS SELECT * FROM employees;
      ```

   **Confirmation:**
   1. In SQL Developer, execute the following command to confirm the creation and compression:
   ```sql
   SELECT table_name, compression FROM user_tables WHERE table_name = 'EMPLOYEES_COMPRESSED';
   ```
   2. In SQL*Plus, use the same SQL command:
   ```sql
   SELECT table_name, compression FROM user_tables WHERE table_name = 'EMPLOYEES_COMPRESSED';
   ```

### 5. **Reclaiming Wasted Space Using Segment Shrink Functionality**

   **In the CDB or PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Connect to `PDBLAB1` in SQL Developer.
   2. Open a **SQL Worksheet** and execute the following SQL command to shrink the space used by a table:
   ```sql
   ALTER TABLE employees ENABLE ROW MOVEMENT;
   ALTER TABLE employees SHRINK SPACE;
   ```
   3. Click the **Run Script** button to execute.

   **Using SQL*Plus:**
   1. Connect to `PDBLAB1` in SQL*Plus:
      ```sql
      CONNECT system@PDBLAB1
      ```
   2. Execute the following commands to shrink the space used by the table:
      ```sql
      ALTER TABLE employees ENABLE ROW MOVEMENT;
      ALTER TABLE employees SHRINK SPACE;
      ```

   **Confirmation:**
   1. In SQL Developer, execute the following command to check the space usage before and after the shrink operation:
   ```sql
   SELECT table_name, bytes/1024/1024 AS size_mb FROM user_segments WHERE segment_name = 'EMPLOYEES';
   ```
   2. In SQL*Plus, use the same SQL command:
   ```sql
   SELECT table_name, bytes/1024/1024 AS size_mb FROM user_segments WHERE segment_name = 'EMPLOYEES';
   ```

### 6. **Managing Resumable Space Allocation**

   **In the CDB or PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Connect to `PDBLAB1` in SQL Developer.
   2. Open a **SQL Worksheet** and execute the following command to enable resumable space allocation:
   ```sql
   ALTER SESSION ENABLE RESUMABLE TIMEOUT 7200;
   ```
   3. Click the **Run Script** button to execute.

   **Using SQL*Plus:**
   1. Connect to `PDBLAB1` in SQL*Plus:
      ```sql
      CONNECT system@PDBLAB1
      ```
   2. Execute the following command to enable resumable space allocation:
      ```sql
      ALTER SESSION ENABLE RESUMABLE TIMEOUT 7200;
      ```

   **Testing Resumable Space Allocation:**
   1. Create a large table to simulate running out of space:
   ```sql
   CREATE TABLE large_table AS SELECT * FROM dba_objects;
   INSERT INTO large_table SELECT * FROM large_table;
   ```
   2. If the operation is interrupted due to space issues, the session will enter a suspended state, allowing you to resolve the issue and resume the operation.

   **Resolving Space Issues:**
   1. In SQL Developer or SQL*Plus, execute the following command to increase the tablespace size:
   ```sql
   ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/CDBLAB/pdb1lab_users01.dbf' RESIZE 50M;
   ```
   2. Once resolved, the suspended session will resume automatically.

   **Confirmation:**
   1. Monitor the status of resumable transactions:
   ```sql
   SELECT session_id, name, sql_text FROM dba_resumable;
   ```

### 7. **Summary:**
In this lab, you learned how to manage advanced storage operations within Oracle Databases. You explored the creation of private temporary tables, implemented data compression to save space, reclaimed wasted space using segment shrink functionality, and managed resumable space allocation for large transactions. These techniques are critical for maintaining an efficient and optimized database environment.
