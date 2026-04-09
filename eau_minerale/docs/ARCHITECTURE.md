# Architecture — Module Eau Minérale

## Vue d'ensemble

Module **usine + distribution** : couvre l'intégralité du cycle
production → stock → ventes → trésorerie → salaires. Architecture Clean
(Domain → Data → Application → Presentation), entièrement
**offline-first** sur Drift, avec sync Firestore en tâche de fond.

C'est le module le plus riche en services métier (calculs de marge,
réconciliation paiements, intégrité stock, calcul des salaires de
production).

## 🏗️ Structure des Couches

### 1. Domain

#### Entities (`domain/entities/`)

**Production** : `ProductionSession`, `ProductionDay`, `ProductionEvent`,
`MachineMaterialUsage`, `MaterialConsumption`, `ProductionPayment`,
`ProductionPaymentPerson`, `ProductionPeriodConfig`, `ProductionSessionStatus`

**Commercial** : `Sale`, `Product`, `CustomerAccount`, `CustomerCredit`,
`CreditPayment`, `ProductSalesSummary`

**Logistique** : `StockItem`, `StockMovement`, `Purchase`, `Supplier`,
`SupplierSettlement`, `Machine`

**RH** : `Employee` (mensuel), `DailyWorker` (journalier),
`SalaryPayment`, `WeeklySalaryInfo`, `WorkerMonthlyStat`

**Finance** : `Expense`, `ExpenseRecord`, `ExpenseReportData`,
`TreasuryMovement`, `Closing`, `ReportData`, `ReportPeriod`,
`ProductionReportData`, `SalaryReportData`, `ActivitySummary`,
`EauMineraleSettings`

#### Domain Services (`domain/services/`)

Le module est riche en services de calcul :

- `DashboardCalculationService` — agrégats du dashboard
- `SaleCalculationService` · `SaleService`
- `CreditCalculationService` · `CreditService`
- `ProductionService` · `ProductionSessionBuilder` · `ProductionSessionStatusCalculator`
- `ProductionMarginCalculator` · `ProfitabilityCalculationService`
- `ProductionPaymentCalculationService` · `ProductionPaymentValidationService`
- `ProductionPeriodService`
- `MachineStockManagementService` · `MachineMaterialCostService`
- `StockHistoryService` · `HistoricalStockService` · `StockIntegrityService` · `StockReconciliationService`
- `PaymentReconciliationService` · `PaymentSplitterService`
- `SalaryCalculationService`
- `ReportCalculationService`
- `TreasuryMovementMapper`
- `validation/` — validateurs métier

### 2. Data — Offline Repositories

`data/repositories/` (17 repositories) :

- `production_session_offline_repository.dart` · `machine_offline_repository.dart`
- `sale_offline_repository.dart` · `product_offline_repository.dart`
- `customer_offline_repository.dart` · `credit_offline_repository.dart`
- `stock_offline_repository.dart`
- `purchase_offline_repository.dart` · `supplier_offline_repository.dart`
- `salary_offline_repository.dart` · `daily_worker_offline_repository.dart`
- `finance_offline_repository.dart` · `treasury_offline_repository.dart`
- `closing_offline_repository.dart`
- `activity_offline_repository.dart`
- `report_offline_repository.dart`
- `settings_offline_repository.dart`

**Caractéristiques** :
- Drift/SQLite local
- Isolation multi-tenant via `enterpriseId`
- `moduleType = 'eau_minerale'`
- Soft delete (`deletedAt`, `deletedBy`)
- Sync Firestore automatique via `SyncManager`

### 3. Application

#### Controllers (`application/controllers/`)

14 controllers — un par domaine :

`activity_controller` · `clients_controller` · `closing_controller` ·
`finances_controller` · `machine_controller` · `product_controller` ·
`production_session_controller` · `purchase_controller` ·
`report_controller` · `salary_controller` · `sales_controller` ·
`stock_controller` · `supplier_controller` · `treasury_controller`

L'UI consomme ces controllers via providers Riverpod. **Jamais d'accès
direct aux repositories.**

### 4. Presentation

Écrans dans `presentation/screens/sections/` : `dashboard_screen`,
`reports_screen`, `sales_screen`, `production_sessions_screen`,
`production_session_detail_screen`, `demarrer_semaine_screen`,
`enregistrer_aujourd_hui_screen`, `terminer_production_screen`,
`stock_screen`, `purchases_screen`, `suppliers_screen`,
`treasury_screen`, `payment_reconciliation_screen`, `finances_screen`,
`salaries_screen`, `clients_screen`, `catalog_screen`, `settings_screen`,
`profile_screen`.

## 🔄 Flux Offline-First

**Écriture** : UI → Provider → Controller → Service métier (calculs) →
OfflineRepository → Drift → SyncManager → Firestore.

**Lecture** : UI → Provider → Controller → OfflineRepository → Drift
(local, jamais d'appel réseau bloquant).

**Sync** : déclenchée à la connectivité, périodiquement et après chaque
écriture critique. Idempotente, résolution de conflits par `updatedAt`.

## 🔐 Multi-Tenancy

- `enterpriseId` filtre toutes les requêtes Drift
- Chemin Firestore : `enterprises/{enterpriseId}/modules/eau_minerale/{collection}`

## 🏭 Particularités Production

### Sessions multi-jours

Une `ProductionSession` couvre **plusieurs jours**. Chaque jour est un
`ProductionDay` qui enregistre personnel présent (avec taux), quantité
produite (packs), conso matière par machine, emballages utilisés et coût
calculé.

`ProductionSessionBuilder` orchestre la création progressive de la session
au fil des jours. `ProductionSessionStatusCalculator` détermine le statut
courant (`draft → inProgress → completed | cancelled`).

### Calcul de coût et marge

`ProductionMarginCalculator` + `MachineMaterialCostService` calculent le
coût de revient (matières + emballages + main d'œuvre) par session.
`ProfitabilityCalculationService` croise avec les ventes pour la marge
réelle.

### Intégrité stock

`StockIntegrityService` + `StockReconciliationService` vérifient en
continu la cohérence entre stock matières premières (consommé en
production), stock produits finis (sortie de session) et mouvements de
vente.

`HistoricalStockService` permet de reconstruire l'état du stock à une date
donnée pour les rapports rétroactifs.

## 📊 Collections Synchronisées

`production_sessions` · `machines` · `sales` · `products` · `customers` ·
`customer_credits` · `credit_payments` · `stock_items` · `stock_movements` ·
`purchases` · `suppliers` · `supplier_settlements` · `employees` ·
`daily_workers` · `salary_payments` · `production_payments` · `expenses` ·
`treasury_movements` · `closings` · `eau_minerale_settings`
