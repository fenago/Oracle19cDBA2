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

### 3. **Transporting Tablespaces Between PDBLAB1 and PDBLAB2 Using the Data PUMP**

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

---

### **Summary:**
In this lab, you learned how to transport data between PDBs (`PDBLAB1` and `PDBLAB2`) that reside within the same CDB (`CDBLAB`) using Data Pump and RMAN. You performed tasks such as exporting and importing data between PDBs, transporting tablespaces, and managing schema remapping. By considering the unique aspects of working within the same CDB, you ensured that data integrity was maintained and potential conflicts were avoided.

These skills are essential for managing data migrations and consolidations within complex Oracle environments, especially in scenarios where multiple PDBs share the same CDB infrastructure.

---

