# Backlog V1.0 — MVP email-first

Backlog révisé avec hypothèses ajustées : **email d'abord** (WhatsApp en V1.x), **Beta livrable en ~40 jours calendaires**, **développement assisté Claude Code**, **paiement Stripe intégré**.

> **Hypothèses de chiffrage**.
> - Estimations en jours-développeur (j-d).
> - **Équipe Beta : 2 devs full-stack expérimentés**, à temps plein, sans blocage externe.
> - **Développement assisté Claude Code** : réduction moyenne ~40-50 % sur le code répétitif (CRUD, UI, glue code, tests). Réduction plus modérée (~20-30 %) sur les tâches à fort enjeu humain : calibration des prompts IA, intégrations OAuth tierces, QA manuelle.
> - Le `styles.css` partagé déjà construit dans la maquette est **réutilisé tel quel** dans le projet final. C'est un gain ferme (économie ~5-7 j de design system).
> - Les estimations ci-dessous sont des **moyennes**. Une fourchette ±20 % reste raisonnable.

---

## Phase 1 — Beta (~40 jours calendaires · ~75 j-d effectifs)

L'objectif de la Beta est d'avoir un produit **utilisable de bout en bout** par 2-5 utilisateurs pilotes, sur lequel on collecte du feedback réel avant de continuer.

**Périmètre fonctionnel volontairement réduit** :
- Connexion **uniquement à Google Workspace** (OAuth 2.0). IMAP générique et WhatsApp arrivent en V1.x.
- Pipeline Relvo : rattachement, création de sujet, tâches déductibles, brouillon de réponse, `triage_hint`, label PJ (Haiku) et résumé PJ (Sonnet). **Pas d'analyse profonde de niveau 3** en Beta.
- Frontend : Dashboard, Sujets (liste + détail), Messages, Contacts (liste + fiche), Activité (fil chronologique uniquement), Paramètres (Compte + Channels).
- **Stripe** : trial 14 jours, plan mensuel unique, gestion via Customer Portal.
- **Hors Beta** : page Domaines (gestion UI), Vue d'ensemble long terme, Notifications avancées, Équipe multi-user, responsive mobile, 2FA active, IMAP générique, WhatsApp.

### Lot B0 — Setup & démarrages externes (J1-J3)

À lancer dès le jour 1, certaines validations sont longues côté providers.

| # | Tâche | j-d |
|---|---|---|
| B0.1 | Choix de la stack (recommandation : Next.js + tRPC ou Hono + Prisma + PostgreSQL + BullMQ) | 0,5 |
| B0.2 | Init repo, environnements dev/staging/prod, secrets manager, CI basique | 1,5 |
| B0.3 | **Démarrage validation OAuth Google Workspace** (process externe long, à lancer day-1) | 0,5 |
| B0.4 | Compte Anthropic + clés, compte Stripe, sandbox Postmark/Resend | 0,5 |

**Sous-total — 3 j-d**

> ⚠️ La validation Google OAuth pour scopes sensibles (`gmail.readonly`, `gmail.send`) prend **4 à 6 semaines**. À démarrer day-1 et avancer en parallèle. La beta peut tourner sur des comptes de test pendant la validation.

### Lot B1 — Modèle de données & API socle (J3-J12)

| # | Tâche | j-d |
|---|---|---|
| B1.1 | Schéma Prisma : 11 entités + relations + enums + index | 1,5 |
| B1.2 | Migrations + seed (6 domaines par défaut) | 0,5 |
| B1.3 | Auth email/password, sessions, reset password | 1,5 |
| B1.4 | API Subject : list filtrée, get, update (status / priority / domain), archive | 1,5 |
| B1.5 | API Message : list par conversation, list "Sans sujet", affecter, ignorer, réaffecter | 1 |
| B1.6 | API Task : list, create, update, complete, delete | 1 |
| B1.7 | API Contact : list (auto/complete), get, update (passage `auto → complete`) | 1 |
| B1.8 | API Action (brouillon) : create, update, send | 1 |
| B1.9 | EventLog : hooks ORM automatiques + API de lecture filtrable | 1,5 |
| B1.10 | Logique cycle de vie (transitions automatiques de statut) | 1 |
| B1.11 | Tests d'intégration API (parcours critiques) | 1,5 |

**Sous-total — 13 j-d**

### Lot B2 — Intégration Email (Google Workspace) (J8-J16, partiellement en parallèle)

| # | Tâche | j-d |
|---|---|---|
| B2.1 | Connexion OAuth Google Workspace (refresh tokens, scopes) | 2 |
| B2.2 | Réception messages : Pub/Sub Gmail, parser MIME, pièces jointes, dédupe | 2 |
| B2.3 | Threading via `In-Reply-To` / `References` / `external_thread_id` | 1 |
| B2.4 | Envoi email via Gmail API + statuts (sent / failed) | 1 |
| B2.5 | Robustesse : retry, alerte channel en erreur | 1 |

**Sous-total — 7 j-d**

### Lot B3 — Pipeline Relvo (J10-J22, en parallèle)

| # | Tâche | j-d |
|---|---|---|
| B3.1 | Pipeline async + queue + idempotence + observabilité | 1,5 |
| B3.2 | Décision de rattachement (Cas A à D) | 2,5 |
| B3.3 | Génération sujet : titre, summary, domaine | 1,5 |
| B3.4 | Création contact `auto` à partir de signature email | 1 |
| B3.5 | Génération de tâches déductibles | 2,5 |
| B3.6 | `triage_hint` pour messages "Sans sujet" | 1 |
| B3.7 | Préparation brouillon de réponse (Action `send_message` avec `payload`) | 1,5 |
| B3.8 | Transitions de statut + `resolution_suggested_at` | 1 |
| B3.9 | PJ niveau 1 — label Haiku | 0,5 |
| B3.10 | PJ niveau 2 — résumé Sonnet caché | 1 |
| B3.11 | Système de prompts versionnés + golden tests minimaux | 1,5 |
| B3.12 | Mesure des coûts (tokens / utilisateur) | 0,5 |

**Sous-total — 16 j-d**

### Lot B4 — Frontend (J14-J32, en parallèle)

Le mockup HTML/CSS sert de référence directe — `styles.css` réutilisé tel quel.

| # | Tâche | j-d |
|---|---|---|
| B4.1 | Setup framework + import `styles.css` + composants atomiques (Badge, Button, Card, Avatar, Modal, Toggle, Table, Form) | 2,5 |
| B4.2 | Layout commun (Sidebar avec compteurs dynamiques, Topbar, routing) | 1 |
| B4.3 | Auth pages (sign-in, reset, setup wizard premier login) | 1,5 |
| B4.4 | Dashboard (KPIs, sujets prioritaires, pipeline, messages sans sujet) | 1,5 |
| B4.5 | Sujets — liste avec status tabs, filtres, marque-page priorité | 1,5 |
| B4.6 | Sujet — détail (header, synthèse, tâches, conversation, panneaux droits) | 3 |
| B4.7 | Composer avec brouillon Relvo (édition, régénération, envoi) | 1,5 |
| B4.8 | Messages — conversations par contact + thread + composer | 2 |
| B4.9 | Tri des messages "Sans sujet" (popover affectation, création contact) | 1,5 |
| B4.10 | Contacts — liste + fiche éditable | 1,5 |
| B4.11 | Activité — fil chronologique avec triptyque + filtres | 1,5 |
| B4.12 | Paramètres — Compte | 0,5 |
| B4.13 | Paramètres — Channels (connexion, reconnexion, domaines associés) | 1,5 |
| B4.14 | Polish minimal : empty states, loading states, error boundaries, toasts | 1,5 |

**Sous-total — 22,5 j-d**

### Lot B5 — Stripe & monétisation (J24-J28)

| # | Tâche | j-d |
|---|---|---|
| B5.1 | Stripe Checkout pour souscription (plan unique mensuel) | 1 |
| B5.2 | Customer Portal pour gestion (changer carte, télécharger factures, annuler) | 0,5 |
| B5.3 | Webhook Stripe (paiement réussi, échec, annulation) → mise à jour statut compte | 1 |
| B5.4 | Trial 14 jours gratuit + bandeau "il vous reste X jours" | 0,5 |
| B5.5 | Garde-fou : compte impayé → accès en lecture seule (pas de blocage hard pour ne pas perdre les données) | 0,5 |

**Sous-total — 3,5 j-d**

### Lot B6 — Sécurité, observabilité, lancement (J28-J40)

| # | Tâche | j-d |
|---|---|---|
| B6.1 | Chiffrement des secrets channels (tokens OAuth) au repos | 0,5 |
| B6.2 | Logging Sentry front + back, structured logging | 0,5 |
| B6.3 | Monitoring basique (uptime, latence API, queue lag, coûts IA) | 1 |
| B6.4 | Notifications in-app minimales (cloche topbar + drawer) | 1,5 |
| B6.5 | Tests E2E parcours critiques (Playwright) | 1,5 |
| B6.6 | QA manuelle des cas A → Q | 2 |
| B6.7 | Bug-bash + fixes | 2,5 |
| B6.8 | Doc utilisateur minimale (page d'aide intégrée, FAQ courte) | 1 |
| B6.9 | Migration prod, runbook, plan rollback | 0,5 |

**Sous-total — 11 j-d**

### Récapitulatif Beta

| Lot | Périmètre | j-d |
|---|---|---|
| B0 | Setup & démarrages externes | 3 |
| B1 | Modèle de données & API | 13 |
| B2 | Email (Google Workspace) | 7 |
| B3 | Pipeline Relvo | 16 |
| B4 | Frontend | 22,5 |
| B5 | Stripe | 3,5 |
| B6 | Sécurité, obs, lancement | 11 |
| | **Total brut Beta** | **~76 j-d** |
| | **+ overhead 15 %** (revues, gestion) | **~87 j-d** |

→ **Avec 2 devs en parallèle = ~44 jours calendaires** (proche de l'objectif 40 j si fluide, sinon 45-50 j en réalité).

> Pour viser strictement 40 j calendaires : **équipe de 3 devs** ou **réduction supplémentaire du périmètre Beta** (ex : retirer la fonction « Tri des messages Sans sujet » avec popover et le résumé PJ niveau 2).

---

## Phase 2 — Compléments V1.0 (~30 j-d effectifs)

À lancer après quelques semaines de Beta, une fois les retours pilotes intégrés. Ces lots peuvent partir en parallèle.

| # | Tâche | j-d |
|---|---|---|
| V1.1 | **WhatsApp Business Twilio** : connexion, réception (texte + médias), envoi, sandbox de dev | 4,5 |
| V1.2 | **IMAP/SMTP générique** (Outlook, OVH, autres) | 2 |
| V1.3 | **PJ niveau 3** : analyse approfondie à la demande explicite | 1 |
| V1.4 | **Page Domaines complète** : grid de cards, création/modification/désactivation, association de channels | 1,5 |
| V1.5 | **Vue d'ensemble long terme** (Activité) : KPIs avec tendances, courbe d'évolution SVG, bénéfices Relvo | 2,5 |
| V1.6 | **Paramètres > Notifications** complètes (9 toggles + email récap quotidien) | 1,5 |
| V1.7 | **Paramètres > Équipe** : multi-user, invitations, rôles | 3 |
| V1.8 | **Responsive mobile** : sidebar collapse, layout 1 col sur mobile, ajustements composer | 3 |
| V1.9 | **2FA active** (TOTP) | 1,5 |
| V1.10 | **RGPD complet** : export des données, droit à l'oubli (suppression complète) | 2,5 |
| V1.11 | **Audit log** d'accès et de modification | 1,5 |
| V1.12 | **Cochage auto tâches `reply`** robuste avec `completion_mode = message_match` | 1 |
| V1.13 | **Polish complet** + onboarding produit (tutoriel premier login, tooltips contextuels) | 3 |
| V1.14 | **Mise à jour temps réel** des compteurs et listes (SSE ou WebSocket) | 1,5 |

**Sous-total V1 compléments — 30 j-d** (~36 j-d avec overhead)

→ Avec 2 devs : **~3 semaines calendaires** supplémentaires.

---

## Récapitulatif global

| Phase | j-d brut | j-d avec overhead | Calendrier (équipe de 2) |
|---|---|---|---|
| **Beta email-first** | 76 | 87 | ~6 semaines (~40-44 j calendaires) |
| **Compléments V1.0** | 30 | 36 | ~3 semaines |
| **Total V1.0 complète** | 106 | 123 | ~9 semaines (~2 mois) |

À titre de comparaison : l'estimation initiale (sans Claude Code, sans découpage Beta, avec WhatsApp dès la V1) était à **~245 j-d**. La nouvelle approche réduit de **~50 %** le coût total tout en accélérant la mise sur le marché.

---

## Hors V1.0 — repoussés en V2

- **Apprentissage Relvo** (cf. doc 04-ia §9) : amélioration progressive des suggestions.
- **App mobile native** (push notifications mobiles incluses).
- **Reformulation IA** des messages utilisateur (cf. 04-ia §3.2).
- **Cochage auto post-réception** : tâche obsolète détectée par l'IA suite à un message entrant (cf. 04-ia §4.2).
- **Multi-organisation** par utilisateur.
- **Slack, SMS, Teams, Outlook 365 natif** (au-delà d'IMAP).
- **Recherche full-text** dans les messages.
- **Plans tarifaires multiples** (Starter / Pro / Enterprise) — V1 = un seul plan.

---

## Risques majeurs à surveiller

1. **OAuth Google validation : 4-6 semaines côté Google.** À démarrer day-1 (Lot B0.3). Pendant la validation, la beta tourne sur des comptes Google "test" (limite : 100 utilisateurs).
2. **Coût Relvo en prod** : sans cache disciplinés (`ai_*_at`), les coûts dérivent vite. Monitorer dès la première semaine de beta. Cible : < 10 €/mois/utilisateur en moyenne.
3. **Qualité de la décision de rattachement** : sous-rattacher (créer trop de sujets) est plus tolérable que sur-rattacher (perte de contexte). Valider sur un jeu de 30 messages annotés avant chaque déploiement (couvert par B3.11).
4. **Latence du pipeline IA** : prévoir un état "Relvo analyse…" sur les messages frais, sinon l'utilisateur a l'impression que rien ne se passe. Cible : pipeline complet < 8 secondes.
5. **Sandbox Postmark / Resend pour le développement** : utiliser un domaine de test pour ne pas spammer pendant le dev.
6. **Stripe webhooks** : sécuriser la vérification de signature, idempotence sur les events. Erreur fréquente.

---

## Plan de déploiement Beta proposé

**Semaine 1 (J1-J7)**
- Lots B0 + début B1 + début B3 (en parallèle)
- Lancement de la procédure OAuth Google
- Architecture validée, premières migrations

**Semaine 2 (J8-J14)**
- B1 finalisé, B2 démarré, B3 en route, B4 démarre (composants atomiques)
- Premier message reçu en dev (boucle complète email → DB)

**Semaine 3 (J15-J21)**
- B3 avance fort (rattachement + génération sujet + tâches)
- B4 prend de la vitesse (Dashboard + Sujets liste fonctionnels)
- Premier sujet créé automatiquement par Relvo en démo interne

**Semaine 4 (J22-J28)**
- B4 finalise les pages restantes (détail sujet, messages, contacts, activité)
- B5 (Stripe) en parallèle
- Démo interne du parcours complet

**Semaine 5 (J29-J35)**
- Polish, B6 (sécu, obs, notifications)
- Tests E2E
- Bug-bash interne

**Semaine 6 (J36-J42)**
- QA manuelle finale
- Migration en prod
- **Beta ouverte à 2-5 pilotes**

---

## Suggestions complémentaires

- **Pré-Beta interne (~7 j)** : avant d'ouvrir aux pilotes externes, faire 7 jours de "dogfooding" par l'équipe sur ses propres emails. Détecte beaucoup de bugs réels.
- **Métrique d'activation** : un pilote qui n'a pas eu au moins 5 sujets créés et 1 brouillon utilisé dans la 1ʳᵉ semaine = signal d'alerte produit.
- **Modèle de pricing** à valider avant la Beta : prix unique mensuel ou par utilisateur ? Mensuel uniquement ou avec réduction annuelle ? Sondage rapide auprès de prospects qualifiés.
