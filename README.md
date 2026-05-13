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

```text
Main.xaml
  ├── Track_Item.xaml             → Handles inventory tracking operations
  ├── Move_Item.xaml              → Processes movement of unique items
  └── Use_Item.xaml               → Manages consumption and billing
```

### Workflow Details

| Workflow | Purpose |
|---|---|
| **Main.xaml** | Entry point. Orchestrates the flow based on action type. |
| **Track_Item.xaml** | Reads and updates inventory from the Stock sheet. |
| **Move_Item.xaml** | Processes the movement logs and updates item locations. |
| **Use_Item.xaml** | Uses items, logs to PatientLog, and generates patient bills. |
| **Debug.xaml** | Debugging utility. |

---

## File Structure

```text
HospitalInventoryManagementUiPath/
├── .gitignore
├── .project/                    # UiPath project metadata
├── .settings/                   # UiPath settings
├── .tmh/                        # UiPath telemetry
├── Main.xaml                    # Main entry-point workflow
├── Track_Item.xaml              # Modular workflow: Track inventory
├── Move_Item.xaml               # Modular workflow: Move items
├── Use_Item.xaml                # Modular workflow: Consume items/Generate bills
├── Debug.xaml                   # Debugging script
├── entry-points.json            # Workflow entry points definition
├── project.json                 # UiPath project configuration
├── project.uiproj               # UiPath project file
├── Old/                         # Archived legacy code (ReadData, ProcessMovement, etc.)
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

### Execution Flow based on Modular Workflows

The codebase is heavily refactored for specific functional capabilities:

1. **Routing** — `Main.xaml` determines which action needs to be processed.
2. **Item Tracking** — In `Track_Item.xaml`, the bot interacts with the `Stock` sheet to read location and property details of a specific UniqueItemID.
3. **Item Movement** — `Move_Item.xaml` manages records. If an item needs moving, it updates its location in the Stock and logs the action in the `MovementLog` sheet.
4. **Item Consumption & Billing** — When items are dispatched for a patient, `Use_Item.xaml` determines the item price from the stock sheet, creates an entry in `PatientLog`, and dynamically auto-generates the `bill.txt` invoice receipt formatting.
5. Legacy capabilities are stored in `/Old/` strictly for archiving reference.

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

3. **Update Excel file paths**: Update the `WorkbookPath` in `ReadData.xaml` and `WriteData.xaml` to point to `hospital.xlsx`.

4. **Place Data**: Ensure `hospital.xlsx` is provided with the required sheets before running.

5. **Run**: Click **Run** in UiPath Studio. The bot will process all logs and generate `bill.txt` output appropriately.
