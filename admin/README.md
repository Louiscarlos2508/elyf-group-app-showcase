# Module Administration

**Console de pilotage 360° du groupe ELYF — vue transverse sur toutes les
entreprises et tous les modules depuis un seul écran.**

> Rôle réservé aux admins du groupe. Aucun accès depuis les apps opérateur.

---

## Fonctionnalités

### Tableau de bord groupe
- 📈 **KPIs consolidés** (CA, Dépenses, Bénéfice net, Marge) filtrables par
  période (Aujourd'hui / Semaine / Mois / Personnalisée)
- 📉 **Graphique d'évolution journalière** CA vs Dépenses
- ⚡ Données en direct (Firestore live — pas de cache Drift côté admin)

### Organisation
- 🏢 **Liste de toutes les entreprises** du groupe avec type et statut
- 🔍 Recherche rapide par nom
- 📊 **Stratégie par type d'entreprise** — chaque fiche adopte les onglets
  adaptés au métier :
  - *Boutique* → Performance · Ventes · Stock · Trésorerie · Audit
  - *Gaz (mère)* → Résumé · Tours · POS · Trésorerie · Versements · Audit
  - *Gaz POS* → Performance · Stock · Trésorerie · Audit
  - *Eau Minérale* → Vue 360° stock, appros, historique
  - *Orange Money* → Dashboard commissions, déclarations
  - *Immobilier* → Propriétés, locataires, encaissements

### Vue Gaz — Tours (exemple)
- 📋 Historique des tours trié : **en cours d'abord**, puis clôturés, annulés
- 🔍 **Fiche tour complète** (bottom sheet) : Bilan · Départ · Recharge
  fournisseur · POS · Grossistes · Dépenses transport · Stock restant · Timeline
- Statuts tous couverts : Ouvert · Collecte · Recharge · Livraison ·
  Encaissement · Clôture · Clôturé · Annulé
- Passages multiples sur le même site **fusionnés** (un grossiste visité
  deux fois n'apparaît qu'une fois, quantités cumulées)
- Bilan « Vides collectées » exclut les échanges grossistes

### Accès & Audit
- 👥 Gestion des utilisateurs et rôles par module
- 🔍 Audit trail complet (toutes les actions critiques tracées)
- 🔔 Notifications groupe

---

## Captures d'écran

### Tableau de bord groupe

![Dashboard groupe](assets/screenshots/01-dashboard-groupe.png)

> Dashboard admin — KPIs groupe consolidés (CA 4 255 690 CFA · Bénéfice
> 2 938 290 CFA · Marge 69%) avec graphique d'évolution journalière sur le mois.

---

### Vue organisation — fiche entreprise

![Organisation entreprise](assets/screenshots/02-organisation-entreprise.png)

> Liste du groupe (à gauche) + fiche **Boutique Accessoires Gaz - Bogandé**
> avec onglets Performance · Ventes · Stock · Trésorerie · Audit.
> KPIs du mois : CA 27 000 CFA, Net 27 000 CFA, Marge 100%.

---

### Fiche tour Gaz — détail complet

![Tour detail](assets/screenshots/03-gaz-tour-detail.png)

> Bottom sheet du tour du 12 avr 2026 (statut Recharge) — section Bilan
> (514 pleines livrées · 312 vides collectées · 688 500 FCFA cash · Net
> −1 021 040 FCFA) et section Recharge Fournisseur avec détail par poids.

---

## Architecture technique (admin)

Le module admin lit **exclusivement Firestore** (pas de Drift) via des
loaders dédiés (`AdminGazLoader`, `AdminBoutiqueLoader`…) pour ne pas
dépendre de la sync layer des modules opérateur, souvent vide sur le device
admin.

```
features/administration/
├── data/services/
│   ├── admin_gaz_loader.dart         # Stream live Firestore (tours, ventes, stocks…)
│   ├── admin_boutique_loader.dart
│   ├── admin_eau_loader.dart
│   └── admin_supply_service.dart     # Appros admin → PO + notification user
├── application/providers/
│   ├── admin_gaz_providers.dart
│   ├── admin_financial_providers.dart
│   └── …
└── presentation/screens/sections/strategies/
    ├── dashboard_strategy.dart       # Dispatcher strategy par type d'entreprise
    └── parts/
        ├── gaz_strategy.dart
        ├── boutique_strategy.dart
        └── …
```

**Pattern Strategy** : chaque type d'entreprise implémente
`EnterpriseDashboardStrategy` (onglets + contenu), ce qui permet d'ajouter
un nouveau type sans modifier le shell.
