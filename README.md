# 🏨 Hotel Management System – Oracle PL/SQL Capstone Project

## 📘 Project Overview

The **Hotel Management System** solves the challenges posed by manual reservation systems in hotels, guesthouses, and resorts. Traditional systems often face issues like double-booking, lost records, inaccurate billing, and a lack of service tracking. This project, built with **Oracle PL/SQL**, automates and secures all essential operations such as:

- Guest registration and room reservations
- Employee and service management
- Booking and payment handling
- Restriction of unauthorized actions during business rules (e.g., holidays or weekdays)
- Activity auditing and reporting

This system utilizes a **relational database model** with multiple tables, business logic via PL/SQL, and role-based operations. Below is the complete design and implementation of the system.

---

## 🔷 Phase 1: Entity Relationship Diagram (ERD)

The **ERD** visually models the structure of the database. It shows how entities like Guests, Bookings, Payments, and Rooms relate to each other.

### Key Entities:
- **Guests**: Stores guest details (ID, name, contact, email)
- **Rooms**: Room types, price, and availability
- **Bookings**: Guest-room assignment and duration
- **Payments**: Amount, method, and date linked to bookings
- **Employees**: Hotel staff information
- **Services**: Optional services requested by guests
- **Service Requests**: Tracks who requested what and when

### Relationships:
- A **Guest** can make multiple **Bookings**
- A **Booking** is linked to one **Room**
- A **Payment** is tied to a **Booking**
- An **Employee** manages multiple **Bookings**
- Guests may make multiple **Service Requests**

![ERD](screenshots/erd_diagram.png)

---

## 🔷 Phase 2: BPMN Diagram (Business Process Model)

The **Business Process Model** illustrates how different actors interact within the system to accomplish hotel operations. Using **BPMN swimlanes**, the flow includes reservation handling, billing, auditing, and report generation.

### Actors:
- **Guest**: Initiates booking, requests services, makes payment
- **Receptionist**: Confirms bookings, updates records
- **Manager**: Views reports and audits
- **System**: Automates billing and restrictions

![BPMN](screenshots/bpmn_diagram.png)

---

## 🔷 Phase 3: Table Creation (DDL)

In this step, the main database structure is created using SQL `CREATE TABLE` commands. Each table is normalized, and appropriate constraints are used.

### Example: `Guests` Table
```sql
CREATE TABLE Guests (
  GuestID NUMBER PRIMARY KEY,
  Name VARCHAR2(100),
  Contact VARCHAR2(15),
  Email VARCHAR2(100)
);
```

### Example: `Rooms` Table
```sql
CREATE TABLE Rooms (
  RoomID NUMBER PRIMARY KEY,
  Type VARCHAR2(50),
  Price NUMBER,
  Status VARCHAR2(20)
);
```

### Other tables created:
- Bookings
- Payments
- Employees
- Services
- ServiceRequests

These table definitions ensure data integrity and support the full functionality of the system.

![Screenshot](screenshots/table_creation.png)

---

## 🔷 Phase 4: Data Insertion (DML)

We insert realistic and meaningful data into all tables to support testing and actual usage.

### Example Insert:
```sql
INSERT INTO Rooms VALUES (112, 'Single', 48000, 'Available');
```

### Another Example:
```sql
INSERT INTO Guests VALUES (1, 'John Doe', '0788000001', 'john@example.com');
```

These data samples allow for verifying procedures, relationships, and triggers through meaningful test cases.

![Screenshot](screenshots/data_insertion.png)

---

## 🔷 Phase 5: Oracle Enterprise Manager (OEM) Setup

We use **Oracle Enterprise Manager** to manage the pluggable database created for the system. OEM helps monitor user sessions, resource usage, and object access in real-time.

### Setup includes:
- Creating database with naming format: `mon_27463_yourname_projectname_db`
- Assigning admin privileges
- Managing session stats, performance, and user activities

![Screenshot](screenshots/oracle_enterprise_manager.png)

---

## 🔷 Phase 6: Enable DBMS Output

```sql
SET SERVEROUTPUT ON;
```

This command is essential for displaying output from `DBMS_OUTPUT.PUT_LINE` used in stored procedures and triggers.

![Screenshot](screenshots/step0_enable_output.png)

---

## 🔷 Step 1: Add Gender Column to Guests Table

```sql
BEGIN
  EXECUTE IMMEDIATE 'ALTER TABLE Guests ADD Gender VARCHAR2(10)';
EXCEPTION
  WHEN OTHERS THEN
    IF SQLCODE = -01430 THEN
      DBMS_OUTPUT.PUT_LINE('Column already exists.');
    ELSE
      RAISE;
    END IF;
END;
/
```

This adds gender data for each guest and handles the "column already exists" error safely.

![Screenshot](screenshots/step1_alter_table.png)

---

## 🔷 Step 2: Perform Sample DML Operations

```sql
UPDATE Guests SET Contact = '0788001121' WHERE GuestID = 1;
DELETE FROM Payments WHERE PaymentID = 10;
INSERT INTO Rooms VALUES (112, 'Single', 48000, 'Available');
```

These operations simulate real-life changes and validate that the tables are functioning properly.

![Screenshot](screenshots/step2_dml_operations.png)

---

## 🔷 Step 3: Procedure to Summarize Guest Bookings and Payments

```sql
CREATE OR REPLACE PROCEDURE GetGuestSummary (
  p_guest_id IN Guests.GuestID%TYPE
) AS
  v_total_bookings NUMBER := 0;
  v_total_paid NUMBER(10,2) := 0;
BEGIN
  SELECT COUNT(*), NVL(SUM(p.Amount), 0)
  INTO v_total_bookings, v_total_paid
  FROM Bookings b
  LEFT JOIN Payments p ON b.BookingID = p.BookingID
  WHERE b.GuestID = p_guest_id;

  DBMS_OUTPUT.PUT_LINE('Guest ID: ' || p_guest_id);
  DBMS_OUTPUT.PUT_LINE('Total Bookings: ' || v_total_bookings);
  DBMS_OUTPUT.PUT_LINE('Total Paid: ' || v_total_paid);
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
END;
/
```

This provides a summarized financial and booking report for each guest.

![Screenshot](screenshots/step3_procedure_summary.png)

---

## 🔷 Step 4: Holidays Table

```sql
CREATE TABLE Holidays (
  HolidayDate DATE PRIMARY KEY,
  Description VARCHAR2(100)
);

INSERT INTO Holidays VALUES (TO_DATE('2025-05-17', 'YYYY-MM-DD'), 'Independence Day');
INSERT INTO Holidays VALUES (TO_DATE('2025-05-23', 'YYYY-MM-DD'), 'National Heroes Day');
COMMIT;
```

Used to restrict operations on holidays.

![Screenshot](screenshots/step4_create_holidays.png)

---

## 🔷 Step 5: Create Audit_Log Table

```sql
CREATE TABLE Audit_Log (
  AuditID NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  UserName VARCHAR2(50),
  ActionDate DATE,
  Operation VARCHAR2(10),
  TableName VARCHAR2(30),
  Status VARCHAR2(10)
);
```

This table stores operation tracking information for auditing and security.

![Screenshot](screenshots/step5_create_audit_log.png)

---

## 🔷 Step 6: Procedure for Audit Logging

```sql
CREATE OR REPLACE PROCEDURE log_audit_action (
  p_user     VARCHAR2,
  p_action   VARCHAR2,
  p_table    VARCHAR2,
  p_status   VARCHAR2
) IS
  PRAGMA AUTONOMOUS_TRANSACTION;
BEGIN
  INSERT INTO Audit_Log (UserName, ActionDate, Operation, TableName, Status)
  VALUES (p_user, SYSDATE, p_action, p_table, p_status);
  COMMIT;
END;
/
```

A reusable module to write logs.

![Screenshot](screenshots/step6_audit_procedure.png)

---

## 🔷 Step 7: Trigger for Restricting Weekday and Holiday Access

```sql
CREATE OR REPLACE TRIGGER trg_block_weekday_holiday_ops
BEFORE INSERT OR UPDATE OR DELETE ON Guests
FOR EACH ROW
DECLARE
  v_day VARCHAR2(10);
  v_holiday NUMBER := 0;
  v_username VARCHAR2(50);
BEGIN
  v_username := SYS_CONTEXT('USERENV', 'SESSION_USER');
  
  SELECT TO_CHAR(SYSDATE, 'DY', 'NLS_DATE_LANGUAGE=''ENGLISH''') INTO v_day FROM DUAL;
  SELECT COUNT(*) INTO v_holiday FROM Holidays WHERE HolidayDate = TRUNC(SYSDATE);

  IF UPPER(v_username) = 'IRABARUTA' AND 
     (v_day IN ('MON', 'TUE', 'WED', 'THU', 'FRI') OR v_holiday > 0) THEN
    log_audit_action(v_username, 'BLOCKED', 'GUESTS', 'DENIED');
    RAISE_APPLICATION_ERROR(-20001, 'Operation not allowed on weekdays or public holidays');
  ELSE
    log_audit_action(v_username, 'ALLOWED', 'GUESTS', 'ALLOWED');
  END IF;
END;
/
```

This prevents unauthorized changes based on rules.

![Screenshot](screenshots/step7_trigger_restriction.png)

---

## 🔷 Step 8: Compound Trigger for Multi-row Audit

```sql
CREATE OR REPLACE TRIGGER trg_multirow_audit
FOR INSERT ON Payments
COMPOUND TRIGGER

  TYPE audit_rec IS RECORD (
    uname VARCHAR2(50),
    op_date DATE,
    op_type VARCHAR2(10),
    tbl_name VARCHAR2(30),
    status VARCHAR2(10)
  );
  TYPE audit_tbl IS TABLE OF audit_rec;
  v_audits audit_tbl := audit_tbl();

BEFORE STATEMENT IS
BEGIN
  NULL;
END BEFORE STATEMENT;

AFTER EACH ROW IS
BEGIN
  v_audits.EXTEND;
  v_audits(v_audits.LAST) := audit_rec(
    SYS_CONTEXT('USERENV', 'SESSION_USER'),
    SYSDATE,
    'INSERT',
    'PAYMENTS',
    'ALLOWED'
  );
END AFTER EACH ROW;

AFTER STATEMENT IS
BEGIN
  FOR i IN 1 .. v_audits.COUNT LOOP
    INSERT INTO Audit_Log (UserName, ActionDate, Operation, TableName, Status)
    VALUES (v_audits(i).uname, v_audits(i).op_date, v_audits(i).op_type, v_audits(i).tbl_name, v_audits(i).status);
  END LOOP;
  COMMIT;
END AFTER STATEMENT;

END;
/
```

Used for auditing multi-row inserts efficiently.

![Screenshot](screenshots/step8_compound_trigger.png)

---

## 🔷 Step 9: Run Summary Procedure

```sql
BEGIN
  GetGuestSummary(1);
END;
/
```

This confirms the working of your reporting procedure.

![Screenshot](screenshots/step9_test_summary.png)

---

## 🔷 Step 10: Test Trigger Blocking

```sql
BEGIN
  INSERT INTO Guests (GuestID, Name, Contact, Email)
  VALUES (99, 'Blocked User', '0700000000', 'block@example.com');
END;
/
```

Checks if weekday/holiday restrictions apply.

![Screenshot](screenshots/step10_test_insert.png)

---

## 🔷 Step 11: View Audit Logs

```sql
SELECT * FROM Audit_Log ORDER BY ActionDate DESC;
```

Used to review all user activity and determine whether access was allowed or denied.

![Screenshot](screenshots/step11_audit_log.png)

---

## ✅ Final Notes

This project combines PL/SQL concepts into a real-world hotel automation system. Key concepts demonstrated include:

- Relational database modeling (ERD)
- Business process modeling (BPMN)
- Stored procedures and triggers
- Compound triggers and auditing
- Oracle Enterprise Manager usage
- Data integrity and user restriction policies

This solution is scalable, secure, and aligned with modern enterprise requirements.

> "Whatever you do, work at it with all your heart, as working for the Lord, not for human masters." — *Colossians 3:23*

---
