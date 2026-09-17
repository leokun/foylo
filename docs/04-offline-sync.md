# Fonctionnement hors ligne et synchronisation

## Décidé

Le pointage doit fonctionner sans réseau. L'action est enregistrée localement et visible immédiatement, puis synchronisée lorsque la connexion revient. Les pointages sont des faits ajoutés au journal.

## À valider

Le flux proposé est : saisie, stockage local durable, affichage, file d'envoi, validation serveur, puis réception des changements sur les autres appareils.

Les identifiants générés sur le client, les clés d'idempotence et un curseur de récupération incrémentale sont proposés. La répétition d'un envoi ne doit pas créer de nouveau fait.

L'interface devra distinguer les données enregistrées sur l'appareil, synchronisées et refusées. Les modalités de reprise après fermeture de l'application, d'erreur et de changement de compte sont à préciser.

| Situation | Arbitrage nécessaire |
| --- | --- |
| Même opération renvoyée | Déduplication par identité de mutation |
| Deux adultes pointent la même récupération | Conserver les deux faits et décider de la lecture métier |
| Deux corrections du même pointage | Définir la correction applicable et le signalement |
| Modification concurrente d'une règle | Choisir versionnement, résolution ou signalement |
| Accès révoqué avant l'envoi | Refuser l'écriture serveur et définir le devenir de la saisie locale |

Le journal append-only préserve les faits, mais ne résout pas à lui seul leur interprétation. « Le plus tôt gagne » ou un seuil d'écart sont des propositions non validées. La dernière écriture par champ pour règles et exceptions reste également à évaluer.

SQLite, PowerSync, ElectricSQL ou une synchronisation dédiée sont des pistes. Aucun choix de bibliothèque locale ou de moteur de synchronisation n'est arrêté.

## Rejeté/repoussé

- Attendre le réseau pour confirmer un pointage local : incompatible avec le besoin.
- Confondre absence de doublon technique et absence de double déclaration métier.
- Concevoir dès la V1 une synchronisation autonome complète pour la Watch.
