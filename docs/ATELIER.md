# Lab 03 — Un checkout simulé et idempotent

**4 h 30 · Java et React.** Une personne double-clique, perd sa connexion ou relance une action dont elle n’a pas vu la réponse. Le code doit pouvoir retrouver la même intention au lieu de fabriquer une nouvelle tentative. Ici, aucun argent ne circule ; le risque étudié est la cohérence entre une action utilisateur et sa trace serveur.

## Contrat de la tentative

La seule requête de création est `POST /api/orders/{id}/payments`, avec `Content-Type: application/json`, un en-tête `Idempotency-Key` et le corps `{ "method": "CARD_SIMULATED" }` ou `{ "method": "BANK_SIMULATED" }`. Les clés sont des chaînes de 8 à 80 caractères ASCII alphanumériques ou tiret ; un UUID convient. La clé ne contient aucune identité et ne joue pas le rôle d’un mot de passe.

| Situation | Résultat demandé |
|---|---|
| Commande existante, méthode connue, nouvelle clé valide | 202, nouvelle tentative `PENDING` |
| Même clé, même commande et même méthode | 202, même identifiant de tentative et même création logique |
| Même clé, commande ou méthode différente | 409, `IDEMPOTENCY_CONFLICT` |
| Nouvelle clé pour une commande qui a déjà une tentative | 409, `PAYMENT_ALREADY_EXISTS` |
| Clé absente/invalide, méthode inconnue ou montant ajouté | 400 |
| Commande ou paiement inconnu | 404 |
| `GET /api/payments/{id}` | 200, état serveur actuel |

Le cursus autorise **une seule tentative par commande**. Pour essayer un autre scénario, créer une nouvelle commande fictive. La valeur d’idempotence porte la réponse de création ; le GET fournit le statut actuel quand le paiement évolue plus tard. Le serveur ne recopie jamais le montant depuis le corps du checkout.

## 1. Décrire l’intention — 25 min

Compléter un exemple : « Pour cette commande, demander une tentative avec CARD_SIMULATED ». La signature logique de cette intention est le couple `(orderId, method)`. Écrire ce qui doit rester identique lors d’une reprise et ce qui rend une nouvelle demande incompatible avec la même clé.

Dessiner les états du domaine : `PENDING`, `SUCCEEDED`, `FAILED`. La création atteint uniquement PENDING. Les deux états terminaux serviront au lab suivant. Côté interface, distinguer l’envoi, une réponse réseau inconnue, une tentative reçue et une erreur contractuelle. « Connexion perdue » ne signifie ni SUCCEEDED ni FAILED.

## 2. Isoler un PSP fictif — 45 min

Dans Java, créer une interface telle que `PspGateway` qui reçoit des informations décidées par le serveur : référence de tentative, montant, devise et méthode. Son adaptateur `MockPspGateway` reste dans le processus local et ne reçoit ni carte ni token de fournisseur. Pour cette séance, il accepte une référence fictive et laisse le résultat PENDING. Un double de test peut compter ses appels.

Créer un service de paiement et ses objets de réponse. La commande est recherchée avant le calcul de l’intention ; le montant et la devise proviennent de l’objet stocké. Ajouter le POST de création et le GET de consultation. Ne retourner pas la Map interne ni une classe de l’adaptateur. Vérifier d’abord un cas nominal avec une commande créée au lab 01.

Le choix de méthode est une donnée métier de simulation, pas un scénario de résultat : BANK_SIMULATED n’est pas synonyme d’échec. La décision SUCCEEDED ou FAILED sera un événement signé de l’atelier suivant.

## 3. Protéger l’intention en une opération — 50 min

Maintenir un registre des clés et une association commande → tentative. Pour chaque clé, conserver l’intention et la réponse de création. Le service doit décider, enregistrer et appeler l’adaptateur dans un ordre cohérent qui empêche deux threads de créer deux tentatives. Dans cette application à une JVM et adaptateur immédiat en mémoire, une section synchronisée est une solution acceptable et visible. Une simple Map concurrente ne rend pas atomique une succession de tests puis écritures sur plusieurs objets.

Contrôler d’abord les entrées invalides, sans enregistrer une fausse réussite. Rechercher ensuite une clé existante : renvoyer sa création si l’intention correspond, sinon 409. Si la clé est nouvelle mais la commande a déjà une tentative, répondre 409. Pour une création, enregistrer une seule tentative et sa réponse. Expliquer ce que votre code garantit lorsqu’un double de test reçoit deux appels concurrents.

Créer un test avec deux travailleurs lancés ensemble, même clé et même intention. Exiger une seule référence de paiement et un seul appel de l’adaptateur. Ajouter les tests de conflit de méthode et de seconde clé sur la même commande. La garantie s’arrête au processus courant ; elle ne survit ni au redémarrage ni à plusieurs instances.

## 4. Construire un checkout sans champ bancaire — 65 min

Après la confirmation de commande, afficher un `fieldset` de deux choix intitulés « Carte simulée » et « Virement simulé », avec la mention visible « Démonstration · aucun paiement réel ». Aucun numéro, date d’expiration, code de sécurité ou IBAN n’est demandé.

Au clic de confirmation, générer une clé une fois pour cette intention, puis envoyer le POST. Bloquer une seconde soumission pendant l’envoi, mais traiter l’idempotence sur le serveur même si le bouton est protégé. Garder l’intention et sa clé dans l’état de la page tant que le résultat est incertain. Le lab 06 prolongera cette mémoire au rechargement.

Quand le POST répond 202, afficher « Tentative enregistrée — résultat en attente » ainsi que la référence retournée. Ne pas écrire « Paiement réussi ». Ajouter une action explicite de lecture de `GET /api/payments/{id}`. Pour ce lab, sans événement terminal, cette lecture reste PENDING. Les tests du frontend utilisent une réponse contrôlée pour vérifier également les variantes d’affichage terminal, sans prétendre que le vrai backend les produit déjà.

## 5. Reprendre une réponse perdue — 55 min

Dans un test Java, créer la tentative puis rejouer la même requête et comparer l’identifiant, le montant et le nombre d’appels au mock. Dans un test React, simuler ce scénario en deux temps : la première requête est considérée comme reçue côté serveur mais sa réponse n’arrive pas au client ; la personne active « Retrouver ma tentative », ce qui renvoie la même requête avec **la même clé**. Vérifier que l’écran affiche l’unique référence renvoyée.

Une reprise n’efface pas la méthode d’origine et ne génère pas automatiquement une autre clé. Si la personne tente de changer la méthode alors que l’état est incertain, expliquer qu’il faut d’abord retrouver la tentative. Prévoir un message spécifique pour un conflit 409 et ne pas conseiller « payer à nouveau ».

Rejouer en terminal, avec l’identifiant de commande réellement obtenu :

```bash
curl -i -X POST http://127.0.0.1:8080/api/orders/ORDER_ID_A_REMPLACER/payments   -H 'Content-Type: application/json'   -H 'Idempotency-Key: demo-key-001'   --data '{"method":"CARD_SIMULATED"}'
```

Exécuter deux fois la commande inchangée ; les deux réponses désignent la même tentative. Sur la même clé, remplacer la méthode et observer 409. Une fois l’essai terminé, utiliser une autre commande pour tout nouveau scénario.

## 6. Démontrer et défendre — 30 min

Exécuter les suites Java et React puis leurs builds. Montrer en revue une requête initiale, un rejeu identique, une intention modifiée et le test concurrent. Demander au relecteur de décrire ce qu’il croit confirmé lorsqu’il lit l’écran PENDING ; ajuster le texte si nécessaire.

Rendre le journal, les preuves de clé stable, le contrat, les rapports et un paragraphe sur les limites de la mémoire. L’extension hors temps est une réflexion sur stockage transactionnel et clé unique durable ; la séance ne prétend pas fournir cette garantie en production.

## Déroulé minuté — 270 minutes

| Plage | Activité | Durée |
|---|---|---:|
| 00:00–00:25 | Définir intention, réponse acceptée et états de paiement | 25 min |
| 00:25–01:10 | Créer l’interface PSP locale, la tentative et son contrat HTTP | 45 min |
| 01:10–02:00 | Rendre atomiques l’idempotence et l’unicité de tentative par commande | 50 min |
| 02:00–03:05 | Construire le choix de méthode fictive et la vue de suivi React | 65 min |
| 03:05–04:00 | Reprendre avec la même clé après une réponse perdue et tester le parcours | 55 min |
| 04:00–04:30 | Démontrer les cas de rejeu, de conflit et les limites | 30 min |

## Contrat commun du fil rouge

La boutique **PilotCrew Démo** ne vend rien. Elle présente le produit `demo-ticket`, libellé « Atelier fictif », à `unitAmountMinor: 2500`, devise `EUR`, `fictional: true`. Une quantité entière de **1 à 5** produit une commande dont le montant est calculé par le serveur. Dans cette simulation EUR seulement, 2500 unités mineures s’affichent 25,00 € fictifs. Aucun autre nombre de décimales ni aucune conversion de devise n’est traité.

| Objet | Champs publics attendus |
|---|---|
| Product | `id`, `label`, `unitAmountMinor`, `currency`, `fictional` |
| Order | `id`, `productId`, `quantity`, `amountMinor`, `currency`, `fictional` |
| Payment | `id`, `orderId`, `amountMinor`, `currency`, `method`, `status`, `fictional` |

Le corps de création de commande accepte uniquement `productId` et `quantity`. Le corps de création de paiement accepte uniquement `method`. Un montant, une devise ou un statut ajoutés par le client ne doivent jamais devenir une autorité : **ce contrat les refuse en 400**. Le code utilise les identifiants retournés, jamais un identifiant d’exemple comme s’il existait déjà.

| Route | Résultat contractuel |
|---|---|
| `GET /api/products` | 200, tableau de produits fictifs |
| `POST /api/orders` | 201, commande créée et en-tête Location |
| `GET /api/orders/{id}` | 200, commande ; 404 si référence absente |
| `POST /api/orders/{id}/payments` | À partir du lab 03 : 202, tentative PENDING ; Idempotency-Key obligatoire |
| `GET /api/payments/{id}` | À partir du lab 03 : 200, état serveur ; 404 si référence absente |
| `POST /api/webhooks/mock-psp` | À partir du lab 04 : réception d’un événement signé par le simulateur local |

Les erreurs structurées emploient `application/problem+json` et au moins `type`, `title`, `status`, `detail`, `code`. `code` est une extension de ce projet. Le texte est destiné à l’utilisateur ; aucune pile d’exception ne sort dans la réponse. Les codes HTTP du cours sont des choix de contrat précis ; ils ne décrivent pas toutes les API de paiement existantes.

Toutes les données métier sont en mémoire dans une JVM : un redémarrage les efface. Cette limite est visible dans la documentation et dans les scénarios. Le cours ne met en place ni identité client, ni base durable, ni gestion de stock, ni remboursement, ni autorisation financière réelle.
