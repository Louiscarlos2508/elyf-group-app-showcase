# Notes d'Implémentation — Module Eau Minérale

## État actuel

Module **production-ready**, le plus mature et le plus riche fonctionnellement
de l'app. Tous les repositories sont migrés en offline-first sur Drift.
17 repositories, 14 controllers, ~25 services métier orchestrés.

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

  static const _moduleType = 'eau_minerale';

  @override
  Future<void> create(X entity) async {
    await driftService.insert(entity);
    await syncManager.enqueue(SyncOperation.create(_moduleType, entity));
  }

  @override
  Stream<List<X>> watchAll() => driftService.watchByEnterprise(enterpriseId);
}
```

### 2. Controller + Service métier

Le module sépare strictement orchestration (Controller) et calculs
(Domain Service) :

```dart
class ProductionSessionController {
  ProductionSessionController(this._repo, this._builder, this._statusCalc);

  Future<ProductionSession> recordDay({
    required String sessionId,
    required ProductionDay day,
  }) async {
    final session = await _repo.findById(sessionId);
    final updated = _builder.appendDay(session, day);            // service
    final newStatus = _statusCalc.compute(updated);              // service
    final next = updated.copyWith(status: newStatus);
    await _repo.update(next);
    return next;
  }
}
```

### 3. Provider

```dart
final productionSessionControllerProvider =
    Provider<ProductionSessionController>((ref) {
  final enterpriseId = ref.watch(activeEnterpriseProvider).value!.id;
  return ProductionSessionController(
    ref.watch(productionSessionRepositoryProvider(enterpriseId)),
    ref.watch(productionSessionBuilderProvider),
    ref.watch(productionSessionStatusCalculatorProvider),
  );
});
```

## ✅ Repositories en place (17)

`production_session` · `machine` · `sale` · `product` · `customer` ·
`credit` · `stock` · `purchase` · `supplier` · `salary` · `daily_worker` ·
`finance` · `treasury` · `closing` · `activity` · `report` · `settings`

## ✅ Controllers en place (14)

`activity` · `clients` · `closing` · `finances` · `machine` · `product` ·
`production_session` · `purchase` · `report` · `salary` · `sales` ·
`stock` · `supplier` · `treasury`

## 🏭 Workflow Production (jour-type)

1. **Démarrer la semaine** (`demarrer_semaine_screen.dart`) →
   `ProductionSessionController.startSession()` crée une `ProductionSession`
   en `draft` avec les machines à utiliser
2. **Chaque jour** (`enregistrer_aujourd_hui_screen.dart`) →
   `recordDay()` ajoute un `ProductionDay` (personnel, packs, conso) ;
   `ProductionSessionStatusCalculator` passe le statut à `inProgress`
3. **Terminer la production** (`terminer_production_screen.dart`) →
   `completeSession()` calcule le coût total
   (`MachineMaterialCostService` + `ProductionMarginCalculator`),
   met à jour le stock matières (consommé) et produits finis (produit),
   passe le statut à `completed`
4. **Salaires** : en fin de période, `SalaryCalculationService` distribue
   les `ProductionPayment` aux `DailyWorker` selon les jours travaillés
   et les taux

## 💰 Workflow Vente

1. UI → `SalesController.createSale()` → `SaleService.validate()` →
   `SaleCalculationService.computeTotals()`
2. `PaymentSplitterService` éclate le paiement entre Cash, Mobile Money
   et crédit
3. Si crédit → `CreditService` crée un `CustomerCredit`
4. Le stock produit fini est décrémenté via `StockController`
5. La trésorerie est mise à jour via `TreasuryMovementMapper`

## 🔍 Intégrité & Réconciliation

Trois services veillent en arrière-plan :
- `StockIntegrityService` — vérifie qu'aucune sortie n'a créé un stock
  négatif (matières + produits finis)
- `StockReconciliationService` — recale le stock après import / sync /
  ajustement manuel
- `PaymentReconciliationService` — vérifie que la somme des
  encaissements = somme des `Sale.amountPaid` du jour

`HistoricalStockService` permet de **reconstruire l'état du stock à une
date donnée** en rejouant les `StockMovement`, utile pour les rapports
rétroactifs et l'audit.

## 📝 Best Practices

1. **Jamais** d'accès direct aux repositories depuis l'UI — toujours via
   provider → controller
2. Les **calculs** restent dans les services `domain/services/`, jamais
   dans les controllers ni l'UI
3. Toute modification de stock **doit** passer par `StockController` (qui
   crée un `StockMovement` traçable)
4. Toute vente **doit** passer par `SalesController.createSale()` pour
   garantir paiement / crédit / stock / trésorerie cohérents
5. Toute session de production **doit** passer par `ProductionSessionBuilder`
   pour préserver la cohérence machines/matières/personnel
6. Soft delete uniquement
7. Filtrer **toujours** par `enterpriseId`
8. Logger les opérations critiques avec `developer.log`
