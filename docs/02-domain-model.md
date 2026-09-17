# Modèle métier

## Décidé

Le prévu est calculé à partir de **ScheduleRule + Calendar + ScheduleException**. Les occurrences attendues sont calculées à la demande, sans table d'occurrences préremplies ni tâche de génération périodique.

Le réel est décrit par des **Punch append-only** : les pointages sont ajoutés à l'historique, sans réécriture silencieuse. Une correction ajoute un fait qui remplace logiquement un précédent. L'état observé est dérivé du journal.

## À valider : entités conceptuelles

| Concept | Responsabilité |
| --- | --- |
| Family | Foyer et périmètre de partage |
| User | Compte authentifié |
| FamilyMembership | Appartenance d'un compte au foyer, rôle et statut |
| Person | Personne représentée, avec ou sans compte |
| Place | Lieu enregistré, adresse et coordonnées éventuelles |
| Activity | Cantine, garderie, crèche, nounou ou autre activité |
| Calendar | Périodes d'ouverture, vacances, jours fériés et fermetures |
| ScheduleRule | Jours de semaine, période de validité, heures locales et calendrier |
| ScheduleException | Modification ponctuelle du prévu, avec motif éventuel |
| Assignment | Responsabilité de dépôt, récupération ou prise en charge |
| Punch | Fait déclaré, auteur, appareil et éventuelle correction |
| ExpectedOccurrence | Résultat calculé du prévu pour une date |
| ObservedOccurrence | Lecture des faits applicables à cette occurrence |

Une Person doit pouvoir exister sans User et recevoir ultérieurement un compte sans perdre son historique.

Pour Punch, les champs envisagés sont l'identifiant, le foyer, la personne, l'activité, le type START/END/PRESENT/ABSENT, l'heure déclarée effectiveAt, l'heure de saisie recordedAt, l'auteur recordedBy, l'appareil deviceId et une référence supersedesId pour les corrections. Ce n'est pas encore un schéma définitif.

## À valider : règles structurantes

- Distinguer l'annulation anticipée du prévu, portée par une exception, de l'absence déclarée après coup, portée par un pointage.
- Définir l'identité d'une occurrence. Le couple règle/date locale est proposé ; plusieurs créneaux dans une journée et les activités ponctuelles doivent être couverts.
- Décider comment une modification de règle ou de calendrier conserve la lecture du passé : périodes de validité, versions ou autre stratégie explicite.
- Définir les priorités entre vacances, jours fériés, fermetures et exceptions individuelles.
- Choisir la source des calendriers scolaires français, le rattachement à une zone, la fréquence de mise à jour et le traitement des ponts propres à un établissement.
- Fixer le traitement des débuts automatiques, des repas présumés et des données incomplètes. Une heure prévue n'est pas automatiquement une heure constatée.
- Définir les corrections concurrentes, les doublons sémantiques et les séquences incohérentes de début/fin.
- Distinguer l'heure de saisie sur l'appareil et l'heure de réception serveur, notamment hors ligne.

Pour le temps, la proposition est de conserver les faits comme instants, les règles en heures locales avec un fuseau de référence, et les regroupements mensuels sur les dates locales. Les passages de minuit et changements d'heure restent des cas à définir.

## Rejeté/repoussé

- Occurrence mutable comme unique source du réel : remplacée par le journal de pointages.
- Matérialisation de plusieurs mois d'occurrences attendues : remplacée par le calcul à la demande.
- RRULE complet : une récurrence hebdomadaire avec période de validité est proposée pour commencer.
- PickupRun/PickupStop en V1 : une liste ordonnée de responsabilités peut être dérivée avant d'introduire une entité tournée.
