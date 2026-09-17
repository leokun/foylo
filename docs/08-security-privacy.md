# Sécurité et confidentialité

## Décidé

Les choix détaillés de sécurité restent à valider. Le domaine contient des données sensibles pour les familles : personnes, lieux habituels, horaires et responsabilités.

## À valider : exigences proposées

- Pas de localisation en temps réel.
- Hébergement en Union européenne.
- Invitations à durée courte et révocation de l'accès serveur dès retrait d'un membre.
- Pas de prénom, lieu ou autre détail sensible dans les contenus push externes.
- Pas de SDK d'analytics tiers.
- Contrôle systématique de l'accès au foyer et traçabilité des corrections.

Ces propositions proviennent de la contre-analyse. Elles ne décrivent pas des protections déjà mises en place.

## Questions à résoudre avant implémentation

1. Quelles données de lieux et de personnes sont réellement nécessaires en V1 ?
2. Comment protéger la base locale, les sessions et les sauvegardes ?
3. Que conserve un appareil hors ligne après une révocation, et quand les données locales sont-elles supprimées ?
4. Quelle durée de conservation choisir pour les pointages, journaux et sauvegardes ?
5. Comment articuler l'historique append-only avec une demande de suppression ou de fermeture du foyer ?
6. Comment gérer le départ du propriétaire et le transfert de responsabilité du compte familial ?
7. Quelles données sont autorisées dans les journaux techniques et outils de diagnostic ?

## Rejeté/repoussé

Le partage inter-foyers et les accès limités de tiers sont proposés après la V1. Leurs implications ne doivent pas être considérées comme résolues par les seuls rôles OWNER/ADULT/MEMBER.
