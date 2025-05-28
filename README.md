# COBOL Banking System (NYC meetup)
---
This project provides a structured training environment for mainframe development on IBM z/OS, with a focus on COBOL and DB2 integration. It includes step-by-step instructions to allocate required datasets, define and load a sample DB2 schema, and compile/run COBOL programs that interface with DB2. Designed for learners, new team members, or mainframe engineers seeking to refresh their skills, this environment simulates real-world enterprise batch processing workflows.

It is compatible with both actual z/OS systems and emulated environments such as Hercules (TK4-).

## 🛠 Development

For development, it is necessary to use the [OpenCOBOL IDE](http://opencobolide.readthedocs.io/en/latest/download.html).

---

## 📡 API Overview

### 1. Person CRUD  
Basic Create, Read, Update, Delete operations for managing people records.

### 2. User Account  
- **1**: Check Balance  
- **2**: Withdrawal  
- **3**: Deposit  
- **4**: Loan Request

### 3. Client Existence Check  
Validate if a client is already registered in the system.

### 4. Client Login  
Authenticate using client login and password.

# 🖥️ z/OS Training Environment Setup Guide

This guide provides step-by-step instructions to create a z/OS training environment, including dataset creation and a basic DB2 database setup. Intended for training and educational use.

---

## 📁 Step 1: Create Required Datasets

Use **TSO/ISPF** or submit via JCL to allocate datasets needed for COBOL and DB2 training.

### COBOL Source Dataset
```tso
ALLOCATE DATASET('TRAIN.COBOL.SRC') NEW SPACE(5,5) TRACKS +
  DSORG(PO) RECFM(FB) LRECL(80) BLKSIZE(800) +
  DSNTYPE(LIBRARY) DIR(5) UNIT(SYSDA) CATALOG
```

### JCL Dataset
```tso
ALLOCATE DATASET('TRAIN.JCL') NEW SPACE(5,5) TRACKS +
  DSORG(PO) RECFM(FB) LRECL(80) BLKSIZE(800) +
  DSNTYPE(LIBRARY) DIR(5) UNIT(SYSDA) CATALOG
```

### DB2 Load File Dataset
```tso
ALLOCATE DATASET('TRAIN.DB2.LOAD') NEW SPACE(5,5) TRACKS +
  DSORG(PS) RECFM(FB) LRECL(80) BLKSIZE(800) +
  UNIT(SYSDA) CATALOG
```

---

## ⚙️ Step 2: Create DB2 Schema and Table

Use **SPUFI** or **DSNTEP2** to run the SQL below:

```sql
CREATE SCHEMA TRAIN;

CREATE TABLE TRAIN.CUSTOMER (
  ID         INTEGER NOT NULL PRIMARY KEY,
  NAME       CHAR(30),
  BALANCE    DECIMAL(9,2),
  CREATED_AT DATE
);
```

---

## 📥 Step 3: Load Sample Data

Prepare a sequential dataset `TRAIN.DB2.LOAD` with fixed-block records like:

```
000000001JOHN DOE                  00001000.002024-01-01
000000002JANE SMITH                00002000.002024-01-01
```

### Sample LOAD JCL
```jcl
//LOADJOB  JOB CLASS=A,MSGCLASS=X,NOTIFY=&SYSUID
//STEP1    EXEC DSNUPROC,UID='TRAIN',DB2=DSN
//SYSREC00 DD DSN=TRAIN.DB2.LOAD,DISP=SHR
//SYSPRINT DD SYSOUT=*
//SYSIN    DD *
  LOAD DATA INDDN SYSREC00
  INTO TABLE TRAIN.CUSTOMER
  (ID         POSITION(1:9)   INTEGER EXTERNAL,
   NAME       POSITION(10:39) CHAR,
   BALANCE    POSITION(40:49) DECIMAL EXTERNAL,
   CREATED_AT POSITION(50:59) DATE EXTERNAL)
/*
```

---

## 👨‍💻 Step 4: Compile and Run COBOL + DB2 Program

### Example Execution JCL
```jcl
//COBDB2  JOB CLASS=A,MSGCLASS=X,NOTIFY=&SYSUID
//STEP1   EXEC PGM=IKJEFT01
//SYSTSPRT DD SYSOUT=*
//SYSPRINT DD SYSOUT=*
//SYSOUT   DD SYSOUT=*
//SYSTSIN  DD *
DSN SYSTEM(DSN)
  RUN PROGRAM(COBDB2) PLAN(COBDB2P) -
  LIB('TRAIN.COBOL.LOADLIB')
  END
/*
```

> 📌 Replace `DSN` with your DB2 subsystem name.

---

## 🔍 Step 5: Verify Data in DB2

Use **SPUFI** or SQL tool to run:

```sql
SELECT * FROM TRAIN.CUSTOMER;
```

Check output and job logs to ensure successful data load and program execution.

---

## ✅ Summary

| Component     | Status         |
|---------------|----------------|
| Datasets      | ✅ Created     |
| DB2 Table     | ✅ Created     |
| Sample Data   | ✅ Loaded      |
| COBOL Program | ✅ Executable  |

---

## 📌 Notes

- Make sure DB2 subsystem is active.
- Use ISPF option 3.4 to browse datasets.
- Ensure proper dataset permissions for load/run jobs.
- For batch COBOL-DB2 programs, perform BIND before execution:
  ```sql
  BIND PLAN(COBDB2P) MEMBER(COBDB2) ACTION(REPLACE)
  ```

---

## 📚 Resources

- [IBM DB2 Documentation](https://www.ibm.com/docs/en/db2)
- [Enterprise COBOL for z/OS](https://www.ibm.com/products/cobol-compiler-zos)
- [Hercules Emulator (TK4-)](http://wotho.ethz.ch/tk4-/)

---

> 🛠 Need help automating this? Let me know and I can provide a script-driven setup using REXX or JCL templates.

