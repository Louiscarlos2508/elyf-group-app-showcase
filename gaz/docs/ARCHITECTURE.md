# Architecture — Module Gaz

## Vue d'ensemble

Module **distribution de bouteilles de gaz** (gros + détail) avec deux
rôles : **Manager** (entreprise mère, possède le camion et le réseau) et
**POS** (point de vente affilié, succursale ou comptoir simple).

Architecture Clean (Domain → Data → Application → Presentation),
entièrement **offline-first** sur Drift, avec sync Firestore en arrière-plan.

Le cœur du module est la **Tournée (Tour)** — un workflow multi-étapes
qui suit le camion de la collecte des vides jusqu'à la clôture des
comptes.

> Voir aussi [DATA_CONSISTENCY_ARCHITECTURE.md](./DATA_CONSISTENCY_ARCHITECTURE.md)
> pour les invariants de cohérence des données (stock bi-modal, échanges,
> chaînage de tournées).

## 🏗️ Structure des Couches

### 1. Domain

#### Entities (`domain/entities/`)

**Tournées** :
- `Tour` — tournée complète (statut, stock initial/résiduel, échange fournisseur, dépenses)
- `TourSiteInteraction` — passage chez un grossiste / POS
- `TransportExpense` — dépenses logistiques (carburant, péage…)
- `SiteLogisticsRecord`

**Stock & Bouteilles** :
- `Cylinder` — type de bouteille (poids, prix achat/vente/consigne, parc enregistré)
- `CylinderStock` — stock bi-modal (pleines / vides) par poids
- `StockMovement` · `StockAlert`
- `ExchangeRecord` — échange vide → pleine
- `CylinderLeak` — déclaration de fuite
- `InventoryAudit`

**Réseau & Ventes** :
- `Wholesaler` — grossiste B2B
- `GazPOSRemittance` — versement POS → maison mère
- `GasSale` — vente détail

**Contrats & RH** :
- `GazContract` — contrat commercial
- `GazCreditPayment` — paiement de créance
- `GazEmployee` · `GazSalaryPayment`

**Finance** :
- `Expense`
- `TreasuryMovement`
- `GazTreasurySynthesis`
- `FinancialReport`
- `GazSettings`

#### Domain Services (`domain/services/`)

- `TourService` — orchestration des tournées
- `TransactionService` — transactions atomiques multi-entités
- `WholesalerService`
- `StockService` · `GazStockCalculationService` · `GazStockReportService`
- `GasValidationService` · `GasAlertService`
- `DataConsistencyService` — invariants stock / tour / échanges
- `GazReconciliationService` — réconciliation entre tours et stock
- `GazDashboardCalculationService`
- `GazFinancialCalculationService` · `FinancialCalculationService`
- `GazSalesCalculationService` · `GazReportCalculationService`
- `GazPrintingService` · `GazSalePdfService` — impression et PDF
- `LeakReportService`
- `RealtimeSyncService` — sync temps réel sur événements critiques
- `filtering/` — filtres réutilisables

### 2. Data — Offline Repositories

`data/repositories/` (18 repositories) :

- `tour_offline_repository`
- `cylinder_stock_offline_repository` · `cylinder_leak_offline_repository`
- `exchange_offline_repository`
- `gas_offline_repository`
- `wholesaler_offline_repository`
- `pos_remittance_offline_repository` · `pos_stock_movement_offline_repository`
- `gaz_contract_offline_repository`
- `gaz_credit_payment_offline_repository`
- `gaz_employee_offline_repository` · `gaz_salary_payment_offline_repository`
- `gaz_settings_offline_repository`
- `inventory_audit_offline_repository`
- `site_logistics_record_offline_repository`
- `expense_offline_repository`
- `treasury_offline_repository`
- `financial_report_offline_repository`

**Caractéristiques** :
- Drift/SQLite local
- Isolation multi-tenant via `enterpriseId`
- `moduleType = 'gaz'`
- Sync Firestore automatique
- Soft delete

### 3. Application

#### Controllers (`application/controllers/`)

- `gas_controller` — opérations gaz génériques
- `cylinder_controller` · `cylinder_stock_controller` · `cylinder_leak_controller`
- `wholesaler_controller`
- `gaz_employee_controller` · `gaz_salary_payment_controller`
- `expense_controller`
- `gaz_settings_controller`
- `financial_report_controller`
- `leak_report_controller`

L'orchestration des tournées passe par `TourService` (couche domain)
exposé via providers Riverpod.

### 4. Presentation

Écrans dans `presentation/screens/sections/`, organisés par rôle :

**Manager** :
- `dashboard_screen.dart`
- `tours_screen.dart` + workflow 6 étapes (Collecte, Recharge, Livraison, Encaissement, Bilan, Clôture)
- `mes_pos_screen.dart` — réseau POS
- `tresorerie_screen.dart`
- `contrats_screen.dart`
- Paramètres (parc bouteilles, profil)

**POS** :
- `retail/` — interface POS détail (accueil, ventes, stock)
- `wholesale/` — interface gros
- `credits_screen.dart`
- `sales_screen.dart`
- `inventory_screen.dart`

**Modules avancés** :
- `cylinder_leak/` — déclarations de fuite
- `finance/` — rapports financiers
- `expenses/` — dépenses opérationnelles

## 🚚 Workflow Tournée (cœur du module)

Une `Tour` traverse 6 statuts :

```
open → collecting → recharging → delivering → encaissement → closing → closed
```

1. **Configuration départ** — date, stock initial dans le camion
2. **Collecte** — passage chez chaque grossiste, récupération des vides
   par poids (3kg, 6kg, 10kg, 12kg) → `TourSiteInteraction`
3. **Recharge fournisseur** — `emptyBottlesReturned` ↔ `fullBottlesReceived`
   + `gasPurchaseCost`
4. **Livraison** — distribution des pleines aux POS et grossistes
5. **Encaissement** — saisie des paiements (cash, mobile money, crédit)
   → impression PDF + partage natif
6. **Bilan & Clôture** — calcul du Flux Financier (Ventes Sites − Coût
   Recharge − Dépenses Trajet = Résultat Net), passage en `closed`

À chaque étape, un popup **État du Camion** est disponible (pleines,
vides collectés, cash en main).

## 🔐 Multi-Tenancy & Multi-Rôle

- `enterpriseId` filtre toutes les requêtes
- Le rôle (Manager / POS) conditionne l'arborescence et les permissions
- Chemin Firestore : `enterprises/{enterpriseId}/modules/gaz/{collection}`
- Les **POS de type entreprise** sont des sous-entreprises Firestore avec
  leur propre `enterpriseId`, rattachées via `parentEnterpriseId`

## 🛢️ Stock bi-modal

Particularité métier : chaque bouteille existe dans **2 états**
(`full` / `emptyAtStore`) et chaque type (poids) a son propre compteur.
`CylinderStock` modélise un (`enterpriseId`, `weight`, `status`, `siteId`)
avec sa quantité. `Cylinder.registeredTotal` représente le parc
**enregistré** par l'entreprise (fixé à la création, modifié uniquement
en cas d'achat ou de réforme).

## 📊 Collections Synchronisées

`tours` · `cylinder_stocks` · `cylinder_leaks` · `exchanges` ·
`wholesalers` · `pos_remittances` · `pos_stock_movements` ·
`gaz_contracts` · `gaz_credit_payments` · `gaz_employees` ·
`gaz_salary_payments` · `gaz_settings` · `inventory_audits` ·
`site_logistics_records` · `expenses` · `treasury_movements` ·
`financial_reports`
