# Assistant IA connecté à vos documents (RAG) — n8n

Chatbot qui répond aux questions des clients **uniquement à partir des documents de
l'entreprise** (menu, horaires, conditions, FAQ), avec source citée, message de repli
en cas d'information absente, et **indexation automatique** dès qu'un document est
ajouté ou modifié.

Projet de démo : restaurant fictif « Le Zébu Doré » (Antananarivo).

## Architecture

```
[Dossier /documents]                    [Chat web n8n]
      │ (détection auto : ajout/modif)         │ question
      ▼                                        ▼
Workflow 1 : Indexation                Workflow 2 : Question-réponse
lire → extraire → découper             Agent IA (Groq) + recherche
(600 car., chevauchement 80)           vectorielle Qdrant (4 passages)
      │                                        │
      ▼                                        ├─ réponse + source ──► client
[Ollama : nomic-embed-text]                    └─ FALLBACK / hors sujet ──► Telegram propriétaire
      │
      ▼
[Qdrant : collection documents_entreprise]
```

Détails complets : voir `docs/architecture.md` (prompt système, découpage, gestion des erreurs).

## Installation

Prérequis : Docker et Docker Compose, une clé API Groq (gratuite), un bot Telegram
(@BotFather) pour la notification propriétaire.

```bash
cp .env.example .env        # puis renseigner N8N_ENCRYPTION_KEY
docker compose up -d
docker exec -it <conteneur_ollama> ollama pull nomic-embed-text
```

Dans n8n (http://localhost:5678) :

1. Créer la credential **Groq** (clé API) — la clé reste dans les credentials, jamais dans les workflows.
2. Importer `workflows/1-indexation-documents.json` puis `workflows/2-assistant-question-reponse.json`.
3. Dans chaque nœud Qdrant : URL `http://host.docker.internal:6333`, collection `documents_entreprise`.
4. Dans chaque nœud Ollama : URL `http://host.docker.internal:11434`, modèle `nomic-embed-text`.
5. Dans le workflow 2 : sélectionner la credential Groq ; renseigner le `chatId` du propriétaire dans le nœud Telegram (avec la credential du bot).
6. Copier les fichiers de `documents/` vers le dossier monté sur `/data/documents` → la première indexation se déclenche (sinon : exécuter le workflow 1 manuellement).
7. Activer les deux workflows, puis ouvrir le chat n8n (« Open chat »).

## Utilisation

- Poser une question en langage naturel : « Quels sont vos horaires le mardi ? »
- La réponse cite sa source : (source : horaires.txt).
- Question hors des documents → message de repli, sans invention, et alerte Telegram au propriétaire.
- Ajouter ou modifier un fichier dans `/data/documents` → réindexation automatique en quelques secondes.

## Tests

Passer le jeu de `tests/questions-test.md` (20 questions dont 5 hors sujet) et viser
au moins 16/20. Tester aussi : ajout d'un nouveau document, modification d'un prix,
question vague, question bilingue, document vide.

## Limites connues

- Un document remplacé ajoute de nouveaux morceaux : brancher une suppression filtrée
  Qdrant (filtre sur la métadonnée `fileId`) avant l'insertion pour purger l'ancienne version.
- Les réponses dépendent de la qualité des documents : un document non à jour donnera
  une réponse fausse « avec confiance ».
- Le repli s'appuie sur le mot-clé FALLBACK du modèle : si des réponses hors documents
  passent, ajouter un seuil de similarité côté recherche vectorielle.
- Le chat web n8n est un canal de démonstration ; pour Telegram ou WhatsApp en canal
  client, remplacer le nœud « Chat web » par le trigger correspondant.
- Données 100 % fictives, prévues pour la démo — ne pas utiliser de vraies données clients.
