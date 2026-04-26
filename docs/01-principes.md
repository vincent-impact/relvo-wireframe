# 1. Principes structurants

## 1. Le produit ne pilote pas des messages, il pilote des sujets

Le cœur de Relvo n'est pas la boîte mail ni WhatsApp.

Le cœur du produit est le **Subject**.

Un sujet est un espace de travail qui rassemble en un seul endroit :

- les **messages**
- les **pièces jointes**
- les **tâches**
- les **événements du journal de bord**

Autrement dit, un sujet représente une **situation métier en cours de traitement**.

Cette logique prolonge bien l'intention initiale du projet : transformer un flux désordonné de messages en dossiers clairs et suivis.

## 2. Le message est le point d'entrée du système

Un message entrant ou sortant est souvent l'élément déclencheur.

Quand un message est reçu, l'IA tente de le traiter :

- elle l'enregistre
- elle tente de le rattacher à un sujet existant ou de créer un nouveau sujet
- si elle y parvient, elle peut proposer des tâches en fonction du contenu

Si l'IA ne parvient pas à comprendre le message (contact inconnu, intention ambiguë, contexte insuffisant), le message reste **"Sans sujet"**. Il est visible dans la page Messages et dans le dashboard, en attente d'une intervention humaine : l'utilisateur peut alors l'affecter à un sujet existant, créer un nouveau sujet à partir de ce message, ou l'ignorer.

Le message n'est donc pas l'unité de pilotage, mais l'élément qui **alimente** le sujet.

## 3. La conversation regroupe les messages par contact

Les messages ne sont pas présentés individuellement, mais regroupés en **conversations par contact**, quel que soit le canal utilisé (email, WhatsApp, etc.).

Un même contact peut écrire par email le lundi et par WhatsApp le mardi : tous ses messages apparaissent dans un seul fil de conversation. Chaque message porte un indicateur de canal pour savoir par où il est passé.

Au sein d'une conversation, les messages peuvent traverser **plusieurs sujets**. Un échange avec un fournisseur peut passer d'un sujet de commande à un sujet de livraison, entrecoupé de messages informels sans rapport professionnel. Chaque message porte un **badge de rattachement** (le sujet auquel il est lié, ou "Sans sujet"), ce qui permet de naviguer vers le sujet correspondant.

Lors de la réponse, le canal est présélectionné sur le **dernier canal utilisé par le contact**, avec la possibilité de le changer via un sélecteur.

## 4. La tâche est l'unité de travail réelle

La logique du produit repose sur une idée simple :

> Un sujet avance parce que des tâches sont identifiées puis réalisées.

Une tâche peut être :

- proposée par l'IA, à partir du contenu d'un message
- créée manuellement par l'utilisateur, à partir de son savoir métier

Cette distinction est importante : l'IA ne peut proposer que des tâches déductibles du contenu du message (par exemple "Confirmer ou refuser le remplacement"). Les tâches qui relèvent de la connaissance du terrain (par exemple "Appeler le shop de Montpellier" ou "Vérifier les stocks de Béziers") ne peuvent venir que de l'utilisateur.

Une tâche sert à matérialiser :

- ce qu'il reste à faire
- ce qui a déjà été fait
- l'avancement réel du sujet

Dans l'interface, l'utilisateur manipule des **tâches**, qu'il peut conserver, supprimer, modifier ou cocher. La source de chaque tâche (IA ou utilisateur) est toujours visible.

## 5. L'IA aide à la décision et à l'exécution

### Aide à la décision

Relvo lit le message et propose des tâches pertinentes, dans la limite de ce que le contenu du message permet de déduire.

### Aide à l'exécution

Relvo peut préparer certaines actions concrètes, en particulier :

- une réponse préremplie (brouillon)
- avec destinataire, canal et contenu déjà préparés

Le brouillon IA est présenté directement dans la zone de rédaction du message, clairement identifié comme une suggestion modifiable. L'utilisateur peut l'éditer librement avant envoi, le régénérer, ou l'effacer pour écrire de zéro.

L'IA ne remplace pas l'utilisateur, elle :

- structure le travail
- prépare des exécutions
- réduit la charge mentale

## 6. L'action est une exécution concrète dans l'interface

Il faut bien distinguer :

### Task

Ce qu'il faut faire.

### Action

L'opération concrète exécutée dans l'outil.

En V1, l'action principale est surtout : **envoyer un message**.

Une tâche comme "Répondre au fournisseur" peut donner lieu à une action :

- ouverture du composer
- envoi effectif du message

L'action n'est pas la tâche. Elle est le **mécanisme d'exécution** de certaines tâches.

## 7. Tout ce qui se passe alimente un journal de bord

Le produit est vivant :

- des messages arrivent
- des tâches sont créées
- des tâches sont cochées
- des actions sont exécutées
- le sujet change d'état

Tout cela produit des **LogEvents**.

Chaque événement est identifié selon deux dimensions :

- **Le type d'événement** : message, tâche, action, changement de statut
- **L'acteur** : Moi (l'utilisateur), IA, ou Externe (le monde extérieur — contacts, fournisseurs, etc.)

Ce triptyque **Moi / IA / Externe** structure la lecture de l'activité dans toute la plateforme. Il permet de comprendre d'un coup d'œil qui agit dans le système et de filtrer l'activité par voix.

Le journal de bord permet :

- d'alimenter la timeline du sujet
- d'alimenter la page Activité dédiée
- de garder une trace claire de l'historique

## 8. La chaîne centrale du produit

L'épine dorsale du projet est la suivante :

> **Message → Task → Action → LogEvent**

### Message

Révèle une situation ou un besoin.

### Task

Formalise ce qu'il faut faire.

### Action

Permet d'exécuter concrètement une partie du travail.

### LogEvent

Trace ce qui s'est passé.

## 9. Cycle de vie d'un sujet

### 1. `new`

Le sujet est fraîchement créé.

- L'utilisateur crée lui-même un nouveau sujet.
- L'IA reçoit un nouveau message, crée un sujet, mais n'identifie aucune tâche à faire. Le sujet reste en `new` en attente que l'utilisateur le consulte.

Note : si l'IA ne parvient pas à comprendre un message (sens ambigu, contact inconnu, contexte insuffisant), elle ne crée pas de sujet. Le message reste "Sans sujet" dans la page Messages, en attente d'une intervention humaine.

### 2. `to_do`

Le sujet est compris, et il existe des choses à faire.

C'est le statut central du produit.

Le sujet passe en `to_do` lorsque :

- l'IA ou l'utilisateur a identifié une ou plusieurs tâches
- le dossier est actionnable
- il reste du travail à réaliser

Exemples :

- confirmer ou non un remplacement produit
- répondre à un fournisseur
- préparer une réponse officielle à une demande RH

Le sujet reste en `to_do` tant qu'il reste des tâches utiles à mener et qu'on n'est pas dans une logique d'attente dominante.

### 3. `waiting`

Le sujet est en attente d'un retour externe ou interne important.

Ce statut s'applique quand :

- une réponse a été envoyée
- une demande a été formulée
- l'avancement dépend désormais d'un tiers

Exemple : on a répondu au fournisseur, et on attend sa confirmation.

Même s'il reste encore quelques tâches secondaires ouvertes, le sujet peut passer en `waiting` si l'état dominant est l'attente.

### 4. `unread`

Le sujet en état `waiting` reçoit un nouveau message qui n'implique aucune nouvelle action (ce sont souvent des messages de validation ou de confirmation). Pour inciter l'utilisateur à fermer le sujet manuellement, on place le sujet en `unread` si aucune nouvelle action n'est suggérée.

### 5. `blocked`

Le sujet ne peut pas avancer.

Ce statut s'applique quand :

- il manque une information indispensable
- une dépendance est bloquée
- aucune action utile ne permet de progresser à court terme

Exemple : pas de stock alternatif, pas de réponse du fournisseur, pas de solution viable.

### 6. `resolved`

Le sujet est traité.

Cela signifie que :

- la situation a été gérée
- les principales tâches ont été faites
- il n'y a plus de travail significatif à mener
- le dossier est stabilisé

### 7. `archived`

Le sujet est clos et rangé.

C'est le statut final d'un sujet déjà résolu, que l'on conserve pour l'historique mais qui ne fait plus partie du flux actif.

### Lecture simple du cycle de vie

- **new** → un nouveau sujet naît
- **to_do** → il y a des tâches à faire
- **waiting** → on attend un retour
- **unread** → un message est arrivé, pas d'action requise
- **blocked** → on ne peut plus avancer
- **resolved** → c'est traité
- **archived** → c'est rangé

Et en amont du cycle : un message que l'IA n'a pas su traiter reste **"Sans sujet"** dans la page Messages, en attente de tri par l'utilisateur.
