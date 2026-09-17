# 004 : fonctionnement hors ligne

Date : 17 septembre 2026. Statut : Décidé.

## Contexte et décision

Un adulte doit pouvoir déclarer une récupération devant l'école même avec une mauvaise connexion. L'enregistrement local précède la synchronisation et l'interface reflète immédiatement la saisie.

## Alternatives et conséquences

Une validation exclusivement en ligne ne répond pas au besoin. La synchronisation devra gérer les reprises, les envois répétés, les doubles déclarations et les opérations refusées.

Le journal append-only fournit l'historique nécessaire sans décider à lui seul de la résolution métier des conflits. Le stockage local, le moteur de synchronisation et les règles de résolution restent à choisir.
