# Déploiement cloud 100 % gratuit (sans installation locale)

Variante « zéro téléchargement » : n8n hébergé sur Hugging Face Spaces, base
vectorielle Qdrant Cloud, embeddings via l'API Google Gemini (gratuite),
documents surveillés dans Google Drive. Le LLM reste Groq.

## Coûts et volumes

| Service | Coût | Quota gratuit suffisant pour la démo |
|---|---|---|
| Hugging Face Spaces (n8n) | 0 € | Oui (CPU gratuit, l'espace s'endort après ~48 h sans visite) |
| Qdrant Cloud free | 0 € | 1 Go de collection |
| Google AI Studio (embeddings) | 0 € | Quota généreux, centré sur du texte court |
| Groq | 0 € | Tier gratuit |
| Google Drive | 0 € | Compte Google standard |

## Étape 1 — Cluster Qdrant Cloud

1. Créer un compte sur https://cloud.qdrant.io puis un cluster gratuit.
2. Noter l'URL HTTPS du cluster (format `https://xxxx.eu-central.aws.cloud.qdrant.io`) et créer une **clé API**.

## Étape 2 — Clé Google Gemini (embeddings)

1. Sur https://aistudio.google.com/apikey → créer une clé API.
2. La clé ne sera saisie que dans les credentials n8n (jamais dans les workflows).

## Étape 3 — n8n sur Hugging Face Spaces

1. Créer un compte https://huggingface.co puis **New Space**.
2. Choisir le template **Docker**, SDK Docker, Space public.
3. Utiliser l'image officielle n8n : `n8nio/n8n:latest`, port applicatif **5678**.
4. Variables d'environnement du Space :
   - `N8N_ENCRYPTION_KEY` : une longue valeur aléatoire (rôle de secret).
   - `WEBHOOK_URL` : `https://<ton-space>.hf.space/`
   - `N8N_HOST` : `<ton-space>.hf.space`
   - `GENERIC_TIMEZONE` : `Indian/Antananarivo`
5. Attendre le build (quelques minutes), puis ouvrir l'URL du Space et créer le **compte propriétaire n8n** (e-mail + mot de passe solide : c'est la protection de l'instance).

## Étape 4 — Dossier Google Drive et documents

1. Dans ton Drive, créer un dossier `documents-zebu-dore`.
2. Y copier les 4 fichiers du dépôt : `documents/` (menu, horaires, conditions, FAQ) en **.txt** ou convertis en **.pdf**.
3. Noter l'**ID du dossier** (dans l'URL du dossier Drive).

## Étape 5 — Credentials dans n8n

Dans n8n (URL du Space) → Settings → Credentials, créer :

| Credential | Valeur |
|---|---|
| **Groq** | ta clé API Groq |
| **Google Gemini (PaLM) API** | clé AI Studio (étape 2) |
| **Qdrant API** | URL du cluster + clé API (étape 1) |
| **Google Drive OAuth2** | connexion Google, autoriser n8n |
| **Telegram bot** (optionnel) | token du bot @BotFather |

## Étape 6 — Import et configuration des workflows

1. Importer `workflows/1-indexation-documents-cloud.json` et `workflows/2-assistant-question-reponse-cloud.json`.
2. Workflow 1 : sélectionner le dossier Drive surveillé (étape 4), l'événement « any change », les credentials Drive / Qdrant / Gemini. Vérifier que l'URL Qdrant du nœud est bien celle du cluster.
3. Workflow 2 : sélectionner les credentials Groq / Qdrant / Gemini, coller l'URL du cluster Qdrant dans le nœud de recherche, et le `chatId` propriétaire dans le nœud Telegram.
4. Activer les deux workflows.
5. Tester : poser une question, puis modifies un prix dans le menu Drive et pose de nouveau la question → la réponse reflète le changement sans action manuelle.
6. Limite connue : la collection Qdrant est créée automatiquement au premier document (sinon, la créer manuellement depuis l'interface Qdrant Cloud).

## Limites de la version gratuite

- **HF Spaces** s'endort après ~48 h sans visite : un clic-relance avant les captures suffit ; les données n8n (workflows, credentials) sont conservées.
- Le chat public est accessible à l'URL `https://<ton-space>.hf.space/...` — prévois une session propriétaire protégée, et n'expose pas les credentials.
- Qdrant Cloud free : 1 Go — largement suffisant pour ces documents.
- L'instance n8n étant hébergée, modifie le dossier Drive uniquement via les documents (pas de montage local).

## Pour la capture ComeUp

URL de chat publique + captures du workflow n8n + schéma d'architecture
(`docs/architecture.md`). La variante locale (`docker-compose.yml`) reste disponible
et documentée dans le README si un jour tu veux tout héberger toi-même.
