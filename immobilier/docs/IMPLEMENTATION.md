# Notes d'Implémentation — Module Immobilier

## État actuel

Module **production-ready**, refactoré en avril 2026 pour simplifier l'UX
(voir refonte UX validée). 8 repositories, 7 controllers, services
métier dédiés (calcul rentabilité, automatisation facturation,
génération de quittances PDF).

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

  static const _moduleType = 'immobilier';

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

```dart
class PaymentController {
  PaymentController(this._paymentRepo, this._contractRepo, this._receiptService);

  Future<Payment> recordRentPayment({
    required String tenantId,
    required String propertyId,
    required List<DateTime> monthsCovered,
    required double amount,
    required PaymentMethod method,
  }) async {
    final contract = await _contractRepo.findActive(tenantId, propertyId);
    final payment = Payment(
      contractId: contract.id,
      monthsCovered: monthsCovered,
      amount: amount,
      method: method,
      paidAt: DateTime.now(),
    );
    await _paymentRepo.create(payment);
    await _receiptService.generate(payment);
    return payment;
  }
}
```

### 3. Provider

```dart
final paymentControllerProvider = Provider<PaymentController>((ref) {
  final enterpriseId = ref.watch(activeEnterpriseProvider).value!.id;
  return PaymentController(
    ref.watch(paymentRepositoryProvider(enterpriseId)),
    ref.watch(contractRepositoryProvider(enterpriseId)),
    ref.watch(receiptServiceProvider),
  );
});
```

## ✅ Repositories en place (8)

`property` · `tenant` · `contract` · `payment` · `property_expense` ·
`maintenance` · `treasury` · `immobilier_settings`

## ✅ Controllers en place (7)

`property` · `tenant` · `contract` · `payment` · `expense` ·
`maintenance` · `immobilier_treasury`

## 💰 Workflow Encaissement (2 étapes)

L'assistant d'encaissement guide le manager :

1. **Étape 1 — Locataire** : sélection du locataire (recherche + filtre
   "à jour / en retard")
2. **Étape 2 — Maison** : sélection de la maison rattachée à ce locataire
3. **Sélection des mois à payer** : la liste des échéances impayées est
   affichée ; l'utilisateur coche les mois à régler
4. `PaymentController.recordRentPayment()` crée le `Payment`, marque les
   échéances comme soldées et déclenche `ReceiptService.generate()` pour
   la quittance PDF

## 🤖 Automatisation

`BillingAutomationService` tourne au démarrage et :

1. **Génère les échéances mensuelles** sur tous les contrats actifs au
   début de chaque mois
2. Met à jour le statut **"en retard"** des locataires dont une échéance
   dépasse une date butoir configurée
3. Recalcule l'agrégat **"à jour / en retard"** affiché sur l'annuaire

`PropertyCalculationService` calcule en parallèle :
- Revenus mensuels par propriété
- Taux d'occupation
- Rentabilité (revenus − dépenses rattachées)
- Bénéfice net par période

## 🧾 Quittances

`ReceiptService` génère un PDF de quittance avec :
- En-tête bailleur
- Locataire + propriété
- Mois couverts par le paiement
- Mode de paiement et référence
- Total

Le PDF est partagé via le quick-share natif Android (Drive, Bluetooth,
Gmail, Imprimer).

## 📝 Best Practices

1. **Jamais** d'accès direct aux repositories depuis l'UI — toujours via
   provider → controller
2. Tout encaissement **doit** passer par `PaymentController.recordRentPayment()`
   (pas de `paymentRepo.create()` direct) pour garantir la cohérence
   échéances ↔ paiements ↔ quittance
3. Les calculs de rentabilité restent dans les services
   `domain/services/`, pas dans les controllers
4. Soft delete uniquement (`deletedAt` / `deletedBy`)
5. Filtrer **toujours** par `enterpriseId`
6. Les contrats **immutables** une fois actifs (pas de modification du
   loyer en cours de bail — créer un avenant via un nouveau contrat)
7. Logger les opérations critiques avec `developer.log`
