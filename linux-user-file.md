# Linux User & Directory Management – Cloud Infrastructure Setup

> **Role:** Cloud Engineer  
> **Task:** Create company users, assign groups, create directories, and configure access permissions.

---

## 📋 Table of Contents

1. [Overview](#overview)
2. [Step 1 – Create Groups](#step-1--create-groups)
3. [Step 2 – Create Users & Assign Groups](#step-2--create-users--assign-groups)
4. [Step 3 – Verify Users & Groups](#step-3--verify-users--groups)
5. [Step 4 – Create Company Directories](#step-4--create-company-directories)
6. [Step 5 – Set Directory Ownership & Permissions](#step-5--set-directory-ownership--permissions)
7. [Step 6 – Verify Directory Permissions](#step-6--verify-directory-permissions)
8. [Screenshots](#screenshots)

---

## Overview

The following employees and their roles were provisioned on the server:

| Employee     | Role                     | Group              |
|--------------|--------------------------|--------------------|
| Andrew       | System Administrator     | `sysadmin`         |
| Julius       | Legal                    | `legal`            |
| Chizi        | Human Resource Manager   | `hr`               |
| Jeniffer     | Sales Manager            | `sales`            |
| Adeola       | Business Strategist      | `strategy`         |
| Bach         | CEO                      | `management`       |
| Gozie        | IT Intern                | `it`               |
| Ogochukwu    | Finance Manager          | `finance`          |

The following company documents (directories) were created and secured:

| Directory                        | Authorized Groups                         |
|----------------------------------|-------------------------------------------|
| `finance_budgets`                | `finance`, `management`                  |
| `contract_documents`             | `legal`, `management`                    |
| `business_projections`           | `strategy`, `sales`, `management`        |
| `business_models`                | `strategy`, `management`                 |
| `employee_data`                  | `hr`, `management`                       |
| `vision_mission_statement`       | All employees (read-only)                 |
| `server_config_scripts`          | `sysadmin`, `it`                         |

---

## Step 1 – Create Groups

Each department/role was assigned a Linux group.

```bash
sudo groupadd sysadmin
sudo groupadd legal
sudo groupadd hr
sudo groupadd sales
sudo groupadd strategy
sudo groupadd management
sudo groupadd it
sudo groupadd finance
```

> **Verify groups were created:**
```bash
cat /etc/group | grep -E "sysadmin|legal|hr|sales|strategy|management|it|finance"
```

---

## Step 2 – Create Users & Assign Groups

Each user was created with a home directory and immediately assigned to their respective group.

```bash
# Andrew – System Administrator
sudo useradd -m -G sysadmin andrew
sudo passwd andrew

# Julius – Legal
sudo useradd -m -G legal julius
sudo passwd julius

# Chizi – Human Resource Manager
sudo useradd -m -G hr chizi
sudo passwd chizi

# Jeniffer – Sales Manager
sudo useradd -m -G sales jeniffer
sudo passwd jeniffer

# Adeola – Business Strategist
sudo useradd -m -G strategy adeola
sudo passwd adeola

# Bach – CEO
sudo useradd -m -G management bach
sudo passwd bach

# Gozie – IT Intern
sudo useradd -m -G it gozie
sudo passwd gozie

# Ogochukwu – Finance Manager
sudo useradd -m -G finance ogochukwu
sudo passwd ogochukwu
```

> **Flag explanation:**  
> `-m` → creates a home directory at `/home/<username>`  
> `-G` → assigns the user to a supplementary group

---

## Step 3 – Verify Users & Groups

```bash
# List all created users
cat /etc/passwd | grep -E "andrew|julius|chizi|jeniffer|adeola|bach|gozie|ogochukwu"

# Confirm group memberships
id andrew
id julius
id chizi
id jeniffer
id adeola
id bach
id gozie
id ogochukwu
```

---

## Step 4 – Create Company Directories

All company document directories were created under `/company_docs/`.

```bash
sudo mkdir -p /company_docs/finance_budgets
sudo mkdir -p /company_docs/contract_documents
sudo mkdir -p /company_docs/business_projections
sudo mkdir -p /company_docs/business_models
sudo mkdir -p /company_docs/employee_data
sudo mkdir -p /company_docs/vision_mission_statement
sudo mkdir -p /company_docs/server_config_scripts
```

> **Verify directories were created:**
```bash
ls -l /company_docs/
```

---

## Step 5 – Set Directory Ownership & Permissions

Each directory was assigned to its authorized group. The `chmod` `770` setting grants full access to the owner and group, while blocking all others. The `vision_mission_statement` directory was set to `750` to allow read access for all employees.

```bash
# Finance Budgets → finance group
sudo chown root:finance /company_docs/finance_budgets
sudo chmod 770 /company_docs/finance_budgets

# Contract Documents → legal group
sudo chown root:legal /company_docs/contract_documents
sudo chmod 770 /company_docs/contract_documents

# Business Projections → strategy group (sales & management added via ACL below)
sudo chown root:strategy /company_docs/business_projections
sudo chmod 770 /company_docs/business_projections

# Business Models → strategy group
sudo chown root:strategy /company_docs/business_models
sudo chmod 770 /company_docs/business_models

# Employee Data → hr group
sudo chown root:hr /company_docs/employee_data
sudo chmod 770 /company_docs/employee_data

# Vision & Mission Statement → readable by all employees
sudo chown root:root /company_docs/vision_mission_statement
sudo chmod 755 /company_docs/vision_mission_statement

# Server Config Scripts → sysadmin group
sudo chown root:sysadmin /company_docs/server_config_scripts
sudo chmod 770 /company_docs/server_config_scripts
```

### Granting CEO (management group) Access to All Directories via ACL

Since the CEO (Bach) needs access to all directories, Access Control Lists (ACLs) were used:

```bash
sudo apt install acl -y   # install ACL tools if not already present

sudo setfacl -m g:management:rwx /company_docs/finance_budgets
sudo setfacl -m g:management:rwx /company_docs/contract_documents
sudo setfacl -m g:management:rwx /company_docs/business_projections
sudo setfacl -m g:management:rwx /company_docs/business_models
sudo setfacl -m g:management:rwx /company_docs/employee_data
sudo setfacl -m g:management:rwx /company_docs/server_config_scripts

# Grant sales group access to business_projections
sudo setfacl -m g:sales:rwx /company_docs/business_projections

# Grant it group access to server_config_scripts
sudo setfacl -m g:it:rx /company_docs/server_config_scripts
```

---

## Step 6 – Verify Directory Permissions

```bash
# Check standard permissions
ls -la /company_docs/

# Check ACL permissions
getfacl /company_docs/finance_budgets
getfacl /company_docs/contract_documents
getfacl /company_docs/business_projections
getfacl /company_docs/business_models
getfacl /company_docs/employee_data
getfacl /company_docs/vision_mission_statement
getfacl /company_docs/server_config_scripts
```

---

## Screenshots

### Users Created
<!-- Replace with your actual screenshot -->
![Users Created](./screenshots/users-created.png)

### Groups Created
<!-- Replace with your actual screenshot -->
![Groups Created](./screenshots/groups-created.png)

### Directories Created
<!-- Replace with your actual screenshot -->
![Directories Created](./screenshots/directories-created.png)

### Directory Permissions
<!-- Replace with your actual screenshot -->
![Directory Permissions](./screenshots/directory-permissions.png)

### ACL Permissions
<!-- Replace with your actual screenshot -->
![ACL Permissions](./screenshots/acl-permissions.png)

---

## Summary

- ✅ **8 users** created with home directories and secure passwords  
- ✅ **8 groups** created matching company departments/roles  
- ✅ **7 company document directories** created under `/company_docs/`  
- ✅ Permissions set using `chmod`, `chown`, and `setfacl` (ACL)  
- ✅ CEO (Bach) granted access to all directories via ACL group rules  
- ✅ IT intern (Gozie) granted read/execute (not write) on server config scripts  
- ✅ Vision & Mission directory readable by all employees  
