# 🏪 Smart Unmanned Convenience Store
### Inventory & Expiry Date Management Program

> A Python 3 CLI application for managing convenience store inventory, sales, customer records, and expiry-based automation through text file data storage.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Usage](#usage)
- [Data Files](#data-files)
- [Program Flow](#program-flow)
- [Data Specifications](#data-specifications)
- [Customer Grade System](#customer-grade-system)
- [Auto Discount Rules](#auto-discount-rules)
- [Team](#team)

---

## Overview

This program stores product information sold at a convenience store as data files and updates the data every time an event such as stock-in, sale, or disposal occurs.

All inventory state is **derived from event logs** rather than stored directly — stock counts are calculated in real time by summing IN, OUT, and DISPOSE events from the event log file.

---

## Features

| Feature | Description |
|---|---|
| 📦 **Inventory Management** | Receive new stock, query by multiple filters, view full inventory, view disposal history |
| 💳 **Sales Processing** | Multi-item transactions with member/non-member support, coupon application, and multiple payment methods |
| 👤 **Customer Management** | Register customers, look up by ID or name, manage loyalty points and coupons |
| 🗂️ **Category Management** | Dynamically add, rename, or delete product categories |
| ⏰ **Auto Expiry Disposal** | Automatically disposes expired stock on program startup |
| 🏅 **Auto Grade Updates** | Monthly customer grade recalculation with automatic coupon issuance |
| 🔍 **Virtual Datetime Query** | Simulate inventory at any arbitrary datetime without modifying actual data |

---

## Requirements

- **Python 3.10+** (CPython standard interpreter)
- **OS:** Windows 10 / Windows 11 (recommended)
  - Linux (Ubuntu) and macOS are also supported with Python 3 installed, but minor differences in file path separators and terminal encoding may occur
- **Terminal:** Supports Unicode (UTF-8) input/output
  - Windows: `cmd.exe`, PowerShell, or Python IDLE

---

## Installation

No installer is required. Simply extract all provided files into a local directory.

```
your-directory/
├── main.py
├── productData.txt
├── productLog.txt
├── customerData.txt
├── customerLog.txt
├── categoryData.txt
└── couponData.txt
```

> ⚠️ All data files must be located in the **same directory** as `main.py`.

---

## Usage

### Windows Terminal (Recommended)

```bash
cd path/to/your-directory
python main.py
```

### Python IDLE

Open `main.py` in Python IDLE, then press **F5** or go to **Run → Run Module**.

---

## Data Files

The program uses 6 text files (UTF-8 without BOM). Fields are separated by `|` (pipe). Records are separated by newlines.

| File | Description | Record Format |
|---|---|---|
| `productData.txt` | Product catalog | `<productID>\|<name>\|<price>\|<category>\|<stockDate>\|<expiryDate>` |
| `productLog.txt` | Inventory event log | `<eventID>\|<productID(s)>\|<eventType>\|<eventTime>` |
| `customerData.txt` | Customer records | `<customerID>\|<name>\|<points>\|<grade>\|<lastGradeUpdate>\|<couponID(s)>` |
| `customerLog.txt` | Purchase history | `<purchaseID>\|<customerID>\|<productID(s)>\|<price>\|<couponID>\|<payMethod>\|<qty>\|<eventTime>` |
| `categoryData.txt` | Category list | `<categoryID>\|<categoryName>` |
| `couponData.txt` | Coupon details | `<couponID>\|<type>\|<discountValue>\|<expiry>\|<usability>` |

### Event Types (`productLog.txt`)
- `IN` — stock received
- `OUT` — item sold
- `DISPOSE` — item disposed

### Default Categories (`categoryData.txt`)

| ID | Category |
|---|---|
| C001 | 냉동식품 (Frozen Food) |
| C002 | 냉장식품 (Refrigerated Food) |
| C003 | 과자 (Snacks) |
| C004 | 음료 (Beverages) |
| C005 | 주류 (Alcohol) |
| C006 | 생활용품 (Daily Necessities) |
| C007 | 기타 (Other) |

> 🔒 Default categories (C001–C007) **cannot be modified or deleted**.

---

## Program Flow

```
Program Start
    │
    ├── File existence & permission check
    ├── Data integrity validation
    ├── Current datetime input (user-defined)
    ├── Auto-dispose expired stock
    ├── Auto-update customer grades + issue coupons
    └── Auto-expire stale coupons
            │
            ▼
        Main Menu
    ┌───────────────────────────┐
    │ 1. Customer Management    │
    │ 2. Inventory Management   │
    │ 3. Sales Processing       │
    │ 4. Virtual Datetime Query │
    │ 5. Change Current Datetime│
    │ 6. Category Management    │
    │ 0. Exit                   │
    └───────────────────────────┘
```

---

## Data Specifications

### Product ID / Customer ID
- 4-digit numeric string (e.g., `0001`–`9999`)
- Auto-assigned: smallest unused value starting from `0001`
- `0000` is reserved as the **non-member ID** for anonymous transactions

### Datetime Format
```
YYYY-MM-DD HH:MM:SS
```
- The "current datetime" is **user-defined** at startup, not the system clock
- It must always be later than the most recent event timestamp in the log files

### Inventory Calculation
```
Stock = (IN events) - (OUT events) - (DISPOSE events)
```
Calculated from `productLog.txt` at runtime; never stored directly.

---

## Customer Grade System

Grades are recalculated monthly based on the **previous month's total purchase amount**. Coupons are automatically issued upon each grade update.

| Grade | Monthly Spend | Coupons Issued |
|---|---|---|
| 🥉 BRONZE | < ₩30,000 | FLAT ₩1,000 × 1 (30-day validity) |
| 🥈 SILVER | ₩30,000 – ₩99,999 | FLAT ₩2,000 × 1, RATE 5% × 1 |
| 🥇 GOLD | ₩100,000 – ₩299,999 | FLAT ₩5,000 × 1, RATE 10% × 1, FLAT ₩1,000 × 2 |
| 💎 VIP | ≥ ₩300,000 | FLAT ₩10,000 × 1, RATE 15% × 1, FLAT ₩2,000 × 3 |

- New customers start at **BRONZE**
- Only **1 coupon** can be used per transaction, applied to **1 item only**
- Coupons and points **cannot be used simultaneously**
- No points are earned when a coupon is used

---

## Auto Discount Rules

Applied automatically at time of sale based on remaining time until expiry:

| Time Until Expiry | Discount |
|---|---|
| > 7 days | No discount |
| 3–7 days | **20% off** |
| ≤ 3 days (72 hours) | **50% off** |

> Discounted prices are **rounded down** to the nearest integer (₩).  
> RATE coupons stack multiplicatively with auto-discounts.

---

## Team

**B08 Team**

| Student ID | Name |
|---|---|
| 202212374 | 정연우 |
| 202212361 | 안진모 |
| 202212382 | 현종화 |
| 202312367 | 이준우 |
| 202510934 | 구지현 |
| 202311347 | 이재원 |
| 202314516 | 루즈한 |

---

## Document Version

| Document | Version | Description |
|---|---|---|
| 기획서 원판 | Original | Base specification |
| 기획서 설계기준 | 1st Edition | Revised design standards with 5 key updates |

### Key Changes in Design Standards (1st Edition)
- **[Terms]** Current datetime validation now includes last grade update timestamp
- **[Flow Diagram]** Added missing Customer Management → Coupon Inquiry branch
- **[5.7.3]** Coupon IDs cannot be shared across customers; only USABLE coupons stored in `customerData.txt`
- **[6.7.4]** Input format for category edit function concretized
- **[6.7.5]** Input format for category delete function concretized
