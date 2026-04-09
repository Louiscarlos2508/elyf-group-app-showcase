# Architecture — Module Immobilier

## Vue d'ensemble

Module **gestion locative** : portefeuille de propriétés en location,
locataires, contrats, encaissement de loyers, dépenses par bien,
maintenance et rapports financiers.

Architecture Clean (Domain → Data → Application → Presentation),
entièrement **offline-first** sur Drift, avec sync Firestore en arrière-plan.

## 🏗️ Structure des Couches

### 1. Domain

#### Entities (`domain/entities/`)

- `Property` — propriété (statut, loyer mensuel, type, surface)
- `Tenant` — locataire (informations, dossier, statut "à jour" / "en retard")
- `Contract` — bail (locataire ↔ propriété, dates, loyer, caution)
- `Payment` — encaissement de loyer (mois couverts, mode, date)
- `Expense` / `PropertyExpense` — dépense rattachée à une propriété
- `MaintenanceTicket` — demande/intervention de maintenance
- `ImmobilierSettings` — configuration du module
- `ReportPeriod` — période de rapport

#### Domain Services (`domain/services/`)

- `DashboardCalculationService` — agrégats du dashboard
- `PropertyCalculationService` — calculs par propriété (revenus, taux d'occupation, rentabilité)
- `ImmobilierValidationService` — validations métier (chevauchement de baux, dates cohérentes…)
- `ImmobilierSettingsService`
- `calculation/` — sous-services de calcul
- `filtering/` — filtres réutilisables (à jour / en retard, par période…)
- `validation/` — validateurs

#### Application Services (`application/services/`)

- `BillingAutomationService` — génération automatique des échéances de loyer
- `ReceiptService` — génération des quittances de loyer (PDF)

### 2. Data — Offline Repositories

`data/repositories/` (8 repositories) :

- `property_offline_repository.dart`
- `tenant_offline_repository.dart`
- `contract_offline_repository.dart`
- `payment_offline_repository.dart`
- `property_expense_offline_repository.dart`
- `maintenance_offline_repository.dart`
- `treasury_offline_repository.dart`
- `immobilier_settings_offline_repository.dart`

**Caractéristiques** :
- Drift/SQLite local
- Isolation multi-tenant via `enterpriseId`
- `moduleType = 'immobilier'`
- Sync Firestore automatique via `SyncManager`
- Soft delete (`deletedAt`, `deletedBy`)

### 3. Application

#### Controllers (`application/controllers/`)

7 controllers :

- `property_controller` — gestion des propriétés
- `tenant_controller` — gestion des locataires
- `contract_controller` — gestion des baux
- `payment_controller` — encaissement de loyers
- `expense_controller` — dépenses par propriété
- `maintenance_controller` — tickets de maintenance
- `immobilier_treasury_controller` — trésorerie consolidée

L'UI consomme ces controllers via providers Riverpod. **Jamais d'accès
direct aux repositories.**

### 4. Presentation

Écrans dans `presentation/screens/sections/` :

- `dashboard_screen.dart` — synthèse revenus, paiements reçus, tendance mensuelle
- `payments_screen.dart` — liste des paiements + assistant d'encaissement 2 étapes (Locataire → Maison → Mois à payer)
- `properties_screen.dart` — catalogue des biens
- `tenants_screen.dart` — annuaire locataires (filtres "À jour / En retard")
- `expenses_screen.dart` — dépenses rattachées à une propriété
- `treasury_screen.dart` — solde caisse + banque
- `reports_screen.dart` — rapports analytiques (revenus, dépenses, bénéfice net, taux d'occupation)
- `settings_screen.dart` · `profile_screen.dart`

## 🔄 Flux Offline-First

**Écriture** : UI → Provider → Controller → Service métier (validation /
calcul) → OfflineRepository → Drift → SyncManager → Firestore.

**Lecture** : UI → Provider → Controller → OfflineRepository → Drift
(local). Aucun appel réseau bloquant.

**Sync** : déclenchée à la connectivité, périodiquement et après chaque
écriture critique. Idempotente, résolution de conflits par `updatedAt`.

## 💰 Workflow Encaissement (2 étapes)

1. **Étape Locataire** — sélection du locataire à encaisser
2. **Étape Maison** — sélection de la maison rattachée au locataire
3. **Sélection des mois** — l'utilisateur coche les mois impayés ;
   `PaymentController` calcule le total dû, applique le paiement et
   marque les échéances correspondantes comme soldées
4. Génération automatique de la quittance via `ReceiptService`

## 🤖 Automatisation

`BillingAutomationService` génère automatiquement :
- Les **échéances mensuelles** sur tous les contrats actifs (le 1er du mois)
- Les **alertes de retard** quand une échéance dépasse une date butoir
- Les **statuts locataire** ("à jour" / "en retard") agrégés depuis les paiements

## 🔐 Multi-Tenancy

- `enterpriseId` filtre toutes les requêtes
- Chemin Firestore : `enterprises/{enterpriseId}/modules/immobilier/{collection}`

## 📊 Collections Synchronisées

`properties` · `tenants` · `contracts` · `payments` · `property_expenses` ·
`maintenance_tickets` · `treasury_movements` · `immobilier_settings`
