# CLAUDE.md

## Projet

Relvo est un assistant IA de pilotage des sollicitations professionnelles. Il transforme le flux désordonné de messages reçus par un dirigeant (e-mails, WhatsApp) en sujets métier structurés, avec tâches, journal de bord et aide à la décision.

Ce dépôt contient la maquette HTML/CSS statique de l'interface Relvo, destinée à valider l'UX avant le développement applicatif.

## Stack technique

- HTML et CSS purs (pas de framework JS)
- Hébergé sur Vercel (site statique)
- Pas de backend, pas de données dynamiques — toutes les données sont simulées en dur dans le HTML

## Structure du projet

```
relvo/
├── CLAUDE.md                    # Ce fichier — point d'entrée pour Claude Code
├── docs/                        # Documents de conception (source de vérité)
│   ├── 01-principes.md          # Principes structurants du produit
│   ├── 02-modele-donnees.md     # Entités et modèles de données
│   ├── 03-cas-usage.md          # Cas d'usage détaillés
│   └── 04-ia.md                 # Interventions de l'IA
├── public/                      # Fichiers statiques servis par Vercel
│   ├── index.html               # Accueil — brief Relvo + KPIs + calendrier semaine + sujets prioritaires
│   ├── sujets.html              # Liste des sujets
│   ├── sujet.html               # Détail d'un sujet
│   ├── planning.html            # Calendrier des tâches (vue mois)
│   ├── messages.html            # Conversations par contact + filtres « non lus » / « sans sujet »
│   ├── dossiers.html            # Dossiers (ex-domaines) — contient Sujets + Connaissances
│   ├── dossier.html             # Fiche d'un dossier (sujets + fichiers + notes)
│   ├── contacts.html            # Liste des contacts
│   ├── contact.html             # Fiche contact
│   ├── parametres.html          # Paramètres
│   └── css/
│       └── styles.css           # Feuille de style partagée
├── vercel.json
└── package.json
```

## Documents de conception

Avant de créer ou modifier un écran, lis toujours les documents de conception dans `docs/`. Ils sont la source de vérité pour toutes les décisions UX et fonctionnelles.

### Lecture obligatoire

- **docs/01-principes.md** — Comprendre l'architecture conceptuelle : le Subject est l'entité centrale, pas le message. La chaîne Message → Task → Action → LogEvent structure tout le produit.
- **docs/02-modele-donnees.md** — Connaître les entités, leurs champs et leurs relations. Toutes les données simulées dans les maquettes doivent être cohérentes avec ce modèle.
- **docs/03-cas-usage.md** — Comprendre les flux utilisateur. Chaque écran illustre un ou plusieurs cas d'usage.
- **docs/04-ia.md** — Comprendre ce que l'IA fait et ne fait pas. Les limites de l'IA sont aussi importantes que ses capacités.

### Points critiques à respecter

> **Note de lecture**. Cette liste capture les invariants produit et UX. Pour le **scope V1** (ce qui ship vs ce qui est V2), voir la section dédiée plus bas.

#### Modèle de données et acteurs

1. **Account tenant et type `Actor`.** Toutes les ressources métier (Subject, Task, Contact, Folder, Channel, Message, Attachment, Action, EventLog, KnowledgeDocument…) portent `account_id` pour le filtrage tenant. Plus de FK utilisateur sur les ressources. Le type partagé `Actor = enum(user, ai, contact, system)` typifie les attributs `source_actor`, `created_by_actor`, `completed_by_actor`, `executed_by_actor`, `updated_by_actor` et `actor` (sur EventLog). Cf. `02-modele-donnees.md §0` et §1.

2. **Triptyque d'acteurs : Moi / Relvo / Externe.** Chaque événement est identifié par son acteur. Côté modèle ce sont les valeurs d'enum `user / ai / contact`. Côté UI on les affiche **Moi / Relvo / Externe** avec des badges colorés `M` (bleu), `R` (violet), `E` (ambre). Utilisé partout : journal de bord du sujet, filtres.

3. **« Relvo » dans l'UI, « IA » dans la doc technique.** Dans toute l'interface (textes, badges, libellés), on utilise « Relvo » comme nom de l'assistant. La doc `04-ia.md` continue de parler de « l'IA » pour les aspects techniques (modèles, coûts, prompts) et le modèle conserve la valeur d'enum `Actor = ai`.

#### Sujets, tâches, messages, contacts

4. **Pas de statut `to_qualify`.** Si Relvo ne comprend pas un message, il ne crée ni sujet ni contact. Le message reste « Sans sujet » dans la page Messages, accompagné d'un `triage_hint` (intention floue, prospection, sans action, expéditeur inconnu, trop court, autre) qui aide au tri humain.

5. **Tâche rattachée au sujet, pas à un utilisateur.** Une tâche existe pour faire avancer le sujet, indépendamment de qui finira par l'exécuter. L'affectation à un utilisateur spécifique est V2. Cf. `01-principes.md §4` et `02-modele-donnees.md §9`.

6. **Sources des tâches.** Relvo ne peut proposer que des tâches déductibles du contenu disponible (message + plus tard documents de Connaissances). Les tâches métier (appeler un magasin, vérifier un stock) sont créées par l'utilisateur. La source est toujours visible via un actor-pill (`✦ Relvo` ou `Moi`). Le champ technique `kind` n'est pas affiché par défaut.

7. **Statuts UI fidèles au modèle.** L'UI affiche les 6 statuts du modèle (`new`, `to_do`, `waiting`, `unread`, `resolved`, `archived`) avec un badge coloré dédié par valeur. Une simplification binaire (Ouvert/Fermé) a été testée et abandonnée : par défaut quasi tous les sujets visibles sont « ouverts », un badge binaire n'apporte donc aucune information utile dans les listes — la nuance (à faire / en attente / non lu) est précisément ce qui aide à trier. Le statut `blocked` du modèle initial a été retiré : il n'incite pas à avancer et finit toujours par se traduire en attente externe (pièces, réponse fournisseur, info client) — ces cas sont représentés par `waiting`.

8. **Priorité UI binaire — drapeau « urgent ».** Côté UI, l'utilisateur ne voit qu'un drapeau **urgent** (rouge) levé uniquement quand `priority = critical` au modèle. Les niveaux `high / medium / low` ne sont **pas exposés** côté UI — ils restent un signal interne pour Relvo. Marque-page latéral 4 px rouge sur les cartes urgentes uniquement. **La rareté est le signal.** Si le drapeau apparaît partout il devient du bruit ; il doit rester l'exception (typiquement 1-2 sujets sur 24 ouverts). Cette règle est **inverse à celle des statuts** (point 7) : les statuts gagnent en valeur par leur nuance, la priorité gagne en valeur par sa rareté.

9. **Brouillon de Relvo dans le composer.** Le brouillon n'est jamais affiché comme un message dans le fil de conversation. Il est directement dans la zone de rédaction du Sujet, identifié comme « Suggestion de Relvo — modifiez librement avant d'envoyer », avec actions « Régénérer » / « Effacer ».

10. **Acquittement implicite des suggestions.** Aucun bouton « valider » sur les suggestions de Relvo. Ouvrir la fiche d'un sujet vaut acquittement. Le badge `✦ N tâches suggérées` / `✦ Réponse suggérée` / `✦ Résolution suggérée` disparaît dès la consultation. La logique repose sur `Subject.last_opened_at` (cf. `04-ia.md §8`).

11. **Conversations par contact, pas par canal.** Un même contact peut écrire par email et WhatsApp — tous ses messages sont dans une seule conversation. Chaque message porte son indicateur de canal **et** son badge de rattachement à un sujet ou « Sans sujet ». Le composer propose un sélecteur de canal présélectionné sur le dernier canal utilisé par le contact.

12. **Contacts liés à la création de sujet.** Un contact n'est créé que lorsqu'un sujet est créé. Un message « Sans sujet » d'un expéditeur inconnu n'engendre pas de contact — il est identifié par `sender_raw`, avec un avatar `?` gris dans l'UI.

13. **Sujets multi-contacts.** Un sujet peut avoir plusieurs contacts (`contact_ids: UUID[]`). Le composer affiche un sélecteur de destinataire quand il y a plusieurs contacts.

#### Dates et Planning

14. **Modèle de date d'une tâche.** Quatre champs nullable : `start_date`, `start_time`, `end_date`, `end_time`. La deadline vit dans `start_*` ; `end_*` exprime uniquement une durée (multi-jours, créneau horaire). « `end_date` seul » n'est pas une configuration valide. Cf. `02-modele-donnees.md §9`.

15. **Planning — deux surfaces calendaires.** Vue **semaine** intégrée à l'Accueil (widget compact dans le brief) et vue **mois** sur la page Planning dédiée. Code couleur par **Dossier**. Drag-and-drop pour replanifier sur les deux surfaces. Pile « Aucune date » accessible en marge. Cf. `01-principes.md §11`.

#### Dossiers et Connaissances

16. **`Folder` côté modèle, « Dossier(s) » côté UI.** Un Folder regroupe à la fois les Sujets (`Subject.folder_id`) et les Connaissances (`KnowledgeDocument.folder_id`) d'un périmètre métier. Page `dossiers.html` (liste) + `dossier.html` (fiche). Cf. `02-modele-donnees.md §2` et `01-principes.md §12`.

17. **Folder « Général » auto-créé.** À la création du compte, un Folder spécial nommé « Général » est auto-créé (`is_default = true`). Il porte les Connaissances transversales (organigramme, charte, ton) et sert de Folder de repli quand Relvo ne sait pas classer.

18. **Connaissances — deux natures de documents.** L'entité `KnowledgeDocument` distingue `kind = file` (PDF/image uploadé, non modifiable sauf suppression) et `kind = note` (texte Markdown rédigé dans l'app, modifiable par l'utilisateur). Les fichiers sont des **références**, les notes sont une **mémoire vivante**. Pas de page « Connaissances » séparée — tout vit dans la fiche d'un Dossier.

19. **Drag-and-drop PDF dans un Dossier.** Pour ajouter un fichier, l'utilisateur glisse un PDF directement dans la fiche du Dossier (Files API d'Anthropic côté backend pour le référencer ensuite via `anthropic_file_id`).

20. **Édition des notes en V1.** Seul l'utilisateur édite les notes. Relvo les consulte mais ne les modifie pas. La proposition de modifications par Relvo (avec acquittement utilisateur) est V2.

#### Chatbot Relvo

21. **Deux modes d'expression — brief et conversation.** L'**Accueil** est un brief structuré compilé par Relvo (KPIs + calendrier semaine + sujets prioritaires) — pas un chat. La **conversation** se fait dans le **drawer** accessible depuis toutes les pages. Icône **maison** pour l'entrée Accueil dans la nav (le robot est réservé au bouton flottant qui ouvre le drawer chatbot, pour bien différencier). Cf. `01-principes.md §13`.

22. **Drawer chatbot accessible partout.** Bouton flottant 🤖 en bas-droite sur **toutes les pages** (Accueil compris). Drawer latéral ~40 % de la largeur à l'ouverture. Pas de page dédiée pour le chat.

23. **Conversations chat éphémères en IndexedDB.** **Aucune entité serveur** `ChatConversation` en V1. Le dialogue vit côté client uniquement. Ce qui persiste = actions effectuées + résultats (`Task`, `Action`, `EventLog`, `KnowledgeDocument`…). Limitations V1 assumées : pas de cross-device, pas de debug support.

24. **Sessions implicites — seuil 5 min.** À l'ouverture du drawer : reprise de la conversation en cours si dernière activité < 5 min, nouvelle conversation sinon. Bouton « + Nouvelle conversation » toujours présent. Liste des N dernières conversations (titre auto-généré + horodatage relatif) accessible dans le drawer.

25. **Chatbot Action-Capable Day-One.** Architecture symétrique UI ↔ tools API : chaque opération a un tool correspondant. Toutes les actions chat-driven sont rendues comme des blocs visuels structurés dans le fil, annulables d'un clic dans une fenêtre de quelques minutes. `EventLog.metadata.source = "chat"` pour la traçabilité. Le brouillon de message atterrit dans le composer du sujet, **n'est jamais envoyé directement**.

26. **Page-aware par défaut.** URL courante + données contextuelles pertinentes transmises à chaque tour. Chip de contexte en haut du drawer (« Contexte : SUB-0142 — Sauce blanche »), bouton × pour basculer en discussion générale.

27. **Stack technique chatbot.** AI SDK Vercel + AI Gateway, tool calls natifs (pas MCP en V1), prompt caching sur system prompt + `KnowledgeDocument`, Files API d'Anthropic pour les PDFs, citations activées via flag. Choix Haiku/Sonnet selon complexité. Cf. `04-ia.md §11.8`.

28. **Empty state du chat.** À l'ouverture d'une nouvelle conversation, afficher 3-4 prompts d'exemple **contextuels à la page courante**, rendus en texte gris italique sans bulle de message, sous un label « Suggestions ». Cliquables pour pré-remplir le champ de saisie. **Ne pas** imiter une bulle de message — il doit être clair que ce ne sont pas des messages historiques.

#### Stratégie technique de contexte

29. **Pas de RAG vectorielle.** Long context + prompt caching pour les `KnowledgeDocument` (statique), tool calls pour les données métier dynamiques (sujets, messages, tâches). Les PDFs passent par la Files API d'Anthropic. Les citations sont activées côté API dès V1 (affichage UI minimal — un lien « Source » discret). Cf. `04-ia.md §10`.

## Conventions CSS

- Toutes les pages partagent `css/styles.css`
- Variables CSS pour les couleurs, rayons, typographie
- Palette : navy (#0A1128), blue (#2B6FE0), red (#E63150), tons neutres
- Police : system font stack (-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif)
- Approche wireframe pour le moment — pas de design final

## Navigation

La sidebar est identique sur toutes les pages. Les liens de navigation doivent pointer vers les bons fichiers HTML :

Ordre de la navigation V1 (par priorité d'usage) :

1. Accueil (🏠) → `index.html`
2. Sujets → `sujets.html`
3. Planning → `planning.html`
4. Messages → `messages.html`
5. Dossiers → `dossiers.html`
6. Contacts → `contacts.html`
7. Paramètres → `parametres.html`

L'icône de l'entrée « Accueil » est une **maison** classique. Le **robot 🤖** est réservé au bouton flottant qui ouvre le drawer chatbot, présent sur les 7 pages — chaque icône a un rôle visuel distinct (la maison signale le brief lu, le robot signale le dialogue actif).

La page active est identifiée par la classe `.active` sur le `nav-item` correspondant.

## Données simulées

Les maquettes utilisent des données fictives mais réalistes, inspirées du cas Tasty Crousty (chaîne de restauration). Les contacts, sujets et messages doivent rester cohérents entre les pages :

- **Karim Benali** (SoGood Distribution) — fournisseur, sujet SUB-0142 (sauce blanche)
- **Sophie Blanchard** — RH, sujet SUB-0148 (congé maternité)
- **ClimaPro Services** — juridique, sujet SUB-0082 (contrat climatisation)
- **Restaurant Le Palais** — business, sujet SUB-0131 (virement client)
- **PackPlus SARL** — fournisseur, sujet SUB-0103 (papier emballage)
- **FroidExpert SA** — production, sujet SUB-0117 (congélateur Narbonne)
- **J. Morel** — contact inconnu, message sans sujet
- **Marie Campos** — comptable, message sans sujet

## Commandes

```bash
# Développement local
npx serve public

# Déploiement
git push  # Vercel déploie automatiquement depuis main
```

## Scope V1

> Cette section consolide les décisions de cadrage prises lors de la phase de design. Elle est la **source de vérité** pour ce qui ship dans la première version vs ce qui est reporté.

### Public cible et posture produit

Les utilisateurs cibles sont issus de secteurs **food** et **bâtiment**. Ils ne sont **pas familiers** des SaaS bureautiques (Notion, Hubspot, Pipedrive…) mais **connaissent ChatGPT et Claude**. La promesse V1 doit donc être lisible immédiatement et minimaliste à la prise en main.

La posture produit assumée : **« l'UI sert à accéder à l'info, Relvo sert à agir »**. Les clients ont indiqué qu'ils feraient l'essentiel de leurs actions via le chatbot plutôt que via les écrans. Cette posture oriente toutes les décisions de scope.

### Pages V1 (7 entrées de navigation)

| # | Page | Rôle |
|---|---|---|
| 1 | **Accueil** (🤖) | Brief Relvo : bandeau KPIs + calendrier semaine + sujets prioritaires du jour. **Pas de chat ici** — pour dialoguer, on ouvre le drawer. |
| 2 | **Sujets** | Liste plate des Sujets + fiche détail. Statuts UI fidèles au modèle (7 valeurs) + drapeau urgent binaire (priority=critical uniquement) avec marque-page latéral rouge. |
| 3 | **Planning** | Calendrier vue mois + drag-and-drop. Tâches avec date riche. |
| 4 | **Messages** | Conversations par contact, filtres « non lus » et « sans sujet » mis en avant, URLs filtrables (`?filter=orphan`). |
| 5 | **Dossiers** | Folders contenant Sujets + Connaissances (PDFs + notes Markdown). |
| 6 | **Contacts** | Liste + fiche détail. |
| 7 | **Paramètres** | Compte, canaux connectés. |

Le **drawer chatbot 🤖** est accessible sur les 7 pages via un bouton flottant en bas-droite.

### Inclus en V1 (MUST)

- Refonte modèle : `Account` tenant, type partagé `Actor` avec suffixe `_actor`, suppression FK utilisateur sur les ressources
- Renommage `Domain` → `Folder`, Folder « Général » auto-créé
- Triptyque Moi / Relvo / Externe avec badges colorés
- Sujets multi-contacts, contacts liés à la création de sujet
- Brouillon Relvo dans le composer
- Acquittement implicite des suggestions
- Modèle de date riche sur Task (4 champs), extraction de date par Relvo
- Calendrier semaine sur l'Accueil + page Planning vue mois + drag-and-drop
- Dossiers contenant Sujets + Connaissances (fichiers PDF via Files API + notes Markdown)
- Statuts UI fidèles au modèle (7 valeurs) + drapeau **urgent** binaire (priority=critical uniquement) avec marque-page latéral rouge — la rareté du drapeau est son signal
- Chatbot drawer accessible partout, page-aware, sessions IndexedDB éphémères, palette tools quasi-complète (lecture + écriture + annulation), empty state avec suggestions contextuelles
- Citations activées côté API (UI minimale)
- Pièces jointes niveau 1 (étiquetage Haiku) uniquement

### Reporté en V2

- Page Activité standalone (KPIs vue d'ensemble long terme, courbe d'évolution 8 semaines, fil chronologique EventLog) — seuls les KPIs essentiels remontent sur le bandeau Accueil
- Pièces jointes niveau 2 (résumé Sonnet) et niveau 3 (analyse profonde à la demande)
- Édition de notes par Relvo (V1 = utilisateur seul édite)
- Cross-device chatbot (V1 = IndexedDB côté client)
- UI riche pour les citations (panneau latéral, surlignage)
- Scope `subject` pour les `KnowledgeDocument` (V1 = `folder_id` uniquement)
- Affectation d'une tâche à un utilisateur spécifique (V1 = un compte = un humain)

### Indicateurs d'arbitrage

Quand un nouveau besoin émerge en cours d'implémentation, le réflexe d'arbitrage est :

- Est-ce que ça **simplifie** l'usage pour un utilisateur food/bâtiment ? → MUST V1
- Est-ce que ça **renforce le chatbot** comme surface d'action principale ? → MUST V1
- Est-ce que ça apporte de la valeur mais peut attendre 2 mois ? → V2
- Est-ce que ça ressemble à une fonctionnalité de power user / analytics ? → V2 par défaut
