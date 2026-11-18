# Architecture et conception

## 1. Vue d'ensemble

Trois chaînes : **indexation** (automatique), **question-réponse** (à la demande),
**notification** (déclenchée par le repli).

| Élément | Choix | Justification |
|---|---|---|
| Orchestration | n8n — Hugging Face Spaces (cloud) ou Docker local | Contrainte du projet, exports JSON, 2 variantes documentées |
| LLM | Groq — `llama-3.3-70b-versatile`, température 0.2 | Tier gratuit, rapide (quelques secondes), clé stockée en credential |
| Embeddings | Cloud : API Gemini (`text-embedding-004`) · Local : Ollama `nomic-embed-text` | Gratuits ; Groq ne propose pas d'embeddings |
| Base vectorielle | Qdrant — Cloud free 1 Go ou Docker, collection `documents_entreprise` | Natif dans n8n, filtre par métadonnée `fileId` |
| Canal | Chat web natif n8n (public) | Zéro configuration, démonstration immédiate |
| Déclencheur d'indexation | Cloud : Google Drive Trigger · Local : Local File Trigger | critère « interrogeable sans intervention manuelle » |
| Notification | Telegram (bot @BotFather) | Gratuit, simple, remplaçable par e-mail SMTP |
| Coût total | 0 € | Tous les services offrent un tier gratuit suffisant |

## 2. Indexation (workflow 1)

Google Drive Trigger sur le dossier documents (variante cloud) ou Local File Trigger
`/data/documents` (variante locale) → lecture/téléchargement du fichier → extraction
(PDF via Extract From File, TXT/MD via opération texte) → métadonnée `fileId`
(nom du fichier) → découpage **600 caractères / chevauchement 80** → embeddings
(Gemini cloud / Ollama local) → Qdrant.

- Le découpage 600/80 garde un item de menu ou une question/réponse de la FAQ entier
  dans un même morceau tout en limitant le bruit.
- La métadonnée `fileId` permet de purger les anciens morceaux d'un document remplacé
  (suppression filtrée avant réinsertion) et garantit une base à jour.
- Test de mise à jour : modifier un prix dans `documents/menu.txt` → le nouveau prix
  est trouvé sans action manuelle.

## 3. Question-réponse (workflow 2)

Chat web → Agent IA (Groq) avec l'outil `recherche_documents` (Qdrant, top-k = 4) et
mémoire 10 échanges → réponse avec source citée.

### Prompt système

```
Tu es « Maki », l'assistant virtuel du restaurant Le Zébu Doré à Antananarivo.

Règles obligatoires :
1. Réponds uniquement avec les informations renvoyées par l'outil de recherche dans
   les documents. N'utilise jamais tes connaissances générales.
2. Cite la source à la fin de ta réponse, entre parenthèses : (source : nom du document).
3. Si l'information ne se trouve pas dans les documents, réponds uniquement : FALLBACK
4. N'invente jamais un prix, un horaire, un plat ou une condition.
5. Ton poli, chaleureux et concis : 2 à 4 phrases maximum.
6. Réponds en français. Si la question est en anglais ou en malgache, réponds dans
   cette langue quand c'est possible, en respectant les règles 1 à 4.
```

### Message de repli

Double barrière, à renforcer avec un seuil de similarité si nécessaire :

1. Le prompt force le modèle à répondre exactement `FALLBACK` hors documents.
2. Le nœud « Question sans réponse ? » détecte la chaîne FALLBACK dans `$json.output`
   et route vers la notification.

Réponse de repli affichée au client : « Je ne trouve pas cette information dans nos
documents. Notre équipe vous répondra rapidement. »

## 4. Notification (branche du workflow 2)

Si le repli est déclenché : message Telegram au propriétaire avec la question et
l'heure, pour compléter la FAQ. Alternative sans Telegram : e-mail SMTP ou ajout dans
un Google Sheet (double usage : tableau des questions fréquentes).

## 5. Couverture des critères de réussite

| Critère | Mécanisme |
|---|---|
| ≥ 80 % sur 20 questions | Grille `tests/questions-test.md` (16/20 minimum) |
| Repli sur les hors-sujets | Prompt (règle 3) + nœud IF + notification |
| Nouveau document interrogeable sans action | Local File Trigger → indexation automatique |
| Temps de réponse acceptable | Inférence Groq rapide + recherche vectorielle (top-k 4) |
| Aucun secret dans les exports | Clés uniquement dans les credentials n8n, jamais dans les nœuds |
| Indication de la source | Règle 2 du prompt (nom du document entre parenthèses) |
| Langues | Règle 6 : français, anglais, malgache |
