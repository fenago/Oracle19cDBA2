# Lab_5_0: Configuring User Resource Limits in Oracle 19c

### Objectives:
- Learn how to create a local profile using SQL Developer.
- Create local users in a Pluggable Database (PDB) and assign resource limits.
- Configure and assign a default role to a user.
- Understand when and why to use profiles and roles to manage user resources and security.

### Use Case:
In a multitenant Oracle Database environment, managing user access and resource limits is crucial for ensuring security, performance, and compliance with organizational policies. By configuring profiles and roles, you can enforce password policies, limit resource usage, and control user permissions across the database. This lab will guide you through creating and configuring user resource limits in a Pluggable Database (PDB) within the Container Database (CDBLAB).

### Prerequisites:
- Access to an Oracle 19c Database environment with CDBLAB and PDBs (PDBLAB1, PDBLAB2, PDBLAB3).
- SQL Developer for executing SQL commands.

---

## Main Steps:

### 1. **Using SQL Developer to Create a Local Profile**

**Why:**
Profiles in Oracle are used to manage and enforce user resource limits such as password policies, session settings, and resource consumption. Creating a local profile within a PDB allows you to enforce these limits at the PDB level.

**Steps:**

1. **Connect to PDBLAB1 using SQL Developer:**
   - Open SQL Developer and create a new connection to `PDBLAB1`.
   - Use the `system` user or another user with DBA privileges.

2. **Create a Local Profile:**
   - In the SQL Worksheet, enter the following command to create a local profile:
   ```sql
   CREATE PROFILE lprofile_PDB1 
   LIMIT
       SESSIONS_PER_USER 2
       CPU_PER_SESSION 10000
       CPU_PER_CALL 1000
       CONNECT_TIME 60
       PASSWORD_LIFE_TIME 30;
   ```
   - **Explanation:**
     - This profile, `lprofile_PDB1`, sets limits on user sessions, CPU usage, connection time, and password life time within `PDBLAB1`. These limits help manage resource consumption and enforce security policies.

---

### 2. **Using SQL Developer to Create Local Users**

**Why:**
Creating local users in a PDB allows you to define specific users with tailored access and resource limits, ensuring they operate within the constraints set by the profile. Local users are essential for managing access control within a PDB.

**Steps:**

1. **Create a Local User and Assign a Profile:**
   - In the same SQL Worksheet, enter the following command:
   ```sql
   CREATE USER john_doe IDENTIFIED BY password123
   PROFILE lprofile_PDB1
   DEFAULT TABLESPACE users
   QUOTA 10M ON users;
   ```
   - **Explanation:**
     - This command creates a local user `john_doe` within `PDBLAB1`, assigns the previously created `lprofile_PDB1`, and sets a default tablespace and quota. The user is now subject to the resource limits defined in the profile.

2. **Grant Necessary Privileges:**
   - Grant the user privileges to connect and create sessions:
   ```sql
   GRANT CONNECT TO john_doe;
   GRANT CREATE SESSION TO john_doe;
   ```
   - **Explanation:**
     - These grants are essential for the user to be able to log in and start a session within the PDB.

---

### 3. **Configuring a Default Role for a User**

**Why:**
Roles in Oracle are used to bundle multiple privileges into a single entity that can be easily assigned to users. Configuring a default role ensures that users automatically have the necessary privileges when they log in.

**Steps:**

1. **Create a Role:**
   - In the SQL Worksheet, create a new role:
   ```sql
   CREATE ROLE read_only_role;
   ```
   - **Explanation:**
     - This role will be used to grant read-only access to certain database objects.

2. **Grant Privileges to the Role:**
   - Assign specific privileges to the role:
   ```sql
   GRANT SELECT ON employees TO read_only_role;
   ```
   - **Explanation:**
     - This command grants the `SELECT` privilege on the `employees` table to the `read_only_role`, allowing users with this role to read data from the table.

3. **Assign the Role to the User:**
   - Assign the `read_only_role` to the user `john_doe` and set it as the default role:
   ```sql
   GRANT read_only_role TO john_doe;
   ALTER USER john_doe DEFAULT ROLE read_only_role;
   ```
   - **Explanation:**
     - This ensures that every time `john_doe` logs in, the `read_only_role` is automatically enabled, giving them read-only access to the `employees` table.

---

### Summary:
In this lab, you learned how to configure user resource limits in Oracle 19c by creating a local profile, setting up local users, and assigning a default role within a Pluggable Database (PDB). These configurations help enforce security and manage resource consumption, ensuring that users operate within the constraints set by your organization's policies.

By following these steps, you can effectively manage user access and resources within your Oracle Database environment, improving security and operational efficiency.

---

### Addendum to Lab_5_0: Advanced User Resource Management Challenge

#### **Challenge Use Case:**

**Scenario:**
You are the Database Administrator (DBA) for a large educational institution that has recently migrated to an Oracle 19c multitenant architecture. The institution uses multiple Pluggable Databases (PDBLAB1, PDBLAB2, PDBLAB3) to manage different departments such as Admissions, Academics, and Finance.

Recently, the institution has faced several challenges:

1. **Resource Contention**: In PDBLAB2 (Academics), a few users are consuming excessive CPU resources, leading to performance degradation for other users.
2. **Security Concerns**: In PDBLAB3 (Finance), the security team has flagged several user accounts for weak password practices, requiring immediate enforcement of stricter password policies.
3. **Access Control**: A new policy requires that all users in PDBLAB1 (Admissions) have read-only access to sensitive student data, without any risk of data modification.

Your task is to resolve these issues by leveraging the skills you've learned in Lab_5_0.

---

### **Tasks:**

1. **Create a Profile to Manage Resource Contention in PDBLAB2:**
   - **Objective**: Create a local profile that limits CPU usage and session time for users in PDBLAB2 to ensure fair resource allocation.
   - **Steps**:
     1. Connect to PDBLAB2 using SQL Developer.
     2. Create a local profile that sets reasonable limits on CPU usage and session time.
     3. Assign this profile to the high-resource users in PDBLAB2.
   - **Reference**: 
     - [Oracle Database SQL Language Reference: CREATE PROFILE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-PROFILE.html)

2. **Enforce Stricter Password Policies in PDBLAB3:**
   - **Objective**: Implement a local profile in PDBLAB3 that enforces password complexity, aging, and history to enhance security.
   - **Steps**:
     1. Connect to PDBLAB3 using SQL Developer.
     2. Create a local profile that includes settings for password complexity, aging, and history.
     3. Assign this profile to all users in PDBLAB3 to enforce the new security policies.
   - **Reference**: 
     - [Oracle Database Security Guide: Managing Passwords with Profiles](https://docs.oracle.com/en/database/oracle/oracle-database/19/dbseg/managing-passwords-profiles-and-resources.html)

3. **Implement Read-Only Access for Users in PDBLAB1:**
   - **Objective**: Create a role in PDBLAB1 that grants read-only access to sensitive student data and assign it as the default role for all users.
   - **Steps**:
     1. Connect to PDBLAB1 using SQL Developer.
     2. Create a role that grants read-only access to the required tables.
     3. Assign this role to the necessary users and configure it as their default role.
   - **Reference**: 
     - [Oracle Database SQL Language Reference: CREATE ROLE](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/CREATE-ROLE.html)
     - [Oracle Database SQL Language Reference: ALTER USER](https://docs.oracle.com/en/database/oracle/oracle-database/19/sqlrf/ALTER-USER.html)

4. **Monitor and Validate the Changes:**
   - **Objective**: After implementing the changes, monitor the user activity and resource usage to ensure the solutions are effective.
   - **Steps**:
     1. Use SQL queries to monitor CPU usage in PDBLAB2 and ensure that the profile limits are being enforced.
     2. Verify password changes in PDBLAB3 to confirm that the new profile settings are applied.
     3. Test user access in PDBLAB1 to ensure that the read-only role is functioning as expected.
   - **Reference**: 
     - [Oracle Database Reference: Dynamic Performance Views](https://docs.oracle.com/en/database/oracle/oracle-database/19/refrn/dynamic-performance-views.html)

---

### **Challenge Notes:**

- **Task 1**: Focus on balancing resource allocation to prevent any single user from monopolizing CPU resources. Consider typical session lengths and CPU demands in PDBLAB2.
- **Task 2**: For password policies, consider the institution's security requirements, such as password length, complexity, and the frequency of changes.
- **Task 3**: Ensure that the read-only role is comprehensive enough to cover all sensitive tables while preventing data modification.

---

This enhanced challenge not only tests your understanding of user resource management in Oracle 19c but also provides valuable reference materials to help guide you through the tasks. By following the tasks outlined and consulting the provided links, you can effectively address the resource, security, and access control challenges faced by the institution, ensuring a secure and efficient database environment.
