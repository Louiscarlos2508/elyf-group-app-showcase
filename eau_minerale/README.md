# Module Eau Minérale — ELYF Group App

## 🎯 Vue d'Ensemble

Le module **Eau Minérale** pilote une **usine de production et de
distribution d'eau en sachets/packs**. Il couvre l'intégralité du cycle :
**approvisionnement matières → sessions de production multi-jours →
stock → ventes (comptant + crédit) → trésorerie → salaires** (employés
fixes + journaliers payés à la production), avec un volet rapports
financiers.

### Caractéristiques Principales

- 🏭 **Sessions de production multi-jours** — machines, bobines, matières premières, personnel
- 📦 **Stock bi-niveau** — matières premières + produits finis avec ajustements
- 🛒 **Approvisionnements** — achats matières au comptant ou à crédit fournisseur
- 💰 **Ventes** — vente directe (cash / mobile money) ou à crédit client
- 💳 **Crédits clients** — encours, encaissements, suivi par client
- 💵 **Trésorerie multi-comptes** — Caisse + Mobile Money avec mouvements
- 💸 **Dépenses** — charges journalières, mensuelles, par catégorie
- 👷 **Salaires** — employés fixes + ouvriers journaliers payés à la production
- 📊 **Rapports** — KPIs Mois, tendances 7 jours, rapport mensuel PDF
- ⚙️ **Configuration** — parc machines, fournisseurs, catalogue produits

Code source : [lib/features/eau_minerale/](../../lib/features/eau_minerale/)

---

## 🧩 Architecture Fonctionnelle

Navigation latérale en 5 sections :

| Section | Écrans |
| --- | --- |
| **Pilotage** | Tableau de Bord · Rapports |
| **Commercial** | Ventes |
| **Production** | Production (sessions) |
| **Logistique** | Stock · Achats |
| **Finances** | Trésorerie · Dépenses · Crédits · Salaires |
| **Configuration** | Fournisseurs · Catalogue · Profil · Paramètres |

---

## ✨ Fonctionnalités Implémentées

### 1. Tableau de Bord

[dashboard_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/dashboard_screen.dart)

- Bloc **Aujourd'hui** : Chiffre d'affaires, Encaissements, Production du jour (packs)
- Actions rapides : **Nouvelle Vente**, **Nouvel Achat**
- Bloc **Ce mois** : CA, Production, Dépenses, Résultat Net
- Graphique **Tendances 7 derniers jours** (ventes vs production)
- Onglets **Stock** : Produits Finis / Matières Premières

### 2. Rapports

[reports_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/reports_screen.dart)

- Sélection de la période (Date début / Date fin) + **Générer PDF**
- KPIs financiers consolidés sur la période
- Onglets : Ventes · Dépenses · Salaires · Rentabilité · Tendances · Productions
- Détail des ventes par produit + tableau ligne à ligne
- Vue **Rapport Mensuel** dédiée avec dépenses par catégorie

### 3. Ventes

[sales_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/sales_screen.dart)

- Liste journalière des ventes (client, produit, montant)
- **+ Nouvelle vente** — formulaire en deux blocs :
  - Informations client (produit, client existant ou ponctuel)
  - Détails de vente (quantité, prix unitaire, stock disponible)
  - Mode de paiement : **Cash / Mobile Money / Mixte**
  - Switch **Crédit** : devient une vente à crédit (pas d'encaissement immédiat)
- Modèle : [sale.dart](../../lib/features/eau_minerale/domain/entities/sale.dart)

### 4. Production — Sessions multi-jours

[production_sessions_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/production_sessions_screen.dart)
· [production_session_detail_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/production_session_detail_screen.dart)

Une **session de production** couvre plusieurs jours et regroupe les machines,
matières et personnel mobilisés.

- État accueil : « Pas de production en cours » + **Démarrer une nouvelle production**
- Historique des sessions précédentes (période, nombre de jours, packs produits)
- **Détail session** :
  - **Informations générales** : date, heures début/fin, durée totale, quantité produite, emballages utilisés
  - **Machines utilisées** + **Utilisation des matières** par machine (bobines installées)
  - **Détail du personnel** par jour de production (nombre, conso, packs, emballages, montant CFA)
  - Onglet **Rapport** : récap finalisé avec statut `Terminée`
- Statuts : `draft → inProgress → completed | cancelled` (cf. [production_session_status.dart](../../lib/features/eau_minerale/domain/entities/production_session_status.dart))
- Modèle : [production_session.dart](../../lib/features/eau_minerale/domain/entities/production_session.dart) — porte `machineMaterials`, `productionDays`, `events`, `coutEmballages`, `machineMaterialCost`

### 5. Stock & Mouvements

[stock_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/stock_screen.dart)

- Vue d'ensemble **Matières Premières** (Bobine, Emballage…) + **Produits Finis** (Pack)
- Alertes seuils (`5 unités` etc.)
- Liste des mouvements pour un produit donné (entrées, sorties, ajustements)
- **Ajustement de stock** via dialog : Ajouter / Retirer + motif/justificatif
- Modèles : [stock_item.dart](../../lib/features/eau_minerale/domain/entities/stock_item.dart) · [stock_movement.dart](../../lib/features/eau_minerale/domain/entities/stock_movement.dart)

### 6. Achats / Approvisionnements

[purchases_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/purchases_screen.dart)

- Onglets **Bon de commande** / **Achats validés**
- **Nouvel approvisionnement** plein écran en deux blocs :
  - **1. Ce que j'achète** : matière première, quantité, prix total payé (multi-lignes)
  - **2. Comment je paie** : `Tout payé` (mode de paiement) ou `Acompte / Crédit`, fournisseur optionnel
- Modèle : [purchase.dart](../../lib/features/eau_minerale/domain/entities/purchase.dart) + [supplier_settlement.dart](../../lib/features/eau_minerale/domain/entities/supplier_settlement.dart)

### 7. Trésorerie

[treasury_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/treasury_screen.dart)

- Soldes consolidés **Caisse** et **Mobile Money**
- 4 actions rapides : **Apport · Retrait · Transfert · Ajuster**
- Filtres temporels (jour/semaine/mois) + historique horodaté
- Modèle : [treasury_movement.dart](../../lib/features/eau_minerale/domain/entities/treasury_movement.dart)

### 8. Dépenses

[expenses_screen.dart (via finances)](../../lib/features/eau_minerale/presentation/screens/sections/finances_screen.dart)

- KPI **Dépenses du jour** + **Résumé Mensuel**
- Historique des dépenses, saisie via bouton flottant
- Modèles : [expense.dart](../../lib/features/eau_minerale/domain/entities/expense.dart) · [expense_record.dart](../../lib/features/eau_minerale/domain/entities/expense_record.dart)

### 9. Crédits clients

- **Total Crédits en cours** + **Nombre de clients avec crédit**
- Tableau **Suivi par client** : contact, total dû, ancienneté, action **Encaisser**
- Modèles : [customer_credit.dart](../../lib/features/eau_minerale/domain/entities/customer_credit.dart) · [credit_payment.dart](../../lib/features/eau_minerale/domain/entities/credit_payment.dart) · [customer_account.dart](../../lib/features/eau_minerale/domain/entities/customer_account.dart)

### 10. Salaires

[salaries_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/salaries_screen.dart)

- Bandeau « Tout est à jour » quand toutes les périodes sont soldées
- Historique des périodes payées (semaines / mois) avec montant total
- Distinction **Employés fixes** (mensuels) vs **Personnel de production** (à la session)
- Modèles : [employee.dart](../../lib/features/eau_minerale/domain/entities/employee.dart) · [daily_worker.dart](../../lib/features/eau_minerale/domain/entities/daily_worker.dart) · [salary_payment.dart](../../lib/features/eau_minerale/domain/entities/salary_payment.dart) · [production_payment.dart](../../lib/features/eau_minerale/domain/entities/production_payment.dart)

### 11. Fournisseurs

[suppliers_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/suppliers_screen.dart)

- Annuaire avec recherche + bouton **+ Nouveau**
- Création : nom complet/établissement, téléphone, email, adresse
- Modèle : [supplier.dart](../../lib/features/eau_minerale/domain/entities/supplier.dart)

### 12. Catalogue Produits

[catalog_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/catalog_screen.dart)

- Liste filtrable (Tous / Produits Finis / Matières Premières)
- Création **Produit Fini** : type, nom, prix unitaire, unité, description
- Création **Matière Première** : conversion d'unité (achat → consommation), seuil d'alerte
- Édition d'un produit existant
- Modèle : [product.dart](../../lib/features/eau_minerale/domain/entities/product.dart)

### 13. Configuration — Machines & Maintenance

[settings_screen.dart](../../lib/features/eau_minerale/presentation/screens/sections/settings_screen.dart)

- Onglet **4 Machines / 4 Actives** — gestion du parc (Machine N°1 … N°4 avec réf.)
- Onglet **Maintenance & Alertes** : signalement de pannes, statut opérationnel par machine
- Modèle : [machine.dart](../../lib/features/eau_minerale/domain/entities/machine.dart)

---

## 📸 Screenshots

> Captures réalisées sur tablette en mode clair, avec des données réelles
> d'exploitation (avril 2026).

### 1. Pilotage — Tableau de Bord

| Aujourd'hui | Ce mois & Tendances | Stock |
| --- | --- | --- |
| ![Dashboard top](assets/screenshots/01-dashboard-top.png) | ![Dashboard mois](assets/screenshots/02-dashboard-mois.png) | ![Dashboard stock](assets/screenshots/03-dashboard-stock.png) |
| CA, encaissements et production du jour + actions rapides. | KPIs du mois (CA 493 020 FCFA, Dépenses 277 200 FCFA, Résultat 215 820 FCFA) et courbes ventes/production. | Onglets Stock — Pack à 301 unités. |

### 2. Pilotage — Rapports

| Période & PDF | Détail Ventes | Rapport Mensuel | Détail Dépenses |
| --- | --- | --- | --- |
| ![Rapports](assets/screenshots/04-rapports-periode.png) | ![Ventes](assets/screenshots/05-rapports-ventes.png) | ![Mensuel](assets/screenshots/06-rapports-mensuel.png) | ![Dépenses](assets/screenshots/07-rapports-depenses.png) |
| Sélection période + génération PDF. | KPIs Ventes/Encaissements/Charges/Trésorerie + tableau ligne à ligne. | Synthèse mensuelle (Productions, Quantité, Dépenses, Salaires, Crédits Vente). | Ventilation Dépenses par catégorie. |

### 3. Commercial — Ventes

| Liste des ventes | Nouvelle vente — client | Nouvelle vente — paiement |
| --- | --- | --- |
| ![Ventes](assets/screenshots/08-ventes-liste.png) | ![Nouvelle client](assets/screenshots/09-ventes-nouvelle-client.png) | ![Nouvelle paiement](assets/screenshots/10-ventes-nouvelle-paiement.png) |
| Liste journalière par client. | Sélection produit + client + qté. | Mode Cash / Gros / Mixte + montant total + bouton Enregistrer. |

### 4. Production

| Accueil Production | Détail session | Personnel session | Rapport session |
| --- | --- | --- | --- |
| ![Accueil](assets/screenshots/11-production-accueil.png) | ![Détail](assets/screenshots/12-production-session-details.png) | ![Personnel](assets/screenshots/13-production-session-personnel.png) | ![Rapport](assets/screenshots/14-production-session-rapport.png) |
| État vide + bouton Démarrer + historique. | Infos générales, machines, matières par machine. | Détail journalier du personnel (packs, conso, montant). | Rapport finalisé avec statut Terminée. |

### 5. Logistique — Stock & Achats

| Vue d'ensemble Stock | Mouvements produit | Ajustement | Achats | Nouvel approvisionnement |
| --- | --- | --- | --- | --- |
| ![Stock](assets/screenshots/15-stock-overview.png) | ![Mouvements](assets/screenshots/16-stock-mouvements.png) | ![Ajustement](assets/screenshots/18-stock-ajustement-dialog.png) | ![Achats](assets/screenshots/17-achats-liste.png) | ![Approvisionnement](assets/screenshots/19-achats-nouvel-approvisionnement.png) |
| Matières premières + produits finis avec alertes. | Tableau des mouvements pour un produit fini. | Ajouter/Retirer avec motif. | Onglets Bon de commande / Achats validés. | Formulaire 2 blocs : Ce que j'achète / Comment je paie. |

### 6. Finances

| Trésorerie | Dépenses | Crédits | Salaires |
| --- | --- | --- | --- |
| ![Trésorerie](assets/screenshots/20-tresorerie.png) | ![Dépenses](assets/screenshots/21-depenses.png) | ![Crédits](assets/screenshots/22-credits.png) | ![Salaires](assets/screenshots/23-salaires-historique.png) |
| Caisse + Mobile Money + mouvements. | Dépenses jour + résumé mensuel. | Total crédits + suivi par client + Encaisser. | Historique des périodes + état "Tout est à jour". |

### 7. Configuration

| Fournisseurs | Nouveau fournisseur | Catalogue | Nouveau produit fini | Nouvelle matière | Édition produit |
| --- | --- | --- | --- | --- | --- |
| ![Fournisseurs](assets/screenshots/24-fournisseurs-liste.png) | ![Nouveau](assets/screenshots/25-fournisseurs-nouveau.png) | ![Catalogue](assets/screenshots/26-catalogue-liste.png) | ![Nouveau PF](assets/screenshots/27-catalogue-nouveau-produit-fini.png) | ![Nouvelle MP](assets/screenshots/28-catalogue-nouvelle-matiere.png) | ![Édition](assets/screenshots/29-catalogue-modifier-produit.png) |
| Annuaire + recherche. | Nom, téléphone, email, adresse. | Pack / Bobine / Emballage. | Type, nom, prix, unité, description. | Conversion d'unité + seuil d'alerte. | Modification d'un emballage existant. |

| Parc Machines | Maintenance & Alertes |
| --- | --- |
| ![Machines](assets/screenshots/30-parametres-machines.png) | ![Maintenance](assets/screenshots/31-parametres-maintenance.png) |
| 4 machines actives avec référence. | Signalement de pannes par machine. |

---

## 🛠️ Modèle de Domaine

Tous les modèles dans
[lib/features/eau_minerale/domain/entities/](../../lib/features/eau_minerale/domain/entities/).

### ProductionSession (cœur du module)
```dart
class ProductionSession {
  final String id;
  final String enterpriseId;
  final DateTime date;
  final int period;                              // Numéro de période
  final DateTime heureDebut;
  final DateTime? heureFin;
  final List<String> machinesUtilisees;
  final List<MachineMaterialUsage> machineMaterials; // Bobines installées
  final double quantiteProduite;                  // Packs produits
  final String quantiteProduiteUnite;
  final double? emballagesUtilises;
  final double? machineMaterialCost;
  final double? coutEmballages;
  final ProductionSessionStatus status;           // draft|inProgress|completed|cancelled
  final List<ProductionEvent> events;
  final List<ProductionDay> productionDays;       // Détail jour par jour
}
```

### Sale
```dart
class Sale {
  final String productId;
  final String? customerId;          // null = client ponctuel
  final double quantity;
  final double unitPrice;
  final double totalAmount;
  final PaymentMethod paymentMethod; // cash | mobileMoney | mixed
  final bool isCredit;               // Si oui → CustomerCredit
}
```

### Purchase
```dart
class Purchase {
  final String supplierId;
  final List<PurchaseLine> lines;    // matière, quantité, prix
  final double totalAmount;
  final double amountPaid;           // Tout payé OU acompte
  final bool isCredit;               // → SupplierSettlement à venir
}
```

Autres entités notables :
- `Machine` / `MachineMaterialUsage` / `MaterialConsumption`
- `Employee` / `DailyWorker` / `SalaryPayment` / `ProductionPayment`
- `CustomerCredit` / `CreditPayment` / `CustomerAccount`
- `Supplier` / `SupplierSettlement`
- `Expense` / `ExpenseRecord` / `ExpenseReportData`
- `TreasuryMovement` / `Closing`
- `ReportPeriod` / `ReportData` / `ProductionReportData` / `SalaryReportData`

---

## 🔄 Flux Métier Clés

### Cycle de production

```mermaid
graph TD
    A[Démarrer session<br/>status = draft] --> B[Affecter machines + bobines]
    B --> C[Affecter personnel du jour]
    C --> D[Production en cours<br/>status = inProgress]
    D --> E{Jour suivant ?}
    E -->|Oui| C
    E -->|Non| F[Terminer la session]
    F --> G[Calcul coûts<br/>matières + emballages]
    G --> H[Rapport finalisé<br/>status = completed]
    H --> I[Stock produits finis ↑]
```

### Vente avec crédit

```mermaid
graph TD
    A[Nouvelle vente] --> B[Sélection client + produit]
    B --> C[Quantité + Prix unitaire]
    C --> D{Mode}
    D -->|Cash/MM/Mixte| E[Encaissement immédiat]
    D -->|Crédit| F[CustomerCredit créé]
    E --> G[Trésorerie ↑ + Stock ↓]
    F --> H[Suivi par client]
    H --> I[Encaisser → CreditPayment]
    I --> G
```

### Approvisionnement

```mermaid
graph TD
    A[Nouvel approvisionnement] --> B[Sélection matière + qté + prix]
    B --> C{Tout payé ?}
    C -->|Oui| D[Trésorerie ↓]
    C -->|Non| E[SupplierSettlement à régler]
    D --> F[Stock matières ↑]
    E --> F
```

---

## 🔗 Liens Utiles

- 📁 [Code source du module](../../lib/features/eau_minerale/)
- 📐 [Architecture](./docs/ARCHITECTURE.md)
- 📋 [Notes d'implémentation](./docs/IMPLEMENTATION.md)
- 🏠 [Retour au Portfolio](../)

---

## 📝 Notes

> **État de la Documentation** : ✅ Aligné sur l'implémentation (avril 2026) — 31 screenshots  
> **Dernière Mise à Jour** : 9 Avril 2026
