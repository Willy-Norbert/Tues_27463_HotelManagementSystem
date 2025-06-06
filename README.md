# Hotel Management System – Oracle PL/SQL Capstone Project

**Student Name:** IRABARUTA Willy Norbert  
**Student ID:** 27463

## Project Overview

The **Hotel Management System** solves the challenges posed by manual reservation systems in hotels, guesthouses, and resorts. Traditional systems often face issues like double-booking, lost records, inaccurate billing, and a lack of service tracking. This project, built with **Oracle PL/SQL**, automates and secures all essential operations such as:

- Guest registration and room reservations
- Employee and service management
- Booking and payment handling
- Restriction of unauthorized actions during business rules (e.g., holidays or weekdays)
- Activity auditing and reporting

This system utilizes a **relational database model** with multiple tables, business logic via PL/SQL, and role-based operations. Below is the complete design and implementation of the system.

---

##  Phase 1: Entity Relationship Diagram (ERD)

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

![ERD](Phase%20III/erd.jpg)

---

##  Phase 2: BPMN Diagram (Business Process Model)

The **Business Process Model** illustrates how different actors interact within the system to accomplish hotel operations. Using **BPMN swimlanes**, the flow includes reservation handling, billing, auditing, and report generation.

### Actors:
- **Guest**: Initiates booking, requests services, makes payment
- **Receptionist**: Confirms bookings, updates records
- **Finance officer**: Deals with payments,validate Payments
- **Manager**: Views reports and audits
- **System**: Automates billing and restrictions

![ERD](Phase%20II/diagram.svg)

---
# Phase 4: Creating a Pluggable Database (PDB) in Oracle

## Step to Create a Pluggable Database (PDB)

### Description

creating pdb 

1. Connect to the Container Database (CDB) as a privileged user:

```sql
sqlplus / as sysdba

SQL> CREATE PLUGGABLE DATABASE mon_27463_irabaruta_hotelMS_db 
2 ADMIN USER irabaruta_admin IDENTIFIED BY irabaruta 
3 ROLES = (DBA) 
4 FILE_NAME_CONVERT = ( 
5 'C:\USERS\PC\DOWNLOADS\COMPRESSED\ORADATA\ORCLL\PDBSEED\', 
6 'C:\USERS\PC\DOWNLOADS\COMPRESSED\ORADATA\ORCLL\mon_27463_irabaruta_hotelMS 
_db\'); 

```
![ERD](Phase%20IV/createPdb.jpg)

---

##  Phase 4: Oracle Enterprise Manager (OEM) Setup

We use **Oracle Enterprise Manager** to manage the pluggable database created for the system. OEM helps monitor user sessions, resource usage, and object access in real-time.

### Setup includes:
- Creating database with naming format: `irabaruta`
- Assigning admin privileges
- Managing session stats, performance, and user activities

![OEM](Phase%20IV/oem.jpg)

---
##  Phase 5: Table Creation (DDL)

In this step, the main database structure is created using SQL `CREATE TABLE` commands. Each table is normalized, and appropriate constraints are used.

### Some of my  Tables
```sql
C:\WINDOWS\system32\cmd.exe - sqlplus irabaruta/irabaruta@localhost:1521/MON_27463_IRABARUTA_HOTELMS_DB 
2 BookingID NUMBER PRIMARY KEY, 
3 GuestID NUMBER REFERENCES Guests (GuestID), 
4 RoomID NUMBER REFERENCES Rooms (RoomID), 
5 EmployeeID NUMBER REFERENCES Employees (EmployeeID), 
6 CheckInDate DATE, 
7 CheckoutDate DATE 
8); 
Table created. 
SQL> 
SQL> Payments Table 

SQL> 
SQL> Services Table 

SQL> CREATE TABLE Services ( 
2 ServiceID NUMBER PRIMARY KEY, 
3 ServiceType VARCHAR2(50), 
4 Cost NUMBER (8,2) 
5); 

Table created. 

SQL> 
SQL> ServiceRequests Table 
SQL> CREATE TABLE ServiceRequests ( 
2 RequestID NUMBER PRIMARY KEY, 
3 GuestID NUMBER REFERENCES Guests (GuestID), 
4 ServiceID NUMBER REFERENCES Services (ServiceID), 
5 EmployeeID NUMBER REFERENCES Employees (EmployeeID), 
6 RequestDate DATE
```

### Other tables created:
- Bookings
- Payments
- Employees
- Services
- ServiceRequests

These table definitions ensure data integrity and support the full functionality of the system.

![table](Phase%20V/createTable.jpg)

---

##  Phase 5: Data Insertion (DML)

We insert realistic and meaningful data into all tables to support testing and actual usage.

### Example Insert in Rooms :
```sql

SQL> INSERT INTO Rooms VALUES (102, 'Double', 70000, 'Booked"); INSERT INTO Rooms VALUES (102, 'Double', 70000, 'Booked') 
ERROR at line 1: 
ORA-00001: unique constraint (IRABARUTA.SYS_C008226) violated 
SQL> INSERT INTO Rooms VALUES (103, 'Suite', 150000, 'Available'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (184, 'Double', 75000, 'Maintenance'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (105, 'Single', 47000, 'Available'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (106, 'Suite', 160000, 'Booked'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (107, 'Single', 46000, 'Available'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (108, 'Double', 73000, 'Booked'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (109, 'Suite', 155000, 'Available'); 
1 row created. 
SQL> INSERT INTO Rooms VALUES (110, 'Double', 72000, 'Available"); 
1 row created. 
SQL> 
다 
X
```
![table](Phase%20V/insert.jpg)

<!-- ---

These data samples allow for verifying procedures, relationships, and triggers through meaningful test cases.

![Screenshot](screenshots/data_insertion.png) -->

---

##  Phase 6: Database Interaction and Transactions

##  Step 1: Add Gender Column to Guests Table

```sql
ALTER TABLE Guests ADD Gender VARCHAR2(10);

```

This adds gender data for each guest and handles the "column already exists" error safely.

![table](Phase%20VI/guest.jpg)

---

##  Step 2: Perform Sample DML Operations

```sql
UPDATE Guests SET Contact = '0788001121' WHERE GuestID = 1;
DELETE FROM Payments WHERE PaymentID = 10;
INSERT INTO Rooms VALUES (112, 'Single', 48000, 'Available');
```

These operations simulate real-life changes and validate that the tables are functioning properly.

![table](Phase%20VI/guest.jpg)

---

##  Step 3: Procedure to Summarize Guest Bookings and Payments

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

![table](Phase%20VI/procedure.jpg)

---

##  Step 4: Holidays Table

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

---

##  Step 5: Create Audit_Log Table

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

![table](Phase%20VI/auditLog.jpg)

---

##  Step 6: Procedure for Audit Logging

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
---

##  Step 7: Trigger for Restricting Weekday and Holiday Access

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
![table](Phase%20VI/triggerHoliday.jpg)

## test if it works 
![table](Phase%20VI/weekdays.jpg)

---

##  Step 8: Compound Trigger for Multi-row Audit

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

![table](Phase%20VI/compoundTrigger.jpg)

---

##  Step 9: Run Summary Procedure

```sql
BEGIN
  GetGuestSummary(1);
END;
/
```

This confirms the working of your reporting procedure.

![table](Phase%20VI/guestTest.jpg)

---

##  Step 10: Test Trigger Blocking

```sql
BEGIN
  INSERT INTO Guests (GuestID, Name, Contact, Email)
  VALUES (99, 'Blocked User', '0700000000', 'block@example.com');
END;
/
```

Checks if weekday/holiday restrictions apply.

![table](Phase%20VI/BlockingTrigger.jpg)


---

##  Step 11: View Audit Logs

```sql
SELECT * FROM Audit_Log ORDER BY ActionDate DESC;
```

Used to review all user activity and determine whether access was allowed or denied.

![table](Phase%20VI/audtest.jpg)

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
