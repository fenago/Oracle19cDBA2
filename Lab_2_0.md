# Lab_2_0: Creating and Managing Tablespaces

## Objectives:
- To understand the process of creating permanent and temporary tablespaces in both the Container Database (CDB) and Pluggable Databases (PDBs).
- To learn how to define default tablespaces for users.
- To manage tablespace information through SQL commands.
- To implement Oracle Managed Files (OMF) for tablespace creation.

## Use Case:
After setting up a CDB and multiple PDBs, the next logical step in database administration is managing tablespaces. Tablespaces are essential for organizing and storing data within the database. This lab will walk you through creating, managing, and querying tablespaces to ensure efficient data management within the CDB and PDBs.

## Prerequisites:
- Completion of the previous lab (Lab 1) where the CDBLAB and PDBLAB1, PDBLAB2, and PDBLAB3 were created.
- Access to the Oracle Database system as the `oracle` user.
- SQL*Plus or SQL Developer for executing SQL commands.

## Optional Steps:

### 1. **Identify and Stop All Running Databases and Listeners**
   (Refer to Lab 1 if needed)

### 2. **Start the CDB and PDBs**
   (Use the same startup commands as Lab 1 to ensure that CDBLAB and the PDBs are running.)

## Main Steps:

### 3. **Creating Permanent Tablespaces**

   **In the CDB:**

   **Using SQL Developer:**
   1. Open **SQL Developer**.
   2. In the **Connections** pane, right-click on `CDBLAB` and select **Connect**.
   3. Once connected, click on the **SQL Worksheet** icon to open a new worksheet.
   4. In the worksheet, execute the following SQL command to create a tablespace:
   ```sql
   CREATE TABLESPACE tbs_CDBLAB_users
   DATAFILE '/u01/app/oracle/oradata/CDBLAB/cdblab_users01.dbf' SIZE 20M;
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
   3. Connect to the CDBLAB:
      ```sql
      CONNECT system@CDBLAB
      ```
   4. Execute the following command:
      ```sql
      CREATE TABLESPACE tbs_CDBLAB_users
      DATAFILE '/u01/app/oracle/oradata/CDBLAB/cdblab_users01.dbf' SIZE 20M;
      ```

   **In a PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. In the **Connections** pane, right-click on `PDBLAB1` and select **Connect**.
   2. Open a new **SQL Worksheet** by clicking the corresponding icon.
   3. Execute the following SQL command to create a tablespace:
   ```sql
   CREATE TABLESPACE tbs_PDBLAB1_users
   DATAFILE '/u01/app/oracle/oradata/CDBLAB/pd1lab/pd1lab_users01.dbf' SIZE 20M;
   ```
   4. Click the **Run Script** button to execute.

   **Using SQL*Plus:**
   1. In the terminal where you are logged into SQL*Plus, connect to PDBLAB1:
      ```sql
      CONNECT system@PDBLAB1
      ```
   2. Execute the following command:
      ```sql
      CREATE TABLESPACE tbs_PDBLAB1_users
      DATAFILE '/u01/app/oracle/oradata/CDBLAB/pd1lab/pd1lab_users01.dbf' SIZE 20M;
      ```

   **Confirmation:**
   1. In SQL Developer, execute the following command to confirm the creation:
   ```sql
   SELECT tablespace_name, status FROM dba_tablespaces;
   ```
   2. In SQL*Plus, use the same SQL command:
   ```sql
   SELECT tablespace_name, status FROM dba_tablespaces;
   ```

### 4. **Defining Default Permanent Tablespaces**

   **In the CDB:**

   **Using SQL Developer:**
   1. Connect to the `CDBLAB` in SQL Developer.
   2. Open a **SQL Worksheet** and execute the following:
   ```sql
   ALTER DATABASE DEFAULT TABLESPACE tbs_CDBLAB_users;
   ```
   3. Run the script to set the default tablespace.

   **Using SQL*Plus:**
   1. Connect to the CDBLAB if not already connected:
      ```sql
      CONNECT system@CDBLAB
      ```
   2. Execute the command:
      ```sql
      ALTER DATABASE DEFAULT TABLESPACE tbs_CDBLAB_users;
      ```

   **In a PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Connect to `PDBLAB1` in SQL Developer.
   2. Open a **SQL Worksheet** and execute:
   ```sql
   ALTER PLUGGABLE DATABASE DEFAULT TABLESPACE tbs_PDBLAB1_users;
   ```
   3. Run the script to set the default tablespace.

   **Using SQL*Plus:**
   1. Connect to PDBLAB1:
      ```sql
      CONNECT pdb1_admin@PDBLAB1
      ```
   2. Execute the command:
      ```sql
      ALTER PLUGGABLE DATABASE DEFAULT TABLESPACE tbs_PDBLAB1_users;
      ```

   **Confirmation:**
   1. In either SQL Developer or SQL*Plus, execute:
   ```sql
   SELECT PROPERTY_VALUE FROM DATABASE_PROPERTIES WHERE PROPERTY_NAME='DEFAULT_PERMANENT_TABLESPACE';
   ```
   2. This should return the name of the tablespace that was set as the default.

### 5. **Creating Temporary Tablespaces**

   **In the CDB:**

   **Using SQL Developer:**
   1. Connect to `CDBLAB` in SQL Developer.
   2. Open a **SQL Worksheet** and execute:
   ```sql
   CREATE TEMPORARY TABLESPACE temp_CDBLAB
   TEMPFILE '/u01/app/oracle/oradata/CDBLAB/temp_CDBLAB01.dbf' SIZE 10M;
   ```

   **Using SQL*Plus:**
   1. Connect to CDBLAB:
      ```sql
      CONNECT system@CDBLAB
      ```
   2. Execute:
      ```sql
      CREATE TEMPORARY TABLESPACE temp_CDBLAB
      TEMPFILE '/u01/app/oracle/oradata/CDBLAB/temp_CDBLAB01.dbf' SIZE 10M;
      ```

   **In a PDB (e.g., PDBLAB1):**

   **Using SQL Developer:**
   1. Connect to `PDBLAB1`.
   2. Open a **SQL Worksheet** and execute:
   ```sql
   ALTER DATABASE DEFAULT TEMPORARY TABLESPACE temp_PDBLAB1;
   ```

   **Using SQL*Plus:**
   1. Connect to PDBLAB1:
      ```sql
      CONNECT pdb1_admin@PDBLAB1
      ```
   2. Execute:
      ```sql
      ALTER DATABASE DEFAULT TEMPORARY TABLESPACE temp_PDBLAB1;
      ```

   **Confirmation:**
   1. In either SQL Developer or SQL*Plus, execute:
   ```sql
   SELECT tablespace_name FROM dba_temp_files;
   ```
   2. This will display the temporary tablespaces created.

### 6. **Viewing Tablespace Information**

   **Tablespace Information:**

   **Using SQL Developer:**
   1. Connect to the desired database (CDB or PDB).
   2. Open a **SQL Worksheet** and execute:
   ```sql
   SELECT * FROM CDB_TABLESPACES WHERE con_id = 1;  -- For CDBLAB
   SELECT * FROM DBA_TABLESPACES;  -- For the connected PDB
   ```

   **Using SQL*Plus:**
   1. Connect to the desired database.
   2. Execute the following:
   ```sql
   SELECT * FROM CDB_TABLESPACES WHERE con_id = 1;  -- For CDBLAB
   SELECT * FROM DBA_TABLESPACES;  -- For the connected PDB
   ```

   **Datafile Information:**
   ```sql
   SELECT file_name, tablespace_name FROM DBA_DATA_FILES;
   ```

   **Temporary File Information:**
   ```sql
   SELECT file_name, tablespace_name FROM DBA_TEMP_FILES;
   ```

### 7. **Implementing Oracle Managed Files (OMF)**

   Oracle Managed Files simplify database file management by allowing the Oracle Database to automatically create and manage files.

   **Set OMF Parameters:**

   **Using SQL Developer:**
   1. Connect to `CDBLAB`.
   2. Open a **SQL Worksheet** and execute:
   ```sql
   ALTER SYSTEM SET DB_CREATE_FILE_DEST='/u01

/app/oracle/oradata';
   ```

   **Using SQL*Plus:**
   1. Connect to CDBLAB:
      ```sql
      CONNECT system@CDBLAB
      ```
   2. Execute:
      ```sql
      ALTER SYSTEM SET DB_CREATE_FILE_DEST='/u01/app/oracle/oradata';
      ```

   **Create a Tablespace using OMF:**
   ```sql
   CREATE TABLESPACE tbs_OMF;
   ```

   **Confirmation:**
   ```sql
   SELECT file_name, tablespace_name FROM DBA_DATA_FILES WHERE tablespace_name = 'TBS_OMF';
   ```

### 8. **Shutdown and Save State**

   As with the previous lab, ensure all databases and listeners are appropriately stopped and that states are saved.

   **Shutdown the CDB and PDBs:**
   ```sql
   SHUTDOWN IMMEDIATE;
   ```

   **Save PDB States:**
   ```sql
   ALTER PLUGGABLE DATABASE ALL SAVE STATE;
   ```

   **Verify Saved States:**
   ```sql
   SELECT con_id, con_name, state FROM DBA_PDB_SAVED_STATES;
   ```

### 9. **Start the CDB and Verify PDB States**

   **Start the CDB:**
   ```sql
   STARTUP;
   ```

   **Open All PDBs:**
   ```sql
   ALTER PLUGGABLE DATABASE ALL OPEN;
   ```

   **Verify:**
   ```sql
   SELECT con_id, name, open_mode FROM v$pdbs;
   ```

## Summary:
In this lab, you learned how to create and manage tablespaces within both CDB and PDB environments. You defined default tablespaces, created temporary tablespaces, and implemented Oracle Managed Files (OMF) for automated file management. These skills are crucial for managing storage within an Oracle Database environment.

### Addendum: Understanding and Verifying Tablespaces in Oracle Databases

#### Objectives:
- To deepen your understanding of tablespaces and their role within Oracle Databases.
- To explore the file locations associated with tablespaces.
- To validate and verify tablespace settings using SQL Developer and SQL*Plus.

#### Introduction to Tablespaces
Tablespaces in Oracle databases are logical storage units that group related data files, helping in organizing and managing the storage of database objects such as tables and indexes. Tablespaces allow for the segregation of data, making it easier to manage different segments of the database. Each tablespace can consist of one or more data files, which physically store the database's data.

#### Navigating and Understanding Tablespace Files

1. **Understanding Tablespace Structure:**
   - A **tablespace** in an Oracle database is a collection of one or more data files.
   - These data files are stored on the disk and are essential for the physical storage of database objects.
   - Tablespaces are classified into **permanent tablespaces** (for storing persistent data), **temporary tablespaces** (for storing temporary data like sorting operations), and **undo tablespaces** (for managing transaction rollbacks).

2. **Navigating to Tablespace Files:**
   - Tablespace files are typically located in a specific directory on the Oracle database server.
   - Based on the lab instructions, the tablespace files for `CDBLAB` and `PDBLAB1` are located in `/u01/app/oracle/oradata/CDBLAB/`.
   - Using a command line or terminal, you can navigate to this directory to view the files:
     ```bash
     cd /u01/app/oracle/oradata/CDBLAB/
     ls -l
     ```
   - Observe the list of `.dbf` files corresponding to the tablespaces you have created. These are the data files physically storing your database's data.

#### Verifying Tablespace Settings

3. **Validating Tablespaces using SQL Developer:**
   - **Step 1:** Open SQL Developer and connect to `CDBLAB` and `PDBLAB1`.
   - **Step 2:** In the **Connections** pane, select your connection and open a **SQL Worksheet**.
   - **Step 3:** Execute the following SQL commands to verify the tablespace settings:
     ```sql
     -- View all tablespaces in CDBLAB
     SELECT tablespace_name, status FROM dba_tablespaces;

     -- View data files associated with tablespaces
     SELECT file_name, tablespace_name FROM dba_data_files;

     -- Verify default tablespace in CDBLAB
     SELECT PROPERTY_VALUE FROM DATABASE_PROPERTIES WHERE PROPERTY_NAME='DEFAULT_PERMANENT_TABLESPACE';
     ```
   - **Step 4:** Review the output to ensure that the tablespaces are correctly created and configured.

4. **Validating Tablespaces using SQL*Plus:**
   - **Step 1:** Open a terminal and connect to SQL*Plus as the `oracle` user.
   - **Step 2:** Connect to the `CDBLAB` and `PDBLAB1` databases:
     ```sql
     CONNECT system@CDBLAB
     ```
   - **Step 3:** Execute the same SQL commands as in SQL Developer to verify the tablespace settings:
     ```sql
     SELECT tablespace_name, status FROM dba_tablespaces;
     SELECT file_name, tablespace_name FROM dba_data_files;
     SELECT PROPERTY_VALUE FROM DATABASE_PROPERTIES WHERE PROPERTY_NAME='DEFAULT_PERMANENT_TABLESPACE';
     ```
   - **Step 4:** Ensure that all the tablespaces are present and correctly configured.

#### Additional Exercise:
- **Explore Datafile Sizes and Locations:**
  Use the following command to explore the size and location of data files associated with each tablespace:
  ```sql
  SELECT file_name, tablespace_name, bytes/1024/1024 AS size_mb FROM dba_data_files;
  ```
  - This query will provide you with the size of each data file in megabytes, allowing you to understand the storage allocation for each tablespace.

#### Conclusion:
Understanding tablespaces is crucial for effective database management. By navigating the filesystem where the tablespace files are stored and verifying their settings through SQL Developer and SQL*Plus, you gain practical experience that is essential for managing Oracle Databases in real-world environments.

This addendum to the lab ensures that you not only create and configure tablespaces but also understand their underlying structure and how to manage them effectively.

