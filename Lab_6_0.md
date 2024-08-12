# Lab_6_0: Implementing and Managing Unified Auditing in Oracle 19c

### Objectives:
- Enable unified auditing in Oracle Database 19c.
- Create and manage unified audit policies.
- Maintain and secure the audit trail to protect sensitive audit data.
- Restrict access to data and services to enhance security.
- Authenticate users and monitor for suspicious activity.

### Use Case:
As a Database Administrator (DBA) for a financial institution, you are tasked with implementing comprehensive auditing across multiple Pluggable Databases (PDBs) within your Container Database (CDBLAB). The institution requires strict monitoring of all activities related to financial transactions, user access, and system changes to ensure compliance with regulatory requirements such as SOX and PCI-DSS. 

This lab will guide you through enabling unified auditing, creating audit policies, maintaining the audit trail, restricting access to sensitive data, and monitoring for suspicious activity.

### Prerequisites:
- Access to an Oracle 19c Database environment with CDBLAB and PDBs (PDBLAB1, PDBLAB2, PDBLAB3).
- SQL*Plus or SQL Developer for executing SQL commands.

---

## Main Steps:

### 1. **Enabling Unified Auditing**

**Why:**
Unified auditing consolidates and simplifies the audit process in Oracle Database, allowing you to define audit policies that cover a wide range of activities and events. Enabling unified auditing is the first step to setting up a robust auditing environment.

**Steps:**

1. **Check if Unified Auditing is Enabled:**
   - Run the following query to check the status of unified auditing:
   ```sql
   SELECT VALUE FROM V$OPTION WHERE PARAMETER = 'Unified Auditing';
   ```
   - **Explanation:**
     - This command checks whether unified auditing is already enabled.

2. **Enable Unified Auditing (if not already enabled):**
   - If unified auditing is not enabled, you will need to enable it. Follow these steps:
   ```bash
   1. Shut down the database:
   SHUTDOWN IMMEDIATE;

   2. Enable unified auditing:
   $ORACLE_HOME/rdbms/lib> make -f ins_rdbms.mk uniaud_on ioracle

   3. Start the database:
   STARTUP;
   ```
   - **Explanation:**
     - These steps re-link the Oracle binaries with unified auditing enabled and restart the database.

---

### 2. **Creating Unified Audit Policies**

**Why:**
Unified audit policies allow you to specify what actions or activities should be audited, providing fine-grained control over monitoring and logging database activities. 

**Steps:**

1. **Create an Audit Policy to Monitor SELECT Statements on Financial Tables:**
   - Connect to PDBLAB1 and create an audit policy:
   ```sql
   CONNECT system@PDBLAB1;
   CREATE AUDIT POLICY select_financial_data
   ACTIONS SELECT ON financial.transactions;
   ```
   - **Explanation:**
     - This policy audits all SELECT operations on the `financial.transactions` table, which is critical for tracking data access.

2. **Create a System-Wide Audit Policy:**
   - Connect to CDBLAB and create a policy that applies across all PDBs:
   ```sql
   CONNECT sys@CDBLAB AS sysdba;
   CREATE AUDIT POLICY audit_sys_admin
   PRIVILEGES CREATE USER, DROP USER, ALTER SYSTEM;
   ```
   - **Explanation:**
     - This policy audits all administrative actions such as creating or dropping users, and altering system settings across the entire CDB and all PDBs.

3. **Create an Audit Policy Based on Specific Conditions:**
   - Create a policy in PDBLAB2 to audit specific actions only during business hours:
   ```sql
   CONNECT system@PDBLAB2;
   CREATE AUDIT POLICY audit_business_hours
   ACTIONS UPDATE ON hr.employees
   WHEN 'SYS_CONTEXT(''USERENV'', ''SESSION_USER'') = ''FINANCE_USER'' AND TO_CHAR(SYSDATE, ''DY'') IN (''MON'', ''TUE'', ''WED'', ''THU'', ''FRI'') AND TO_CHAR(SYSDATE, ''HH24'') BETWEEN ''08'' AND ''18''';
   ```
   - **Explanation:**
     - This policy audits `UPDATE` operations on the `hr.employees` table only when performed by `FINANCE_USER` during business hours, ensuring that critical updates are tracked during the most active periods.

---

### 3. **Maintaining the Audit Trail**

**Why:**
The audit trail is essential for ensuring the integrity and accountability of audit data. Maintaining and securing this trail prevents unauthorized access and tampering, ensuring that audit logs are reliable.

**Steps:**

1. **View the Current Audit Trail Configuration:**
   - Query the current audit trail settings:
   ```sql
   SELECT * FROM DBA_AUDIT_TRAIL;
   ```
   - **Explanation:**
     - This query returns the existing audit logs, allowing you to review the recorded activities.

2. **Configure the Audit Trail to Write to a Secure Location:**
   - Ensure the audit trail is written to a secure file system location:
   ```sql
   ALTER SYSTEM SET AUDIT_FILE_DEST = '/secure_location/audit_logs' SCOPE=BOTH;
   ```
   - **Explanation:**
     - This command redirects the audit logs to a secure directory, protecting them from unauthorized access.

3. **Set the Retention Period for Audit Logs:**
   - Set a retention policy for the audit logs:
   ```sql
   DBMS_AUDIT_MGMT.SET_AUDIT_TRAIL_PROPERTY(
       AUDIT_TRAIL_TYPE => DBMS_AUDIT_MGMT.AUDIT_TRAIL_UNIFIED,
       AUDIT_TRAIL_PROPERTY => DBMS_AUDIT_MGMT.AUDIT_TRAIL_AGE,
       AUDIT_TRAIL_PROPERTY_VALUE => 30);
   ```
   - **Explanation:**
     - This command sets the retention period for audit logs to 30 days, ensuring that old logs are archived or deleted as per policy.

---

### 4. **Restricting Access to Data and Services**

**Why:**
Restricting access to sensitive data and services is critical for securing the database environment and preventing unauthorized access to confidential information.

**Steps:**

1. **Create a Role with Restricted Access:**
   - Create a role in PDBLAB3 that restricts access to specific tables:
   ```sql
   CONNECT system@PDBLAB3;
   CREATE ROLE restricted_access;
   GRANT SELECT ON hr.salary TO restricted_access;
   GRANT EXECUTE ON hr.raise_salary_proc TO restricted_access;
   ```
   - **Explanation:**
     - This role limits users to only select operations on the `hr.salary` table and execution of the `hr.raise_salary_proc` procedure.

2. **Assign the Restricted Role to a User:**
   - Assign the restricted role to a user in PDBLAB3:
   ```sql
   GRANT restricted_access TO john_smith;
   ```
   - **Explanation:**
     - This ensures that `john_smith` can only perform the operations allowed by the `restricted_access` role, enhancing data security.

---

### 5. **Authenticating Users**

**Why:**
Authentication is the first line of defense in securing the database. Ensuring that only authorized users can access the database is crucial for protecting data and maintaining compliance.

**Steps:**

1. **Enable Strong Authentication Mechanisms:**
   - Enforce strong password policies in CDBLAB:
   ```sql
   ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIME 90 PASSWORD_REUSE_TIME 365 PASSWORD_REUSE_MAX 5 PASSWORD_VERIFY_FUNCTION ora12c_verify_function;
   ```
   - **Explanation:**
     - This policy ensures strong passwords, preventing users from reusing old passwords frequently, and enforces regular password changes.

2. **Implement Multi-Factor Authentication (MFA) for Critical Users:**
   - While MFA configuration varies by implementation, ensure that key administrative users in PDBLAB1 are configured for MFA through Oracle's Identity Management solutions.

---

### 6. **Monitoring for Suspicious Activity**

**Why:**
Continuous monitoring for suspicious activity is essential for detecting and responding to potential security incidents before they cause significant harm.

**Steps:**

1. **Create a Policy to Audit Suspicious Logins:**
   - Monitor for failed login attempts in PDBLAB1:
   ```sql
   CREATE AUDIT POLICY audit_failed_logins
   ACTIONS LOGON WHENEVER NOT SUCCESSFUL;
   AUDIT POLICY audit_failed_logins;
   ```
   - **Explanation:**
     - This policy audits all failed login attempts, which can indicate potential unauthorized access attempts.

2. **Query the Audit Trail for Suspicious Activity:**
   - Regularly review the audit logs for signs of suspicious activity:
   ```sql
   SELECT USERNAME, ACTION_NAME, RETURN_CODE, TIMESTAMP
   FROM DBA_AUDIT_TRAIL
   WHERE RETURN_CODE != 0;
   ```
   - **Explanation:**
     - This query helps identify failed actions and potential security issues by focusing on actions that did not succeed.

---

### Summary:
In this lab, you learned how to enable unified auditing, create and manage audit policies, maintain the audit trail, restrict access to sensitive data, authenticate users, and monitor for suspicious activities in Oracle 19c. These steps are critical for ensuring that your database environment is secure, compliant, and capable of detecting and responding to potential threats.

By following these steps, you can implement a robust auditing framework that provides comprehensive oversight of your database activities, ensuring the integrity, confidentiality, and availability of your data.
