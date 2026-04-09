# Notes d'Implémentation — Module Boutique

## État actuel

Module **production-ready** : tous les repositories sont migrés en
offline-first sur Drift, le chaînage cryptographique des tickets est en
place, l'impression Sunmi est intégrée nativement, et la sync Firestore
tourne automatiquement.

## 🏗️ Patterns

### 1. OfflineRepository

Tous les repositories étendent un pattern commun :

```dart
class XOfflineRepository implements XRepository {
  XOfflineRepository({
    required this.driftService,
    required this.syncManager,
    required this.connectivityService,
    required this.enterpriseId,
  });

  final DriftService driftService;
  final SyncManager syncManager;
  final ConnectivityService connectivityService;
  final String enterpriseId;

  static const _moduleType = 'boutique';

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
class StoreController {
  StoreController(this._productRepo, this._saleRepo, this._stockRepo);

  Future<Sale> checkout(Cart cart, PaymentMethod method) async {
    final sale = SaleBuilder(cart, method)
      .withTicketHash(_lastHash)         // chaînage
      .withNumber(_numberingService.next())
      .build();
    await _saleRepo.create(sale);
    await _stockRepo.decrementBatch(cart.items);
    return sale;
  }
}
```

### 3. Provider

```dart
final storeControllerProvider = Provider<StoreController>((ref) {
  final enterpriseId = ref.watch(activeEnterpriseProvider).value!.id;
  return StoreController(
    ref.watch(productRepositoryProvider(enterpriseId)),
    ref.watch(saleRepositoryProvider(enterpriseId)),
    ref.watch(stockRepositoryProvider(enterpriseId)),
  );
});
```

## ✅ Repositories en place

- `ProductOfflineRepository`
- `CategoryOfflineRepository`
- `SaleOfflineRepository`
- `StockOfflineRepository` · `StockMovementOfflineRepository`
- `PurchaseOfflineRepository`
- `SupplierOfflineRepository` · `SupplierSettlementOfflineRepository`
- `ExpenseOfflineRepository`
- `TreasuryOfflineRepository`
- `ClosingOfflineRepository`
- `BoutiqueSettingsOfflineRepository`
- `ReportOfflineRepository`

## 🧾 Chaînage de tickets

Implémenté dans `domain/services/security/`. Avant d'enregistrer une
vente :

1. Lire le `ticketHash` de la dernière vente de la boutique
2. Calculer `ticketHash = sha256(saleContent || previousHash)`
3. Stocker `previousHash` et `ticketHash` sur la nouvelle vente

Toute vérification d'intégrité rejoue la chaîne du début à la fin.

## 🖨️ Impression Sunmi

L'impression passe par `core/printing/sunmi_printer_service.dart`. Le
service prend une `Sale` et délègue le formatage du ticket à
`BoutiqueExportService`. L'impression est non-bloquante et silencieuse en
cas d'erreur (le ticket reste réimprimable depuis l'historique).

## 📝 Best Practices

1. **Jamais** d'appel direct aux repositories depuis l'UI — toujours via
   provider → controller
2. Toujours filtrer par `enterpriseId` (vient de `activeEnterpriseProvider`)
3. Logger les opérations critiques avec `developer.log`
4. Utiliser `ErrorHandler` pour les erreurs utilisateur
5. Soft-delete via `deletedAt` / `deletedBy`, jamais de `DELETE` SQL direct
6. Toute nouvelle vente **doit** passer par `StoreController.checkout()`
   pour préserver le chaînage des tickets
