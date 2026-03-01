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

Key Calculations
•	WarehouseStock = onhand - reserved
•	TotalInventory = WarehouseStock + RegionalStock

Salesforce
•	Custom Object: Inventory__c
•	External ID Field: ItemId__c
•	Fields:
o	ItemId__c (Text)
o	WarehouseStock__c (Number)
o	RegionalStock__c (Number)
o	TotalInventory__c (Number)
o	Status__c (Picklist/Text: LowStock/Normal/High

JMS Alerts
Queue: procurement.alert.queue
Message body:
•	itemId
•	stockLevel
•	timestamp

Error Handling
Record-level
•	Validation/transform errors are handled with On Error Continue per record
•	Failed records are logged with structured JSON log lines

Global - for errros that we are not caught at the flow or processor level
•	DB/Salesforce connectivity failures propagate and fail the run
•	Structured logging included for fatal connectivity issues
Munit Testing:

Architecture diagram:
<img width="1510" height="955" alt="image" src="https://github.com/user-attachments/assets/6dfac5b7-de88-4d78-8de7-a7d0c395821a" />
<img width="468" height="612" alt="image" src="https://github.com/user-attachments/assets/f1f83ff4-9d72-41e5-914c-122ae7e2c57a" />
