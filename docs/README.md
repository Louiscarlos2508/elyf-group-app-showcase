# Documentation du Portfolio

Documentation transverse du portfolio **ELYF Group App** (Carlos Simporé —
Scalario Labs).

## 📁 Structure

```
Project_Portfolio/
├── README.md                  # Index principal du portfolio
├── docs/
│   ├── README.md              # Ce fichier
│   └── assets/                # Screenshots transverses (login, switch, splash)
├── eau_minerale/   ✅         # 31 screenshots — production / ventes / salaires
├── gaz/            ✅         # 38 screenshots — vues Manager + POS, tournées
├── orange_money/   ✅         # 32 screenshots — vues Dealer + Agent, cash-in/out
├── boutique/       ✅         # 10 screenshots — POS commerce détail + chaînage
├── immobilier/     ✅         # 13 screenshots — locations + encaissement loyers
└── admin/          ✅         # 3 screenshots — console de pilotage 360° du groupe
```

Chaque module suit la même structure :

```
<module>/
├── README.md                  # Vue d'ensemble + features + gallery + modèle de domaine
├── docs/
│   ├── ARCHITECTURE.md        # Détails d'architecture (couches, flux, patterns)
│   └── IMPLEMENTATION.md      # Notes d'implémentation (patterns, best practices)
└── assets/screenshots/        # Captures alignées sur l'implémentation
```

## 📸 Écrans transverses

Les captures dans [`assets/`](./assets/) sont communes à toute l'app :

- `01-login.png` — écran d'authentification Firebase
- `02-workspace-switcher.png` — popup de switch multi-tenant
- `03-module-loading.png` — splash de chargement d'un module

Ils sont référencés depuis le [README principal](../README.md).

## 📝 Statut

Tous les modules sont **alignés sur l'implémentation** (avril 2026) et
documentés à partir du code source réel (`lib/features/<module>/`), pas de
spécifications théoriques.

## 🔗 Liens

- [← Portfolio principal](../README.md)
- [Code source](../../lib/features/)
