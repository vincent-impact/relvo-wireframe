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
│   ├── index.html               # Dashboard (page d'accueil)
│   ├── sujets.html              # Liste des sujets
│   ├── sujet.html               # Détail d'un sujet (SUB-0142)
│   ├── messages.html            # Conversations par contact
│   ├── contacts.html            # Liste des contacts
│   ├── contact.html             # Fiche contact
│   ├── domaines.html            # Gestion des domaines
│   ├── activite.html            # Journal d'activité global
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

1. **Pas de statut `to_qualify`.** Si Relvo ne comprend pas un message, il ne crée ni sujet ni contact. Le message reste "Sans sujet" dans la page Messages, accompagné d'un `triage_hint` (intention floue, prospection, sans action, expéditeur inconnu, trop court, autre) qui aide au tri humain.

2. **Sources des tâches.** Relvo ne peut proposer que des tâches déductibles du contenu du message. Les tâches métier (appeler un magasin, vérifier un stock) sont créées par l'utilisateur. La source (Relvo ou utilisateur) est toujours visible via un actor-pill (`✦ Relvo` ou `Moi`). Le champ technique `kind` (decision, reply, check…) **n'est pas affiché par défaut** — il sert aux automatisations et au filtrage futur.

3. **Brouillon de Relvo dans le composer.** Le brouillon n'est jamais affiché comme un message dans le fil de conversation. Il est directement dans la zone de rédaction, identifié comme « Suggestion de Relvo — modifiez librement avant d'envoyer », avec actions « Régénérer » / « Effacer ».

4. **Acquittement implicite des suggestions.** Aucun bouton « valider » à cliquer sur les suggestions de Relvo. Ouvrir la fiche d'un sujet vaut acquittement de toutes les suggestions présentes. Sur les listes (Dashboard, Sujets), le badge `✦ N tâches suggérées` / `✦ Réponse suggérée` / `✦ Résolution suggérée` disparaît dès la consultation. La logique repose sur `Subject.last_opened_at` (cf. `04-ia.md §8`).

5. **Conversations par contact, pas par canal.** Un même contact peut écrire par email et WhatsApp — tous ses messages sont dans une seule conversation. Chaque message porte son indicateur de canal **et** son badge de rattachement à un sujet (linked) ou « Sans sujet » (orphan). Le composer propose un sélecteur de canal présélectionné sur le dernier canal utilisé par le contact.

6. **Contacts liés à la création de sujet.** Un contact n'est créé que lorsqu'un sujet est créé. Un message « Sans sujet » d'un expéditeur inconnu n'engendre pas de contact — il est identifié par `sender_raw` (email ou téléphone), avec un avatar `?` gris dans l'UI.

7. **Sujets multi-contacts.** Un sujet peut avoir plusieurs contacts (`contact_ids: UUID[]`). Le composer affiche un sélecteur de destinataire quand il y a plusieurs contacts.

8. **Triptyque d'acteurs : Moi / Relvo / Externe.** Chaque événement est identifié par son acteur. Côté modèle ce sont les valeurs d'enum `user / ai / contact`. Côté UI on les affiche **Moi / Relvo / Externe** avec des badges colorés `M` (bleu), `R` (violet), `E` (ambre). Utilisé partout : journal de bord du sujet, page Activité, filtres.

9. **Pièces jointes — 3 niveaux d'analyse.** Label automatique (Haiku, à la réception, badge discret affiché à côté du nom), résumé au premier accès (Sonnet, caché et affiché en italique sous le titre dans un accordéon `<details>`), analyse approfondie à la demande explicite via bouton « Analyser avec Relvo pour plus de détails ».

10. **Vue d'ensemble long terme sur la page Activité.** En plus du fil chronologique d'événements, la page Activité présente une section « Vue d'ensemble » avec KPIs (sujets résolus, délai moyen, charge actuelle vs capacité, **% d'aide Relvo**), une **courbe d'évolution** sur 8 semaines, et les bénéfices Relvo (tâches suggérées, brouillons préparés, PJ étiquetées, temps économisé). Cf. `01-principes.md §10`.

11. **Priorité matérialisée par marque-page latéral.** Sur les cartes de sujets (Dashboard et Sujets), la priorité critique/haute est rendue par une **bordure gauche colorée de 4px** (rouge `--brand-accent` pour critique, ambre pour haute). Sur la fiche du sujet (header), la priorité est affichée comme un badge classique. Le statut reste toujours rendu par un badge `sb-*` à part.

12. **Sujets non lus mis en évidence.** Les cartes de sujets en statut `unread` ont un fond légèrement teinté (`#F4FAEC`) et un titre en gras, façon « email non lu ».

13. **« Relvo » dans l'UI, « IA » dans la doc technique.** Dans toute l'interface (textes, badges, libellés), on utilise « Relvo » comme nom de l'assistant. La doc `04-ia.md` continue de parler de « l'IA » pour les aspects techniques (modèles, coûts, prompts) et le modèle conserve `actor_type = ai`.

## Conventions CSS

- Toutes les pages partagent `css/styles.css`
- Variables CSS pour les couleurs, rayons, typographie
- Palette : navy (#0A1128), blue (#2B6FE0), red (#E63150), tons neutres
- Police : system font stack (-apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif)
- Approche wireframe pour le moment — pas de design final

## Navigation

La sidebar est identique sur toutes les pages. Les liens de navigation doivent pointer vers les bons fichiers HTML :

- Dashboard → `index.html`
- Sujets → `sujets.html`
- Messages → `messages.html`
- Contacts → `contacts.html`
- Domaines → `domaines.html`
- Activité → `activite.html`
- Paramètres → `parametres.html`

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
