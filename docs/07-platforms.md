# Plateformes

## Décidé

Le socle mobile retenu est Expo / React Native / TypeScript. La logique métier doit pouvoir être partagée.

## À valider

La proposition de livraison commence sur iPhone. Les versions minimales, les appareils cibles et le mode de distribution restent à choisir.

Android demeure une possibilité future, cohérente avec le choix mobile. Une interface web pourrait faciliter la configuration des semaines types et la consultation de l'historique.

La Watch est envisagée comme une interface légère pour la prochaine mission et les actions rapides. La piste de conception est une application SwiftUI reliée à l'iPhone par WatchConnectivity. La faisabilité, les contraintes de distribution et le comportement téléphone absent restent à vérifier avant tout engagement.

Préserver une logique orientée actions simples dès la conception est proposé pour cette évolution, sans créer de client Watch ni figer des routes d'API aujourd'hui.

## Rejeté/repoussé

Android, web et Watch sont proposés hors V1 pour éviter de mener plusieurs clients de front. Widgets, complications et Live Activities ne sont pas inclus dans un engagement de livraison.
