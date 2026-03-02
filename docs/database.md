# Schéma base de données (PostgreSQL)

## Principes
- UUID comme PK pour éviter exposition d’IDs séquentiels.
- Timestamps partout (`created_at`, `updated_at`).
- Soft delete uniquement sur entités sensibles si nécessaire (`deleted_at`).
- Contraintes + index pour intégrité et performance.

## Tables minimales demandées

## `users`
- `id` (uuid, pk)
- `email` (unique, not null)
- `password_hash` (not null)
- `full_name` (not null)
- `phone` (nullable, unique partiel)
- `city` (nullable)
- `is_email_verified` (bool, default false)
- `is_phone_verified` (bool, default false)
- `verification_level` (enum: L0, L1, L2)
- `status` (enum: active, blocked)
- `created_at`, `updated_at`

## `roles`
- `id` (uuid, pk)
- `name` (enum/string unique: user, campaign_owner, moderator, admin)

## `user_roles`
- `user_id` (fk -> users)
- `role_id` (fk -> roles)
- PK composite (`user_id`, `role_id`)

## `categories`
- `id` (uuid, pk)
- `slug` (unique)
- `name_fr` (not null)
- `name_ht` (nullable)
- `name_en` (nullable)
- `is_active` (bool)

## `campaigns`
- `id` (uuid, pk)
- `owner_id` (fk -> users)
- `category_id` (fk -> categories)
- `title` (varchar 140)
- `slug` (unique)
- `story` (text)
- `city` (nullable)
- `goal_amount` (numeric(14,2), check > 0)
- `raised_amount` (numeric(14,2), default 0)
- `currency` (char(3), default 'USD')
- `is_urgent` (bool)
- `is_verified_badge` (bool)
- `status` (enum: draft, submitted, under_review, published, suspended, closed)
- `submitted_at`, `published_at`, `closed_at` (nullable)
- `created_at`, `updated_at`

## `campaign_media`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `type` (enum: image, video)
- `storage_key` (not null)
- `mime_type`
- `size_bytes`
- `sort_order`
- `created_at`

## `campaign_documents`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `document_type` (enum: identity, selfie, medical_proof, ownership_proof, other)
- `storage_key`
- `review_status` (enum: pending, accepted, rejected)
- `review_note` (nullable)
- `created_at`, `updated_at`

## `campaign_updates`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `author_id` (fk -> users)
- `title`
- `content`
- `is_published` (bool)
- `created_at`

## `donations`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `donor_id` (fk -> users, nullable pour guest futur)
- `amount` (numeric(14,2), check > 0)
- `currency` (char(3))
- `is_anonymous` (bool)
- `message` (varchar 280, nullable)
- `status` (enum: pending, succeeded, failed, refunded)
- `payment_transaction_id` (fk -> payment_transactions)
- `created_at`

## `payment_transactions`
- `id` (uuid, pk)
- `provider` (varchar)
- `provider_txn_id` (unique)
- `type` (enum: donation, payout)
- `amount` (numeric(14,2))
- `currency` (char(3))
- `status` (enum: initiated, succeeded, failed, pending_review)
- `raw_payload` (jsonb)
- `created_at`, `updated_at`

## `withdrawals`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `requester_id` (fk -> users)
- `amount` (numeric(14,2), check > 0)
- `status` (enum: requested, in_review, approved, paid, rejected, frozen)
- `risk_flags` (jsonb)
- `reviewed_by` (fk -> users, nullable)
- `review_note` (text, nullable)
- `paid_at` (nullable)
- `created_at`, `updated_at`

## `reports`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns)
- `reporter_id` (fk -> users)
- `category` (enum: scam, abuse, misleading, duplicate, other)
- `description` (text)
- `status` (enum: open, investigating, resolved, dismissed)
- `handled_by` (fk -> users, nullable)
- `created_at`, `updated_at`

## `admin_notes`
- `id` (uuid, pk)
- `campaign_id` (fk -> campaigns, nullable)
- `withdrawal_id` (fk -> withdrawals, nullable)
- `admin_id` (fk -> users)
- `note` (text)
- `visibility` (enum: internal)
- `created_at`

## `audit_logs`
- `id` (uuid, pk)
- `actor_user_id` (fk -> users)
- `action` (varchar, ex: CAMPAIGN_APPROVED)
- `entity_type` (varchar: campaign, withdrawal, user, report)
- `entity_id` (uuid)
- `metadata` (jsonb)
- `ip_address` (inet)
- `user_agent` (text)
- `created_at`

## Relations clés
- 1 user -> N campaigns
- 1 campaign -> N donations / N updates / N media / N documents / N reports / N withdrawals
- 1 donation -> 1 payment_transaction
- N users <-> N roles via `user_roles`
- Actions admin sensibles -> `audit_logs` obligatoires

## Index recommandés
- `campaigns(status, category_id, city, is_urgent)`
- `campaigns(raised_amount desc)` pour tendances
- `donations(campaign_id, status)`
- `withdrawals(status, created_at)`
- `reports(status, created_at)`
- `audit_logs(entity_type, entity_id, created_at)`
