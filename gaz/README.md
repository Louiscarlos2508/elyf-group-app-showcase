# Module Gaz — ELYF Group App

## 🎯 Vue d'Ensemble

Le module **Gaz** pilote une activité de **distribution de bouteilles de gaz**
en gros et au détail. Il modélise une chaîne logistique complète :
- Une **entreprise mère (Manager)** qui possède le camion, le stock de
  bouteilles et le réseau de **points de vente (POS)** affiliés.
- Des **POS** rattachés (succursales internes ou simples comptoirs) qui
  vendent au détail et **versent** leurs encaissements à la maison mère.
- Des **Grossistes** (clients B2B) qui achètent par lots.
- Un **Fournisseur** central chez qui le camion va recharger les bouteilles
  pleines en échange des vides.

Le cœur du module est la **Tournée (Tour)** : un workflow multi-étapes qui
suit le camion de la collecte des vides jusqu'à la clôture des comptes.

### Caractéristiques Principales

- 🚚 **Tournées multi-étapes** — Collecte → Recharge → Livraison → Encaissement → Clôture
- 🏪 **Réseau de POS** — succursales (entreprise) ou comptoirs simples
- 🧑‍💼 **Grossistes** — annuaire B2B et collecte de vides
- 🛢️ **Stock bi-modal** — bouteilles pleines / vides par poids (3kg, 6kg, 10kg, 12kg…)
- 💰 **Versements POS** — flux financier des comptoirs vers la maison mère
- 💵 **Trésorerie & Contrats** — pilotage financier
- ⚙️ **Parc de bouteilles** — registre du nombre total enregistré par poids

---

## 🧩 Architecture Fonctionnelle

Navigation latérale en 4 sections (rôle Manager illustré ici) :

| Section | Écrans |
| --- | --- |
| **Pilotage** | Accueil (Tableau de Bord) · Tours |
| **Réseau** | Mes POS |
| **Finances** | Trésorerie · Contrats |
| **Configuration** | Paramètres (parc de bouteilles) · Profil |

L'écran **Tableau de Bord** est rôle-conscient :
- **Manager** : CTA "Nouveau Tour", KPIs encaissements, dernier tour, versements POS
- **POS** : 3 cards (Stock, Ventes du jour, Crédits)

Code source : [lib/features/gaz/](../../lib/features/gaz/)

---

## ✨ Fonctionnalités Implémentées

### 1. Tableau de Bord

[dashboard_screen.dart](../../lib/features/gaz/presentation/screens/sections/dashboard_screen.dart)

- Encaissement du jour, dernier tour effectué, versements POS reçus
- CTA permanent **+ Nouveau Tour** pour démarrer la journée

### 2. Tournée — Workflow 6 étapes

Le workflow `Tour` est implémenté avec un stepper visuel à 6 étapes
(constantes [TourStatus](../../lib/features/gaz/domain/entities/tour.dart) :
`collecting → recharging → delivering → encaissement → closing → closed`).

#### Configuration du départ
Saisie de la date du tour, puis bouton **Démarrer le tour** qui crée le `Tour`
en statut `open` et redirige vers l'étape Collecte.

#### Étape 1 — Collecte des vides
Liste des **grossistes** chez qui le camion passe pour récupérer les
bouteilles vides. Chaque interaction est enregistrée comme
`TourSiteInteraction`.

- Recherche / sélection d'un grossiste existant
- Création à la volée d'un **Nouveau Grossiste** (nom, téléphone, adresse)
- Saisie des quantités collectées **par poids** (3kg, 6kg, 10kg, 12kg) avec
  steppers ±
- Bouton flottant **État du Camion** (popup) pour voir à tout moment :
  - Total bouteilles pleines (par poids)
  - Total bouteilles vides collectées
  - Cash en main

#### Étape 2 — Recharge fournisseur
Le camion se rend chez le fournisseur :
- Bouteilles vides retournées (`emptyBottlesReturned`)
- Bouteilles pleines reçues (`fullBottlesReceived`)
- Coût d'achat (`gasPurchaseCost`) et nom du fournisseur

#### Étapes 3-4 — Livraison & Encaissement
Distribution des pleines aux POS et grossistes, encaissement immédiat ou à
crédit.

#### Étapes 5-6 — Résumé & Clôture
Consolidation et passage en statut `closed`.

Modèle complet :
[tour.dart](../../lib/features/gaz/domain/entities/tour.dart) — porte aussi
les dépenses logistiques (`TransportExpense` : carburant, péage…) et le
stock résiduel dans le camion.

### 3. Réseau de POS

[mes_pos_screen.dart](../../lib/features/gaz/presentation/screens/sections/mes_pos_screen.dart)

- Liste des POS rattachés à l'entreprise mère (`enterprisesByParentAndType`
  filtré sur `gasPointOfSale`)
- Création d'un POS via dialog avec deux variantes :
  - **Simple** : nom, localisation, contact
  - **Entreprise** : crée une véritable sous-entreprise
    (gestion des données autonomes)
- Détail d'un POS en bottom sheet : stock, historique des versements
- Bouton **Versement** : enregistrement d'un encaissement reversé à la
  maison mère
  - Modèle :
    [pos_remittance.dart](../../lib/features/gaz/domain/entities/pos_remittance.dart)
    — `pending → validated → rejected`, mode de paiement, référence,
    rattachement optionnel à un `tourId`

### 4. Trésorerie

[tresorerie_screen.dart](../../lib/features/gaz/presentation/screens/sections/tresorerie_screen.dart)

- Solde global de la maison mère
- Historique des mouvements filtrable (`Aujourd'hui / Semaine / Mois / Tout`)
- Inclut encaissements et dépenses du module gaz

### 5. Contrats

[contrats_screen.dart](../../lib/features/gaz/presentation/screens/sections/contrats_screen.dart)

Liste des contrats commerciaux (grossistes, fournisseurs) avec recherche.

### 6. Configuration — Parc de bouteilles

L'écran **Paramètres** permet de configurer les **types de bouteilles**
gérés par l'entreprise :
- Pour chaque poids (1kg, 3kg, 6kg, 10kg, 12kg…) :
  - Prix d'achat / prix de vente / prix de consigne
  - **Parc total enregistré** (`registeredTotal`) — fixé à la création,
    mis à jour uniquement en cas d'achat ou de réforme
- Modèle :
  [cylinder.dart](../../lib/features/gaz/domain/entities/cylinder.dart)
- Le suivi du stock pleines/vides est délégué au `CylinderStockRepository`
  (stock bi-modal)

### 7. Modules avancés (présents dans le code)

Le code source contient des écrans/répertoires non capturés ici :

- [retail/](../../lib/features/gaz/presentation/screens/sections/retail/) — interface POS détail
- [wholesale/](../../lib/features/gaz/presentation/screens/sections/wholesale/) — gros
- [credits_screen.dart](../../lib/features/gaz/presentation/screens/sections/credits_screen.dart) — créances clients
- [sales_screen.dart](../../lib/features/gaz/presentation/screens/sections/sales_screen.dart) — historique des ventes
- [inventory_screen.dart](../../lib/features/gaz/presentation/screens/sections/inventory_screen.dart) — inventaires (`gaz_inventory_audit.dart`)
- [cylinder_leak/](../../lib/features/gaz/presentation/screens/sections/cylinder_leak/) — déclaration de fuites
- [finance/](../../lib/features/gaz/presentation/screens/sections/finance/) — rapports financiers
- [expenses/](../../lib/features/gaz/presentation/screens/sections/expenses/) — dépenses opérationnelles

---

## 📸 Screenshots — Vue Manager

> Captures réalisées sur tablette en mode clair, **avec des données réelles
> de tournées** pour illustrer le workflow complet de bout en bout.

### 1. Pilotage

| Tableau de Bord | Liste des Tours |
| --- | --- |
| ![Dashboard](assets/screenshots/19-dashboard-avec-donnees.png) | ![Tours](assets/screenshots/01-tours-liste.png) |
| Encaissements du jour, dernier tour, versements POS, CTA **+ Nouveau Tour**. | Historique des tournées avec encaissé / dépensé / bénéfice par ligne. |

### 2. Démarrage d'un Tour

| Configuration du départ |
| --- |
| ![Configuration](assets/screenshots/04-tour-configuration-depart.png) |
| Sélection de la date puis **Démarrer le tour** pour entrer dans le workflow 6 étapes. |

### 3. Étape 1 — Collecte des vides

| Liste des grossistes | Saisie par poids | Grossiste validé |
| --- | --- | --- |
| ![Collecte](assets/screenshots/05-tournee-collecte-grossistes.png) | ![Saisie](assets/screenshots/06-tournee-collecte-saisie.png) | ![Validé](assets/screenshots/07-tournee-collecte-validee.png) |
| Stepper 6 étapes en tête, liste des grossistes à visiter. | Compteurs `3kg / 6kg / 10kg / 12kg` avec ± pour chaque grossiste. | Cocher ✓ et activer **Aller à la recharge**. |

### 4. Étape 2 — Recharge fournisseur

| Recharge + État du Camion |
| --- |
| ![Recharge](assets/screenshots/08-tournee-recharge-etat-camion.png) |
| Saisie de l'échange vides → pleines avec popup live **État du Camion** (pleines, vides, cash). |

### 5. Étapes 3-4 — Livraison & Encaissement

| Livraison Grossistes | Livraison POS | Encaissement Grossistes |
| --- | --- | --- |
| ![Livraison Gros](assets/screenshots/09-tournee-livraison-grossistes.png) | ![Livraison POS](assets/screenshots/10-tournee-livraison-pos.png) | ![Encaissement](assets/screenshots/11-tournee-encaissement-grossistes.png) |
| Onglet Grossistes — auto-distribution des bouteilles collectées. | Onglet Points de Vente — distribution aux POS affiliés. | Détail par poids et total dû par grossiste. |

| Dialog Encaisser | Encaissement validé | Reçu PDF |
| --- | --- | --- |
| ![Encaisser](assets/screenshots/12-tournee-encaisser-dialog.png) | ![Confirmé](assets/screenshots/13-tournee-encaissement-confirme.png) | ![PDF](assets/screenshots/03-tour-recu-pdf.png) |
| Saisie du montant et du mode (Espèces / Mobile Money). | Confirmation et accès au reçu généré. | **REÇU D'ENCAISSEMENT** PDF avec table par poids et total. |

| Reçu — popup | Partage du PDF |
| --- | --- |
| ![Reçu popup](assets/screenshots/02-tour-recu-popup.png) | ![Partage](assets/screenshots/14-tournee-partage-pdf.png) |
| Récap du paiement (client, date, montant, mode). | Quick share : Imprimer, Drive, Bluetooth, Gmail. |

### 6. Étapes 5-6 — Bilan & Clôture

| Bilan Final | Détail & Clôture | Confirmation |
| --- | --- | --- |
| ![Bilan](assets/screenshots/15-tournee-bilan-final.png) | ![Détail](assets/screenshots/16-tournee-bilan-cloture.png) | ![Confirmer](assets/screenshots/17-tournee-confirmer-cloture.png) |
| Flux financier : Ventes Sites − Coût Recharge − Dépenses Trajet = **Résultat Net**. | Détail par site, reçus grossistes, impression rapport et **Clôturer définitivement**. | Dialog de confirmation avant passage en `closed`. |

| Tour Clôturé |
| --- |
| ![Clôturé](assets/screenshots/18-tournee-cloture.png) |
| Badge vert **Tour Clôturé** — la tournée est verrouillée et archivée. |

### 7. Réseau — Mes POS

| Liste des POS | Détail POS | Versement |
| --- | --- | --- |
| ![POS](assets/screenshots/29-pos-liste.png) | ![Détail](assets/screenshots/30-pos-detail.png) | ![Versement](assets/screenshots/31-pos-versement-formulaire.png) |
| Annuaire des points de vente affiliés. | Stock et historique des versements d'un POS. | Saisie d'un versement reversé à la maison mère. |

| POS Simple | POS Entreprise |
| --- | --- |
| ![Simple](assets/screenshots/32-pos-nouveau-simple.png) | ![Entreprise](assets/screenshots/33-pos-nouveau-entreprise.png) |
| Comptoir simple : nom, localisation, contact. | Sous-entreprise complète avec données autonomes. |

### 8. Finances

| Trésorerie — Historique | Trésorerie — Actions | Contrats |
| --- | --- | --- |
| ![Historique](assets/screenshots/21-tresorerie-historique.png) | ![Actions](assets/screenshots/20-tresorerie-actions.png) | ![Contrats](assets/screenshots/34-contrats.png) |
| Solde consolidé et mouvements (recharges, recettes, dépenses). | Popup : Nouvelle dépense, Apport, Retrait, Transfert, Ajustement. | Liste des contrats grossistes / fournisseurs. |

### 9. Configuration

| Paramètres — Parc de bouteilles |
| --- |
| ![Paramètres](assets/screenshots/35-parametres-bouteilles.png) |
| Types de bouteilles (1kg, 3kg, 6kg, 10kg, 12kg) : prix achat / vente / consigne et parc total enregistré. |

---

## 📸 Screenshots — Vue POS (Détail & Gros)

> Vue d'un **point de vente affilié** (compte POS Bogandé). Le POS gère son
> propre stock, ses ventes au détail et au gros, ses crédits clients et la
> tarification de ses bouteilles.

### 1. Accueil POS

| Tableau de bord POS |
| --- |
| ![Accueil POS](assets/screenshots/40-pos-accueil.png) |
| 3 cards rôle POS : **Stock actuel**, **Ventes du jour** (Détail / Gros / Total), **Crédits en cours**. |

### 2. Stock & Mouvements

| Stock par poids | Mouvement — Entrée | Mouvement — Retour |
| --- | --- | --- |
| ![Stock](assets/screenshots/41-pos-stock.png) | ![Entrée](assets/screenshots/42-pos-mouvement-entree.png) | ![Retour](assets/screenshots/43-pos-mouvement-retour.png) |
| Pleines / vides par poids (3kg, 6kg, 10kg, 12kg) + bouton **Nouveau Mouvement**. | Vides livrées (retour fournisseur) + Pleines reçues (entrée). | Vides reçues du client + Empruntées (pleine → vide). |

### 3. Ventes

| Détail | Gros | Historique |
| --- | --- | --- |
| ![Détail](assets/screenshots/44-pos-ventes-detail.png) | ![Gros](assets/screenshots/45-pos-ventes-gros.png) | ![Historique](assets/screenshots/46-pos-ventes-historique.png) |
| Recette du jour + ventes par poids avec stepper rapide. | Onglet Gros avec **Nouvelle vente**. | Historique des ventes du POS. |

### 4. Crédits

| Crédits clients |
| --- |
| ![Crédits](assets/screenshots/47-pos-credits.png) |
| Total des crédits en cours et recherche par client. |

### 5. Configuration POS

| Gestion des bouteilles | Modifier le prix |
| --- | --- |
| ![Bouteilles](assets/screenshots/48-pos-parametres-bouteilles.png) | ![Prix](assets/screenshots/49-pos-parametres-prix-dialog.png) |
| Liste des poids gérés par le POS avec prix achat / détail / gros. | Édition du prix de vente détail pour un poids donné. |

---

## 🛠️ Modèle de Domaine

Tous les modèles dans
[lib/features/gaz/domain/entities/](../../lib/features/gaz/domain/entities/).

### Tour (Journal du Camion)
```dart
enum TourStatus {
  open, collecting, recharging, delivering,
  encaissement, closing, closed, cancelled,
}

class Tour {
  final String id;
  final DateTime tourDate;
  final TourStatus status;
  // Stock initial dans le camion au départ
  final Map<int, int> initialFullBottles;   // {6: 50, 12: 30}
  final Map<int, int> initialEmptyBottles;
  // Stock résiduel à la clôture
  final Map<int, int> remainingFullBottles;
  final Map<int, int> remainingEmptyBottles;
  // Échange fournisseur (Recharge)
  final Map<int, int> fullBottlesReceived;
  final Map<int, int> emptyBottlesReturned;
  final double? gasPurchaseCost;
  final String? supplierName;
  // Passages POS / grossistes
  List<TourSiteInteraction> get siteInteractions;
  // Dépenses logistiques (carburant, péage…)
  List<TransportExpense> get transportExpenses;
}
```

### Cylinder (type de bouteille)
```dart
class Cylinder {
  final int weight;              // 3, 6, 10, 12…
  final double buyPrice;
  final double sellPrice;
  final double depositPrice;     // Consigne
  final int registeredTotal;     // Parc total enregistré
}
```

### GazPOSRemittance (versement POS → mère)
```dart
enum RemittanceStatus { pending, validated, rejected }

class GazPOSRemittance {
  final String posId;
  final double amount;
  final DateTime remittanceDate;
  final RemittanceStatus status;
  final PaymentMethod paymentMethod;
  final String? reference;       // OM/Momo/Bancaire
  final String? tourId;          // Lien au tour
  final String? validatedBy;
  final DateTime? validatedAt;
}
```

### Wholesaler (client B2B)
```dart
class Wholesaler {
  final String name;
  final String? phone;
  final String? address;
  final String? email;
  final bool isActive;
}
```

Autres entités notables :
- `CylinderStock` — stock bi-modal pleines/vides par poids
- `StockMovement` / `StockAlert` — mouvements et alertes
- `ExchangeRecord` — échanges de bouteilles
- `GasSale` — vente détail
- `GazContract` — contrats commerciaux
- `GazCreditPayment` — paiements de créances
- `GazEmployee` / `GazSalaryPayment` — RH du module
- `CylinderLeak` — déclarations de fuite
- `GazInventoryAudit` — inventaires physiques
- `GazTreasurySynthesis` — synthèse trésorerie
- `SiteLogisticsRecord` — log logistique des sites visités

---

## 🔄 Flux Métier Clés

### Workflow d'une Tournée

```mermaid
graph TD
    A[Démarrer le tour<br/>status = open] --> B[Collecte des vides<br/>chez les grossistes]
    B --> C{Plus de grossistes ?}
    C -->|Oui| B
    C -->|Non| D[Recharge fournisseur<br/>vides → pleines]
    D --> E[Livraison aux POS / clients]
    E --> F[Encaissement<br/>cash + crédits]
    F --> G[Clôture<br/>résumé + résidus]
    G --> H[status = closed]
    B -.->|popup| I[État du Camion<br/>pleines / vides / cash]
    E -.->|popup| I
```

### Versement POS → Maison Mère

```mermaid
graph TD
    A[POS encaisse au détail] --> B[Constitution du fond à reverser]
    B --> C[Saisie versement<br/>montant + mode de paiement]
    C --> D[GazPOSRemittance.pending]
    D --> E{Validation Manager}
    E -->|OK| F[validated]
    E -->|Refus| G[rejected]
    F --> H[Mise à jour trésorerie maison mère]
```

### Cycle d'une bouteille

```mermaid
graph LR
    A[Vide chez client] --> B[Collecte par camion]
    B --> C[Retour au dépôt]
    C --> D[Recharge fournisseur]
    D --> E[Pleine reçue]
    E --> F[Livrée au POS / client]
    F --> A
```

---

## 🔗 Liens Utiles

- 📁 [Code source du module](../../lib/features/gaz/)
- 📐 [Architecture](./docs/ARCHITECTURE.md)
- 📋 [Notes d'implémentation](./docs/IMPLEMENTATION.md)
- 🏠 [Retour au Portfolio](../)

---

## 📝 Notes

> **État de la Documentation** : ✅ Aligné sur l'implémentation (avril 2026)  
> **Rôle illustré dans les screenshots** : Manager (entreprise mère)  
> **Dernière Mise à Jour** : 9 Avril 2026
