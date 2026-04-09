# Architecture — Module Boutique

## Vue d'ensemble

Le module Boutique suit une **Clean Architecture** strict (Domain → Data →
Application → Presentation), entièrement **offline-first** sur Drift/SQLite
avec synchronisation Firestore en arrière-plan.

## 🏗️ Structure des Couches

### 1. Domain

#### Entities (`domain/entities/`)

- `Product` — produit avec stock, prix d'achat/vente, marge, soft delete, code-barres
- `Category` — catégorie produit
- `Sale` — vente complète avec items, paiement, **chaînage de tickets** (`ticketHash`/`previousHash`)
- `CartItem` — item du panier avant validation
- `Purchase` — achat fournisseur
- `Supplier` / `SupplierSettlement` — fournisseurs et règlements
- `Expense` — dépense opérationnelle
- `StockMovement` — entrées/sorties/ajustements
- `Closing` — clôture journée (Z-Report)
- `BoutiqueSettings` — configuration par boutique
- `ReportData` — agrégats pour rapports

#### Domain Services (`domain/services/`)

- `BoutiqueCalculationService` — calculs unifiés Dashboard & Reports
- `ProductCalculationService` — calculs liés produits (marge, valorisation stock)
- `CartService` — calculs panier (sous-total, remise, total)
- `NumberingService` — génération des numéros de facture (`FAC-YYYYMMDD-NNN`)
- `BoutiqueExportService` — export PDF / partage
- `BoutiqueSettingsService` — accès aux paramètres boutique
- `ProductFilterService` — recherche / filtrage produits
- `SupplierSettlementService` — règlements fournisseurs
- `calculation/` — sous-services de calcul spécialisés
- `validation/` — validateurs (produit, vente, etc.)
- `security/` — chaînage de tickets, hash cryptographique

### 2. Data — Offline Repositories

Tous les repositories sont **offline-first** dans
`data/repositories/` :

- `product_offline_repository.dart`
- `category_offline_repository.dart`
- `sale_offline_repository.dart`
- `stock_offline_repository.dart` · `stock_movement_offline_repository.dart`
- `purchase_offline_repository.dart`
- `supplier_offline_repository.dart` · `supplier_settlement_offline_repository.dart`
- `expense_offline_repository.dart`
- `treasury_offline_repository.dart`
- `closing_offline_repository.dart`
- `boutique_settings_offline_repository.dart`
- `report_offline_repository.dart` (calculs basés sur les données locales)

**Caractéristiques** :
- Stockage local Drift/SQLite
- Isolation multi-tenant via `enterpriseId`
- `moduleType = 'boutique'` pour le routing sync
- Soft delete (`deletedAt`, `deletedBy`)
- Synchronisation automatique via `SyncManager`

### 3. Application

#### Controllers (`application/controllers/`)

- `StoreController` — orchestration produits / ventes / stock
- `CartController` — gestion du panier en cours

#### Providers (`application/providers/` + `providers.dart`)

Tous les providers Riverpod résolvent les controllers et services. **L'UI
ne doit jamais accéder directement aux repositories.**

### 4. Presentation

Écrans Flutter dans `presentation/screens/sections/` :
- `pos_screen.dart` — caisse temps réel + scan code-barres
- `catalog_screen.dart` — catalogue produit + édition
- `stock_screen.dart` · `stock_movement_screen.dart` — inventaire
- `treasury_screen.dart` — soldes Caisse / Mobile Money
- `credits_screen.dart` — créances clients
- `expenses_screen.dart` — dépenses opérationnelles
- `purchases_screen.dart` · `suppliers_screen.dart`
- `sales_screen.dart` — historique des ventes
- `reports_screen.dart` — rapports & clôtures
- `category_management_screen.dart`

## 🔄 Flux de Données Offline-First

**Écriture** : UI → Provider → Controller → OfflineRepository → Drift →
SyncManager (file d'attente) → FirebaseSyncHandler → Firestore.

**Lecture** : UI → Provider → Controller → OfflineRepository → Drift
(toujours local, jamais d'appel réseau bloquant).

**Sync** : déclenchée par changement de connectivité, périodique, et après
chaque écriture critique. Idempotente, gère les conflits par
`updatedAt` / version.

## 🔐 Multi-Tenancy

- `enterpriseId` filtre toutes les requêtes Drift
- Chemin Firestore : `enterprises/{enterpriseId}/modules/boutique/{collection}`
- L'`activeEnterpriseProvider` injecte l'enterprise courante dans tous les providers

## 🧾 Chaînage cryptographique des tickets

Chaque `Sale` porte deux champs :
- `previousHash` — hash du ticket précédent
- `ticketHash` — hash de cette vente (calculé à partir du contenu + previousHash)

Cela forme une **chaîne séquentielle vérifiable** : impossible d'insérer,
supprimer ou modifier une vente sans casser la chaîne. Utilisé pour audit
et conformité fiscale.

## 📊 Collections Synchronisées

`products` · `categories` · `sales` · `purchases` · `suppliers` ·
`supplier_settlements` · `expenses` · `stock_movements` · `closings` ·
`treasury_movements` · `boutique_settings`
