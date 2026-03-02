# Architecture MVP — Plateforme de financement participatif (Haïti)

## 1) Objectifs produit MVP
- Permettre la création, soumission, modération et publication de campagnes.
- Permettre des dons one-time sécurisés avec option anonyme + message.
- Gérer les retraits avec workflow de validation et traçabilité complète.
- Offrir un backoffice modération/admin orienté anti-fraude.

## 2) Propositions de stack

## Option A — Next.js + Node + Postgres + Prisma + Auth + Admin panel

### Stack proposée
- **Frontend**: Next.js (App Router), TypeScript, Tailwind CSS, i18n FR-first
- **Backend API**: Next.js Route Handlers (ou NestJS si séparation stricte), TypeScript
- **ORM/DB**: Prisma + PostgreSQL
- **Auth**: Auth.js (credentials email/password, reset password)
- **Paiement**: abstraction `PaymentProvider` + 1 provider initial (ex: MonCash/Stripe selon disponibilité locale)
- **Storage**: S3 compatible (Cloudflare R2, MinIO, AWS S3) + URLs signées
- **Backoffice**: module `/admin` avec RBAC (moderator/admin)
- **Infra**: Docker Compose (MVP), puis déploiement sur Railway/Fly.io/Render

### Avantages
- Vitesse d’exécution élevée (monorepo TypeScript full-stack).
- Réutilisation types front/back (zod schemas + types Prisma).
- Très bon DX pour équipe produit/UX/front.

### Risques
- Discipline d’architecture nécessaire pour ne pas mélanger couches métier/UI.
- Ecosystème auth/paiement à configurer proprement pour cas réglementaires locaux.

## Option B — Laravel + Postgres + Admin + S3

### Stack proposée
- **Frontend**: Blade + Livewire/Alpine ou SPA séparée (Inertia + Vue)
- **Backend API**: Laravel 11, PHP 8.3
- **ORM/DB**: Eloquent + PostgreSQL
- **Auth**: Laravel Breeze/Fortify (email/password + reset)
- **Paiement**: service `PaymentProviderInterface` + 1 implémentation MVP
- **Storage**: Laravel Filesystem (S3 driver) + URLs temporaires
- **Backoffice**: Filament Admin (RBAC intégré)
- **Infra**: Docker + Nginx + PHP-FPM

### Avantages
- Excellente productivité backoffice avec Filament.
- Workflow robuste pour modération/revue métier.
- Écosystème mature pour permissions, queues, notifications.

### Risques
- Si équipe très orientée JS/TS, ramp-up plus long.
- Double stack possible si front SPA séparée.

## Recommandation
**Option A (Next.js + Node + Postgres + Prisma)** pour ce MVP, car:
1. Time-to-market rapide avec équipe full-stack JS/TS.
2. Parcours public + dashboard utilisateur très fluides en SSR/ISR.
3. Facilité d’itérer sur UX mobile-first et composants partagés.

> Si le besoin principal devient un backoffice complexe multi-rôles dès J1, Option B redevient très compétitive.

## 3) Architecture logique (recommandée)

```text
[Web Next.js]
  ├─ Public pages (home, explorer, campaign)
  ├─ User dashboard (campaigns, donations, withdrawals)
  ├─ Admin backoffice (moderation, reports, settings)
  └─ API routes / services
         ├─ AuthService
         ├─ CampaignService
         ├─ DonationService
         ├─ WithdrawalService
         ├─ ModerationService
         ├─ FraudService (rules engine MVP)
         ├─ PaymentProvider (interface + impl)
         ├─ StorageService (S3 signed URLs)
         └─ AuditLogService

[PostgreSQL]
  ├─ Core business tables
  └─ Audit + fraud signals

[Object Storage S3]
  ├─ campaign media
  └─ KYC / justificatifs

[Queue/Jobs (optionnel MVP+)]
  ├─ emails
  ├─ webhooks paiements
  └─ checks anti-fraude asynchrones
```

## 4) Structure de repo (squelette)

```text
apps/
  web/                 # UI Next.js (public + user + admin)
  api/                 # services, domain, API handlers (si séparation)
packages/
  shared/              # types, schemas zod, constants statuts
infra/
  docker/              # compose, reverse proxy
  ci/                  # pipeline lint/test/build
docs/
  architecture.md
  database.md
  api.md
  security.md
  qa.md
```

## 5) Rôles et permissions MVP
- **Donor/User**: consulter, donner, signaler, gérer profil.
- **CampaignOwner**: créer/éditer campagne, soumettre, publier updates, demander retrait.
- **Moderator**: review campagnes/docs, approuver/suspendre, traiter signalements.
- **Admin**: droits moderator + paramètres (catégories, commissions) + audit complet.

## 6) États métier
- **Campaign**: `draft -> submitted -> under_review -> published | suspended | closed`
- **Withdrawal**: `requested -> in_review -> approved -> paid | rejected | frozen`
- **Verification level**: `L0, L1, L2`

## 7) Internationalisation
- FR par défaut.
- Préparer namespace i18n (`fr`, `ht`, `en`) avec fallback FR.
- Contenus légaux versionnés et localisables.

## 8) Instructions de déploiement (MVP)

### Prérequis
- PostgreSQL managé
- Bucket S3 compatible
- SMTP transactionnel
- Provider paiement configuré (clé API + webhook secret)

### Variables d’environnement minimales
- `DATABASE_URL`
- `AUTH_SECRET`
- `APP_URL`
- `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET_PUBLIC`, `S3_BUCKET_PRIVATE`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`
- `PAYMENT_PROVIDER`, `PAYMENT_API_KEY`, `PAYMENT_WEBHOOK_SECRET`
- `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASS`

### Pipeline
1. `pnpm install`
2. `pnpm prisma migrate deploy`
3. `pnpm build`
4. `pnpm start`

### Post-déploiement
- Créer rôles initiaux (admin/moderator).
- Seeder catégories.
- Configurer webhook paiement vers `/api/v1/payments/webhook/:provider`.
- Vérifier headers sécurité et HTTPS.
