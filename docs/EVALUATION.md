# Évaluation — jr-03-checkout · 20 points

| Critère | Points | Preuve attendue |
|---|---:|---|
| Contrat de tentative, montant serveur et frontière PSP locale | 4 | POST 202 PENDING, GET conforme, aucune donnée bancaire ni réseau PSP. |
| Rejeu et conflits d’intention | 4 | Même clé → même id ; méthode différente et seconde clé même commande → 409. |
| Atomicité dans une JVM | 3 | Test concurrent avec une seule tentative et un seul appel mock. |
| Interface des deux méthodes et statuts explicites | 3 | Choix accessibles ; PENDING jamais présenté comme réussite. |
| Reprise avec clé stable après réponse perdue | 4 | Test React vérifiant les deux headers et la référence retrouvée. |
| Revue et limites correctement expliquées | 2 | Journal distinguant 202, état courant, mémoire et absence de PSP réel. |
| **Total** | **20** | |
## Règles de restitution

Un fichier présent n’obtient pas à lui seul tous les points : montrer son comportement et pouvoir l’expliquer. Les cas négatifs sont aussi importants que le nominal. Le code rendu ne contient aucune vraie donnée bancaire ou identité PSP ; tous les scénarios restent locaux et fictifs.

Associer chaque preuve à la version remise. Le journal mentionne l’acquis repris du lab précédent et toute base fournie par l’encadrant. Une revue locale est recevable lorsqu’aucun dépôt distant n’est utilisé. Ne pas créer de fausse URL de commit, de pull request ou de rapport.

En cas de panne d’outillage documentée, l’encadrant distingue les critères vérifiés sur un poste de secours, les contrôles relus mais non exécutés et les preuves reportées. Un test navigateur non exécuté ne reçoit pas l’étiquette « E2E réussi ». Une reprise de preuve peut être organisée ; le résultat obtenu reste daté et attribué au bon environnement.

Pour valider la séance, le formateur dispose d’un ensemble fonctionnel sur les objectifs obligatoires, de tests pertinents, d’un diagnostic expliqué et de limites honnêtes. Une extension ne compense pas un montant client accepté, une duplication masquée ou un faux succès affiché.

## Questions d’oral

1. Expliquer la durée de vie de la clé d’une intention.
2. Montrer l’opération rendue atomique.
3. Distinguer réponse de création rejouée et état actuel de tentative.
