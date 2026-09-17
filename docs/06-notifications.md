# Notifications

## Décidé

Le niveau de notifications inclus en V1 n'est pas encore figé.

## À valider

Deux catégories sont proposées :

- Notifications locales pour les informations déjà connues : événement imminent, récupération et rappel de pointage.
- Notifications serveur pour les changements ou faits enregistrés par un autre adulte.

Exemple de comportement attendu à préciser : lorsqu'une récupération est enregistrée puis synchronisée, le second téléphone actualise la journée et supprime le rappel devenu inutile.

Un push silencieux a été envisagé pour faciliter la synchronisation. La reprise effective doit également être définie à l'ouverture de l'application et au retour du réseau ; les garanties de livraison et d'exécution seront vérifiées lors du choix technique.

Expo Notifications et Expo Push sont des candidats, sans sélection définitive. Les préférences, délais, horaires de silence et comportements après modification du planning restent à décider.

La contre-analyse propose des messages externes génériques, sans prénom, lieu ni horaire sensible. Cela remplace la piste initiale de notifications détaillant la récupération d'un enfant, sous réserve de validation du cadrage confidentialité.

## Rejeté/repoussé

Les notifications riches et actions avancées sont proposées après la V1. Une notification ne constitue pas la source de vérité du pointage.
