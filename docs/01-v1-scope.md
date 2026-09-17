# Périmètre V1

## Décidé

La V1 et le schéma de données doivent être cadrés avant de commencer à coder. Le principe de fonctionnement hors ligne est retenu.

## À valider : périmètre proposé

| Domaine | Proposition V1 |
| --- | --- |
| Foyer | Une famille par compte dans l'usage V1, deux comptes adultes |
| Personnes | Enfants et adultes représentés indépendamment des comptes |
| Référentiel | Lieux et activités |
| Planning | Semaine type, calendriers scolaires, jours fériés, fermetures et exceptions |
| Responsabilités | Qui dépose ou récupère chaque personne |
| Aujourd'hui | Activités, horaires, lieux, responsables et prochaine action |
| Pointage | Début/fin, fin seule, présence/absence, ajustement rapide |
| Historique | Corrections traçables et séparation prévu/réel |
| Récapitulatif | Quantités et durées mensuelles par personne et activité |

Les actions rapides envisagées sont « Maintenant », « -5 minutes », « -10 minutes » et le choix d'une heure. Pour une garderie commençant à 16:30, une récupération à 17:48 peut être saisie seule. La provenance du début utilisé dans le calcul devra rester explicite.

La cantine doit pouvoir être prévue automatiquement sans demander une confirmation quotidienne. La qualification des repas sans pointage reste à décider : attendus ou présumés, mais pas présentés sans distinction comme des présences constatées.

## Scénarios proposés pour valider la V1

1. Configurer deux enfants avec des lieux et des semaines types différents.
2. Exclure automatiquement les périodes où l'école est fermée.
3. Changer le responsable un jeudi sans affecter les jeudis suivants.
4. Enregistrer une récupération hors ligne puis retrouver le fait sur le second téléphone.
5. Corriger 17:48 en 17:43 en conservant l'historique de la correction.
6. Comparer prévu et déclaré sur un mois, avec les données manquantes identifiables.

## Rejeté/repoussé

Tournées explicites, navigation, tarification, Android, web, Watch, multi-foyers, permissions externes sophistiquées et push riche sont proposés après la V1. Voir [les évolutions](09-future-scope.md).
