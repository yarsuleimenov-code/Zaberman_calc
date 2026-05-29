# Developer Handoff для Zaberman Calculator

## 1. Статус репозитория

Этот репозиторий является hi-fi wireframe и visual technical specification для будущей production-системы Zaberman Broker Calculator.

Репозиторий предназначен для передачи команде разработки как визуальное и бизнес-техническое задание.

Текущий код является:

- hi-fi prototype;
- visual specification;
- workflow prototype;
- reference UI;
- mock frontend;
- основой для обсуждения бизнес-логики и UX.

Текущий код не является:

- production-ready реализацией;
- рабочим pricing engine;
- backend-системой;
- источником проверенных production-формул;
- источником production-data model;
- готовым PDF-generation pipeline;
- системой с реальным state management, persistence, validation и audit trail.

В репозитории используются:

- mock data;
- hardcoded values;
- статичные customer records;
- статичные items;
- статичные routes;
- статичные prices;
- статичные dates;
- статичные statuses;
- placeholders для будущих production-функций.

## 2. Critical Warning

Do not reuse hardcoded calculations.

Do not treat static HTML values as validated business formulas.

Do not use current screen values as production pricing logic.

All pricing formulas must be implemented in a separate tested pricing engine.

HTML files in this repository may be used to understand:

- screen structure;
- broker workflow;
- UX expectations;
- field grouping;
- naming conventions;
- visual hierarchy;
- business intent.

HTML files must not be used as source of truth for:

- pricing formulas;
- margin logic;
- insurance logic;
- fuel allocation;
- route cost allocation;
- PDF generation;
- estimate validity;
- legal liability;
- backend architecture.

## 3. Назначение репозитория

Репозиторий показывает, как должна выглядеть и работать будущая система на уровне workflow и UX.

Основные цели:

- показать workflow брокера;
- показать структуру экранов;
- показать UX quote creation;
- показать customer-facing estimate;
- показать quote lifecycle;
- показать draft / estimate / invoice / order / eBOL flow;
- показать expected domain entities and fields;
- показать expected pricing breakdown structure;
- показать, какие данные должны попадать в production pricing engine;
- показать, какие snapshots нужны для auditability;
- показать, какие areas требуют backend implementation.

## 4. Screens

| Screen | Назначение | Текущий статус | Production-команда должна реализовать |
|---|---|---|---|
| `index.html` | Основной экран создания quote. Broker вводит customer data, route, items, access conditions и quote options. | Wireframe / mock | Реальный QuoteDraft, item CRUD, validation, recalculation через pricing engine, autosave, Generate Estimate from snapshot. |
| `breakdown.html` | Экран объяснения pricing breakdown: operational cost, additional charges, margin, rounding. | Reference / mock | Read-only breakdown из pricing engine, component-level explanation, formula version, audit trail. |
| `estimate-document.html` | Customer-facing estimate document. Показывает, как должен выглядеть estimate для клиента. | Wireframe / mock | Генерация estimate document из immutable EstimateSnapshot, PDF artifact, versioning, legal text ownership. |
| `drafts.html` | Список сохраненных quote drafts. | Wireframe / mock | Backend draft persistence, search, filters, pagination, ownership, duplicate, soft delete, resume draft. |
| `estimates.html` | Список customer-facing estimates, которые были generated/sent. | Wireframe / mock | Estimate history, status lifecycle, expiration, versioning, view/send/reopen/convert actions. |
| `invoices.html` | Будущий invoice workflow. | Wireframe / mock | Invoice entity, invoice generation, payment status, provider integration, reconciliation. |
| `orders.html` | Будущий order workflow после approved estimate. | Wireframe / mock | Order lifecycle, dispatch state, operational assignment, route execution, conversion from Estimate. |
| `ebol.html` | Operational eBOL / proof workflow: item verification, pickup/delivery statuses, photos, signatures, exceptions. | Wireframe / mock | eBOL state machine, photo upload, signature capture, item-level verification, POD generation. |
| `formulas.html` | Reference documentation для pricing formulas и business rules. | Reference | Превратить business formulas в executable, versioned, tested pricing rules. |
| `variables.html` | Admin/reference screen для pricing variables: fuel, vehicles, margins, helper rules, COI, stackability. | Reference / mock | Backend configuration model, versioning, permissions, audit log, variable snapshots. |
| `references.html` | Reference data area для operational/pricing lookup tables. | Reference / mock | Production reference data source, admin editing, import/export, version control. |
| `lifecycle.html` | Visual explanation of quote lifecycle and transitions. | Reference | Formal state machines для QuoteDraft, Estimate, Invoice, Order и eBOL. |

## 5. Domain Entities

### QuoteDraft

Рабочий черновик quote до отправки клиенту.

Должен хранить:

- customer data;
- route data;
- items;
- access conditions;
- quote options;
- current pricing result;
- draft owner;
- draft status;
- last saved timestamp.

### QuoteItem

Одна строка груза внутри quote.

Должен хранить:

- item name;
- dimensions;
- weight;
- quantity;
- packaging selection;
- insurance/protection selection;
- declared value;
- storage days;
- fragile flag;
- non-stackable flag;
- crate flag;
- comments;
- photos or photo references.

### PricingVariables

Версионированный набор переменных для pricing engine.

Примеры:

- fuel rates;
- fuel surcharge;
- vehicle parameters;
- cost per mile;
- margin rules;
- minimum quote;
- helper hourly rate;
- helper minimum hours;
- COI fees;
- stackability coefficients;
- packaging rates;
- crating rates;
- access thresholds.

### PricingBreakdown

Детальный результат расчета pricing engine.

Должен объяснять:

- pickup stage cost;
- interstate stage cost;
- delivery stage cost;
- additional charges;
- packaging;
- storage;
- insurance/protection;
- COI;
- access fees;
- helper fees;
- margin;
- discounts;
- raw price;
- rounding delta;
- final customer price.

### Estimate

Customer-facing версия quote.

Estimate создается из QuoteDraft только через EstimateSnapshot.

Должен хранить:

- estimate ID;
- estimate version;
- customer-facing price;
- status;
- created date;
- expiration date;
- sent/viewed/approved/rejected state;
- PDF/document artifact reference.

### EstimateSnapshot

Immutable snapshot данных на момент генерации Estimate.

Должен хранить:

- customer snapshot;
- route snapshot;
- item snapshot;
- access conditions snapshot;
- quote options snapshot;
- pricing variables snapshot;
- formula version;
- pricing breakdown;
- final rounded price;
- generated document metadata.

Snapshot не должен изменяться после генерации Estimate.

### Customer

Клиент или lead.

Должен хранить:

- name;
- phone;
- email;
- CRM ID;
- contact preferences;
- billing/contact metadata.

### Order

Operational entity после approval/conversion Estimate.

Должен хранить:

- order ID;
- linked estimate;
- order status;
- pickup address;
- delivery address;
- route assignment;
- execution workflow;
- item verification state.

### Invoice

Billing entity на основе approved Estimate или Order.

Должен хранить:

- invoice ID;
- linked estimate/order;
- amount;
- payment status;
- due date;
- provider payment ID;
- invoice document artifact;
- adjustments/refunds.

### eBOL

Electronic Bill of Lading / Proof of Delivery workflow.

Должен хранить:

- linked order;
- item-level pickup verification;
- item-level delivery verification;
- photos;
- signatures;
- exceptions;
- POD status;
- final eBOL/POD artifact.

## 6. Required Production Architecture

Production implementation должна строиться отдельно от текущих static HTML screens.

Required architecture components:

### Pricing Engine

Отдельный tested service/module для всех pricing calculations.

Pricing engine должен:

- принимать normalized QuoteDraft input;
- применять versioned PricingVariables;
- возвращать PricingBreakdown;
- возвращать validation warnings/errors;
- быть покрыт unit tests;
- быть воспроизводимым;
- сохранять formula version.

### Quote Snapshot

Перед генерацией Estimate система должна собрать quote snapshot.

Snapshot должен быть immutable после создания customer-facing Estimate.

### Estimate Generation From Snapshot

Estimate document должен генерироваться только из EstimateSnapshot.

Нельзя генерировать customer-facing estimate напрямую из текущего editable UI state.

### Backend Persistence

Production-система должна иметь backend/database для:

- QuoteDraft;
- QuoteItem;
- PricingVariables;
- PricingBreakdown;
- Estimate;
- EstimateSnapshot;
- Customer;
- Invoice;
- Order;
- eBOL;
- audit events;
- PDF/document artifacts.

### Validation

Validation должна быть на frontend и backend.

Минимально требуется:

- ZIP validation;
- email validation;
- phone validation;
- dimensions validation;
- weight validation;
- quantity validation;
- declared value validation;
- date validation;
- access conditions validation;
- required fields validation;
- pricing sanity validation.

### Audit Trail

Система должна фиксировать:

- кто создал quote;
- кто изменил quote;
- кто изменил variables;
- какая formula version применена;
- какой variables snapshot применен;
- когда был generated estimate;
- кому был sent estimate;
- какие versions были созданы;
- какие changes привели к new estimate version.

### PDF / Export Flow

Production должна реализовать одно из двух:

- backend PDF artifact generation;
- controlled print/PDF flow with saved artifact metadata.

В любом случае нужно хранить:

- generated_at;
- generated_by;
- snapshot_id;
- estimate_version;
- document artifact ID;
- document hash или equivalent integrity marker.

## 7. Pricing Engine Requirements

Pricing engine должен быть отдельным от UI.

Минимальные требования:

- no hardcoded UI calculations;
- all formulas versioned;
- all variables versioned;
- reproducible output;
- deterministic calculation for same input and same variables;
- test fixtures for common scenarios;
- edge-case tests;
- explanation output for breakdown screen;
- validation output for broker workflow.

Pricing engine должен покрывать:

- physical volume;
- effective volume;
- total weight;
- vehicle fit;
- crew logic;
- helper logic;
- pickup stage cost;
- interstate stage cost;
- delivery stage cost;
- fuel cost;
- route cost allocation;
- packaging;
- crating;
- storage;
- insurance/protection;
- COI fees;
- access fees;
- margin;
- discounts;
- rounding;
- minimum quote;
- manual review flags.

## 8. Snapshot Architecture

Recommended flow:

```text
Broker Input
↓
QuoteDraft
↓
Validation
↓
Pricing Engine
↓
PricingBreakdown
↓
Generate EstimateSnapshot
↓
Estimate Document
↓
PDF / Export Artifact
↓
Customer Approval
↓
Invoice / Order / eBOL
