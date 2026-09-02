# Module Orange Money — ELYF Group App

## 🎯 Vue d'Ensemble

Le module **Orange Money** pilote l'activité d'un **Dealer** Mobile Money et de
son réseau d'agents (succursales internes ou partenaires externes). Il couvre
le cycle complet : pointages de liquidité, transactions inter-agents, contrats,
réconciliation journalière, trésorerie multi-comptes et gestion des commissions
opérateur.

L'architecture est **role-aware** : les mêmes écrans s'adaptent automatiquement
selon que l'utilisateur connecté est un **Dealer** (vue réseau, agrégée) ou un
**Agent** (vue personnelle, opérationnelle). Les screenshots ci-dessous
illustrent le parcours **Dealer**.

### Caractéristiques Principales

- 🏢 **Vue Dealer / Vue Agent** — un seul module, deux rôles
- 🔁 **Transactions Dealer ↔ Agent** — dépôt / retrait en assistant 3 étapes
- 👥 **Réseau d'Agents** — annuaire, classement, badge "app installée"
- 📄 **Contrats** — affiliation des agents avec taux de commission
- ⏱️ **Pointages de Liquidité** — déclaration matin + bilan soir avec calcul théorique
- 💵 **Trésorerie** — soldes Cash / Orange Money + recharges agents
- 💰 **Commissions** — saisie mensuelle manuelle avec preuve photo
- 📊 **Historique & Réconciliation** — écarts détectés et justifiés

---

## 🧩 Architecture Fonctionnelle

Le module est organisé en **5 sections** dans la navigation latérale, chacune
adaptée au rôle de l'utilisateur :

| Section | Écrans Dealer | Écrans Agent |
| --- | --- | --- |
| **Vue d'ensemble** | Tableau de Bord (vue réseau) | Tableau de Bord (vue perso) |
| **Opérations** | Transaction **Agent** (3 étapes) · Réseau d'Agents | Transaction **Client** (4 étapes) · Historique Transactions |
| **Gestion** | Contrats | — |
| **Réconciliation** | Déclaration Matin · Bilan Journée · Historique Pointages | Identique au Dealer (mêmes écrans, contexte agent) |
| **Finances** | Trésorerie · Commissions (vue réseau) | Trésorerie · Commissions (déclaration mensuelle) |
| **Configuration** | Profil | Profil |

Code source : [lib/features/orange_money/](../../lib/features/orange_money/)

---

## ✨ Fonctionnalités Implémentées

### 1. Tableau de Bord (rôle-conscient)

Implémenté dans
[dashboard_screen.dart](../../lib/features/orange_money/presentation/screens/sections/dashboard_screen.dart).

**Vue Dealer** :
- Bandeau "Pointage Matin Requis" tant que la déclaration n'est pas validée
- Soldes consolidés Cash / Orange Money calculés à partir du checkpoint du
  matin et des transactions de la journée
- Classement des agents par volume
- Badge `DEALER` dans le header

**Vue Agent** :
- Soldes personnels et transactions récentes
- Actions rapides vers les opérations courantes

### 2. Transaction Agent — Assistant 3 étapes

Implémenté dans
[dealer_transaction_screen.dart](../../lib/features/orange_money/presentation/screens/sections/dealer_transaction_screen.dart).

Flow strict en 3 étapes avec stepper visuel :

1. **Agent** — recherche dans le réseau, agents récents épinglés en haut
2. **Montant + Type** — sélection `Dépôt` ou `Retrait`, saisie du montant en FCFA
3. **Confirmer** — récapitulatif puis validation

Chaque transaction met à jour les soldes théoriques utilisés par le bilan de
journée.

### 3. Réseau d'Agents

Implémenté dans
[agents_screen.dart](../../lib/features/orange_money/presentation/screens/sections/agents_screen.dart).

- Annuaire des agents affiliés (succursales internes ou partenaires externes)
- Recherche, filtres `Actifs / Inactifs / Tous`
- KPIs réseau : nombre d'agents actifs, volume mensuel
- Création / fiche agent avec :
  - Identification (nom, téléphone, n° SIM, code agent)
  - Statut (`Actif`, `Inactif`, `Suspendu`)
  - Documents (pièces d'identité, contrat scanné)
  - Notes libres
- Modèle : [agent.dart](../../lib/features/orange_money/domain/entities/agent.dart)

### 4. Contrats

Implémenté dans
[contracts_screen.dart](../../lib/features/orange_money/presentation/screens/sections/contracts_screen.dart).

- Liste filtrée par statut (`Actif`, `Expiré`, `Résilié`)
- Création d'un contrat rattaché à un agent avec taux de commission
- Détail en bottom sheet avec actions imprimer / partager
- Modèle : [contract.dart](../../lib/features/orange_money/domain/entities/contract.dart)

### 5. Réconciliation des Liquidités

#### 5.1 Session Journalière (Déclaration Matin + Comptage + Bilan)

[daily_session_screen.dart](../../lib/features/orange_money/presentation/screens/sections/daily_session_screen.dart)

Flux guidé en 3 étapes :

1. **Résumé du jour** — KPIs (dépôts, retraits, nb transactions, volume total)
   + **déclaration matin** (soldes Cash + OM en début de journée, référence aux
   soldes de la veille)
2. **Comptage** — saisie des soldes réels (Cash + OM)
3. **Bilan / Validation** — comparaison **théorique vs réel** (théorique
   `= solde matin + cash-in − cash-out`), affichage de l'écart et **validation
   avec justification obligatoire** si l'écart dépasse le seuil

- **Auto-clôture** : si la veille n'a pas été clôturée, le système la ferme
  automatiquement avec les valeurs théoriques et demande validation

#### 5.2 Historique Pointages
[liquidity_screen.dart](../../lib/features/orange_money/presentation/screens/sections/liquidity_screen.dart)

- Statut du jour en lecture seule avec liens vers les écrans dédiés
- Historique des 7 derniers jours et liste complète filtrable
- Filtres période + date personnalisée

Modèle :
[liquidity_checkpoint.dart](../../lib/features/orange_money/domain/entities/liquidity_checkpoint.dart)
— supporte les pointages `morning`, `evening`, `full`, le calcul des soldes
théoriques, les écarts, la justification et l'auto-clôture.

### 6. Trésorerie

Implémenté dans
[treasury_tab.dart](../../lib/features/orange_money/presentation/screens/sections/treasury_tab.dart).

- Soldes consolidés **Cash** et **Orange Money** en haut d'écran
- **Vue Dealer** : grille 4 opérations (Recharge, Retrait, Apport, Payer commission)
  + section **Recharges agents**
- **Vue Agent** : 2 opérations principales + historique incluant les recharges reçues
- **Apport** : formulaire avec compte d'application (Espèces, Mobile Money…),
  motif, bénéficiaire / fournisseur, notes
- Historique filtrable `Aujourd'hui / Cette semaine / Ce mois`

### 7. Commissions — Saisie Mensuelle Manuelle

Implémenté dans
[commissions_screen.dart](../../lib/features/orange_money/presentation/screens/sections/commissions_screen.dart).

Chaque mois, l'agent saisit **manuellement** la commission reçue de l'opérateur
en joignant une preuve (photo du SMS ou capture). Aucun calcul automatique
n'est appliqué : la valeur saisie est la valeur officielle. Modèle :
[commission.dart](../../lib/features/orange_money/domain/entities/commission.dart).

**KPIs en haut d'écran** :

- **Ce mois-ci** — montant déclaré pour le mois courant
- **À encaisser** — total déclaré non encore payé
- **Encaissé (12 mois)** — total payé sur les 12 derniers mois
- **Cumul total** — total payé toutes périodes confondues

**Saisie d'une commission** :

- Bouton « + » dans le header → choix du mois + montant + photo de preuve + notes
- Statut initial : `Déclarée`

**Validation par le superviseur** :

- Une commission `Déclarée` (ou `Contestée`) peut être marquée `Payée`
- Suivi : liste filtrable, tri par statut (contestée → déclarée → payée)

### 8. Configuration

Implémenté dans
[settings_screen.dart](../../lib/features/orange_money/presentation/screens/sections/settings_screen.dart).

Profil utilisateur, sécurité (mot de passe) et déconnexion.

---

## 📸 Screenshots — Parcours Dealer

> Captures réalisées sur tablette en mode sombre, locale FR, sur un compte
> Dealer fraîchement initialisé (montants à zéro pour ne pas exposer de
> données réelles).

### 1. Vue d'Ensemble

| Tableau de Bord — Dealer |
| --- |
| ![Dashboard](assets/screenshots/01-dashboard.png) |
| Vue réseau avec bandeau "Pointage Matin Requis", soldes Cash / OM, dépôts, retraits et classement des agents. |

### 2. Opérations

#### Transaction Agent (assistant 3 étapes)

| Étape 1 — Choisir l'agent | Étape 2 — Dépôt | Étape 2 — Retrait |
| --- | --- | --- |
| ![Agent](assets/screenshots/02-transaction-step1-agent.png) | ![Dépôt](assets/screenshots/05-transaction-step2-depot.png) | ![Retrait](assets/screenshots/06-transaction-step2-retrait.png) |
| Recherche dans le réseau avec agents récents épinglés. | Saisie du montant pour un dépôt vers l'agent. | Bascule vers un retrait depuis l'agent. |

#### Réseau d'Agents

| Liste des agents | Nouvel agent | Détails agent |
| --- | --- | --- |
| ![Agents](assets/screenshots/03-reseau-agents.png) | ![Nouveau](assets/screenshots/07-agent-nouveau-formulaire.png) | ![Détail](assets/screenshots/08-agent-detail.png) |
| Annuaire avec filtres et KPIs réseau. | Création : identification, contact, statut, documents. | Fiche complète d'un agent existant. |

### 3. Gestion

| Contrats |
| --- |
| ![Contrats](assets/screenshots/04-contrats-vide.png) |
| Liste des contrats avec filtres `Actifs / Expirés / Tous` et création d'un nouveau contrat. |

### 4. Réconciliation

| Déclaration Matin | Bilan Journée | Bilan — détail agent |
| --- | --- | --- |
| ![Déclaration](assets/screenshots/09-declaration-matin.png) | ![Bilan](assets/screenshots/10-bilan-journee.png) | ![Bilan détail](assets/screenshots/11-bilan-journee-detail-agent.png) |
| Saisie des soldes Cash + OM en début de journée. | KPIs du jour, soldes théoriques, écarts. | Détail par agent avec calcul des soldes théoriques. |

| Historique — vue jour | Historique — vide | Historique — liste |
| --- | --- | --- |
| ![Pointages jour](assets/screenshots/12-historique-pointages-jour.png) | ![Vide](assets/screenshots/13-historique-pointages-vide.png) | ![Liste](assets/screenshots/14-historique-pointages-liste.png) |
| Statut du jour en lecture seule avec actions rapides. | État vide quand aucun pointage n'a encore été fait. | Historique complet avec filtres période / date. |

### 5. Finances

#### Trésorerie

| Vue trésorerie | Historique trésorerie |
| --- | --- |
| ![Trésorerie](assets/screenshots/15-tresorerie-vue.png) | ![Historique](assets/screenshots/16-tresorerie-historique.png) |
| Soldes Cash / OM, grille des 4 opérations Dealer. | Historique filtrable des mouvements. |

| Apport — Espèces | Apport — Mobile Money |
| --- | --- |
| ![Espèces](assets/screenshots/17-apport-especes.png) | ![Mobile Money](assets/screenshots/18-apport-mobile-money.png) |
| Saisie d'un apport en cash sur le compte d'application. | Saisie d'un apport via Mobile Money avec bénéficiaire. |

#### Commissions

| Saisie de commission | Vue réseau Dealer |
| --- | --- |
| ![Saisie](assets/screenshots/19-commissions-declaration.png) | ![Réseau](assets/screenshots/20-commissions-reseau.png) |
| Formulaire mensuel : choix du mois, montant reçu, photo de preuve, notes. | Vue agrégée du réseau : commissions déclarées et payées par agent. |

---

## 📸 Screenshots — Parcours Agent

> Captures du même module mais connecté en tant qu'**Agent** (succursale du
> Dealer). On retrouve la même charpente Material 3 mais une navigation et des
> écrans adaptés à l'opérationnel terrain.
>
> Les écrans **Réconciliation** et **Finances** sont strictement identiques à
> ceux du Dealer (mêmes composants Flutter, contexte d'entreprise différent).
> Seuls les écrans spécifiques au rôle Agent sont illustrés ici.

### 1. Vue d'Ensemble

| Tableau de Bord — haut | Tableau de Bord — bas |
| --- | --- |
| ![Dashboard haut](assets/screenshots/agent-01-dashboard-haut.png) | ![Dashboard bas](assets/screenshots/agent-02-dashboard-bas.png) |
| Soldes Cash / OM personnels et bandeau "Déclaration matin manquante". | KPIs du jour (dépôts, retraits, clients), transactions récentes et actions rapides. |

### 2. Opérations — Transaction Client (assistant 4 étapes)

L'agent enregistre les opérations face à un **client final**. Le flow est en
4 étapes avec auto-complétion par numéro de téléphone (recherche dans la base
clients existante).

Code :
[transactions_v2_screen.dart](../../lib/features/orange_money/presentation/screens/sections/transactions_v2_screen.dart).

| Étape 1 — Type | Étape 2 — Montant (Dépôt) | Étape 2 — Montant saisi |
| --- | --- | --- |
| ![Type](assets/screenshots/agent-03-transaction-step1-type.png) | ![Dépôt vide](assets/screenshots/agent-04-transaction-step2-depot.png) | ![Dépôt saisi](assets/screenshots/agent-05-transaction-step2-depot-saisi.png) |
| Choix `Dépôt` ou `Retrait`. | Saisie du montant pour un dépôt. | Montant rempli, prêt à continuer. |

| Étape 2 — Montant (Retrait) | Étape 3 — Client |
| --- | --- |
| ![Retrait](assets/screenshots/agent-08-transaction-step2-retrait.png) | ![Client](assets/screenshots/agent-06-transaction-step3-client.png) |
| Variante retrait avec saisie du montant. | Saisie / auto-complétion : N° téléphone, nom, prénom, ville, N° CNIB. |

#### Historique Transactions

| Historique |
| --- |
| ![Historique](assets/screenshots/agent-07-historique-transactions.png) |
| Recherche par nom / téléphone / montant, filtres `Tous / Dépôts / Retraits / Personnels`, périodes `Aujourd'hui / Cette semaine / Ce mois`. |

### 3. Réconciliation (identique au Dealer)

Mêmes écrans, mêmes composants — seul le contexte d'entreprise change.

| Déclaration Matin | Bilan de Journée | Historique Pointages |
| --- | --- | --- |
| ![Déclaration](assets/screenshots/agent-09-declaration-matin.png) | ![Bilan](assets/screenshots/agent-10-bilan-journee.png) | ![Historique](assets/screenshots/agent-11-historique-pointages.png) |
| Saisie soldes Cash + OM en début de journée. | Réconciliation théorique vs réel. | Statut du jour et historique complet. |

### 4. Finances — Commissions (Vue Agent)

| Commission Mensuelle |
| --- |
| ![Commission](assets/screenshots/agent-12-commission-mensuelle.png) |
| Saisie mensuelle manuelle : bouton « + » pour enregistrer le montant reçu de l'opérateur en joignant une photo du SMS comme preuve. |

> Les écrans **Trésorerie** côté Agent réutilisent
> [treasury_tab.dart](../../lib/features/orange_money/presentation/screens/sections/treasury_tab.dart)
> avec une grille à 2 opérations (au lieu de 4) et une section "Recharges
> reçues" en remplacement de "Recharges agents".

---

## 🛠️ Modèle de Domaine

Tous les modèles sont définis dans
[lib/features/orange_money/domain/entities/](../../lib/features/orange_money/domain/entities/).

### Transaction
```dart
class Transaction {
  final String id;
  final String enterpriseId;
  final TransactionType type;        // cashIn | cashOut
  final int amount;                   // FCFA
  final String phoneNumber;
  final DateTime date;
  final TransactionStatus status;     // pending | completed | failed
  final String? customerName;
  final int? commission;
  final int? fees;
  final String? reference;
  // + audit (createdBy, deletedAt, …)
}
```

### Agent
```dart
class Agent {
  final String id;
  final String name;
  final String phoneNumber;
  final String simNumber;
  final MobileOperator operator;      // orange | mtn | moov | other
  final int liquidity;                // FCFA
  final double commissionRate;        // %
  final AgentStatus status;           // active | inactive | suspended
  final AgentType type;               // internal (succursale) | external (partenaire)
  final bool hasApp;                  // a installé l'app dealer
  final List<String> attachmentUrls;
}
```

### Contract
```dart
class Contract {
  final String id;
  final String agentId;
  final String contractNumber;
  final ContractStatus status;        // active | expired | terminated
  final DateTime startDate;
  final DateTime? endDate;
  final double commissionRate;
  final String? pdfUrl;
  final String? signatureUrl;
}
```

### LiquidityCheckpoint
```dart
class LiquidityCheckpoint {
  final DateTime date;
  final LiquidityCheckpointType type; // morning | evening | full
  final int? morningCashAmount;
  final int? morningSimAmount;
  final int? eveningCashAmount;
  final int? eveningSimAmount;
  final int? theoreticalCash;         // calcul auto
  final int? theoreticalSim;          // calcul auto
  final int? cashDiscrepancy;         // réel − théorique
  final int? simDiscrepancy;
  final bool requiresJustification;
  final String? justification;
  final bool autoClosed;              // clôturé automatiquement
}
```

### Commission (saisie manuelle mensuelle)

```dart
class Commission {
  final String id;
  final String period;            // "YYYY-MM"
  final String enterpriseId;
  final int declaredAmount;       // montant saisi par l'agent (FCFA)
  final CommissionStatus status;  // declared | paid | disputed
  final String? smsProofUrl;      // photo du SMS / reçu (Firebase Storage)
  final DateTime? declaredAt;
  final String? declaredBy;
  final DateTime? paidAt;
  final String? paymentProofUrl;
  final String? notes;
  // + soft delete (deletedAt, deletedBy) et timestamps (createdAt, updatedAt)
}
```

---

## 🔄 Flux Métier Clés

### Transaction Dealer → Agent

```mermaid
graph TD
    A[Sélectionner Agent] --> B[Choisir Type: Dépôt / Retrait]
    B --> C[Saisir Montant]
    C --> D[Confirmer]
    D --> E[Mise à jour soldes théoriques]
    E --> F[Apparaît dans Bilan Journée]
```

### Cycle Journalier de Réconciliation

```mermaid
graph TD
    A[Début de journée] --> B[Déclaration Matin]
    B --> C[Opérations: transactions, recharges]
    C --> D[Fin de journée]
    D --> E[Saisir soldes réels]
    E --> F[Calcul théorique]
    F --> G{Écart > seuil ?}
    G -->|Non| H[Validation directe]
    G -->|Oui| I[Justification obligatoire]
    I --> H
    H --> J[Bilan clôturé]
```

### Cycle Mensuel de Commission

```mermaid
graph TD
    A[Fin de mois] --> B[Agent reçoit SMS opérateur]
    B --> C[Saisie manuelle: mois + montant + preuve]
    C --> D[Statut: Déclarée]
    D --> E{Paiement reçu ?}
    E -->|Oui| F[Statut: Payée]
    E -->|Conteste| G[Statut: Contestée]
    G --> H[Résolution / nouvelle saisie]
    H --> F
```

---

## 🔗 Liens Utiles

- 📁 [Code source du module](../../lib/features/orange_money/)
- 📐 [Architecture](./docs/ARCHITECTURE.md)
- 📋 [Notes d'implémentation](./docs/IMPLEMENTATION.md)
- 🏠 [Retour au Portfolio](../)

---

## 📝 Notes

> **État de la Documentation** : ✅ Aligné sur l'implémentation (avril 2026)  
> **Rôle illustré dans les screenshots** : Dealer  
> **Dernière Mise à Jour** : 27 Avril 2026
