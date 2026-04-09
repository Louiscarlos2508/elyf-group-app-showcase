# Notes d'Implémentation — Module Orange Money

## État actuel

Module **production-ready**, avec deux rôles complets (Dealer + Agent),
tous les repositories migrés en offline-first sur Drift, et un workflow
quotidien complet : déclaration matinale → transactions → bilan journée →
commissions mensuelles.

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

  static const _moduleType = 'orange_money';

  @override
  Future<void> create(X entity) async {
    await driftService.insert(entity);
    await syncManager.enqueue(SyncOperation.create(_moduleType, entity));
  }

  @override
  Stream<List<X>> watchAll() => driftService.watchByEnterprise(enterpriseId);
}
```

### 2. Controller

```dart
class OrangeMoneyController {
  OrangeMoneyController(this._txRepo, this._liquidityRepo);

  Future<Transaction> recordCashIn({
    required double amount,
    required String? customerPhone,
  }) async {
    final tx = Transaction.cashIn(
      enterpriseId: _enterpriseId,
      amount: amount,
      customerPhone: customerPhone,
    );
    await _txRepo.create(tx);
    await _liquidityRepo.applyTransaction(tx);
    return tx;
  }
}
```

### 3. Provider

```dart
final orangeMoneyControllerProvider = Provider<OrangeMoneyController>((ref) {
  final enterpriseId = ref.watch(activeEnterpriseProvider).value!.id;
  return OrangeMoneyController(
    ref.watch(transactionRepositoryProvider(enterpriseId)),
    ref.watch(liquidityRepositoryProvider(enterpriseId)),
  );
});
```

## ✅ Repositories en place

- `TransactionOfflineRepository`
- `AgentOfflineRepository`
- `ContractOfflineRepository`
- `CommissionOfflineRepository`
- `LiquidityOfflineRepository`
- `TreasuryOfflineRepository`
- `SettingsOfflineRepository`

## ✅ Controllers en place

- `OrangeMoneyController` — transactions
- `AgentsController` — réseau d'agents
- `ContractController` — contrats Dealer-Agent
- `CommissionsController` — calcul et paiement
- `LiquidityController` — checkpoints matin/soir
- `SettingsController` — configuration

## 🔄 Workflow Agent (journée type)

1. **Matin** : `morning_declaration_screen.dart` → `LiquidityController.declareMorning()`
   crée un `LiquidityCheckpoint(type: morning)` avec soldes Cash + MM
2. **Journée** : `transactions_v2_screen.dart` (4 étapes : Type → Montant
   → Client → Confirmation) → `OrangeMoneyController.recordCashIn/Out()`
3. **Soir** : `daily_summary_screen.dart` → `LiquidityController.closeDailySummary()`
   crée un `LiquidityCheckpoint(type: evening)`, calcule l'écart,
   alerte si > seuil
4. **Fin de mois** : `CommissionsController.computeFor(agent, period)` →
   propose le montant, le Dealer valide et déclare le paiement

## 🔄 Workflow Dealer

1. Supervise plusieurs Agents via `agents_screen.dart`
2. Voit la liquidité agrégée du réseau (`liquidity_screen.dart`)
3. Saisit ses propres transactions Dealer (`dealer_transaction_screen.dart`)
4. Déclare et paie les commissions mensuelles
5. Suit la trésorerie consolidée (`treasury_tab.dart`)

## 📝 Best Practices

1. **Jamais** d'accès direct aux repositories depuis l'UI
2. Toujours filtrer par `enterpriseId` (`activeEnterpriseProvider`)
3. Le rôle utilisateur (Dealer / Agent) conditionne les routes — ne pas
   mélanger les écrans
4. Toute transaction **doit** mettre à jour la liquidité du jour
5. Soft delete uniquement (jamais de DELETE physique)
6. Logger les opérations critiques avec `developer.log`
7. Les commissions sont calculées **à partir des transactions persistées**,
   jamais à la volée — toujours passer par `CommissionsController.computeFor()`
