# Notes d'Implémentation — Module Gaz

## État actuel

Module **production-ready** avec deux rôles complets (Manager + POS).
18 repositories, 11 controllers, ~20 services métier. Le workflow
**Tournée 6 étapes** est opérationnel de bout en bout, avec génération
PDF, partage natif et impression Sunmi.

## 🏗️ Patterns

### 1. OfflineRepository

```dart
class XOfflineRepository implements XRepository {
  XOfflineRepository({
    required this.driftService,
    required this.syncManager,
    required this.connectivityService,
    required this.enterpriseId,
  });

  static const _moduleType = 'gaz';

  @override
  Future<void> create(X entity) async {
    await driftService.insert(entity);
    await syncManager.enqueue(SyncOperation.create(_moduleType, entity));
  }
}
```

### 2. TransactionService — opérations atomiques

Le module manipule régulièrement plusieurs entités en cascade
(stock + tour + trésorerie + interactions). `TransactionService` garantit
l'atomicité : toutes les écritures réussissent, ou tout est rollback.

```dart
class TourService {
  Future<Tour> recordEncaissement({
    required String tourId,
    required String wholesalerId,
    required double amount,
    required PaymentMethod method,
  }) async {
    return _txService.run((txn) async {
      final tour = await _tourRepo.findById(tourId, txn: txn);
      final interaction = tour.findInteraction(wholesalerId);
      final updated = interaction.markPaid(amount, method);
      await _tourRepo.updateInteraction(tourId, updated, txn: txn);
      await _treasuryRepo.recordIncome(amount, source: tourId, txn: txn);
      return _tourRepo.findById(tourId, txn: txn);
    });
  }
}
```

### 3. Provider

```dart
final tourServiceProvider = Provider<TourService>((ref) {
  final enterpriseId = ref.watch(activeEnterpriseProvider).value!.id;
  return TourService(
    tourRepo: ref.watch(tourRepositoryProvider(enterpriseId)),
    stockRepo: ref.watch(cylinderStockRepositoryProvider(enterpriseId)),
    treasuryRepo: ref.watch(treasuryRepositoryProvider(enterpriseId)),
    txService: ref.watch(transactionServiceProvider),
  );
});
```

## ✅ Repositories en place (18)

`tour` · `cylinder_stock` · `cylinder_leak` · `exchange` · `gas` ·
`wholesaler` · `pos_remittance` · `pos_stock_movement` · `gaz_contract` ·
`gaz_credit_payment` · `gaz_employee` · `gaz_salary_payment` ·
`gaz_settings` · `inventory_audit` · `site_logistics_record` · `expense` ·
`treasury` · `financial_report`

## ✅ Controllers en place (11)

`gas` · `cylinder` · `cylinder_stock` · `cylinder_leak` · `wholesaler` ·
`gaz_employee` · `gaz_salary_payment` · `expense` · `gaz_settings` ·
`financial_report` · `leak_report`

## 🚚 Workflow Tournée

```
open → collecting → recharging → delivering → encaissement → closing → closed
```

### Étape 1 — Collecte

`TourService.recordCollect(tourId, wholesalerId, byWeight)` :
1. Crée ou met à jour un `TourSiteInteraction(type: collect)`
2. Incrémente `cylinderStock(emptyAtStore)` du camion
3. Met à jour le statut du tour si nécessaire

### Étape 2 — Recharge

`TourService.recordRecharge(tourId, supplier, full, empty, cost)` :
1. Stocke `fullBottlesReceived`, `emptyBottlesReturned`, `gasPurchaseCost`
   sur le `Tour`
2. Décrémente `cylinderStock(emptyAtStore)` (vides retournés)
3. Incrémente `cylinderStock(full)` (pleines reçues)
4. Mouvement trésorerie négatif pour le coût

### Étape 3-4 — Livraison & Encaissement

Distribution automatique des pleines aux grossistes/POS selon ce qui a
été collecté en étape 1, puis saisie des encaissements (cash / MM /
crédit). Génération du PDF via `GazSalePdfService` et partage natif via
`GazPrintingService` (Drive, Bluetooth, Gmail, imprimante Sunmi).

### Étape 5-6 — Bilan & Clôture

`TourService.computeBilan(tourId)` calcule via
`GazFinancialCalculationService` :

```
Résultat Net = Σ Ventes Sites − Coût Recharge − Σ Dépenses Trajet
```

`closeTour(tourId)` passe en statut `closed` (verrouillé, plus aucune
modification).

## 🔍 Cohérence & Réconciliation

- `DataConsistencyService` vérifie en continu : aucun stock négatif,
  somme `full + empty ≤ registeredTotal` par poids, cohérence
  collectes/échanges/livraisons par tour
- `GazReconciliationService` rapproche les tours clôturés avec les
  mouvements de stock réels
- `GazAlertService` lève des alertes (stock bas, fuite déclarée, écart
  d'inventaire)

## 🖨️ PDF & Partage

- `GazSalePdfService` génère un **Reçu d'encaissement** PDF (logo,
  client, table par poids, total, mode de paiement)
- `GazPrintingService` ouvre le quick share natif Android (Imprimer
  Sunmi, Drive, Bluetooth, Gmail) sur le PDF généré

## 📝 Best Practices

1. **Jamais** d'accès direct aux repositories depuis l'UI
2. Toute opération multi-entités (tour + stock + trésorerie) **doit**
   passer par `TransactionService.run()` pour l'atomicité
3. Les calculs de bilan/marge restent dans les services
   `domain/services/`, pas dans les controllers
4. Tout mouvement de bouteille **doit** créer un `StockMovement`
   traçable, jamais d'écriture directe sur `CylinderStock`
5. Une `Tour` clôturée est **immutable** — toute correction passe par
   un nouveau tour de régulation
6. Filtrer **toujours** par `enterpriseId` (POS de type entreprise = sub
   enterprise avec son propre id)
7. Soft delete uniquement
8. Logger les opérations critiques avec `developer.log`
