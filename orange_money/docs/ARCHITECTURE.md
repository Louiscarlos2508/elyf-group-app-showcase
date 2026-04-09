# Architecture — Module Orange Money

## Vue d'ensemble

Module **multi-rôle** (Dealer ↔ Agent) couvrant les opérations cash-in /
cash-out d'un réseau Orange Money. Architecture Clean (Domain → Data →
Application → Presentation), entièrement **offline-first** sur Drift, avec
sync Firestore en tâche de fond.

## 🏗️ Structure des Couches

### 1. Domain

#### Entities (`domain/entities/`)

- `Transaction` — opération cash-in / cash-out / dépôt / retrait
- `Agent` — agent rattaché à un Dealer
- `Customer` — client final (optionnel sur transaction)
- `Contract` — contrat commercial Dealer-Agent (taux de commission, conditions)
- `Commission` — commission calculée mensuellement
- `LiquidityCheckpoint` — pointage de liquidité (déclaration matinale, bilan journée)
- `OrangeMoneySettings` — configuration du module
- `orange_money_enterprise_extensions.dart` — extensions sur `Enterprise`

### 2. Data — Offline Repositories

`data/repositories/` :

- `transaction_offline_repository.dart`
- `agent_offline_repository.dart`
- `contract_offline_repository.dart`
- `commission_offline_repository.dart`
- `liquidity_offline_repository.dart`
- `treasury_offline_repository.dart`
- `settings_offline_repository.dart`

**Caractéristiques** :
- Stockage Drift/SQLite local
- Isolation multi-tenant via `enterpriseId`
- `moduleType = 'orange_money'`
- Sync Firestore automatique (SyncManager + FirebaseSyncHandler)
- Soft delete

### 3. Application

#### Controllers (`application/controllers/`)

- `OrangeMoneyController` — orchestration des transactions
- `AgentsController` — gestion du réseau d'agents
- `ContractController` — contrats Dealer-Agent
- `CommissionsController` — calcul / déclaration / paiement des commissions
- `LiquidityController` — déclaration matinale + bilan journée + checkpoints
- `SettingsController` — paramètres et seuils du module

L'UI ne consomme **jamais** les repositories directement — toujours via
provider → controller.

### 4. Presentation

Écrans dans `presentation/screens/`, organisés par rôle :

**Dealer** :
- `dashboard_screen.dart` — vue agrégée du réseau
- `agents_screen.dart` — annuaire et gestion des agents
- `dealer_transaction_screen.dart` — saisie de transactions Dealer
- `treasury_tab.dart` — trésorerie consolidée
- `commissions_screen.dart` — déclaration et paiement
- `liquidity_screen.dart` — supervision liquidité réseau

**Agent** :
- `transactions_v2_screen.dart` — flow client en 4 étapes (Type → Montant → Client → Confirmation)
- `morning_declaration_screen.dart` — déclaration matinale
- `daily_summary_screen.dart` — bilan de fin de journée

## 🔄 Flux Offline-First

**Écriture** : UI → Provider → Controller → OfflineRepository → Drift →
SyncManager (queue) → FirebaseSyncHandler → Firestore.

**Lecture** : UI → Provider → Controller → OfflineRepository → Drift
(local). Aucun appel réseau bloquant en lecture.

**Sync** : déclenchée à la connectivité, périodiquement, et après chaque
écriture critique. Idempotente, résolution de conflit par `updatedAt`.

## 🔐 Multi-Tenancy & Multi-Rôle

- `enterpriseId` filtre toutes les requêtes
- Chemin Firestore : `enterprises/{enterpriseId}/modules/orange_money/{collection}`
- Le rôle utilisateur (Dealer / Agent) est résolu au login et conditionne
  l'arborescence des écrans + les permissions
- Les commissions calculées remontent automatiquement du niveau Agent vers
  le niveau Dealer via les contrats

## 💰 Liquidité & Commissions

### Liquidité

Chaque journée commence par une **déclaration matinale**
(`LiquidityCheckpoint` matin) et finit par un **bilan journée**
(`LiquidityCheckpoint` soir). Les soldes Cash + Mobile Money sont
comparés aux mouvements pour détecter les écarts.

### Commissions

Le `CommissionsController` calcule en fin de mois (ou à la demande) le
montant dû à chaque Agent selon son `Contract` (taux, paliers, bonus). Le
Dealer valide et déclare le paiement, qui crée un mouvement de trésorerie.

## 📊 Collections Synchronisées

`transactions` · `agents` · `contracts` · `commissions` ·
`liquidity_checkpoints` · `treasury_movements` · `orange_money_settings`
