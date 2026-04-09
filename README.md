# Portfolio — ELYF Group App

**Application Flutter multi-entreprises (ERP multi-tenant) pour piloter
plusieurs activités commerciales depuis une seule app — offline-first,
role-aware, intégration matérielle native (imprimantes thermiques Sunmi).**

> Conçue et développée par **Carlos Simporé** — Founder, [Scalario Labs](https://scalario.com).
> ELYF Group est l'entreprise cliente exploitant l'application.

---

## 🎯 Vue d'Ensemble

ELYF Group App regroupe **6 modules métier** indépendants partageant une
même base technique (auth, multi-tenant, sync offline, trésorerie, audit
trail, permissions, notifications). Chaque module est conçu pour un métier
réel exercé sur le terrain au Burkina Faso.

### Stack Technique

| Couche | Technologies |
| --- | --- |
| **Frontend** | Flutter 3.9+ · Dart 3.9+ · Material 3 |
| **State management** | Riverpod (providers, AsyncValue) |
| **Local DB / offline** | Drift (SQLite) — offline-first sur tous les modules |
| **Backend** | Firebase (Firestore, Cloud Functions, Auth, FCM) |
| **Architecture** | Clean Architecture · Feature-First · Repository pattern |
| **Matériel** | Sunmi V3 Mix (impression thermique native), partage PDF système |
| **CI/CD** | GitHub Actions · Shorebird (hotfix OTA) |

### Fonctionnalités Transverses

- 🔐 **Auth Firebase** — login email/password, sessions sécurisées
- 🏢 **Multi-tenant** — un utilisateur peut switcher entre plusieurs
  entreprises (espaces) sans se reconnecter
- 📱 **Offline-first** — toutes les écritures passent par Drift puis sont
  poussées vers Firestore en arrière-plan
- 🛂 **Permissions granulaires** — système de rôles par module
  (Manager / POS / Agent / Dealer…)
- 🖨️ **Impression thermique** — tickets de caisse, reçus, rapports
- 📊 **Tableaux de bord rôle-conscients** — chaque rôle voit son propre
  dashboard
- 🔍 **Audit trail** — traçabilité des actions critiques
- 💰 **Trésorerie multi-comptes** — Caisse + Mobile Money par module

---

## 📸 Écrans Transverses

> Trois écrans communs à toute l'application — point d'entrée, switch
> multi-tenant, splash de chargement de module.

| Login | Switch d'espace | Chargement module |
| --- | --- | --- |
| ![Login](docs/assets/01-login.png) | ![Switch](docs/assets/02-workspace-switcher.png) | ![Loading](docs/assets/03-module-loading.png) |
| Écran d'accueil **« Bienvenue sur Elyf »** — auth Firebase email/mot de passe. | Popup **Changer d'espace** — passage rapide entre Orange Money Principal, Elyf Gaz, Elyf Immobilier… | Splash module-aware (ici Gaz) — initialisation Drift, sync, permissions. |

---

## 🏢 Modules d'Entreprise

Tous les modules ci-dessous sont **alignés sur l'implémentation réelle**
(avril 2026), avec des screenshots capturés sur tablette en conditions
réelles d'exploitation.

### [Eau Minérale](./eau_minerale/) ✅

**Usine de production et distribution d'eau en sachets/packs**

- 🏭 Sessions de production multi-jours (machines, bobines, matières, personnel)
- 📦 Stock bi-niveau (matières premières + produits finis)
- 🛒 Achats fournisseurs (comptant ou crédit)
- 💰 Ventes (cash / mobile money / mixte / crédit client)
- 💳 Crédits clients + encaissements
- 💵 Trésorerie multi-comptes
- 💸 Dépenses & rapports mensuels
- 👷 Salaires (employés fixes + ouvriers payés à la production)
- 📊 Rapports PDF (KPIs, tendances, ventilations)
- ⚙️ Parc machines + maintenance

**31 screenshots** · [📖 Documentation →](./eau_minerale/)

---

### [Gaz](./gaz/) ✅

**Distribution de bouteilles de gaz — gros & détail**

- 🚚 **Tournées multi-étapes** : Collecte vides → Recharge fournisseur → Livraison → Encaissement → Bilan → Clôture
- 🏪 Réseau de **POS affiliés** (succursales ou comptoirs simples)
- 🧑‍💼 Annuaire **grossistes B2B**
- 🛢️ Stock bi-modal (pleines / vides) par poids (3kg, 6kg, 10kg, 12kg)
- 💰 Versements POS → maison mère
- 🧾 Reçus PDF et partage natif (Drive, Bluetooth, Gmail…)
- ⚙️ Parc de bouteilles enregistré + tarification consigne

**Vues Manager + POS** · [📖 Documentation →](./gaz/)

---

### [Orange Money](./orange_money/) ✅

**Opérations cash-in / cash-out pour agents Orange Money agréés**

Deux rôles distincts :
- **Dealer** — pilote le réseau d'agents, supervise les liquidités, distribue les commissions
- **Agent** — exécute les transactions client (4 étapes), gère sa caisse quotidienne

Fonctionnalités :
- 💵 Transactions cash-in / cash-out / dépôt / retrait
- 🏧 **Déclaration matinale** + **bilan quotidien**
- 📊 Calcul automatique des commissions (par contrat)
- 💰 Suivi de liquidité (checkpoints horodatés)
- 📈 Trésorerie consolidée par dealer / agent

[📖 Documentation →](./orange_money/)

---

### [Boutique](./boutique/) ✅

**Point de vente (POS) commerce de détail**

- 🛒 Caisse temps réel avec scan code-barres + panier persistant
- 📦 Catalogue produits (catégories, photos, historique des prix, soft delete)
- 📥 Inventaire & mouvements de stock (réception, ajustement)
- 💵 Trésorerie multi-comptes (Caisse + Mobile Money)
- 🧾 **Chaînage cryptographique des tickets** (`ticketHash` / `previousHash`) pour audit
- 💳 Créances clients (ventes à crédit, encaissements partiels)
- 💸 Dépenses opérationnelles avec impact direct trésorerie
- 🖨️ Impression Sunmi native

[📖 Documentation →](./boutique/)

---

### [Immobilier](./immobilier/) ✅

**Gestion d'un portefeuille de propriétés en location**

- 🏠 Catalogue de propriétés (statut occupation, loyer mensuel)
- 👥 Annuaire locataires (à jour / en retard)
- 💰 Encaissement de loyers — assistant 2 étapes (Locataire → Maison → Mois à payer)
- ⚠️ Suivi des loyers en retard avec relance
- 💸 Dépenses rattachées à une propriété
- 💵 Trésorerie consolidée (caisse + banque)
- 📊 Rapports analytiques (revenus, dépenses, bénéfice net, taux d'occupation)

[📖 Documentation →](./immobilier/)

---

## 🏗️ Architecture Globale

### Clean Architecture & Feature-First

```
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

Chaque feature suit la même structure en couches :

```
features/<module>/
├── presentation/    # Screens, widgets, dialogs
├── application/     # Controllers Riverpod, services métier
├── domain/          # Entités pures, value objects, énumérés
└── data/            # Repositories (Firestore + Drift), datasources
```

---

## 🔐 Sécurité & Conformité

- Auth Firebase avec tokens sécurisés et règles Firestore par tenant
- Validation côté serveur (Cloud Functions)
- **Chaînage cryptographique** des tickets de caisse (boutique)
- Audit trail des actions critiques
- Permissions granulaires par module et par rôle
- Soft delete avec `deletedAt` / `deletedBy`

---

## 📊 Métriques

| Métrique | Valeur |
| --- | --- |
| **Modules métier** | 6 (5 opérationnels + administration) |
| **Screens** | 100+ |
| **Lignes de code** | ~50 000+ |
| **Plateformes** | Android (cible principale, tablettes Sunmi) · iOS prêt |
| **Mode offline** | ✅ Complet sur tous les modules |
| **Synchronisation** | ✅ Automatique en arrière-plan |
| **Hotfix OTA** | ✅ Shorebird |

---

## 👤 Auteur

**Carlos Simporé** — Founder, **Scalario Labs**

Architecte, développeur et designer unique de l'application ELYF Group App.
Conception produit, modélisation métier, implémentation Flutter/Firebase,
intégration matérielle (Sunmi), CI/CD et déploiement — réalisés en solo.

> ELYF Group est l'entreprise cliente exploitant l'application sur le
> terrain au Burkina Faso.

### Licence

Propriétaire — Scalario Labs © 2026 · Exploité sous licence par ELYF Group.

---

## 📝 Notes

> **État de la documentation** : ✅ Tous les modules alignés sur
> l'implémentation (avril 2026)  
> **Dernière mise à jour** : 9 Avril 2026
