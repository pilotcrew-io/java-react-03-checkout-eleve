# Dépannage — jr-03-checkout

Relever la première erreur, la commande, la version et le contexte. Une correction conserve le comportement demandé et ajoute ou rejoue une preuve ; retirer une assertion ne répare pas le problème.

| Symptôme | Cause plausible | Diagnostic discriminant | Résolution attendue |
|---|---|---|---|
| Le double-clic produit deux références | La clé est générée dans chaque appel ou le serveur n’est pas atomique | Comparer les clés Réseau puis le nombre d’appels du mock | Stabiliser l’intention côté React et verrouiller l’opération de création côté Java. |
| Le rejeu donne une autre méthode avec le même id | Le registre ne compare que la clé | Rejouer la clé avec BANK_SIMULATED après CARD_SIMULATED | Comparer toute l’intention et renvoyer IDEMPOTENCY_CONFLICT. |
| L’écran confirme une réussite après HTTP 202 | Il confond réussite de requête et statut métier | Lire status dans le JSON de la réponse | Afficher PENDING puis lire l’état serveur ; pas de succès optimiste. |
| Le test concurrent passe parfois seulement | Le démarrage des travailleurs ou leur attente est mal contrôlé | Vérifier la barrière de départ et les futures avec timeout | Rendre la concurrence reproductible et contrôler appels mock + références. |
| Un retry réseau crée une autre intention | La clé est régénérée dans la fonction de retry | Inspecter l’en-tête lors des deux appels du test | Conserver clé, commande et méthode de la demande incertaine. |
| Un nouveau scénario renvoie 409 | La commande possède déjà une tentative | Consulter la référence existante et lire le code applicatif | Créer une nouvelle commande fictive pour un nouveau scénario. |
| Le registre est vide après arrêt | L’idempotence est uniquement mémoire | Comparer processus, heure de démarrage et journal | Documenter la limite et démarrer une nouvelle commande de démonstration. |

Si un port est occupé, identifier le processus avant de l’arrêter. Ne jamais arrêter globalement toutes les JVM ou tous les processus Node d’un poste partagé. Si une commande dépend d’un fichier demandé dans l’atelier, le construire d’abord : les dépôts élèves sont des supports, pas des solutions prêtes à lancer.
