# 🏥 Hospital Inventory Management — UiPath RPA

An RPA (Robotic Process Automation) solution built with **UiPath Studio** to automate the tracking and management of hospital inventory item movements across departments. The bot reads inventory data and movement logs from the provided Excel file sheets, processes each transfer, updates stock levels, validates business rules, and auto-generates invoice outputs.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Workflow Architecture](#workflow-architecture)
- [File Structure](#file-structure)
- [Excel Data Files](#excel-data-files)
- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Setup & Usage](#setup--usage)

---

## Overview

Hospitals need to track the movement of medical supplies (syringes, gloves, masks, etc.) between departments like **Store**, **ICU**, **Surgery**, and **Emergency**. Manually updating spreadsheets is error-prone and time-consuming.

This UiPath automation:

1. **Reads** inventory, department stock, and movement log data from different sheets in `hospital.xlsx`.
2. **Processes** each movement — deducting stock from the source and adding it to the destination department.
3. **Validates** that stock never goes negative (throws a `BusinessRuleException` if it does).
4. **Auto-generates bill output** as `.txt` files replacing the legacy purchase orders.
5. **Writes** the updated inventory and department stock back to the Excel sheets.

---

## Workflow Architecture

```
Main.xaml
  ├── InitAllApplications.xaml    → Initializes required applications
  ├── ReadData.xaml               → Reads all 3 sheets into DataTables
  ├── ProcessMovement.xaml        → Processes each movement row (loop)
  │     ├── Deduct from Inventory (Current_Stock)
  │     ├── Add quantity to destination department (DeptStock)
  │     ├── Subtract quantity from source department (DeptStock)
  │     └── If Current_Stock < Min_Stock → Generate Bill (.txt)
  └── WriteData.xaml              → Writes updated Inventory & DeptStock back to Excel
```

### Workflow Details

| Workflow | Purpose |
|---|---|
| **Main.xaml** | Entry point. Orchestrates the full process inside a Try-Catch-Finally block. |
| **InitAllApplications.xaml** | Placeholder for initializing applications (e.g., opening Excel). |
| **ReadData.xaml** | Opens `hospital.xlsx` using Excel Process Scope and reads each sheet into a DataTable (`dtInventory`, `dtMovement`, `dtDeptStock`). |
| **ProcessMovement.xaml** | For each movement row: extracts `Item_ID`, `From`, `To`, `Quantity`; updates inventory stock; updates department-level quantities; generates the `bill.txt` invoice text file. |
| **WriteData.xaml** | Writes the updated `dtInventory` and `dtDeptStock` DataTables back to their respective Excel sheets. |

---

## File Structure

```
HospitalInventoryManagementUiPath/
├── .gitignore
├── .project/                    # UiPath project metadata
├── .settings/                   # UiPath settings
├── .tmh/                        # UiPath telemetry
├── Main.xaml                    # Main entry-point workflow
├── InitAllApplications.xaml     # Application initialization workflow
├── ReadData.xaml                # Excel data reading workflow
├── ProcessMovement.xaml         # Movement processing & PO generation workflow
├── WriteData.xaml               # Excel data writing workflow
├── project.json                 # UiPath project configuration
├── project.uiproj               # UiPath project file
├── entry-points.json            # Workflow entry points definition
├── hospital.xlsx                # Master data file containing Inventory, Logs, and Stock sheets (input/output)
└── bill.txt                     # Output invoice format
```

---

## Excel Data Files

### `hospital.xlsx` 
A unified Excel file containing different sheets for our datasets.

#### 1. **Stock Sheet**: 
Tracks each unique item's properties, expiry, location, and price.

| ProductType | UniqueItemID | ItemName      | ExpiryDate | CurrentLocation | Price | TakeHomeAllowed |
|-------------|--------------|---------------|------------|-----------------|-------|-----------------|
| SU          | UID-5001     | Syringe       | 2028-12-01 | Pharmacy        | 5     | N               |
| RE          | UID-8001     | Stethoscope   | 2030-01-01 | ICU             | 150   | N               |
| CO          | UID-9001     | Glucose IV    | 2026-05-10 | Consumed/Billed | 25    | Y               |
| CO          | UID-9002     | Expired Meds  | 2025-01-01 | Pharmacy        | 10    | Y               |

#### 2. **MovementLog Sheet**: 
Each row represents a physical transfer of a unique item between departments.

| UniqueItemID | FromDept   | ToDept     | Timestamp  |
|--------------|------------|------------|------------|
| UID-8001     | Pharmacy   | Cardiology | 2026-04-15 |
| UID-8001     | Cardiology | ICU        | 2026-04-17 |

#### 3. **PatientLog Sheet**: 
Logs consumed items that are tied to specific patients for billing.

| PatientID | UniqueItemID | ItemName   | TotalBill | Timestamp  |
|-----------|--------------|------------|-----------|------------|
| P-001     | UID-9001     | Glucose IV | 25.0      | 2026-04-15 |
| P-001     | UID-9001     | Glucose IV | 25.0      | 2026-04-17 |

---

## Example Output: `bill.txt`

When items are processed, the system replaces legacy `.txt` PO outputs with hospital invoices using the format in this file:

```text
Patient: P-001
Item: Glucose IV
Total Due: $25
```

*(Note: Also supports `Sample_Invoice_P-001.txt` format with date and precise UID data depending on the workflow invocation).*

---

## How It Works

### Step-by-step Execution

1. **Initialize** — `InitAllApplications.xaml` is invoked to set up the environment.
2. **Read Data** — `ReadData.xaml` opens the multi-sheet `hospital.xlsx` and loads them into in-memory DataTables:
   - `dtInventory` ← `Inventory` sheet
   - `dtMovement` ← `MovementLog` sheet
   - `dtDeptStock` ← `DepartmentStock` sheet
3. **Process Movements** — For each row in `dtMovement`, `ProcessMovement.xaml` performs the tracking operations.
4. **Output Generation** — Instead of generating generic purchase orders, the bot auto-generates the `bill.txt` styled patient invoices.
5. **Write Data** — `WriteData.xaml` writes the updated `dtInventory` and `dtDeptStock` back to the sheets in `hospital.xlsx`.

### Error Handling

- The entire process runs inside a **Try-Catch-Finally** block.
- Any exception is caught, and the error message is logged via `LogMessage`.

---

## Prerequisites

- **UiPath Studio** (version 26.0 or later recommended)
- **Microsoft Excel** installed on the machine
- **UiPath Dependencies**:
  - `UiPath.Excel.Activities` v3.4.1
  - `UiPath.System.Activities` v26.2.1

---

## Setup & Usage

1. **Clone the repository**:
   `ash
   git clone https://github.com/Shyam5002b/HospitalInventoryManagementUiPath.git
   `

2. **Open in UiPath Studio**: Open the `project.json` file in UiPath Studio.

3. **Update Excel file paths**: Update the `WorkbookPath` in `ReadData.xaml` and `WriteData.xaml` to point to `hospital.xlsx`.

4. **Place Data**: Ensure `hospital.xlsx` is provided with the required sheets before running.

5. **Run**: Click **Run** in UiPath Studio. The bot will process all logs and generate `bill.txt` output appropriately.
