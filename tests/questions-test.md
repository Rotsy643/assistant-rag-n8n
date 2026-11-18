# Jeu de 20 questions de test — Le Zébu Doré

Score de réussite visé : **au moins 16 / 20 (80 %)**.
Une question hors sujet est « correcte » si le message de repli (FALLBACK) est déclenché,
sans invention. Les questions 12, 14 et 15 sont volontairement reformulées ou bilingues.

## Questions sur les documents (15)

| # | Question | Réponse attendue (éléments clés) | Source attendue |
|---|----------|----------------------------------|-----------------|
| 1 | Quels sont vos horaires le mardi ? | 11h–15h et 18h30–22h | horaires.txt |
| 2 | Êtes-vous ouvert le lundi ? | Non, fermé le lundi | horaires.txt |
| 3 | Jusqu'à quelle heure êtes-vous ouvert le samedi ? | 23h | horaires.txt |
| 4 | Combien coûte le zébu sauce poivre vert ? | 38 000 Ar | menu.txt |
| 5 | Avez-vous des plats végétariens ? Lesquels ? | Curry de légumes 18 000 Ar, riz sauté 15 000 Ar, nems 12 000 Ar | menu.txt |
| 6 | Qu'est-ce que la formule déjeuner et à quel prix ? | Entrée + plat + boisson, 18 000 Ar, mar–ven 11h30–14h30 | menu.txt / horaires.txt |
| 7 | Que proposez-vous comme dessert ? | Gâteau de manioc 8 000 Ar, beignets de banane 7 000 Ar, sorbet 9 000 Ar | menu.txt |
| 8 | Puis-je réserver pour 20 personnes ? | Oui, sous 48h, menu groupe 35 000 Ar/pers | conditions-reservation.txt |
| 9 | Quels moyens de paiement acceptez-vous ? | Espèces, MVola, Orange Money, Visa/Mastercard ≥ 50 000 Ar | conditions-reservation.txt |
| 10 | Puis-je annuler ma réservation demain matin pour ce soir ? | Gratuite jusqu'à 24h avant ; au-delà, 20 % pour groupes de 10+ | conditions-reservation.txt |
| 11 | Faites-vous la livraison ? | Non, uniquement sur place ou à emporter | faq.txt |
| 12 | Est-ce que je peux venir avec mon enfant de 6 ans ? | Oui, chaise haute, menu enfant 12 000 Ar, coin lecture | faq.txt |
| 13 | Avez-vous du wifi et un parking ? | Wifi gratuit (code chez le serveur), parking 25 places gratuit derrière | faq.txt |
| 14 | Proposez-vous quelque chose pour les personnes allergiques ? | Allergènes signalés, chef adapte si possible ; options sans gluten | faq.txt |
| 15 | What time do you close on Saturday? (Amin'ny firy ny famaranana ny asabotsy ?) | 23h | horaires.txt |

## Questions hors sujet (5) — le repli doit être déclenché

| # | Question | Comportement attendu |
|---|----------|----------------------|
| 16 | Qui a gagné le dernier match de football ? | Message de repli, aucune invention |
| 17 | Quel temps fera-t-il demain ? | Message de repli |
| 18 | Raconte-moi une blague. | Message de repli |
| 19 | Quelle est la capitale de l'Espagne ? | Message de repli |
| 20 | Recommande-moi un film à ce soir. | Message de repli |

## Cas limites à tester en plus (phase 5)

- Question vague : « Vous êtes ouverts ? » (sans jour) → réponse + invite à préciser le jour.
- Question à deux sujets : « Prix du romazava et parking ? » → 30 000 Ar (menu) + parking gratuit (faq).
- Document vide ajouté dans le dossier → aucune erreur bloquante, aucune indexation corrompue.

## Grille de notation

Pour chaque question sur les documents : 1 point si tous les éléments clés sont présents,
0,5 si la réponse est partiellement correcte avec la bonne source, 0 sinon.
Les questions 16 à 20 : 1 point si le repli est déclenché sans invention, 0 sinon.
