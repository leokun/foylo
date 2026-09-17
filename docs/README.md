# Foylo : dossier de conception

Mis à jour le 17 septembre 2026. Phase actuelle : documentation et décisions, sans implémentation.

Ces documents reprennent la conversation « Applications de pointage familial » du 17 septembre 2026, notamment son brief complet, sa contre-analyse et sa synthèse finale. Les propositions antérieures sont conservées comme pistes lorsqu'elles n'ont pas été confirmées. Aucune nouvelle recherche technique, commerciale ou juridique n'a été effectuée pour cette transcription.

## Lecture

1. [Vision produit](00-product-vision.md)
2. [Périmètre V1](01-v1-scope.md)
3. [Modèle métier](02-domain-model.md)
4. [Architecture](03-architecture.md)
5. [Fonctionnement hors ligne et synchronisation](04-offline-sync.md)
6. [Authentification et permissions](05-auth-permissions.md)
7. [Notifications](06-notifications.md)
8. [Plateformes](07-platforms.md)
9. [Sécurité et confidentialité](08-security-privacy.md)
10. [Évolutions futures](09-future-scope.md)
11. [Registre des décisions](decisions/README.md)

## Statuts

- **Décidé** : choix retenu dans la synthèse finale ou confirmé explicitement depuis.
- **À valider** : proposition, détail de conception ou arbitrage encore ouvert.
- **Rejeté/repoussé** : option abandonnée ou différée, avec sa raison.

Les exemples et critères de validation précisent le plan ; ils ne constituent pas une validation supplémentaire du périmètre.

## Prochaines décisions, dans l'ordre

1. Confirmer le périmètre V1 et les parcours quotidiens indispensables.
2. Définir les règles du prévu/réel : repas non pointés, début automatique, corrections, doublons et changement d'une semaine type.
3. Figer le modèle de données conceptuel et la gestion du calendrier scolaire.
4. Choisir le backend, l'authentification, le stockage local et le mécanisme de synchronisation.
5. Confirmer les exigences de confidentialité, les notifications et les critères de recette.

Le passage au développement fera l'objet d'une demande distincte.
