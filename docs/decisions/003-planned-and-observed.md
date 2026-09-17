# 003 : séparation du prévu et du réel

Date : 17 septembre 2026. Statut : Décidé.

## Contexte et décision

Le produit doit conserver ce qui était prévu et ce qui a été déclaré, avec un historique exploitable pour les récapitulatifs.

Le prévu est calculé à la demande à partir des règles, calendriers et exceptions. Le réel est porté par un journal de pointages append-only. Une correction ajoute un nouveau fait au lieu de réécrire silencieusement le précédent.

## Alternatives et conséquences

La table d'occurrences préremplies et l'occurrence mutable comme source unique du réel sont abandonnées. Le calendrier devient une dépendance du calcul pour éviter de compter des activités lorsque l'établissement est fermé.

Il reste à définir les identités d'occurrences, la préservation du planning historique, les corrections concurrentes et la lecture des données incomplètes. L'absence de pointage ne prouve pas une présence.
