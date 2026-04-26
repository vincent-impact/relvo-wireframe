# 2. Entités et modèles de données

## 1. Domain

### Propriétés

- `id: UUID`
- `name: string`
- `slug: string`
- `description: string nullable`
- `is_active: boolean`
- `created_at: datetime`
- `updated_at: datetime`

### Exemples

- RH
- Juridique
- Fournisseurs
- Support
- Business
- Production

### Rôle

Permet de catégoriser un sujet dans un périmètre métier principal. Le domaine est aussi associé aux contacts (via `default_domain_id`) pour faciliter le classement automatique des sujets.

## 2. User

### Propriétés

- `id: UUID`
- `email: string`
- `first_name: string`
- `last_name: string`
- `role: enum(admin, ceo, manager, operator, viewer)`
- `is_active: boolean`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Utilisateur qui consulte les sujets, crée des tâches, exécute des actions et intervient dans le traitement. Dans le journal de bord, ses actions sont identifiées par l'acteur **"Moi"**.

## 3. Contact

### Propriétés

- `id: UUID`
- `name: string`
- `email: string nullable`
- `phone: string nullable`
- `company: string nullable`
- `job_title: string nullable`
- `default_domain_id: UUID nullable`
- `status: enum(auto, complete)`
- `source: enum(ai, user)`
- `notes: text nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Représente l'émetteur ou le destinataire d'un message, et souvent le contact principal d'un sujet. Dans le journal de bord, ses actions sont identifiées par l'acteur **"Externe"**.

Un contact n'est créé que lorsqu'un sujet est créé (automatiquement par l'IA ou manuellement par l'utilisateur). La réception d'un message seule ne suffit pas à créer un contact. Cela évite de polluer la base avec des spammers et des démarcheurs.

Les contacts en statut `auto` apparaissent dans la page Contacts avec un indicateur "À compléter" pour inviter l'utilisateur à enrichir la fiche.

### **Définition des statuts**

- **auto** : contact créé automatiquement par l'IA à partir d'informations extraites du message (signature email, nom d'expéditeur, etc.). Les informations sont partielles et non vérifiées.
- **complete** : l'utilisateur a vérifié et complété la fiche contact.

### Point important — Contact et canaux

Un contact est une personne, indépendamment du canal par lequel elle communique. Un même contact peut avoir un email et un numéro WhatsApp. Les messages de tous les canaux sont regroupés dans une **seule conversation par contact** dans l'interface.

Le contact n'est pas dupliqué par canal. Les identifiants de canaux (email, téléphone) sont portés directement par les champs `email` et `phone` du contact.

## 4. Channel

Point d'entrée de communication connecté à la plateforme.

### Propriétés

- `id: UUID`
- `name: string`
- `type: enum(email, whatsapp)`
- `identifier: string`
- `domain_ids: UUID[]`
- `is_active: boolean`
- `created_at: datetime`
- `updated_at: datetime`

### Exemples

- Boîte RH → `rh@tastycrousty.fr`
- Boîte Support → `support@tastycrousty.fr`
- WhatsApp perso CEO → `+336...`
- WhatsApp réservé aux fournisseurs → `+337...`

### Rôle

Le channel représente un compte de communication côté utilisateur (une boîte mail, un numéro WhatsApp). Il aide à :

- identifier la source d'entrée du message
- orienter la classification du domaine
- distinguer les différents comptes connectés

### Remarque

Le channel est distinct du contact. Un channel est un point d'entrée côté utilisateur ("ma boîte Fournisseurs"). Un contact est une personne extérieure qui peut écrire sur n'importe lequel de ces channels.

## 5. ChannelConfig

Configuration technique du channel.

### Propriétés

- `id: UUID`
- `channel_id: UUID`
- `provider: string`
- `connection_data: jsonb`
- `status: enum(pending, connected, error, disabled)`
- `last_sync_at: datetime nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Exemples de `connection_data`

- email : host, port, login, secret, api_key
- WhatsApp : numéro, session, token, clé API

### Remarque

Cette entité porte les informations techniques de connexion, sans alourdir `Channel`.

## 6. Subject

Entité centrale du produit.

### Propriétés

- `id: UUID`
- `reference: string`
- `title: string`
- `summary: text nullable`
- `domain_id: UUID`
- `contact_ids: UUID[] default []`
- `status: enum(new, to_do, waiting, unread, blocked, resolved, archived)`
- `priority: enum(low, medium, high, critical)`
- `source_channel_id: UUID nullable`
- `opened_at: datetime`
- `resolved_at: datetime nullable`
- `last_activity_at: datetime nullable`
- `last_opened_at: datetime nullable`
- `resolution_suggested_at: datetime nullable`
- `created_by_user_id: UUID nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Conteneur métier principal.

Un sujet rassemble :

- les messages
- les pièces jointes
- les tâches
- les événements

### Remarques

`reference` est un identifiant lisible métier. Exemples : `SUB-00124`, `RH-0042`.

Si l'IA ne comprend pas un message (sens ambigu, contact inconnu, contexte insuffisant), elle ne crée pas de sujet. Le message reste "Sans sujet" (`subject_id = null` sur le Message) en attente d'une intervention humaine dans la page Messages.

Un sujet n'est créé que si l'IA a suffisamment compris la situation pour l'identifier. Il démarre alors en `new` (pas de tâche identifiée) ou directement en `to_do` (tâches identifiées).

Un sujet peut impliquer un ou plusieurs contacts. Le tableau `contact_ids` porte cette relation directement, sans table de liaison.

- Sujet mono-contact (cas courant) : `contact_ids = [UUID de Karim]`
- Sujet multi-contacts : `contact_ids = [UUID de Julien, UUID de Karim, UUID de Youssef]`
- Sujet sans contact (créé par l'utilisateur, pas encore de destinataire) : `contact_ids = []`

### Distinction `last_activity_at` / `last_opened_at`

Ces deux timestamps portent des informations différentes et ne doivent pas être confondus.

- `last_activity_at` : horodatage du dernier événement sur le sujet, quel qu'il soit (message reçu/envoyé, tâche créée/cochée, statut modifié, brouillon IA généré, etc.). C'est ce qui est affiché comme **"Dernière activité"** dans l'interface.
- `last_opened_at` : horodatage de la dernière fois où **l'utilisateur a ouvert la fiche du sujet**. Sert à distinguer les éléments IA déjà consultés des éléments à examiner.

### `resolution_suggested_at`

Renseigné par l'IA quand elle estime que le sujet est candidat à la résolution (cf. doc 04-ia §5.5). Si `resolution_suggested_at > last_opened_at`, l'interface affiche le badge **"Résolution suggérée"** dans les listes. Une fois le sujet ouvert, le badge disparaît des listes mais la suggestion reste visible dans la fiche jusqu'à ce que l'utilisateur résolve, archive ou que l'IA la révoque suite à une nouvelle activité.

### Cycle de vie des suggestions IA — vue modèle

Une **suggestion IA** est tout élément créé par l'IA qui appelle une décision de l'utilisateur :

- une `Task` avec `source = ai`
- un brouillon de réponse — `Action` de type `send_message` avec `status = open` et `payload` renseigné
- une suggestion de résolution — portée par `Subject.resolution_suggested_at`

Une suggestion est dite **"à examiner"** tant que sa date de création (ou de mise à jour pour la résolution suggérée) est postérieure au `Subject.last_opened_at`. Dès que l'utilisateur ouvre la fiche du sujet, `last_opened_at` est mis à jour, et toutes les suggestions présentes deviennent **"examinées"** (sans intervention explicite de l'utilisateur).

Les badges agrégés sur les listes (Dashboard, Sujets) ne comptent que les suggestions **"à examiner"** :

- `✦ N tâches suggérées` — `count(Task WHERE source='ai' AND status='open' AND created_at > Subject.last_opened_at)`
- `✦ Réponse suggérée` — il existe une `Action` send_message avec `status='open'`, `payload IS NOT NULL` et `created_at > Subject.last_opened_at`
- `✦ Résolution suggérée` — `resolution_suggested_at > last_opened_at`

Cf. doc 04-ia §8 pour le détail UX et les règles d'invalidation.

## 7. Message

Événement brut reçu ou envoyé.

### Propriétés

- `id: UUID`
- `subject_id: UUID nullable`
- `channel_id: UUID`
- `sender_contact_id: UUID nullable`
- `sender_raw: string nullable`
- `recipient_contact_id: UUID nullable`
- `direction: enum(incoming, outgoing)`
- `external_id: string nullable`
- `external_thread_id: string nullable`
- `subject_line: string nullable`
- `content: text nullable`
- `received_at: datetime nullable`
- `sent_at: datetime nullable`
- `status: enum(received, linked, sent, failed, ignored)`
- `triage_hint: enum(too_short, ambiguous, prospection, unknown_sender, informative_only, other) nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Remarques

- `subject_line` est surtout utile pour l'email.
- `external_thread_id` aide au rattachement d'un email à un fil existant.
- `subject_id` reste **nullable** : c'est le mécanisme qui porte les messages "Sans sujet". Un message avec `subject_id = null` est un message que l'IA n'a pas su traiter. Il est visible dans la page Messages et dans le dashboard, en attente d'affectation manuelle par l'utilisateur.
- Un message avec `status = ignored` est un message que l'utilisateur a volontairement écarté (spam, non pertinent) sans lui affecter de sujet.
- `sender_contact_id` est **nullable**. Un message peut exister sans contact associé : c'est le cas quand l'expéditeur est inconnu et qu'aucun sujet n'a encore été créé. L'information brute de l'expéditeur (adresse email ou numéro de téléphone) est conservée dans `sender_raw` pour permettre la création ultérieure du contact si l'utilisateur décide de traiter le message.
- `triage_hint` est renseigné **uniquement** quand `subject_id = null`, c'est-à-dire pour les messages que l'IA n'a pas su rattacher à un sujet. Il porte la raison synthétique de cette décision et aide l'utilisateur à trier rapidement (afficher dans la liste des messages "Sans sujet", choisir d'ignorer ou d'affecter). Il n'est pas affiché pour les messages rattachés à un sujet. Valeurs :
  - `too_short` — message trop court pour être exploitable ("Ok merci", "Bien reçu")
  - `ambiguous` — intention floue, sens non identifiable
  - `prospection` — démarchage commercial probable
  - `unknown_sender` — expéditeur inconnu sans contexte suffisant
  - `informative_only` — message compris mais purement informatif, sans accroche pour ouvrir un sujet
  - `other` — autre cas non couvert par les valeurs ci-dessus

### Affichage par conversation

Dans l'interface, les messages sont regroupés par `sender_contact_id` / `recipient_contact_id` pour former des **conversations par contact**, tous canaux confondus. Ce regroupement est un concept d'affichage, pas une entité de données distincte.

## 8. Attachment

Pièce jointe liée à un message.

### Propriétés

- `id: UUID`
- `message_id: UUID`
- `subject_id: UUID nullable`
- `name: string`
- `mime_type: string nullable`
- `file_url: string`
- `file_size: integer nullable`
- `ai_label: string nullable`
- `ai_summary: text nullable`
- `ai_analysis: text nullable`
- `ai_label_at: datetime nullable`
- `ai_summary_at: datetime nullable`
- `ai_analysis_at: datetime nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Permet de retrouver les documents :

- depuis le message
- depuis le sujet
- même après clôture
- `ai_label` — Étiquette courte attribuée automatiquement à la réception (facture, bon de livraison, contrat, planning, photo, justificatif, autre). Générée par Haiku à coût quasi nul. Affichée comme **badge discret** à côté du nom du fichier dans la fiche du sujet : utile quand le nom du document est peu explicite (ex: `scan_001.pdf` ou nom généré aléatoirement). Sert aussi de critère de filtrage et recherche.
- `ai_summary` — Résumé court du document, généré par Sonnet **une seule fois** au premier accès par l'utilisateur, puis caché. Exemples : "Facture SoGood Distribution — 1 240€ TTC — échéance 15 mai", "Contrat de maintenance — renouvellement tacite au 30 avril". Affiché dans le panneau pièces jointes du sujet.
- `ai_analysis` — Analyse approfondie du document, générée par Sonnet **uniquement à la demande explicite** de l'utilisateur (bouton "Analyser avec l'IA"). Contient l'extraction détaillée du contenu : clauses d'un contrat, lignes d'une facture, écarts identifiés, etc.
- `ai_label_at`, `ai_summary_at`, `ai_analysis_at` — Horodatages de chaque niveau d'analyse. Servent de flag de cache : si le champ datetime est renseigné, l'analyse a déjà été faite et le résultat est en cache. L'IA n'est jamais sollicitée deux fois pour le même document au même niveau.

## 9. Task

Unité de travail du sujet.

### Propriétés

- `id: UUID`
- `subject_id: UUID`
- `message_id: UUID nullable`
- `title: string`
- `description: text nullable`
- `source: enum(ai, user, system)`
- `kind: enum(decision, reply, check, call, inform, follow_up, other)`
- `status: enum(open, done, deleted)`
- `owner_user_id: UUID nullable`
- `completion_mode: enum(manual, message_match, action_match)`
- `due_at: datetime nullable`
- `completed_at: datetime nullable`
- `completed_by_user_id: UUID nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Représente une chose à faire dans le cadre du sujet.

### Exemples

- Confirmer ou refuser le remplacement sauce algérienne _(source: IA — déduit du message)_
- Appeler le shop de Montpellier _(source: utilisateur — savoir métier)_
- Vérifier les stocks de Béziers _(source: utilisateur — savoir métier)_
- Répondre au fournisseur _(source: IA — déduit du message)_

### Point important — Source des tâches

Le champ `source` est systématiquement affiché dans l'interface. Il permet de distinguer :

- **ai** : tâche proposée par l'IA, déductible du contenu du message. L'IA ne peut proposer que ce que le texte du message permet de déduire.
- **user** : tâche créée manuellement par l'utilisateur, souvent issue de son savoir métier ou de sa connaissance du terrain. L'IA n'a pas accès à ces informations.
- **system** : tâche créée automatiquement par le système (cas rares, workflows automatisés).

### Point important — Le champ `kind` n'est pas affiché par défaut

Le champ `kind` (`decision`, `reply`, `check`, `call`, `inform`, `follow_up`, `other`) est conservé dans le modèle pour deux usages techniques :

- **Automatisation `completion_mode = message_match`** : une tâche `kind = reply` peut être cochée automatiquement lorsqu'un message sortant est envoyé au même contact dans le même sujet.
- **Filtrage et statistiques** futures (ex : "voir mes tâches d'appel cette semaine").

En revanche, `kind` **n'est pas affiché** dans les cartes de tâches de l'interface principale. Les libellés (« décision », « réponse », « vérif »…) sont trop spécifiques pour apporter une lecture rapide et utile à l'utilisateur — la formulation du titre de la tâche suffit. Ce qui est affiché systématiquement, c'est :

- l'**actor-pill** (`✦ IA` ou `Moi`) qui matérialise la `source`
- une **action suggérée** contextuelle quand pertinent (« Aller au brouillon » pour les tâches de réponse, « Voir le message » pour les tâches qui pointent vers un message déclencheur, etc.)

## 10. Action

Exécution concrète déclenchée depuis l'interface.

### Propriétés

- `id: UUID`
- `subject_id: UUID`
- `task_id: UUID nullable`
- `message_id: UUID nullable`
- `type: enum(send_message, other)`
- `title: string`
- `payload: jsonb nullable`
- `status: enum(open, in_progress, done, cancelled, failed)`
- `owner_user_id: UUID nullable`
- `executed_at: datetime nullable`
- `created_at: datetime`
- `updated_at: datetime`

### Rôle

Trace une opération exécutée dans l'interface.

### Exemple

Task : "Répondre au fournisseur"
Action : "Envoyer le message de réponse"

### Remarque — Brouillon IA

Lorsque l'IA prépare un brouillon de réponse, celui-ci est stocké dans le `payload` de l'action (destinataire, canal, contenu). Le brouillon est présenté directement dans la zone de rédaction (composer) de l'interface, clairement identifié comme une suggestion modifiable. Il ne constitue pas un message tant qu'il n'a pas été envoyé.

## 11. EventLog

Journal de bord du système.

### Propriétés

- `id: UUID`
- `subject_id: UUID nullable`
- `message_id: UUID nullable`
- `task_id: UUID nullable`
- `action_id: UUID nullable`
- `entity_type: enum(subject, message, task, action, attachment, system)`
- `entity_id: UUID nullable`
- `event_type: string`
- `title: string`
- `description: text nullable`
- `actor_type: enum(user, ai, system, contact)`
- `actor_id: UUID nullable`
- `metadata: jsonb nullable`
- `created_at: datetime`

### Rôle

Historise tout ce qui doit apparaître dans la timeline du sujet ou dans la page Activité.

### Le triptyque d'acteurs

Le champ `actor_type` porte l'information "qui a agi". Il est structurant pour toute la plateforme :

- **user** → "Moi" — actions de l'utilisateur (tâche créée, tâche cochée, message envoyé, statut modifié)
- **ai** → "IA" — actions de l'intelligence artificielle (tâche proposée, sujet créé, brouillon préparé, domaine suggéré)
- **contact** → "Externe" — actions du monde extérieur (message reçu, pièce jointe reçue)
- **system** → événements techniques automatiques (changement de statut automatique, archivage)

Ce triptyque **Moi / IA / Externe** est utilisé dans l'interface pour filtrer l'activité et pour identifier visuellement l'acteur de chaque événement (badges colorés).

### Exemples d'event_type

- `message_incoming_received` (actor: contact)
- `message_outgoing_sent` (actor: user)
- `subject_created` (actor: ai)
- `subject_status_changed` (actor: ai ou user)
- `task_created_by_ai` (actor: ai)
- `task_created_by_user` (actor: user)
- `task_completed` (actor: user)
- `action_draft_prepared` (actor: ai)
- `action_send_message_done` (actor: user)
