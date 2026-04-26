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

1. **Pas de statut `to_qualify`.** Si l'IA ne comprend pas un message, elle ne crée ni sujet ni contact. Le message reste "Sans sujet" dans la page Messages.

2. **Sources des tâches.** L'IA ne peut proposer que des tâches déductibles du contenu du message. Les tâches métier (appeler un magasin, vérifier un stock) sont créées par l'utilisateur. La source (IA ou utilisateur) est toujours visible.

3. **Brouillon IA dans le composer.** Le brouillon n'est jamais affiché comme un message dans le fil de conversation. Il est directement dans la zone de rédaction, identifié comme "Suggestion IA".

4. **Conversations par contact, pas par canal.** Un même contact peut écrire par email et WhatsApp — tous ses messages sont dans une seule conversation. Chaque message porte son indicateur de canal. Le composer propose un sélecteur de canal.

5. **Contacts liés à la création de sujet.** Un contact n'est créé que lorsqu'un sujet est créé. Un message "Sans sujet" d'un expéditeur inconnu n'engendre pas de contact.

6. **Sujets multi-contacts.** Un sujet peut avoir plusieurs contacts (`contact_ids: UUID[]`). Le composer affiche un sélecteur de destinataire quand il y a plusieurs contacts.

7. **Triptyque d'acteurs dans l'activité.** Chaque événement est identifié par son acteur : Moi (utilisateur), IA, Externe (contacts). Ce triptyque est utilisé partout : journal de bord du sujet, page Activité, filtres.

8. **Pièces jointes — 3 niveaux d'analyse.** Label automatique (Haiku, à la réception), résumé au premier accès (Sonnet, caché), analyse approfondie à la demande (Sonnet, explicite).

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
