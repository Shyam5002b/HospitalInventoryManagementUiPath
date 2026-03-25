# 🏥 Hospital Inventory Management — UiPath RPA

An RPA (Robotic Process Automation) solution built with **UiPath Studio** to automate the tracking and management of hospital inventory item movements across departments. The bot reads inventory data and movement logs from Excel files, processes each transfer, updates stock levels, validates business rules, and auto-generates purchase orders when supplies run low.

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

1. **Reads** inventory, department stock, and movement log data from Excel files.
2. **Processes** each movement — deducting stock from the source and adding it to the destination department.
3. **Validates** that stock never goes negative (throws a `BusinessRuleException` if it does).
4. **Auto-generates purchase orders** as `.txt` files when an item's stock falls below its minimum threshold.
5. **Writes** the updated inventory and department stock back to Excel.

---

## Workflow Architecture

```
Main.xaml
  ├── InitAllApplications.xaml    → Initializes required applications
  ├── ReadData.xaml               → Reads all 3 Excel files into DataTables
  ├── ProcessMovement.xaml        → Processes each movement row (loop)
  │     ├── Deduct from Inventory (Current_Stock)
  │     ├── Add quantity to destination department (DeptStock)
  │     ├── Subtract quantity from source department (DeptStock)
  │     └── If Current_Stock < Min_Stock → Generate Purchase Order (.txt)
  └── WriteData.xaml              → Writes updated Inventory & DeptStock back to Excel
```

### Workflow Details

| Workflow | Purpose |
|---|---|
| **Main.xaml** | Entry point. Orchestrates the full process inside a Try-Catch-Finally block. |
| **InitAllApplications.xaml** | Placeholder for initializing applications (e.g., opening Excel). |
| **ReadData.xaml** | Opens `Inventory.xlsx`, `MovementLog.xlsx`, and `DepartmentStock.xlsx` using Excel Process Scope and reads each into a DataTable (`dtInventory`, `dtMovement`, `dtDeptStock`). |
| **ProcessMovement.xaml** | For each movement row: extracts `Item_ID`, `From`, `To`, `Quantity`; updates inventory stock; updates department-level quantities; generates a purchase order text file if stock drops below the minimum. |
| **WriteData.xaml** | Writes the updated `dtInventory` and `dtDeptStock` DataTables back to their respective Excel files. |

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
├── Inventory.xlsx               # Master inventory data (input/output)
├── MovementLog.xlsx             # Item movement log (input)
└── DepartmentStock.xlsx         # Per-department stock levels (input/output)
```

---

## Excel Data Files

### 1. `Inventory.xlsx` — Master Inventory

Tracks each item's current stock level, minimum stock threshold, and vendor contact for reordering.

| Item_ID | Item_Name         | Current_Stock | Min_Stock | Vendor_Email       |
|---------|-------------------|---------------|-----------|--------------------|
| 101     | Syringe           | 100           | 50        | vendor1@gmail.com  |
| 102     | Surgical Scissors | 40            | 20        | vendor2@gmail.com  |
| 103     | Gloves            | 200           | 100       | vendor3@gmail.com  |
| 104     | Face Mask         | 300           | 150       | vendor4@gmail.com  |

---

### 2. `DepartmentStock.xlsx` — Department-Level Stock

Tracks how many units of each item are held by each department.

| Dept_Name | Item_ID | Quantity |
|-----------|---------|----------|
| Store     | 101     | 100      |
| ICU       | 101     | 20       |
| Surgery   | 102     | 15       |
| ICU       | 103     | 50       |
| Emergency | 104     | 80       |
| Store     | 102     | 40       |
| Store     | 103     | 200      |
| Store     | 104     | 300      |

---

### 3. `MovementLog.xlsx` — Movement Transactions

Each row represents a transfer of items from one department to another.

| Item_ID | From  | To        | Quantity |
|---------|-------|-----------|----------|
| 101     | Store | ICU       | 30       |
| 102     | Store | Surgery   | 10       |
| 103     | Store | ICU       | 50       |
| 104     | Store | Emergency | 100      |
| 101     | ICU   | Store     | 10       |

---

## How It Works

### Step-by-step Execution

1. **Initialize** — `InitAllApplications.xaml` is invoked to set up the environment.
2. **Read Data** — `ReadData.xaml` opens all three Excel files and loads them into in-memory DataTables:
   - `dtInventory` ← `Inventory.xlsx`
   - `dtMovement` ← `MovementLog.xlsx`
   - `dtDeptStock` ← `DepartmentStock.xlsx`
3. **Process Movements** — For each row in `dtMovement`, `ProcessMovement.xaml` performs:
   - **Inventory Update**: Finds the matching `Item_ID` in `dtInventory` and reduces `Current_Stock` by the movement `Quantity`.
   - **Negative Stock Check**: If `Current_Stock - Quantity < 0`, throws a `BusinessRuleException("Stock cannot be negative")`.
   - **Department Stock Update (To)**: Adds the `Quantity` to the destination department's stock in `dtDeptStock`.
   - **Department Stock Update (From)**: Subtracts the `Quantity` from the source department's stock in `dtDeptStock`.
   - **Purchase Order Generation**: If the updated `Current_Stock` falls below `Min_Stock`, a purchase order `.txt` file is generated with:
     - Filename: `PO_<ItemName>.txt`
     - Order Quantity: `Min_Stock × 2`
     - Contains item name and order quantity details.
4. **Write Data** — `WriteData.xaml` writes the updated `dtInventory` and `dtDeptStock` back to `Inventory.xlsx` and `DepartmentStock.xlsx`.

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
   ```bash
   git clone https://github.com/Shyam5002b/HospitalInventoryManagementUiPath.git
   ```

2. **Open in UiPath Studio**: Open the `project.json` file in UiPath Studio.

3. **Update Excel file paths**: The workflows reference Excel files at an absolute path. Update the `WorkbookPath` in `ReadData.xaml` and `WriteData.xaml` to match your local directory.

4. **Place Excel files**: Ensure `Inventory.xlsx`, `MovementLog.xlsx`, and `DepartmentStock.xlsx` are in the project directory with the sample data shown above.

5. **Run**: Click **Run** in UiPath Studio. The bot will process all movements and update the Excel files. Check the project folder for any generated `PO_*.txt` purchase order files.
