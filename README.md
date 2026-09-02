<div align="center">

# 🏭 ELYF Group App

### Un ERP multi-entreprises **Flutter + Firebase**, offline-first, pour piloter 6 activités commerciales réelles depuis une seule application.

**Conçue, architecturée et développée de A à Z par [Carlos Simporé](https://scalario.com) — Founder, Scalario Labs.**

</div>

<div align="center">

![Flutter](https://img.shields.io/badge/Flutter-3.29+-02569B?style=for-the-badge&logo=flutter&logoColor=white)
![Dart](https://img.shields.io/badge/Dart-3.7+-0175C2?style=for-the-badge&logo=dart&logoColor=white)
![Riverpod](https://img.shields.io/badge/Riverpod-3-white?style=for-the-badge&logo=riverpod&logoColor=black)
![Drift/SQLite](https://img.shields.io/badge/Drift%20SQLite-offline--first-4479A1?style=for-the-badge&logo=sqlite&logoColor=white)
![Firebase](https://img.shields.io/badge/Firebase-backend-FFCA28?style=for-the-badge&logo=firebase&logoColor=black)
![Clean Architecture](https://img.shields.io/badge/Architecture-Clean%20%2F%20Feature--First-6DB33F?style=for-the-badge)
![GitHub Actions](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)

**Télécharger l'app** · [📲 APK (Firebase Storage)](https://elyf-app-portal--elyf-group-app.us-east4.hosted.app/) &nbsp;·&nbsp; **Console admin** · [🌐 Web app](https://elyf-group-app.web.app/)

</div>

---

## 💡 Le projet en une phrase

> Un seul code Flutter, **6 ERP métier**, **7 rôles**, **120+ écrans**, fonctionnant **hors-ligne** sur des tablettes Android (Sunmi), synchronisés en arrière-plan avec Firebase — utilisé **en production quotidienne** par ELYF Group au Burkina Faso.

Chaque module modélise un **métier réel exercé sur le terrain** : production d'eau, distribution de gaz en bouteilles, opérations Orange Money, commerce de détail, gestion locative, et un back-office de pilotage transverse du groupe.

---

## 🎯 Pourquoi ce projet est différent

Ce n'est pas une démo. C'est un **produit en production**, avec de vraies contraintes terrain résolues proprement :

| Défi réel | Solution technique |
| --- | --- |
| 📶 **Réseau instable sur le terrain** | Architecture **offline-first** : Drift (SQLite) en local, toutes les opérations passent par la base locale, sync Firestore automatique en arrière-plan. |
| 🏢 **Plusieurs activités, un seul groupe** | **Multi-tenant** : switch d'entreprise sans reconnexion, isolation stricte par `enterpriseId`. |
| 👥 **Rôles très différents** | **Permissions granulaires** et **dashboards rôle-conscients** : Manager, POS, Agent, Dealer… chacun voit le sien. |
| 🖨️ **Impression de tickets sur le terrain** | **Intégration matérielle native** des imprimantes thermiques **Sunmi V3 Mix**. |
| 🧾 **Fraude / audit** | **Chaînage cryptographique des tickets de caisse** (`ticketHash` / `previousHash`) + audit trail des actions critiques. |
| 📱 **Mises à jour immédiates** | **Hotfix OTA** via Shorebird, **CI/CD** GitHub Actions, **APK signé** distribué par Firebase Storage. |

---

## 🏗️ Les modules

| Module | Activité | Screenshots | Statut |
| --- | --- | --- | --- |
| 💧 [**Eau Minérale**](./eau_minerale/) | Usine de production & distribution d'eau | 31 | ✅ Production |
| 🏪 [**Gaz**](./gaz/) | Distribution de bouteilles de gaz (gros & détail) | 38 | ✅ Production |
| 📱 [**Orange Money**](./orange_money/) | Dealer & agents Mobile Money | 32 | ✅ Production |
| 🛒 [**Boutique**](./boutique/) | Point de vente commerce de détail | 10 | ✅ Production |
| 🏠 [**Immobilier**](./immobilier/) | Gestion locative de propriétés | 13 | ✅ Production |
| 🧭 [**Administration**](./admin/) | Console de pilotage 360° du groupe | 3 | ✅ Production |

---

### 💧 Eau Minérale — Usine de production & distribution

> Cycle complet : approvisionnement matières → sessions de production multi-jours → stock → ventes → trésorerie → salaires.

- 🏭 **Sessions de production multi-jours** (machines, bobines, matières, personnel)
- 📦 Stock **bi-niveau** (matières premières + produits finis) avec alertes de seuil
- 🛒 Achats fournisseurs (comptant ou crédit, support lots)
- 💰 Ventes cash / **mobile money** / mixte / **crédit client**
- 👷 **Salaires** (employés fixes + ouvriers payés à la production)
- 📊 **Rapports PDF** (KPIs, tendances, ventilations)
- ⚙️ Parc machines + maintenance

**31 screenshots** · [📖 Documentation complète →](./eau_minerale/)

---

### 🏪 Gaz — Distribution de bouteilles (gros & détail)

> Des **tournées multi-étapes** sur le terrain à la gestion d'un réseau de points de vente affiliés.

- 🚚 **Tournées** : Collecte vides → Recharge fournisseur → Livraison → Encaissement → Bilan → Clôture
- 🏪 Réseau de **POS affiliés** (succursales ou comptoirs simples)
- 🧑‍💼 Annuaire **grossistes B2B**
- 🛢️ Stock **bi-modal** (pleines / vides) par poids (3–12 kg)
- 💰 **Versements POS → maison mère** avec validation manager
- 🧾 Reçus **PDF** et partage natif (Drive, Bluetooth, Gmail…)
- ⚙️ Parc de bouteilles + tarification consigne

**Vues Manager + POS** · [📖 Documentation complète →](./gaz/)

---

### 📱 Orange Money — Dealer & agents Mobile Money

> Deux rôles dans un seul module : le **Dealer** pilote son réseau, l'**Agent** exécute les transactions client.

- 💵 Transactions cash-in / cash-out / dépôt / retrait (assistant 3 & 4 étapes)
- 🏧 **Déclaration matinale** + **bilan quotidien** avec réconciliation théorique vs réel
- 📊 Saisie mensuelle manuelle des **commissions** (avec preuve photo)
- 💰 Suivi de **liquidité** (checkpoints horodatés, auto-clôture)
- 📈 **Trésorerie consolidée** par dealer / agent, recharges, apports

**Vues Dealer + Agent** · [📖 Documentation complète →](./orange_money/)

---

### 🛒 Boutique — Point de vente commerce de détail

> Une caisse moderne pour le commerce de détail — du scan code-barres à l'audit anti-fraude.

- 🛒 **Caisse temps réel** : scan code-barres + panier persistant
- 📦 Catalogue produits (catégories, photos, historique des prix, soft delete)
- 📥 Inventaire & mouvements de stock (réception, ajustement)
- 🧾 **Chaînage cryptographique des tickets** pour audit
- 💳 Créances clients (ventes à crédit, encaissements partiels)
- 💵 Trésorerie multi-comptes + dépenses
- 🖨️ Impression Sunmi native

[📖 Documentation complète →](./boutique/)

---

### 🏠 Immobilier — Gestion locative

> Gestion d'un portefeuille de propriétés en location, de l'encaissement des loyers aux rapports de rentabilité.

- 🏠 Catalogue de propriétés (statut d'occupation, loyer mensuel)
- 👥 Annuaire locataires (**à jour / en retard**)
- 💰 **Encaissement de loyers** — assistant 2 étapes (Locataire → Maison → Mois à payer)
- ⚠️ Suivi des loyers en retard + **facturation automatique** des échéances
- 🧾 Génération automatique de **quittances PDF**
- 💵 Trésorerie consolidée (caisse + banque)
- 📊 Rapports analytiques (revenus, dépenses, bénéfice net, taux d'occupation)

**13 screenshots** · [📖 Documentation complète →](./immobilier/)

---

### 🧭 Administration — Console 360° du groupe

> Vue transverse sur **toutes** les entreprises et **tous** les modules, depuis un seul écran réservé aux admins.

- 📈 **Dashboard groupe** — KPIs consolidés (CA, Dépenses, Bénéfice net, Marge) + graphique d'évolution journalière
- 🏢 **Organisation** — toutes les entreprises, fiche live par entreprise avec onglets adaptés au métier (**pattern Strategy**)
- 🚚 **Vue Gaz** — historique des tours + fiche tour live (Bilan · Recharge · POS · Grossistes · Timeline)
- 📦 **Vue Eau Minérale** — stock + appros admin directes avec notification automatique
- 🛒 **Vue Boutique** — performance, stock, trésorerie par point de vente
- 👥 **Accès** — gestion des utilisateurs et rôles par module
- 🔍 **Audit trail** complet + ⚡ lecture **Firestore live** (données toujours fraîches)

**3 screenshots** · [📖 Documentation complète →](./admin/)

---

## 🖼️ Écrans transverses

Trois écrans communs à toute l'application — le point d'entrée, le switch multi-tenant, et le splash de chargement.

| Login | Switch d'espace | Chargement module |
| --- | --- | --- |
| ![Login](docs/assets/01-login.png) | ![Switch](docs/assets/02-workspace-switcher.png) | ![Loading](docs/assets/03-module-loading.png) |
| Écran « Bienvenue sur Elyf » — auth Firebase email/mot de passe. | Popup « Changer d'espace » — bascule rapide entre Orange Money Principal, Elyf Gaz, Elyf Immobilier… | Splash module-aware — initialisation Drift, sync, permissions. |

---

## ⚙️ Stack technique

| Couche | Technologies |
| --- | --- |
| **Frontend** | Flutter 3.29+ · Dart 3.7+ · Material Design 3 |
| **State management** | Riverpod 3 (providers, AsyncValue, family) |
| **Local DB / offline** | Drift (SQLite) — offline-first sur tous les modules opérateur |
| **Backend** | Firebase Auth · Firestore · Cloud Functions · FCM · Firebase Storage |
| **Architecture** | Clean Architecture · Feature-First · Repository pattern |
| **Matériel** | Sunmi V3 Mix (impression thermique native), partage PDF système |
| **CI/CD** | GitHub Actions · Shorebird (hotfix OTA) |
| **Distribution** | APK signé → Firebase Storage (lien permanent) + GitHub Release |

### Fonctionnalités transverses

- 🔐 **Auth Firebase** — login email/password, sessions sécurisées
- 🏢 **Multi-tenant** — switch entre entreprises sans reconnexion
- 📱 **Offline-first** — Drift (SQLite) en local, sync Firestore en arrière-plan
- 🛂 **Permissions granulaires** — rôles par module (Manager / POS / Agent / Dealer…)
- 🖨️ **Impression thermique** — tickets, reçus, rapports via Sunmi V3 Mix
- 📊 **Dashboards rôle-conscients** — chaque rôle voit son propre tableau de bord
- 🔍 **Audit trail** — traçabilité des actions critiques
- 💰 **Trésorerie multi-comptes** — Caisse + Mobile Money par module
- 🔔 **Notifications push** — FCM + notifications en-app

---

## 🏗️ Architecture globale

### Clean Architecture & Feature-First

```text
lib/
├── app/                       # Configuration globale (router, theme)
├── core/                      # Services transverses
│   ├── auth/                 # Authentification Firebase
│   ├── firebase/             # Wrappers Firestore, Functions, FCM
│   ├── offline/              # Drift (SQLite) + sync layer
│   ├── printing/             # Intégration Sunmi V3
│   ├── permissions/          # Système de permissions par module
│   └── tenant/               # Multi-tenant (active enterprise)
├── features/                  # Modules métier (feature-first)
│   ├── eau_minerale/
│   ├── gaz/
│   ├── orange_money/
│   ├── boutique/
│   ├── immobilier/
│   └── administration/
└── shared/                    # Widgets et utilitaires partagés
```

### Structure d'un module

Chaque feature suit la même organisation en couches :

```text
features/<module>/
├── presentation/    # Screens, widgets, dialogs
├── application/     # Controllers Riverpod, services métier
├── domain/          # Entités pures, value objects, énumérés
└── data/            # Repositories (Firestore + Drift), datasources
```

> 💡 **Lecture du flux** : UI → Provider → Controller → Service métier (validation / calcul) → Repository offline (Drift) → SyncManager → Firestore. Aucun appel réseau bloquant en lecture.

---

## 🧪 Tests

**52 fichiers de tests** couvrant les couches critiques — les règles métier (services de calcul purs) sont prioritairement couvertes car elles concentrent la complexité.

| Catégorie | Fichiers | Périmètre |
| --- | --- | --- |
| **Domain services** | 17 | Calculs métier (production, ventes, crédits, rapports, validation) |
| **Controllers** | 12 | Logique applicative Riverpod — Gaz, Immobilier, Administration |
| **Repositories** | 4 | Couche Drift (SQLite) — Boutique, Eau Minérale, Gaz, Orange Money |
| **Core offline** | 3 | Sync Firestore → Drift, résolution de conflits |
| **Intégration** | 2 | Multi-tenant, offline sync end-to-end |
| **Widgets** | 8 | Composants partagés (navigation, layout, états vides, erreurs) |

---

## 🔒 Sécurité & conformité

- Auth Firebase avec tokens sécurisés et règles Firestore par tenant
- Validation côté serveur (Cloud Functions)
- **Chaînage cryptographique** des tickets de caisse (Boutique)
- **Audit trail** des actions critiques
- Permissions granulaires par module et par rôle
- Soft delete avec `deletedAt` / `deletedBy`

---

## 📊 Métriques

| Métrique | Valeur |
| --- | --- |
| **Modules métier** | 6 opérationnels + console administration |
| **Screens** | 120+ |
| **Lignes de code** | ~60 000+ |
| **Plateformes** | Android (cible principale, tablettes Sunmi) · Web (admin) |
| **Mode offline** | ✅ Complet sur tous les modules opérateur |
| **Synchronisation** | ✅ Automatique en arrière-plan (Drift → Firestore) |
| **Hotfix OTA** | ✅ Shorebird |
| **Distribution APK** | ✅ Firebase Storage (lien permanent par release) |
| **CI/CD** | ✅ GitHub Actions — build APK + upload Storage + GitHub Release |

---

## 👤 Auteur

**Carlos Simporé** — Founder, Scalario Labs

Architecte, développeur et designer unique de l'application ELYF Group App :
**conception produit**, modélisation métier, implémentation **Flutter/Firebase**,
intégration **matérielle (Sunmi)**, **CI/CD** et **déploiement** — le tout réalisé en solo.

> **ELYF Group** est l'entreprise cliente exploitant l'application sur le terrain au Burkina Faso.

### Licence

Propriétaire — Scalario Labs © 2026 · Exploité sous licence par ELYF Group.

---

> **État** : ✅ Tous les modules alignés sur l'implémentation — Avril 2026 · v0.1.24
