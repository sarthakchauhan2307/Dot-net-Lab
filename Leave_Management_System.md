# 🗂️ Leave Management System — Database Design

> A structured SQL Server database to manage employee leave requests, balances, and approvals across departments.

---

## 📋 Table of Contents

- [Introduction](#introduction)
- [Database Creation](#database-creation)
- [ER Diagram](#er-diagram)
- [Table Relationships](#table-relationships)
- [Table Definitions](#table-definitions)
  - [1. Departments](#1-departments-table)
  - [2. Employees](#2-employees-table)
  - [3. LeaveTypes](#3-leavetypes-table)
  - [4. LeaveRequests](#4-leaverequests-table)
  - [5. LeaveBalances](#5-leavebalances-table)
- [Normalization](#normalization)
- [Key Formula](#key-formula)
- [Summary](#summary)

---

## Introduction

The **Leave Management System** is a relational database that handles:

- Employee and department records
- Different types of leaves (paid/unpaid)
- Leave requests with approval status
- Leave balance tracking per employee

---

## Database Creation

```sql
CREATE DATABASE LeaveManagementSystem;
USE LeaveManagementSystem;
```

---

```mermaid
erDiagram
    DEPARTMENTS {
        int DepartmentID PK
        varchar DepartmentName
    }

    EMPLOYEES {
        int EmployeeID PK
        varchar FirstName
        varchar LastName
        varchar Email
        varchar Gender
        date HireDate
        int DepartmentID FK
        int ManagerID FK
    }

    LEAVETYPES {
        int LeaveTypeID PK
        varchar LeaveTypeName
        int MaxDays
        bit IsPaid
    }

    LEAVEREQUESTS {
        int LeaveRequestID PK
        int EmployeeID FK
        int LeaveTypeID FK
        date StartDate
        date EndDate
        int TotalDays
        varchar Reason
        varchar Status
        datetime AppliedDate
    }

    LEAVEBALANCES {
        int BalanceID PK
        int EmployeeID FK
        int LeaveTypeID FK
        int TotalLeave
        int UsedLeave
        int RemainingLeave
    }

    DEPARTMENTS ||--o{ EMPLOYEES : "has"
    EMPLOYEES ||--o{ EMPLOYEES : "manages"
    EMPLOYEES ||--o{ LEAVEREQUESTS : "submits"
    LEAVETYPES ||--o{ LEAVEREQUESTS : "categorizes"
    EMPLOYEES ||--o{ LEAVEBALANCES : "has"
    LEAVETYPES ||--o{ LEAVEBALANCES : "tracks"
` ``


## Table Relationships

| Parent Table   | Child Table      | Relationship Type | Description                         |
|----------------|------------------|-------------------|-------------------------------------|
| `Departments`  | `Employees`      | One-to-Many       | A department has many employees     |
| `Employees`    | `Employees`      | Self-referencing  | An employee can manage others       |
| `Employees`    | `LeaveRequests`  | One-to-Many       | An employee submits many requests   |
| `LeaveTypes`   | `LeaveRequests`  | One-to-Many       | A leave type applies to many requests |
| `Employees`    | `LeaveBalances`  | One-to-Many       | An employee has balances per leave type |
| `LeaveTypes`   | `LeaveBalances`  | One-to-Many       | A leave type tracks multiple balances |

---

## Table Definitions

---

### 1. Departments Table

> Stores department/team information.

```sql
CREATE TABLE Departments (
    DepartmentID   INT PRIMARY KEY IDENTITY(1,1),
    DepartmentName VARCHAR(100) NOT NULL UNIQUE
);
```

| Column           | Data Type     | Constraint        | Description              |
|------------------|---------------|-------------------|--------------------------|
| `DepartmentID`   | INT           | PK, Auto-Increment | Unique department ID     |
| `DepartmentName` | VARCHAR(100)  | NOT NULL, UNIQUE  | Name of the department   |

---

### 2. Employees Table

> Stores all employee records including self-referencing manager relationship.

```sql
CREATE TABLE Employees (
    EmployeeID   INT PRIMARY KEY IDENTITY(1,1),
    FirstName    VARCHAR(50)  NOT NULL,
    LastName     VARCHAR(50)  NOT NULL,
    Email        VARCHAR(100) UNIQUE NOT NULL,
    Gender       VARCHAR(10),
    HireDate     DATE         NOT NULL,
    DepartmentID INT,
    ManagerID    INT NULL,
    FOREIGN KEY (DepartmentID) REFERENCES Departments(DepartmentID),
    FOREIGN KEY (ManagerID)    REFERENCES Employees(EmployeeID)
);
```

| Column         | Data Type    | Constraint        | Description                         |
|----------------|--------------|-------------------|-------------------------------------|
| `EmployeeID`   | INT          | PK, Auto-Increment | Unique employee ID                  |
| `FirstName`    | VARCHAR(50)  | NOT NULL          | Employee's first name               |
| `LastName`     | VARCHAR(50)  | NOT NULL          | Employee's last name                |
| `Email`        | VARCHAR(100) | UNIQUE, NOT NULL  | Employee's email address            |
| `Gender`       | VARCHAR(10)  | —                 | Employee's gender                   |
| `HireDate`     | DATE         | NOT NULL          | Date of joining                     |
| `DepartmentID` | INT          | FK → Departments  | Department the employee belongs to  |
| `ManagerID`    | INT          | FK → Employees    | Self-reference to manager employee  |

> 💡 `ManagerID` is a **self-referencing foreign key** — it points back to `Employees(EmployeeID)`.

---

### 3. LeaveTypes Table

> Defines the categories of leave available (e.g., Sick Leave, Casual Leave).

```sql
CREATE TABLE LeaveTypes (
    LeaveTypeID   INT PRIMARY KEY IDENTITY(1,1),
    LeaveTypeName VARCHAR(50) NOT NULL UNIQUE,
    MaxDays       INT         NOT NULL,
    IsPaid        BIT         DEFAULT 1
);
```

| Column         | Data Type   | Constraint        | Description                       |
|----------------|-------------|-------------------|-----------------------------------|
| `LeaveTypeID`  | INT         | PK, Auto-Increment | Unique leave type ID              |
| `LeaveTypeName`| VARCHAR(50) | NOT NULL, UNIQUE  | Name of leave type                |
| `MaxDays`      | INT         | NOT NULL          | Maximum days allowed per year     |
| `IsPaid`       | BIT         | DEFAULT 1         | `1` = Paid Leave, `0` = Unpaid    |

---

### 4. LeaveRequests Table

> Records every leave application submitted by employees.

```sql
CREATE TABLE LeaveRequests (
    LeaveRequestID INT PRIMARY KEY IDENTITY(1,1),
    EmployeeID     INT          NOT NULL,
    LeaveTypeID    INT          NOT NULL,
    StartDate      DATE         NOT NULL,
    EndDate        DATE         NOT NULL,
    TotalDays      INT          NOT NULL,
    Reason         VARCHAR(300),
    Status         VARCHAR(20)  DEFAULT 'Pending',
    AppliedDate    DATETIME     DEFAULT GETDATE(),
    FOREIGN KEY (EmployeeID)  REFERENCES Employees(EmployeeID),
    FOREIGN KEY (LeaveTypeID) REFERENCES LeaveTypes(LeaveTypeID)
);
```

| Column           | Data Type    | Constraint        | Description                              |
|------------------|--------------|-------------------|------------------------------------------|
| `LeaveRequestID` | INT          | PK, Auto-Increment | Unique request ID                       |
| `EmployeeID`     | INT          | FK → Employees    | Who submitted the request                |
| `LeaveTypeID`    | INT          | FK → LeaveTypes   | Type of leave requested                  |
| `StartDate`      | DATE         | NOT NULL          | Leave start date                         |
| `EndDate`        | DATE         | NOT NULL          | Leave end date                           |
| `TotalDays`      | INT          | NOT NULL          | Number of leave days                     |
| `Reason`         | VARCHAR(300) | —                 | Reason for leave                         |
| `Status`         | VARCHAR(20)  | DEFAULT 'Pending' | `Pending` / `Approved` / `Rejected`      |
| `AppliedDate`    | DATETIME     | DEFAULT GETDATE() | Auto-stamped date when request was made  |

---

### 5. LeaveBalances Table

> Tracks how many leave days each employee has used and has remaining.

```sql
CREATE TABLE LeaveBalances (
    BalanceID      INT PRIMARY KEY IDENTITY(1,1),
    EmployeeID     INT NOT NULL,
    LeaveTypeID    INT NOT NULL,
    TotalLeave     INT NOT NULL,
    UsedLeave      INT DEFAULT 0,
    RemainingLeave AS (TotalLeave - UsedLeave),
    FOREIGN KEY (EmployeeID)  REFERENCES Employees(EmployeeID),
    FOREIGN KEY (LeaveTypeID) REFERENCES LeaveTypes(LeaveTypeID)
);
```

| Column           | Data Type | Constraint         | Description                          |
|------------------|-----------|--------------------|--------------------------------------|
| `BalanceID`      | INT       | PK, Auto-Increment | Unique balance record ID             |
| `EmployeeID`     | INT       | FK → Employees     | Which employee                       |
| `LeaveTypeID`    | INT       | FK → LeaveTypes    | Which leave type                     |
| `TotalLeave`     | INT       | NOT NULL           | Total days allocated                 |
| `UsedLeave`      | INT       | DEFAULT 0          | Days used so far                     |
| `RemainingLeave` | INT       | **Computed Column**| Auto-calculated: `TotalLeave - UsedLeave` |

> 💡 `RemainingLeave` is a **computed/derived column** in SQL Server — no manual update needed.

---

## Key Formula

```
RemainingLeave = TotalLeave - UsedLeave
```

This is enforced at the database level as a **computed column**, so it stays consistent automatically.

---

## Normalization

This database follows the first three normal forms:

| Normal Form | Rule Applied | How It's Met |
|-------------|-------------|--------------|
| **1NF** | Atomic values, no repeating groups | Every column holds a single value |
| **2NF** | No partial dependency on composite key | All non-key columns depend on the full PK |
| **3NF** | No transitive dependency | Non-key columns depend only on the PK, not on each other |

---

## Summary

This database covers the full lifecycle of employee leave management:

| Feature                   | Table Responsible     |
|---------------------------|-----------------------|
| Department tracking       | `Departments`         |
| Employee records          | `Employees`           |
| Manager hierarchy         | `Employees` (self-FK) |
| Leave categories & limits | `LeaveTypes`          |
| Leave request workflow    | `LeaveRequests`       |
| Leave balance tracking    | `LeaveBalances`       |
| Paid/unpaid leave support | `LeaveTypes.IsPaid`   |

---

> 📌 **Tech Stack:** SQL Server (T-SQL) | **Diagram:** Mermaid ERD  
> 📁 This file is part of a database design series for teaching assistants.
