# Authentification et permissions

## Décidé

Aucun fournisseur d'authentification ni matrice de permissions n'est encore arrêté.

## À valider

Le foyer serait le périmètre principal d'accès. Le modèle proposé passe par FamilyMembership entre User et Family, même si l'usage V1 se limite à une famille par compte et deux adultes. Cela évite de figer une appartenance unique directement dans User.

Une Person représentée dans le planning ne dispose pas nécessairement d'un compte. Un grand-parent pourrait être désigné comme responsable sans recevoir pour autant un accès à l'application.

Les rôles envisagés sont OWNER, ADULT et MEMBER. Leurs droits précis restent à définir : invitation, gestion du foyer, modification du planning, pointage, correction et suppression de données.

Les méthodes proposées sont Apple, éventuellement Google, et email avec lien de connexion. Invitations par lien ou QR code, expiration, usage unique, retrait d'accès et récupération du compte propriétaire restent à spécifier.

Les accès serveur devront vérifier l'appartenance active au foyer pour chaque opération. La gestion des données déjà présentes sur un appareil déconnecté après révocation doit être traitée séparément de la coupure d'accès serveur.

## Rejeté/repoussé

Multi-foyers, partage d'un enfant entre foyers et permissions fines pour intervenants sont proposés après la V1. Une table Grant a été suggérée dans la contre-analyse ; sa création immédiate n'est pas décidée.
