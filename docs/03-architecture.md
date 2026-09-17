# Architecture

## Décidé

- Application mobile : Expo, React Native et TypeScript.
- Base de données : PostgreSQL.
- Accès aux données côté serveur : Drizzle.
- Logique du prévu à la demande et pointages append-only.
- Conception offline-first.

Ces choix privilégient le partage de la logique métier TypeScript et un modèle relationnel adapté aux liens entre foyers, personnes, activités et faits.

## À valider

La séparation proposée comprend une application mobile, une API, un domaine TypeScript indépendant de l'interface, une couche de données et un client d'API. Les futurs clients web et Watch ne doivent pas imposer de développement immédiat.

Le domaine porterait l'expansion du planning, l'application des exceptions, la lecture du journal et les agrégats mensuels. Les règles doivent être utilisables localement et côté serveur avec les mêmes résultats.

Les écritures métier passeraient par une API ou des fonctions serveur pour garantir autorisations et idempotence. La sécurité des tables servirait de défense complémentaire. Le contrat exact reste ouvert.

Supabase est une option pour l'authentification et PostgreSQL managé. Hono et Fastify sont des pistes pour l'API. Aucun fournisseur, framework serveur, hébergeur, gestionnaire de monorepo ou outil de déploiement n'est arrêté.

La sélection devra comparer la simplicité d'exploitation, la localisation des données, les sauvegardes, la compatibilité avec la synchronisation et le coût. Les capacités actuelles des solutions seront vérifiées au moment de l'arbitrage.

## Rejeté/repoussé

L'option iPhone entièrement native n'est pas retenue dans la synthèse finale. L'argument en sa faveur était la proximité avec une future Watch ; le choix TypeScript privilégie le partage du domaine mobile. Les détails d'intégration native restent à étudier.

Aucun squelette applicatif, dépendance, schéma SQL ou configuration d'infrastructure n'est créé à ce stade.
