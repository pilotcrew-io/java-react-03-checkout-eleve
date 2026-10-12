# Mémos 03 — Une intention doit rester identifiable

## Trois références, trois sens

| Référence | Utilité dans le cours | Qui la produit ? |
|---|---|---|
| orderId | Retrouver la commande et son montant | Serveur |
| paymentId | Suivre la tentative fictive | Serveur |
| Idempotency-Key | Reconnaître la répétition d’une intention de création | Client, une fois par intention |

La clé n’est pas une identité utilisateur et ne donne aucun droit d’accès. Dans la démonstration locale sans comptes, toutes les données sont fictives. Ne pas déduire qu’une application publique pourrait consulter les paiements de n’importe qui avec une simple référence.

## Accepté n’est pas terminé

Le code HTTP décrit l’échange avec le service ; le statut du paiement décrit l’état métier. Dans le contrat du lab, 202 signifie qu’une tentative a été acceptée pour traitement. Il ne prouve pas son résultat final. L’écran lit le statut rendu par le serveur et conserve PENDING tant qu’aucun fait terminal n’est enregistré. [Sémantique HTTP](https://www.rfc-editor.org/rfc/rfc9110.html).

## L’idempotence est une règle observée

Dans notre exercice, répéter une requête portant la même clé et la même intention retrouve une création. La clé est associée à `(orderId, method)`. La réutiliser avec une intention différente est une erreur explicite. Cette règle est définie par notre API ; écrire le nom d’un en-tête ne la met pas en œuvre tout seul.

Le bouton désactivé évite une maladresse dans un écran donné. Un autre onglet, un client HTTP ou une reprise réseau peut encore envoyer une requête. Seul le service voit toutes les requêtes qui arrivent dans son processus. Une nouvelle clé ne résout pas une réponse perdue : elle change l’intention que le serveur est invité à reconnaître.

## Le piège de « vérifier puis écrire »

Deux threads peuvent lire « clé absente », créer chacun une tentative puis écraser la même entrée. Employer une collection concurrente empêche certaines corruptions de structure, mais ne rend pas toutes les décisions successives indivisibles. Le cours demande que vérification, choix de tentative et enregistrement forment une opération atomique à l’échelle de la JVM.

Cette solution est acceptable parce que le mock est local et immédiat. Un verrou tenu pendant un appel réseau réel poserait d’autres problèmes. Un système durable demanderait des contraintes de stockage, des transitions transactionnelles et une stratégie pour l’incertitude entre services ; ces éléments ne sont pas simulés par une simple Map.

## Un adaptateur rend une dépendance remplaçable

Le domaine demande une tentative à une interface. Le mock exécute ce contrat sans parler à un prestataire. Les tests peuvent compter ses appels et choisir une réponse. Cette frontière aide à raisonner sur le code ; elle ne prouve aucune compatibilité avec une API bancaire.

## Questions de compréhension

1. Pourquoi une intention modifiée avec la même clé répond-elle 409 ?
2. Qu’est-ce qui empêche deux threads de créer chacun leur Payment ?
3. Quelle différence faites-vous entre réessayer une requête et créer une nouvelle intention ?
4. Pourquoi faut-il consulter l’état actuel après un rejeu de la réponse de création ?
5. Que perd le registre d’idempotence quand le processus redémarre ?
