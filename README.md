## Problem Statement: Why the Ordering System "Fails" the Manager

The current inventory software lacks real-world context, forcing managers to override the system manually. This "blindness" leads to significant financial loss through **shrink** (wasted stock) and **lost revenue** (out-of-stocks).

### Core Pain Points

* **One-Size-Fits-All Logic:** The system relies on a simple 4-week rolling average, ignoring seasonal shifts or upcoming public holidays.
* **The "Weather Gap":** Inventory doesn't sync with forecasts; storm-driven footfall drops result in overstocking perishables.
* **Festival Blindness:** The system fails to predict "Product Switching" (e.g., buying Turkey instead of Beef during holidays).
* **Mixing Up Customers:** Failure to distinguish between **Loyal Customers** (consistent demand) and **Random Customers** (weather/event-dependent demand).
* **Promotion Paradox:** A total disconnect from marketing. If Strawberries are on the front page of a flyer, the system orders for "normal" demand (5 cases) instead of "promo" demand (40 cases).
* **Manual Stress:** Managers are forced to calculate orders on paper, leading to human error and data silos.

---

## Data Model

| Table Name | Column Name | Data Type | Key/Notes |
| --- | --- | --- | --- |
| **Product** | Item_id | VARCHAR(50) | Primary Key (SKU) |
|  | Item_desc | TEXT |  |
|  | Color | VARCHAR(20) |  |
|  | Supplier | VARCHAR(100) |  |
|  | Shelf_Life_Days | INT | Crucial for shrink prevention |
|  | Category | VARCHAR(50) |  |
|  | SubCategory | VARCHAR(50) |  |
|  | Width / Height / Weight | DECIMAL(10,2) | Logistics dimensions |
| **Customer** | Customer_id | INT | Primary Key |
|  | First_name / Last_name | VARCHAR(50) |  |
|  | Email / Phone | VARCHAR(100) |  |
| **CustomerAddress** | Customer_id | INT | Foreign Key |
|  | Shipping / Billing Address | TEXT |  |
| **Sales** | Item_id | VARCHAR(50) | Foreign Key |
|  | Warehouse_site_id | INT | Foreign Key |
|  | Customer_id | INT | Foreign Key |
|  | Quantity | INT |  |
|  | SalesAmount | DECIMAL(12,2) |  |
|  | Unit Of Measure | VARCHAR(20) |  |
|  | Date / Shipping Date | DATE |  |
| **WarehouseSite** | Warehouse_site_id | INT | Primary Key |
|  | Site_location / City | VARCHAR(100) |  |
|  | State | VARCHAR(2) |  |
| **WarehouseProduct** | Item_id | VARCHAR(50) | Foreign Key |
|  | Warehouse_site_id | INT | Foreign Key |
|  | Quantity | INT | Current Stock on Hand |
|  | Date | DATE |  |
| **ProductPrice** | Item_id | VARCHAR(50) | Foreign Key |
|  | Warehouse_site_id | INT | Foreign Key |
|  | Price | DECIMAL(10,2) |  |
|  | Date | DATE |  |
|  | Min_order_quantity | INT |  |
| **UnitOfMeasure** | Item | VARCHAR(50) | e.g., "Coke" |
|  | Case_Size | INT | e.g., 30, 12, 6, 1 |
| **ProductPromotion** | Item_id | VARCHAR(50) | Foreign Key |
|  | Warehouse_Site_id | INT | Foreign Key |
|  | Price | DECIMAL(10,2) | Promo Price |
|  | StartDate / EndDate | DATE | Promotion window |

---

## The Approach: Predictive Promotional Ordering

To solve the "blindness" of the current system, we implement a **Lift-Based** forecasting model.

### 1. Data Cleaning & Preparation

* **De-duplication:** Ensure unique records for `Item_id` + `Date` + `Warehouse_site_id`.
* **Integrity Check:** Filter out negative units (returns) to avoid dragging down demand averages.
* **Null Handling:** Impute missing values so average sales reflect reality.

### 2. Promotional Analysis

We calculate the impact of marketing using the **Lift Factor**:

1. **Baseline ADS:** Average Daily Sales when **no** promotion is active.
2. **Promo ADS:** Average Daily Sales during active promotion periods.
3. **Lift Factor:** 


*(Example: If sales jump from 10 to 80 units, the Lift Factor is 8x).*

### 3. Prediction Logic for Next Week

The system checks the upcoming marketing calendar:

* **Scenario A (On Promotion):** Forecast = Historical Promo ADS.
* **Scenario B (Not on Promotion):** Forecast = Baseline ADS.

### 4. Final Order Calculation (Gap Analysis)

To prevent overstocking, the system looks at what is already on the shelf:

> **Result:** If a promo requires 150 units and the warehouse has 100, the system orders exactly **50 units**, ensuring shelves stay full without creating a surplus.
