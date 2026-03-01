# gocs-inventory-papi-2

## Overview
This Mule 4 application processes regional inventory CSV files, updates a MySQL database, sends JMS alerts for low stock, and upserts inventory records into Salesforce.

## Architecture - Overview of steps involved in this project
- File Listener: monitors `Regional-Updates/` for new CSV files
- Batch Job: processes each CSV row as a record
- Database:
  - SELECT existing onhand/reserved by (item_id, location_id)
  - UPSERT regional_stock + last_updated_at
  - Maintains onhand/reserved and supports warehouse stock calculations
- Routing:
  - TotalInventory < 10: Send JMS alert + Upsert to Salesforce
  - TotalInventory > 100: Upsert to Salesforce
  - Otherwise: DB only
- File Move: moves processed file to `Regional-Processed/`

## Input CSV Format
Headers:
- ItemId
- LocationId
- RegionalStock
- LastUpdatedAt (yyyy-MM-dd HH:mm:ss)

Example:
```csv
ItemId,LocationId,RegionalStock,LastUpdatedAt
1203,WH1,7,2026-03-01 10:10:00
1204,WH2,125,2026-03-01 10:11:00
