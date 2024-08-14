# Lab_9_0: Transporting Data Between PDBs in the Same CDB Using Data Pump and RMAN

### Objectives:
- Learn how to use Oracle Data Pump to export and import data between Pluggable Databases (PDBs) within the same Container Database (CDB).
- Understand the process of transporting tablespaces using RMAN within the same CDB.
- Manage schema remapping and directory paths to avoid conflicts during the data transfer process.

### Use Case:
You are a Database Administrator responsible for moving data between PDBs that reside within the same Container Database (`CDBLAB`). Your task is to transport data from `PDBLAB1` to `PDBLAB2` using Data Pump and RMAN. This lab will guide you through the process, ensuring that you manage file paths, schema names, and other considerations to maintain data integrity and avoid conflicts.

### Prerequisites:
- Access to Oracle 19c Database environment with `PDBLAB1` and `PDBLAB2` under the same `CDBLAB`.
- SQL*Plus or SQL Developer for executing SQL commands.
- Basic knowledge of Oracle Data Pump and RMAN.

---

## Main Steps:

### 1. **Exporting Data from PDBLAB1 Using Data Pump**

**Why:**
Data Pump is a highly efficient utility for exporting and importing data within Oracle databases. Exporting data from `PDBLAB1` allows you to create a transportable dump file that can be imported into `PDBLAB2`.

**Steps:**

1. **Create a Directory Object for Data Pump in PDBLAB1:**
   - Connect to `PDBLAB1` and create a directory object where the export dump file will be stored:
   ```sql
   CONNECT system@PDBLAB1;
   CREATE OR REPLACE DIRECTORY data_pump_dir_pdb1 AS '/u01/app/oracle/oradata/PDBLAB1/dpump';
   ```
   - **Explanation:**
     - The directory object `data_pump_dir_pdb1` is used by Data Pump to store the dump file specific to `PDBLAB1`.

2. **Export the Data Using Data Pump:**
   - Use the Data Pump `expdp` utility to export the full content of `PDBLAB1`:
   ```bash
   expdp system@PDBLAB1 FULL=Y DUMPFILE=pdb1_exp.dmp DIRECTORY=data_pump_dir_pdb1 LOGFILE=expdp_pdb1.log
   ```
   - **Explanation:**
     - `FULL=Y`: Exports the entire PDB.
     - `DUMPFILE`: Specifies the name of the dump file.
     - `DIRECTORY`: Points to the directory object where the dump file is stored.
     - `LOGFILE`: Generates a log file to record the export process.

---

### 2. **Importing Data into PDBLAB2 Using Data Pump**

**Why:**
Importing the data into `PDBLAB2` ensures that the same data is available across multiple PDBs, facilitating data sharing and migration within the same CDB.

**Steps:**

1. **Create a Directory Object in PDBLAB2:**
   - Connect to `PDBLAB2` and create a directory object where the dump file will be imported:
   ```sql
   CONNECT system@PDBLAB2;
   CREATE OR REPLACE DIRECTORY data_pump_dir_pdb2 AS '/u01/app/oracle/oradata/PDBLAB2/dpump';
   ```
   - **Explanation:**
     - This directory will be used to access the dump file during the import process.

2. **Copy the Dump File to PDBLAB2:**
   - Ensure that the `pdb1_exp.dmp` file from `PDBLAB1` is accessible from `PDBLAB2` by copying it to the directory specified in `PDBLAB2`.
   - Here is the copy:  `cp /u01/app/oracle/oradata/PDBLAB1/dpump/pdb1_exp.dmp /u01/app/oracle/oradata/PDBLAB2/dpump/pdb1_exp.dmp`

3. **Import the Data Using Data Pump:**
   - Use the Data Pump `impdp` utility to import the data into `PDBLAB2`:
   ```bash
   impdp system@PDBLAB2 FULL=Y DUMPFILE=pdb1_exp.dmp DIRECTORY=data_pump_dir_pdb2 LOGFILE=impdp_pdb2.log REMAP_SCHEMA=old_schema:new_schema
   ```
   - **Explanation:**
     - `FULL=Y`: Imports the entire PDB.
     - `REMAP_SCHEMA`: Remaps schemas from the source PDB to the target PDB if needed.
     - `LOGFILE`: Generates a log file to record the import process.

---

### 3. **Transporting Tablespaces Between PDBLAB1 and PDBLAB2 Using RMAN**

**Why:**
Transporting tablespaces is a common requirement when moving data between environments. RMAN provides a reliable method to perform this operation within the same CDB with minimal downtime.

**Steps:**

1. **Prepare the Tablespace for Transport in PDBLAB1:**
   - Connect to `PDBLAB1` and make the tablespace read-only:
   ```sql
   CONNECT system@PDBLAB1;
   ALTER TABLESPACE users READ ONLY;
   ```
   - **Explanation:**
     - Setting the tablespace to read-only ensures data consistency during transport.

2. **Export the Tablespace Metadata Using Data Pump:**
   - Use the `expdp` utility to export the tablespace metadata:
   ```bash
   expdp system@PDBLAB1 TRANSPORT_TABLESPACES=users DUMPFILE=ts_users_exp.dmp DIRECTORY=data_pump_dir_pdb1 LOGFILE=expdp_ts_users.log
   ```
   - **Explanation:**
     - `TRANSPORT_TABLESPACES`: Specifies the tablespace to be transported.

3. **Copy the Data Files and Dump File to PDBLAB2:**
   - Ensure the data files associated with the `users` tablespace and the `ts_users_exp.dmp` file are copied to the corresponding directory in `PDBLAB2`.

4. **Import the Tablespace Metadata into PDBLAB2:**
   - Connect to `PDBLAB2` and import the tablespace metadata using Data Pump:
   ```bash
   impdp system@PDBLAB2 TRANSPORT_DATAFILES='/u01/app/oracle/oradata/PDBLAB2/users01.dbf' DIRECTORY=data_pump_dir_pdb2 DUMPFILE=ts_users_exp.dmp LOGFILE=impdp_ts_users.log
   ```
   - **Explanation:**
     - `TRANSPORT_DATAFILES`: Specifies the path to the data file being imported.

5. **Make the Tablespace Read-Write in PDBLAB2:**
   - Set the transported tablespace to read-write mode:
   ```sql
   ALTER TABLESPACE users READ WRITE;
   ```
   - **Explanation:**
     - Changing the tablespace back to read-write mode makes it fully operational in the target PDB.

---

### **Summary:**
In this lab, you learned how to transport data between PDBs (`PDBLAB1` and `PDBLAB2`) that reside within the same CDB (`CDBLAB`) using Data Pump and RMAN. You performed tasks such as exporting and importing data between PDBs, transporting tablespaces, and managing schema remapping. By considering the unique aspects of working within the same CDB, you ensured that data integrity was maintained and potential conflicts were avoided.

These skills are essential for managing data migrations and consolidations within complex Oracle environments, especially in scenarios where multiple PDBs share the same CDB infrastructure.

---

This lab provides a comprehensive guide to data transport within the same CDB, focusing on practical steps and considerations necessary for a successful operation.


---

### **Lab: Transporting Tablespaces Between PDBLAB1 and PDBLAB2 Using RMAN**

#### **Objectives:**
- Learn how to transport tablespaces between PDBs using RMAN.
- Understand the process of making tablespaces read-only, exporting metadata, and using RMAN to transport data files between environments.

#### **Use Case:**
Transporting tablespaces is a common requirement when moving data between environments, especially when minimizing downtime is critical. RMAN provides a reliable method to perform this operation within the same CDB, ensuring data consistency and integrity.

#### **Prerequisites:**
- Access to Oracle 19c Database environment with `PDBLAB1` and `PDBLAB2` under the same `CDBLAB`.
- SQL Developer and SQL*Plus installed and configured.
- Directory objects created in both `PDBLAB1` and `PDBLAB2`.

#### **Steps:**

### **1. Prepare the Tablespace for Transport in PDBLAB1**

**Task:** Connect to `PDBLAB1` and make the `users` tablespace read-only.

**Commands:**
```sql
CONNECT system@PDBLAB1;
ALTER TABLESPACE users READ ONLY;
```

**Explanation:**  
Setting the tablespace to read-only ensures data consistency during the transport process. This step prevents any modifications to the data while it’s being prepared for transport.

### **2. Convert the Tablespace Data Files Using RMAN**

**Task:** Use RMAN to convert the tablespace data files for transport.

**Commands:**
```sql
CONNECT system@CDBLAB AS SYSDBA;
RMAN TARGET sys@PDBLAB1;

RMAN> CONVERT TABLESPACE users TO PLATFORM 'Linux x86 64-bit' 
  FORMAT '/u01/app/oracle/oradata/PDBLAB1/users01.dbf';
```

**Explanation:**  
The `CONVERT TABLESPACE` command prepares the tablespace data files for transport by converting them into a format that can be imported into another PDB. This step ensures compatibility between different environments or platforms if necessary.

### **3. Export the Tablespace Metadata Using Data Pump**

**Task:** Use the `expdp` utility to export the tablespace metadata.

**Commands:**
```bash
expdp system@PDBLAB1 TRANSPORT_TABLESPACES=users DUMPFILE=ts_users_exp.dmp DIRECTORY=data_pump_dir_pdb1 LOGFILE=expdp_ts_users.log
```

**Explanation:**  
The `TRANSPORT_TABLESPACES` parameter specifies which tablespace to export. The metadata is saved to a dump file, which will be used later during the import process.

### **4. Copy the Data Files and Dump File to PDBLAB2**

**Task:** Ensure the data files associated with the `users` tablespace and the `ts_users_exp.dmp` file are copied to the corresponding directory in `PDBLAB2`.

**Explanation:**  
This step involves physically transferring the converted data files and the dump file to the target environment (`PDBLAB2`). This can be done via any method suitable for your environment, such as SCP or a direct copy.

### **5. Import the Tablespace Metadata into PDBLAB2**

**Task:** Connect to `PDBLAB2` and import the tablespace metadata using Data Pump.

**Commands:**
```bash
impdp system@PDBLAB2 TRANSPORT_DATAFILES='/u01/app/oracle/oradata/PDBLAB2/users01.dbf' DIRECTORY=data_pump_dir_pdb2 DUMPFILE=ts_users_exp.dmp LOGFILE=impdp_ts_users.log
```

**Explanation:**  
The `TRANSPORT_DATAFILES` parameter specifies the path to the data files being imported. This step registers the tablespace in `PDBLAB2` and associates it with the existing data files.

### **6. Make the Tablespace Read-Write in PDBLAB2**

**Task:** Set the transported tablespace to read-write mode in `PDBLAB2`.

**Commands:**
```sql
ALTER TABLESPACE users READ WRITE;
```

**Explanation:**  
Changing the tablespace back to read-write mode makes it fully operational in the target PDB (`PDBLAB2`), allowing normal operations to resume.

### **7. Cleanup**

**Task:** After the transport is complete, clean up any temporary files and verify the integrity of the data.

**Commands:**
- Remove temporary files:
  ```bash
  rm /u01/app/oracle/oradata/PDBLAB1/users01.dbf
  rm /u01/app/oracle/oradata/PDBLAB2/users01.dbf
  ```
- Verify data integrity by checking the contents of the `users` tablespace in `PDBLAB2`:
  ```sql
  CONNECT system@PDBLAB2;
  SELECT * FROM users.employees; -- Example query to check data
  ```

**Why:**  
Cleaning up unnecessary files ensures that your environment remains tidy and secure, and verifying data integrity ensures that the transport operation was successful.

### **References:**
- [RMAN Transportable Tablespaces Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-transportable-tablespace-backup.html)
- [Oracle Data Pump Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/dmpug/oracle-database-utilities.html#GUID-B1E5AE1C-76D3-41B9-A38D-B3EDE21E718C)

---

This revised lab now properly includes RMAN for the transport of the tablespaces, ensuring that all steps align with the specified technology and providing explicit instructions to avoid confusion.


# Challenge Lab: Transporting Data and Managing Schemas in PDBLAB3

### Objectives:
- Apply your knowledge of Data Pump and RMAN to transport data and manage schemas in a new PDB (`PDBLAB3`).
- Use SQL Developer and SQL*Plus to perform all tasks.
- Test your ability to handle real-world scenarios involving schema remapping, directory management, and tablespace transport.
- Reinforce best practices for maintaining data integrity and avoiding conflicts in a shared CDB environment.

### Use Case:
You are tasked with setting up a new Pluggable Database (`PDBLAB3`) within the `CDBLAB`. Your goal is to transport specific schemas and tablespaces from `PDBLAB1` and `PDBLAB2` into `PDBLAB3`. You need to ensure that the data is correctly imported, with no conflicts in schema names or file paths, and that the tablespaces are properly configured and operational in the new PDB.

### Prerequisites:
- Access to Oracle 19c Database environment with `PDBLAB1`, `PDBLAB2`, and `PDBLAB3` under the same `CDBLAB`.
- SQL Developer and SQL*Plus installed and configured.
- **Note:** Before starting, you will create the `HR` and `FINANCE` schemas in `PDBLAB1` with sample tables and data.

### References:
- [Oracle Data Pump Documentation](https://docs.oracle.com/en/database/oracle/oracle-database/19/dmpug/oracle-database-utilities.html#GUID-B1E5AE1C-76D3-41B9-A38D-B3EDE21E718C)
- [RMAN Transportable Tablespaces](https://docs.oracle.com/en/database/oracle/oracle-database/19/bradv/rman-transportable-tablespace-backup.html#GUID-9F63A070-D2ED-4A5F-8668-E9F4BFF4A2AC)
- [SQL Developer User Guide](https://docs.oracle.com/en/database/oracle/sql-developer/19.2/rptug/sql-developer-user-guide.html)

---

## Main Steps:

### 1. **Setup Directory Objects in PDBLAB3**

**Task:** Create a directory object in `PDBLAB3` for Data Pump operations.

**Steps:**

1. **Connect to PDBLAB3 using SQL Developer:**
   - Open SQL Developer, connect to `PDBLAB3`, and create a directory object:
   ```sql
   CONNECT system@PDBLAB3;
   CREATE OR REPLACE DIRECTORY data_pump_dir_pdb3 AS '/u01/app/oracle/oradata/PDBLAB3/dpump';
   ```

**Why:**
This directory will be used to store Data Pump export and import files for `PDBLAB3`.

---

### 2. **Create and Populate HR and FINANCE Schemas in PDBLAB1**

**Task:** Create the `HR` and `FINANCE` schemas in `PDBLAB1` and populate them with sample data.

**Steps:**

1. **Connect to PDBLAB1 using SQL*Plus:**
   ```sql
   CONNECT system@PDBLAB1;
   ```

2. **Create the HR and FINANCE schemas:**
   ```sql
   CREATE USER HR IDENTIFIED BY hr_password;
   CREATE USER FINANCE IDENTIFIED BY finance_password;
   GRANT CONNECT, RESOURCE TO HR;
   GRANT CONNECT, RESOURCE TO FINANCE;
   ```

3. **Create tables in the HR schema:**
   ```sql
   CONNECT HR/hr_password@PDBLAB1;

   CREATE TABLE employees (
       employee_id NUMBER PRIMARY KEY,
       first_name VARCHAR2(50),
       last_name VARCHAR2(50),
       email VARCHAR2(100),
       phone_number VARCHAR2(20),
       hire_date DATE,
       job_id VARCHAR2(10),
       salary NUMBER(8, 2),
       manager_id NUMBER,
       department_id NUMBER
   );

   INSERT INTO employees VALUES (1, 'John', 'Doe', 'johndoe@example.com', '555-1234', SYSDATE, 'IT_PROG', 60000, NULL, 10);
   INSERT INTO employees VALUES (2, 'Jane', 'Smith', 'janesmith@example.com', '555-5678', SYSDATE, 'HR_REP', 55000, 1, 20);
   COMMIT;
   ```

4. **Create tables in the FINANCE schema:**
   ```sql
   CONNECT FINANCE/finance_password@PDBLAB1;

   CREATE TABLE transactions (
       transaction_id NUMBER PRIMARY KEY,
       transaction_date DATE,
       amount NUMBER(10, 2),
       employee_id NUMBER,
       FOREIGN KEY (employee_id) REFERENCES HR.employees(employee_id)
   );

   INSERT INTO transactions VALUES (1001, SYSDATE, 1500.00, 1);
   INSERT INTO transactions VALUES (1002, SYSDATE, 2000.00, 2);
   COMMIT;
   ```

**Why:**
These schemas and tables with sample data are necessary for the export and import tasks in this lab.

---

### 3. **Export Specific Schemas from PDBLAB1**

**Task:** Export the `HR` and `FINANCE` schemas from `PDBLAB1` using Data Pump.

**Steps:**

1. **Create a Directory Object in PDBLAB1 (if not already done):**
   ```sql
   CONNECT system@PDBLAB1;
   CREATE OR REPLACE DIRECTORY data_pump_dir_pdb1 AS '/u01/app/oracle/oradata/PDBLAB1/dpump';
   ```

2. **Export the Schemas Using Data Pump:**
   - Use SQL*Plus or a terminal to run the export:
   ```bash
   expdp system@PDBLAB1 SCHEMAS=HR,FINANCE DUMPFILE=schemas_exp.dmp DIRECTORY=data_pump_dir_pdb1 LOGFILE=expdp_schemas.log
   ```

**Why:**
This step captures all objects and data within the specified schemas, which will be imported into `PDBLAB3`.

---

### 4. **Import Schemas into PDBLAB3**

**Task:** Import the `HR` schema from `PDBLAB1` into `PDBLAB3`, remapping the schema name to `HR_DEV`.

**Steps:**

1. **Ensure the Dump File is Accessible in PDBLAB3:**
   - Copy the `schemas_exp.dmp` file to the directory associated with `PDBLAB3`.

2. **Import the Schema Using Data Pump:**
   - Use SQL*Plus or a terminal to run the import:
   ```bash
   impdp system@PDBLAB3 SCHEMAS=HR DUMPFILE=schemas_exp.dmp DIRECTORY=data_pump_dir_pdb3 LOGFILE=impdp_hr.log REMAP_SCHEMA=HR:HR_DEV
   ```

**Why:**
Remapping the schema allows you to avoid conflicts with existing schemas and demonstrates how to manage schema names during import.

---

### 5. **Transport a Tablespace from PDBLAB2 to PDBLAB3 Using RMAN**

**Task:** Transport the `SALES` tablespace from `PDBLAB2` to `PDBLAB3`.

**Steps:**

1. **Prepare the Tablespace in PDBLAB2:**
   - Connect to `PDBLAB2` using SQL*Plus and make the tablespace read-only:
   ```sql
   CONNECT system@PDBLAB2;
   ALTER TABLESPACE sales READ ONLY;
   ```

2. **Export the Tablespace Metadata Using Data Pump:**
   - Export the metadata for the `SALES` tablespace:
   ```bash
   expdp system@PDBLAB2 TRANSPORT_TABLESPACES=SALES DUMPFILE=ts_sales_exp.dmp DIRECTORY=data_pump_dir_pdb2 LOGFILE=expdp_ts_sales.log
   ```

3. **Copy the Data Files and Dump File to PDBLAB3:**
   - Ensure the data files and the `ts_sales_exp.dmp` file are copied to the directory associated with `PDBLAB3`.

4. **Import the Tablespace into PDBLAB3:**
   - Connect to `PDBLAB3` and import the tablespace metadata:
   ```bash
   impdp system@PDBLAB3 TRANSPORT_DATAFILES='/u01/app/oracle/oradata/PDBLAB3/sales01.dbf' DIRECTORY=data_pump_dir_pdb3 DUMPFILE=ts_sales_exp.dmp LOGFILE=impdp_ts_sales.log
   ```

5. **Make the Tablespace Read-Write in PDBLAB3:**
   - Finalize the transport by making the tablespace read-write:
   ```sql
   ALTER TABLESPACE sales READ WRITE;
   ```

**Why:**
Transporting tablespaces allows you to move large datasets efficiently between PDBs. This process showcases how to manage tablespace-level data transfer within the same CDB.

---

### 6. **Validate the Import and Transport**

**Task:** Verify that the `HR_DEV` schema and the `SALES` tablespace are correctly imported and operational in `PDBLAB3`.

**Steps:**

1. **Check the HR_DEV Schema:**
   - Connect to `PDBLAB3` using SQL Developer or SQL*Plus and verify the schema:
   ```sql
   SELECT table_name FROM all_tables WHERE owner = 'HR_DEV';


   ```

2. **Check the SALES Tablespace:**
   - Verify the tablespace status:
   ```sql
   SELECT tablespace_name, status FROM dba_tablespaces WHERE tablespace_name = 'SALES';
   ```

**Why:**
Validation ensures that the data has been correctly imported and that the tablespace is functional in `PDBLAB3`.

---

### 7. **Clean Up**

**Task:** After completing the tasks, clean up any temporary files or directories used during the process.

**Steps:**

1. **Remove Temporary Files:**
   - Delete any dump files or temporary data files created during the process.

2. **Review and Document:**
   - Document the process and review the results to ensure everything is correctly configured.

---

## **Challenge Questions:**

1. **Schema Remapping:** Why is schema remapping important when importing data into a new PDB, and what are potential pitfalls if not handled correctly?
2. **Tablespace Transport:** What are the key considerations when transporting a tablespace between PDBs in the same CDB, and how do you ensure data integrity during the process?
3. **Directory Management:** How does proper directory management in each PDB contribute to a successful Data Pump operation, and what are the consequences of directory conflicts?

---

### **Summary:**
This challenge lab tests your ability to apply the skills learned in previous labs to a new scenario involving `PDBLAB3`. By exporting, importing, and transporting data between PDBs within the same CDB, you will reinforce your understanding of Data Pump, RMAN, and schema management in a shared Oracle environment.

Upon completion, you should be able to confidently manage complex data migrations and consolidations, ensuring data integrity and avoiding conflicts in a multi-PDB setup.

---

This challenge scenario provides an opportunity to apply advanced Oracle database management techniques, preparing you for real-world scenarios where precision and attention to detail are critical.
