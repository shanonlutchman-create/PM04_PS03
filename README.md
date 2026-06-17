# PM04 PS03 — Mzansi Bank FICA Compliance Bot

> **UiPath RPA Automation** | Windows Process | UiPath Studio

---

## 📋 Project Overview

This automation processes FICA (Financial Intelligence Centre Act) compliance checks for Mzansi Bank clients. The bot reads a CSV file of client records, validates each client's South African ID number, simulates submission to the FICA compliance portal, determines the compliance status, and writes an updated output CSV along with a detailed processing log.

---

## 🗂️ Project Structure

```
PS04_PS03/
│
├── Main.xaml                          # Main entry point / orchestrator
│
├── Workflows/
│   ├── ProcessClient.xaml             # Processes a single client on the FICA portal
│   └── ValidateSAID.xaml              # Validates a South African ID number
│
├── Config/
│   └── Config.csv                     # Configuration key-value settings
│
├── Data/
│   ├── Input/
│   │   └── FICA_Compliance_Data.csv   # Input: Client records to process
│   └── Output/
│       ├── FICA_Compliance_Data_Updated.csv  # Output: Updated records with FICA status
│       └── processing_log.txt                # Output: Detailed run log
│
└── README.md                          # This file
```

---

## ⚙️ Configuration

The bot reads its settings from **`Config\Config.csv`**. Update this file to change runtime behaviour without modifying any workflow.

| Key             | Default Value                          | Description                              |
|-----------------|----------------------------------------|------------------------------------------|
| `InputFilePath` | `Data\Input\FICA_Compliance_Data.csv`  | Relative path to the input CSV file      |
| `PortalURL`     | `https://ficaportal.mzansibank.co.za`  | URL of the FICA compliance portal        |
| `OutputFolder`  | `Data\Output\`                         | Folder where output files are written    |
| `MaxRetries`    | `3`                                    | Maximum retry attempts per client        |

---

## 📥 Input File Format

**File:** `Data\Input\FICA_Compliance_Data.csv`

| Column          | Type   | Description                          |
|-----------------|--------|--------------------------------------|
| `Client_ID`     | String | Unique client identifier             |
| `Full_Name`     | String | Client's full name                   |
| `SA_ID_Number`  | String | South African 13-digit ID number     |
| `FICA_Status`   | String | Initially empty — filled by the bot  |

**Example:**
```csv
Client_ID,Full_Name,SA_ID_Number,FICA_Status
C001,John Smith,8001015009087,
C002,Jane Doe,9203064800088,
```

---

## 📤 Output Files

### 1. Updated CSV — `Data\Output\FICA_Compliance_Data_Updated.csv`
Same structure as the input file, with the `FICA_Status` column populated for each client.

**Possible FICA Status values:**

| Status            | Meaning                                              |
|-------------------|------------------------------------------------------|
| `Compliant`       | Client has passed FICA compliance check              |
| `Non-Compliant`   | Client has failed FICA compliance check              |
| `Pending Review`  | Client requires manual review                        |
| `Invalid_ID`      | SA ID number failed validation — record skipped      |
| `Error`           | Unexpected system error during processing            |

### 2. Processing Log — `Data\Output\processing_log.txt`
A timestamped log of every action taken during the run.

**Log format:**
```
yyyy-MM-dd HH:mm:ss | LEVEL | Message
```

**Log levels:** `INFO`, `SUCCESS`, `WARNING`, `ERROR`, `CRITICAL`

---

## 🔄 Workflow Logic

```
Main.xaml
│
├── 1. Config Section
│   ├── Read Config\Config.csv
│   ├── Parse into DataTable
│   └── Assign: InputFilePath, PortalURL, OutputFolder, LogFilePath
│
└── 2. Try Read CSV File
    ├── Read input CSV → Parse to DataTable
    ├── Log: Process started + total records
    │
    ├── For Each Client Row:
    │   ├── Extract: Client_ID, SA_ID_Number
    │   ├── Mask SA ID (for secure logging)
    │   ├── Invoke ValidateSAID.xaml
    │   │   ├── [Valid]   → Invoke ProcessClient.xaml
    │   │   │               └── Returns: Compliant / Non-Compliant / Pending Review
    │   │   └── [Invalid] → Set FICA_Status = "Invalid_ID", log WARNING
    │   │
    │   └── Update FICA_Status in DataTable row
    │
    ├── Save Results Section
    │   ├── Build output CSV string from DataTable
    │   └── Write to Data\Output\FICA_Compliance_Data_Updated.csv
    │
    ├── Catch FileNotFoundException → Log CRITICAL error
    └── Catch Exception            → Log unexpected error
```

---

## 🧩 Sub-Workflows

### `Workflows/ValidateSAID.xaml`
Validates a South African ID number.

| Argument       | Direction | Type    | Description                        |
|----------------|-----------|---------|------------------------------------|
| `in_SAIDNumber`| In        | String  | The SA ID number to validate       |
| `out_IsValid`  | Out       | Boolean | `True` if valid, `False` otherwise |

---

### `Workflows/ProcessClient.xaml`
Simulates the submission of a client to the FICA compliance portal and returns the compliance status.

| Argument        | Direction | Type   | Description                         |
|-----------------|-----------|--------|-------------------------------------|
| `in_ClientID`   | In        | String | Client unique identifier            |
| `in_SAIDNumber` | In        | String | SA ID number of the client          |
| `in_PortalURL`  | In        | String | URL of the FICA portal              |
| `out_FICAResult`| Out       | String | Compliance result from the portal   |

**Status determination logic (simulation):**
- Last digit of SA ID `<= 6` → `Compliant`
- Last digit of SA ID `<= 8` → `Non-Compliant`
- Last digit of SA ID `= 9` → `Pending Review`

---

## 🚀 How to Run

1. **Open** the project in UiPath Studio
2. **Verify** `Config\Config.csv` has the correct paths (relative to project root)
3. **Place** your input CSV in `Data\Input\FICA_Compliance_Data.csv`
4. **Click Run** (`F5`) or **Debug** (`F7`) from `Main.xaml`
5. **Check results** in `Data\Output\` after the run completes

---

## 🛠️ Troubleshooting

| Error | Cause | Fix |
|-------|-------|-----|
| `Could not find part of path ... Input\FICA_Compliance_Data.csv` | Wrong path in Config.csv | Ensure `InputFilePath` starts with `Data\` |
| `CRITICAL: Input file not found` | CSV file is missing | Place the file in `Data\Input\` |
| All rows show `Error` | Portal connection issue | Check `PortalURL` in Config.csv |
| All rows show `Invalid_ID` | Wrong CSV column names | Ensure columns are named `Client_ID`, `SA_ID_Number` |

---

## 📦 Dependencies

| Package                    | Purpose                              |
|----------------------------|--------------------------------------|
| `UiPath.System.Activities` | File I/O, DataTable, logging         |
| `UiPath.Excel.Activities`  | Excel/CSV operations                 |
| `UiPath.UIAutomation.Activities` | UI interaction activities      |

---

## 👤 Author

**Shanon Lutchman**
- Email: shanonlutchman@gmail.com
- Project: PM04 PS03 — Alexander Forbes Group Services

---

## 📅 Version History

| Version | Date       | Description              |
|---------|------------|--------------------------|
| 1.0.0   | 2026-06-17 | Initial release          |
