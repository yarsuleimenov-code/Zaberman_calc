# Zaberman Broker Calculator MVP — Pricing Engine Flow

## 1. Purpose

Документ описывает последовательность работы pricing engine.

Цель:
- стандартизировать pricing calculations;
- исключить неоднозначность;
- зафиксировать calculation order;
- подготовить backend implementation.

---

# 2. Core Principle

Pricing engine:
- получает input data;
- рассчитывает pricing;
- возвращает calculation result.

UI не должен самостоятельно выполнять pricing calculations.

---

# 3. High-Level Pricing Flow

```text
Customer Input
↓
Route Input
↓
Items Input
↓
Physical Volume Calculation
↓
Effective Volume Calculation
↓
Vehicle Fit
↓
Operational Cost
↓
Additional Charges
↓
Discounts
↓
Margin
↓
Rounding
↓
Final Price
↓
Estimate Snapshot
```

---

# 4. Input Stage

## Customer Input

Pricing engine получает:
- customer info;
- pickup address;
- delivery address;
- route options;
- delivery preferences.

---

## Items Input

Pricing engine получает:
- dimensions;
- weight;
- quantity;
- packaging type;
- fragile flag;
- non-stackable flag;
- crated flag;
- declared value.

---

# 5. Physical Volume Calculation

## Formula

```text
Physical Volume =
(Length × Width × Height × Quantity) / 1728
```

Unit:
```text
cu ft
```

---

# 6. Effective Volume Calculation

## Purpose

Effective volume показывает реальный space usage внутри vehicle.

---

## Base Formula

```text
Effective Volume =
Physical Volume × Stackability Coefficient
```

---

## Stackability Logic

### Standard Item

```text
Coefficient = 1.0
```

---

### Fragile Item

```text
Coefficient > 1.0
```

Причина:
- требуется безопасное размещение;
- ограничивается stacking.

---

### Non-Stackable Item

```text
Coefficient significantly > 1.0
```

Причина:
- над предметом учитывается воздух;
- stacking запрещён.

---

### Crated Item

Crating:
- увеличивает volume;
- увеличивает handling complexity;
- может автоматически делать item non-stackable.

---

# 7. Total Shipment Aggregation

## Total Weight

```text
SUM(Item Weight × Quantity)
```

---

## Total Physical Volume

```text
SUM(Item Physical Volume)
```

---

## Total Effective Volume

```text
SUM(Item Effective Volume)
```

---

# 8. Vehicle Fit Logic

## Purpose

Автоматически подобрать vehicle.

---

## Vehicle Selection Rule

```text
Select first active vehicle where:

Vehicle Capacity ≥ Total Effective Volume
AND
Vehicle Max Weight ≥ Total Weight
```

---

## Multi-Vehicle Logic

Если shipment не помещается:

```text
Vehicle Count =
CEILING(
Total Effective Volume / Vehicle Capacity
)
```

---

# 9. Route Logic

Route calculation использует:
- pickup location;
- delivery location;
- route zones;
- distance;
- operational region.

---

## Distance Source

Future:
```text
Google Maps / Mapbox API
```

---

## Zone Logic

Zones могут влиять на:
- delivery fee;
- NYC surcharge;
- parking reserve;
- toll reserve;
- access complexity.

---

# 10. Operational Cost Calculation

Operational Cost состоит из:

```text
Pickup Cost
+
Interstate Cost
+
Delivery Cost
```

---

## 10.1 Pickup Cost

Может включать:
- labor;
- loading time;
- access complexity;
- stairs;
- long carry;
- parking;
- tolls.

---

## 10.2 Interstate Cost

Может включать:
- mileage;
- fuel;
- vehicle wear;
- driver cost;
- route sharing logic.

---

## 10.3 Delivery Cost

Может включать:
- unloading;
- stairs;
- access;
- delivery scheduling;
- exclusive delivery.

---

# 11. Additional Charges

Additional Charges не должны дублировать Operational Cost.

---

## Included Charges

```text
Packaging
Crating
Storage
Insurance
Exclusive Delivery
Access Fees
Zone Fees
```

---

# 12. Packaging Logic

Packaging рассчитывается:
- per item;
- per packaging type;
- per quantity.

---

## Formula

```text
Packaging Cost =
SUM(
Packaging Price × Quantity
)
```

---

# 13. Crating Logic

Crating рассчитывается отдельно от packaging.

---

## Formula

```text
Crate Cost =
Material Cost
+
Labor Cost
```

---

## Crating Effects

Crating может:
- увеличивать effective volume;
- увеличивать handling complexity;
- делать item non-stackable.

---

# 14. Insurance Logic

## Basic Liability

```text
Coverage =
Total Weight × $0.60
```

---

## Full Coverage

```text
Insurance Fee =
Declared Value × Coverage Rate
```

---

# 15. Storage Logic

## Formula

```text
Storage Cost =
Storage Days
×
Storage Daily Rate
×
Total Effective Volume
```

---

# 16. Consolidation Discount Logic

Discount применяется если:
- несколько заказов с одного pickup address;
- несколько заказов на один delivery address;
- shared route.

---

## Formula

```text
Discount Amount =
Eligible Charges × Discount %
```

---

# 17. Margin Calculation

Margin рассчитывается после:
- operational cost;
- additional charges.

---

## Formula

```text
Margin =
(
Operational Cost
+
Additional Charges
)
×
Margin Rate
```

---

# 18. Raw Final Price

## Formula

```text
Raw Final Price =
Operational Cost
+
Additional Charges
+
Margin
-
Discounts
```

---

# 19. Rounding Logic

## Purpose

Сделать customer-facing price cleaner.

---

## Formula

```text
Rounded Final Price =
CEILING(
Raw Final Price,
Rounding Rule
)
```

---

# 20. Estimate Snapshot Generation

После расчёта система фиксирует snapshot:

```text
Customer
Addresses
Items
Vehicle
Variables
Formula Version
Calculation Result
Final Price
```

Estimate становится immutable customer-facing document.

---

# 21. Future Expansion

Pricing engine должен поддерживать:
- dynamic pricing;
- regional pricing models;
- route optimization;
- warehouse logic;
- dispatch optimization;
- ML/rule-based adjustments;
- seasonal pricing;
- live fuel updates.

---

# 22. Key Architectural Rules

## Rule 1

```text
Pricing calculations belong to backend pricing engine.
```

---

## Rule 2

```text
Estimate must store frozen calculation snapshot.
```

---

## Rule 3

```text
Operational layer must not recalculate pricing.
```

---

## Rule 4

```text
Breakdown screen must explain calculations,
not perform calculations.
```
