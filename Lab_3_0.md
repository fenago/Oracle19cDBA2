### Real-World Use Case: Optimizing a Financial Data Warehouse

#### Scenario:
Imagine you're a Database Administrator (DBA) at a financial institution that processes large volumes of transactions daily. The organization relies on an Oracle 19c data warehouse to store and analyze transaction data from various sources, including customer purchases, account transfers, and stock trades. This data is used for real-time reporting, fraud detection, and regulatory compliance.

Over time, the database grows rapidly due to the continuous influx of transaction data. As the DBA, you face several challenges:

1. **Managing Temporary Data**:
   - **Challenge**: During nightly batch processing, the ETL (Extract, Transform, Load) jobs generate a significant amount of temporary data. This data is only needed for the duration of the processing and should not persist in the database afterward.
   - **Solution**: Use **Global Temporary Tables** to store intermediate results during the ETL process. These tables allow the temporary data to be discarded automatically at the end of each session or transaction, freeing up space and ensuring that no unnecessary data persists in the database.

2. **Isolated Data Analysis**:
   - **Challenge**: Analysts frequently run complex queries to detect patterns, identify fraud, or simulate financial scenarios. Each analyst’s queries may require working with large, temporary datasets that should remain private to their session.
   - **Solution**: Use **Private Temporary Tables** to provide analysts with session-specific storage that automatically disappears when their session ends. This prevents data from one analyst’s session from interfering with others, maintaining security and privacy.

3. **Reclaiming Wasted Space**:
   - **Challenge**: Over time, as transactions are archived or deleted, tables and indexes can accumulate unused space, leading to inefficient storage usage. For example, the `transactions` table, which logs every customer interaction, has grown substantially, but recent data purges have left it with a lot of unused space.
   - **Solution**: Use **Segment Shrink** to reclaim this wasted space. By shrinking the `transactions` table, you reduce its physical footprint on the disk, optimizing storage and improving query performance. This operation also lowers the storage costs associated with the data warehouse.

4. **Handling Large Operations Without Failure**:
   - **Challenge**: Periodically, large bulk operations are required, such as re-indexing tables or performing mass updates to reflect new financial regulations. These operations are critical but can fail due to insufficient space, disrupting the ETL processes and impacting business operations.
   - **Solution**: Enable **Resumable Space Allocation** to prevent these operations from failing. If an operation runs out of space, it enters a suspended state, allowing you to resolve the issue (e.g., by adding more storage) and then resume the operation without losing progress. This feature ensures that critical bulk operations can be completed smoothly, even under constrained resources.

#### Implementation:

1. **Global Temporary Tables**:
   - During the ETL process, a global temporary table is used to hold transaction data that needs to be transformed. Once the transformation is complete and the data is loaded into the permanent table, the temporary table’s data is automatically purged.

2. **Private Temporary Tables**:
   - Each financial analyst can create a private temporary table to hold the results of their complex queries. These tables are private to their session, and any sensitive information processed is automatically deleted when the session ends.

3. **Segment Shrink**:
   - After monthly data archiving, where older transaction records are moved to a different storage, the `transactions` table is shrunk to reclaim the unused space. This reduces the size of the table on disk and improves query performance.

4. **Resumable Space Allocation**:
   - During the annual re-indexing of the `transactions` table, resumable space allocation is enabled to handle any potential space issues. This ensures that the operation does not fail mid-process, and you have the opportunity to resolve space constraints without restarting the operation.

#### Benefits:

- **Improved Performance**: By reclaiming unused space and optimizing storage, query performance improves, leading to faster report generation and data processing.
- **Resource Efficiency**: Temporary data storage is managed efficiently, ensuring that disk space is only used when necessary and automatically freed up after the session or transaction ends.
- **Operational Resilience**: Resumable space allocation allows critical operations to proceed without interruption, even in the face of space limitations, ensuring business continuity.
- **Security and Privacy**: Private temporary tables ensure that sensitive data is securely managed within individual sessions, reducing the risk of data leakage or unauthorized access.

This real-world scenario demonstrates how the advanced database operations covered in **Lab_3_0** are essential tools for maintaining an efficient, secure, and high-performing financial data warehouse.

# Lab_3_0: Advanced Database Operations in Oracle 19c

## Objectives:
- To create and manage global temporary tables.
- To create and utilize private temporary tables.
- To reclaim space by shrinking segments.
- To manage resumable space allocation for large operations.

## Use Case:
Efficient database management often requires advanced operations to optimize performance and resource usage. This lab will guide you through creating temporary tables for session-specific data handling, shrinking segments to reclaim unused space, and managing resumable space allocation for large operations. These techniques are crucial for maintaining optimal database performance, especially in environments with high transaction volumes or complex queries.

---

## Prerequisites:
- Access to an Oracle 19c Database.
- SQL*Plus or SQL Developer for executing SQL commands.

---

## Main Steps:

### 1. **Creating Global Temporary Tables**

   **Global temporary tables** are useful for storing temporary data that needs to persist for the duration of a session or transaction. The data stored in these tables is session-specific, meaning that different sessions can use the same table name without affecting each other.

   **Using SQL Developer or SQL*Plus:**

   1. **Create a Global Temporary Table:**
      ```sql
      CREATE GLOBAL TEMPORARY TABLE trans_buff_area (
          date1 DATE,
          data_value NUMBER
      ) ON COMMIT DELETE ROWS;
      ```
      - This creates a temporary table where data is deleted at the end of the transaction.

   2. **Session-Specific Data Persistence:**
      - To create a temporary table where data persists for the session:
      ```sql
      CREATE GLOBAL TEMPORARY TABLE session_data_area (
          date1 DATE,
          data_value NUMBER
      ) ON COMMIT PRESERVE ROWS;
      ```

   **Use Case:**
   - Use global temporary tables when you need a workspace for temporary data manipulation during transactions or sessions. For example, during data migration or complex data processing tasks that require temporary storage.

---

### 2. **Creating Private Temporary Tables**

   **Private temporary tables** are session-specific tables that are automatically dropped at the end of a session or transaction. These tables are isolated to the session and cannot be accessed by other sessions.

   **Using SQL Developer or SQL*Plus:**

   1. **Create a Private Temporary Table:**
      ```sql
      CREATE PRIVATE TEMPORARY TABLE ora$ptt_temp_table (
          col1 DATE,
          col2 NUMBER
      ) ON COMMIT PRESERVE DEFINITION;
      ```
      - This table is private to your session and will be automatically dropped when your session ends.

   **Use Case:**
   - Use private temporary tables when you need a secure, session-specific storage space that is not shared with other users or sessions. This is ideal for performing operations that should not persist beyond the current session.

---

### 3. **Reclaiming Space with Segment Shrink**

   Over time, database tables and indexes can accumulate unused space. The **shrink space** feature allows you to reclaim this space, thus optimizing storage usage.

   **Using SQL Developer or SQL*Plus:**

   1. **Shrink a Table Segment:**
      ```sql
      ALTER TABLE employees SHRINK SPACE COMPACT;
      ```
      - This compacts the table, reclaiming unused space while allowing DML operations to continue.

   2. **Full Space Reclamation:**
      ```sql
      ALTER TABLE employees SHRINK SPACE;
      ```
      - This operation reclaims space and adjusts the High Water Mark (HWM), but blocks DML operations during the process.

   **Use Case:**
   - Use segment shrink when you need to reclaim space from tables or indexes that have grown significantly over time and now contain unused space. This is particularly useful after large data deletions or when optimizing storage for better performance.

---

### 4. **Managing Resumable Space Allocation**

   **Resumable space allocation** allows certain database operations to be suspended rather than fail when they run out of space. This feature provides an opportunity to resolve space issues and resume the operation without losing progress.

   **Using SQL Developer or SQL*Plus:**

   1. **Enable Resumable Mode:**
      ```sql
      ALTER SESSION ENABLE RESUMABLE TIMEOUT 7200;
      ```
      - This enables resumable space allocation with a timeout of 2 hours.

   2. **Test Resumable Operation:**
      - You can test this by running a large insert operation that could potentially run out of space:
      ```sql
      INSERT INTO large_table SELECT * FROM source_table;
      ```

   **Use Case:**
   - Use resumable space allocation when performing large operations that could potentially run out of space, such as bulk inserts, index creation, or partition operations. This ensures that you have time to address space issues without losing the operation's progress.

---

## Summary:
In this lab, you have learned how to perform advanced Oracle Database operations such as creating and managing global and private temporary tables, shrinking segments to reclaim space, and managing resumable space allocation. These skills are essential for efficient database management, particularly in environments with high transaction volumes or complex queries.

### Addendum to Lab_3_0: Verifying and Managing Files Created on Disk

#### Objectives:
- To identify and explore the specific files created or modified on the disk as a result of operations in **Lab_3_0**.
- To validate the creation and management of these files using SQL Developer or SQL*Plus.

---

### 1. **Identifying Specific Files Created on Disk**

After performing the operations in **Lab_3_0**, several files on your disk will have been created or modified. Here’s a breakdown of what you should expect:

1. **Global Temporary Tables**:
   - When you create a global temporary table, the data associated with these tables is stored in the temporary tablespace. The specific file you should look for is:
   - **Temporary Tablespace Data File:**
     - **File Name:** Typically named `TEMP01.DBF` or similar.
     - **Location:** 
       ```bash
       /u01/app/oracle/oradata/ORCL/temp/
       ```
     - **Command to Locate:**
       ```bash
       ls -l /u01/app/oracle/oradata/ORCL/temp/
       ```
     - **Expected Result:** A file like `TEMP01.DBF` should be present in this directory, which is used by Oracle to manage data for global temporary tables.

2. **Private Temporary Tables**:
   - Private temporary tables do not persist beyond the session and do not create permanent data files on the disk. However, their activity will still involve the temporary tablespace during their session. Therefore, the same `TEMP01.DBF` file is used, but no new files are created specifically for private temporary tables.
     - **Key Point:** No additional files are created for private temporary tables. The `TEMP01.DBF` file handles all temporary data, both for global and private temporary tables.

3. **Shrinking Segments**:
   - When you perform a segment shrink operation, the data files associated with the affected tables are modified to reclaim unused space. For example, if you performed a shrink operation on the `employees` table:
   - **Data File for `employees` Table:**
     - **File Name:** The file name will vary depending on how your database is configured, but it will typically have a `.DBF` extension and be located in the data directory for the tablespace that houses the `employees` table.
     - **Location:**
       ```bash
       /u01/app/oracle/oradata/ORCL/users01.dbf
       ```
     - **Command to Locate:**
       ```bash
       ls -l /u01/app/oracle/oradata/ORCL/
       ```
     - **Expected Result:** Look for files like `users01.dbf` (if the `employees` table is stored in the `USERS` tablespace). This file should reflect the shrink operation, with its size reduced accordingly.

---

### 2. **Validating Files and Operations Using SQL Developer or SQL*Plus**

After identifying the specific files on disk, validate that the operations were successful and that the database is properly managing these files.

**Steps to Validate Using SQL Developer or SQL*Plus:**

1. **List Temporary Tablespaces:**
   - To confirm which temporary tablespaces are being used by the global and private temporary tables:
   ```sql
   SELECT tablespace_name, file_name
   FROM dba_temp_files;
   ```
   - **Expected Output:** You should see the `TEMP` tablespace and its associated data files like `TEMP01.DBF`.

2. **Validate Segment Shrink for the `employees` Table:**
   - Confirm the space usage before and after the shrink operation:
   ```sql
   SELECT segment_name, tablespace_name, bytes/1024/1024 AS size_mb
   FROM user_segments
   WHERE segment_name = 'EMPLOYEES';
   ```
   - **Expected Output:** The `size_mb` value should be smaller after the shrink operation, indicating that space has been reclaimed.

3. **Check Data File Locations:**
   - Verify the location and size of the data files after performing the shrink operation:
   ```sql
   SELECT file_name, tablespace_name, bytes/1024/1024 AS size_mb
   FROM dba_data_files;
   ```
   - **Expected Output:** The file associated with the `USERS` tablespace, such as `users01.dbf`, should show a reduced size, reflecting the space reclaimed by the shrink operation.

4. **Monitor Resumable Operations:**
   - If you have enabled resumable operations, monitor any suspended operations:
   ```sql
   SELECT session_id, sql_text, start_time, error_number, error_message
   FROM dba_resumable;
   ```
   - **Expected Output:** If there were any resumable operations triggered by lack of space, they will be listed here along with their status.

---

### 3. **Exploring and Managing Data Files on Disk**

If you need to manage or further explore these files, follow these steps:

1. **Resize Data Files:**
   - To manually resize data files after confirming the space reclaimed:
   ```sql
   ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/ORCL/users01.dbf' RESIZE 200M;
   ```

2. **Move Data Files:**
   - If you need to move data files to a different directory:
   ```sql
   ALTER DATABASE DATAFILE '/u01/app/oracle/oradata/ORCL/users01.dbf'
   TO '/u02/app/oracle/oradata/ORCL/users01.dbf';
   ```

3. **Check Free Space in Tablespaces:**
   - To monitor the free space available in your tablespaces after performing these operations:
   ```sql
   SELECT tablespace_name, file_name, bytes/1024/1024 AS size_mb, 
          maxbytes/1024/1024 AS max_size_mb, 
          autoextensible 
   FROM dba_data_files;
   ```

---

### Summary:
This addendum has provided specific instructions to identify, validate, and manage the files created or modified on disk as a result of the operations in **Lab_3_0**. By understanding the exact files involved—like `TEMP01.DBF` for temporary operations and `users01.dbf` for the `employees` table—you gain deeper insights into how Oracle manages storage and how to effectively monitor and manage these resources using SQL Developer or SQL*Plus.

---
